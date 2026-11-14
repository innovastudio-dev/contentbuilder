# Enabling Code Chat Engine v2

```javascript
const builder = new ContentBuilder({
    // ...

    container: '.container',

    chatEngine: 'v2',               // enable Code Chat engine (v2)
    chatAttachments: true,          // let users attach images, PDFs and documents
    
    sendCommandUrl: '/openrouter',  // your AI endpoint, can be changed to use OpenAI. See docs/README.md
                                    
    codeModels: [                   // models offered in Chat Settings
        { id: '~anthropic/claude-opus-latest', label: 'Claude Opus (Latest)' },
        { id: '~anthropic/claude-sonnet-latest', label: 'Claude Sonnet (Latest)' },
        { id: '~google/gemini-pro-latest', label: 'Gemini Pro (Latest)' },
        { id: '~openai/gpt-latest', label: 'GPT (Latest)' },
    ],

    defaultChatSettings: {
        codeModel: '~anthropic/claude-opus-latest',
        imageModel: 'google/gemini-3-pro-image',
        imageSize: '',
    },
});
```

Code Chat needs a server-side endpoint to communicate with an AI provider. 
Examples are provided in **Node.js, PHP, and Next.js**, but you can implement 
the endpoint in any language.

If you already have an endpoint from Code Chat v1, you need to update 
the existing endpoint (`sendCommandUrl`) to enable Code Chat v2.

---

# Node.js

## Using OpenRouter

Replace this in your server file:

```javascript
app.post('/openrouter', async (req, res) => {
    ...
});
```

with this:

```javascript
/*
Marks the end of the conversation as cacheable, so the next call in the same
Code Chat request re-uses everything sent so far instead of paying for it again.
*/
function cbMarkConversationCacheable(messages) {
    const out = messages.slice();
    for (let i = out.length - 1; i >= 0; i--) {
        const m = out[i];
        if (!m || !m.role) continue;
        if (Array.isArray(m.content)) {
            const parts = m.content.map(p => ({ ...p }));
            const last = parts[parts.length - 1];
            if (last && last.type === 'text') {
                parts[parts.length - 1] = { ...last, cache_control: { type: 'ephemeral' } };
                out[i] = { ...m, content: parts };
                return out;
            }
            continue;
        }
        if (typeof m.content === 'string' && m.content.length > 0) {
            out[i] = { ...m, content: [{ type: 'text', text: m.content, cache_control: { type: 'ephemeral' } }] };
            return out;
        }
    }
    return out;
}

/*
Streams the reply back to Code Chat, so the answer appears as it is written.
Used automatically when the request has `stream: true`.
*/
async function serveConversationStream(req, res, { url, apiKey, defaultModel }) {

    const { messages, tools, system, model, temperature, topP } = req.body;

    if (!Array.isArray(messages) || messages.length === 0) {
        return res.json({ error: 'messages must be a non-empty array' });
    }

    const cacheable = system && system.length > 2000 && req.body.cacheContext !== false;
    const systemMessage = cacheable
        ? { role: 'system', content: [{ type: 'text', text: system, cache_control: { type: 'ephemeral' } }] }
        : { role: 'system', content: system };

    const streamMessages = cacheable ? cbMarkConversationCacheable(messages) : messages;

    const payload = {
        model: model || defaultModel,
        messages: system ? [systemMessage, ...streamMessages] : streamMessages,
        temperature: temperature === undefined ? 0.7 : parseFloat(temperature),
        top_p: topP === undefined ? 0.9 : parseFloat(topP),
        stream: true,
        stream_options: { include_usage: true },
    };
    if (Array.isArray(tools) && tools.length > 0) {
        payload.tools = tools.map(t => ({ type: 'function', function: t }));
        payload.tool_choice = 'auto';
    }

    const send = () => axios.post(url, payload, {
        headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${apiKey}` },
        responseType: 'stream',
        // An attached image makes the request bigger than axios sends by default
        maxBodyLength: Infinity,
        maxContentLength: Infinity,
    });

    let upstream;
    try {
        try {
            upstream = await send();
        } catch (first) {
            /*
            Same retry as the non-streaming path: some newer OpenAI models apply
            a default reasoning effort their chat/completions endpoint will not
            combine with function tools, and say so. Older models reject the
            argument outright, so it cannot always be sent — the provider's own
            message decides, once.

            The rejection body is a stream here, so it has to be read before it
            can be matched against.
            */
            const body = await (async () => {
                const d = first.response && first.response.data;
                if (d && typeof d.pipe === 'function') {
                    return await new Promise(resolve => {
                        let buf = '';
                        d.on('data', c => { buf += c.toString(); });
                        d.on('end', () => resolve(buf));
                        d.on('error', () => resolve(''));
                    });
                }
                return d ? (typeof d === 'string' ? d : JSON.stringify(d)) : '';
            })();

            if (/reasoning_effort/i.test(body) && payload.reasoning_effort === undefined) {
                payload.reasoning_effort = 'none';
                upstream = await send();
            } else {
                first.__body = body;   // reuse below rather than re-reading a spent stream
                throw first;
            }
        }
    } catch (e) {
        // Pass the provider's status through, so Code Chat waits and retries
        // when you are rate limited instead of giving up.
        const status = (e.response && e.response.status) || 0;
        const retryAfter = e.response && e.response.headers && e.response.headers['retry-after'];

        // Include what the provider actually said. Without it a rejected
        // request reads only as "status code 400", which says nothing about
        // which part of the request it refused.
        let detail = e.__body || '';
        try {
            const d = detail ? null : (e.response && e.response.data);
            if (d && typeof d.pipe === 'function') {
                detail = await new Promise(resolve => {
                    let buf = '';
                    d.on('data', c => { buf += c.toString(); });
                    d.on('end', () => resolve(buf));
                    d.on('error', () => resolve(''));
                });
            } else if (d) {
                detail = typeof d === 'string' ? d : JSON.stringify(d);
            }
        } catch (err) { /* keep the plain message */ }

        if (detail) console.log('[provider error]', String(detail).slice(0, 800));
        const parsed = (() => { try { return JSON.parse(detail); } catch (err) { return null; } })();
        const message = (parsed && parsed.error && parsed.error.message) || e.message;

        return res.json({ error: message, status, retryAfter: retryAfter ? Number(retryAfter) : undefined });
    }

    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    /*
    A chunk can end mid-line. Hold the remainder until the rest arrives, or the
    line is forwarded truncated and its tail is dropped — which corrupts long
    values such as the HTML in a tool call.
    */
    let pending = '';
    upstream.data.on('data', chunk => {
        pending += chunk.toString();
        const lines = pending.split('\n');
        pending = lines.pop();
        lines.forEach(line => {
            if (line.trim().startsWith('data:')) res.write(`${line}\n`);
        });
    });
    upstream.data.on('end', () => {
        if (pending.trim().startsWith('data:')) res.write(`${pending}\n`);
        res.write('data: [DONE]\n\n');
        res.end();
    });
    upstream.data.on('error', () => { try { res.end(); } catch (e) { /* already closed */ } });
}

/*
────────────────────────────────────────────────────────────────────────────
Code Chat v2  (chatEngine: 'v2')

Handles a v2 request: the conversation is passed through to your AI provider
and the model's reply is returned as-is.
────────────────────────────────────────────────────────────────────────────
*/
async function serveConversation(req, res, { url, apiKey, defaultModel }) {

    // Streaming is a flag on the same request, not a second endpoint.
    if (req.body && req.body.stream) {
        return serveConversationStream(req, res, { url, apiKey, defaultModel });
    }

    const { messages, tools, system, model, temperature, topP } = req.body;

    if (!Array.isArray(messages) || messages.length === 0) {
        return res.json({ error: 'messages must be a non-empty array' });
    }

    // Mark the instructions as cacheable. One Code Chat request makes several
    // calls with the same instructions, so this cuts the cost significantly.
    const cacheable = system && system.length > 2000 && req.body.cacheContext !== false;
    const systemMessage = cacheable
        ? { role: 'system', content: [{ type: 'text', text: system, cache_control: { type: 'ephemeral' } }] }
        : { role: 'system', content: system };

    // Do the same for a message carrying an attached image: it is large, and
    // it would otherwise be re-sent on every call.
    const conversation = messages.slice();
    if (cacheable) {
        const i = conversation.findIndex(m => m && m.role === 'user' && Array.isArray(m.content)
            && m.content.some(part => part && part.type === 'image_url'));
        if (i !== -1) {
            const parts = conversation[i].content.map(p => ({ ...p }));
            parts[parts.length - 1] = { ...parts[parts.length - 1], cache_control: { type: 'ephemeral' } };
            conversation[i] = { ...conversation[i], content: parts };
        }
    }

    // Cache what has been sent so far, so later calls in the same request do not
    // pay for the whole conversation again.
    const cached = cacheable ? cbMarkConversationCacheable(conversation) : conversation;

    const payload = {
        model: model || defaultModel,
        messages: system ? [systemMessage, ...cached] : cached,
        temperature: temperature === undefined ? 0.7 : parseFloat(temperature),
        top_p: topP === undefined ? 0.9 : parseFloat(topP),
    };

    // Code Chat sends plain tool definitions; providers expect each wrapped.
    if (Array.isArray(tools) && tools.length > 0) {
        payload.tools = tools.map(t => ({ type: 'function', function: t }));
        payload.tool_choice = 'auto';
    }

    const send = () => axios.post(url, payload, {
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${apiKey}`,
        },
        // An attached image makes the request bigger than axios sends by default
        maxBodyLength: Infinity,
        maxContentLength: Infinity,
        validateStatus: () => true,
    });

    let response = await send();

    /*
    Some newer OpenAI models apply a default reasoning effort that their
    chat/completions endpoint will not combine with function tools, and answer:
    "Function tools with reasoning_effort are not supported ... set
    reasoning_effort to 'none'". Older models reject that argument outright, so
    it cannot simply always be sent, and which models need it is not something
    to guess from a name. The provider says so plainly, so the retry is driven
    by what it said — once, and only for that message.
    */
    if (response.status >= 400
        && /reasoning_effort/i.test((response.data && response.data.error && response.data.error.message) || '')
        && payload.reasoning_effort === undefined) {
        payload.reasoning_effort = 'none';
        response = await send();
    }

    if (response.status >= 400) {
        const retryAfter = response.headers && response.headers['retry-after'];
        return res.json({
            error: (response.data && response.data.error && response.data.error.message) || `Request failed with status code ${response.status}`,
            status: response.status,
            retryAfter: retryAfter ? Number(retryAfter) : undefined,
        });
    }

    if (response.data && response.data.error) {
        return res.json({ error: response.data.error.message || String(response.data.error) });
    }

    const choice = response.data.choices && response.data.choices[0];
    if (!choice || !choice.message) {
        return res.json({ error: 'no message in provider response' });
    }

    // Return the whole message: Code Chat needs `tool_calls` to continue.
    return res.json({ message: choice.message, usage: response.data.usage });
}

/*
════════════════════════════════════════════════════════════════════════════
IMAGE GENERATION VIA OPENAI

Answered in one call, like OpenRouter — no queue, so no request id to poll.
The one difference is that OpenAI splits the two operations across endpoints:
/images/generations for a new image, /images/edits for one derived from a
reference. The route picks between them by whether references were sent.

An edit takes multipart form data rather than JSON, so a reference arriving as
a data URL is decoded back to bytes here.

Either way the API answers with base64. It is saved here and only a URL goes
back to the browser — base64 in the page would be written into every save of
that content.
════════════════════════════════════════════════════════════════════════════
*/
async function serveOpenAIImage(req, res) {
    const OPENAI_API_KEY = process.env.OPENAI_API_KEY;
    const { imageModel, prompt, imageSize, input_references, customData } = req.body;

    const folderPath = (customData && customData.folderPath) || '';
    const imageQuality = 70;
    const references = Array.isArray(input_references) ? input_references : [];

    // A data URL or an http(s) URL, either way ending as bytes for the form.
    const toBuffer = async (url) => {
        if (/^data:/i.test(url)) return Buffer.from(url.split(',')[1] || '', 'base64');
        const r = await axios.get(url, { responseType: 'arraybuffer' });
        return Buffer.from(r.data);
    };

    let data;
    try {
        if (references.length) {
            // Required here rather than at the top: an edit is the only path
            // that needs it, and this keeps the helper self-contained.
            const FormData = require('form-data');
            const form = new FormData();
            form.append('model', imageModel);
            form.append('prompt', prompt || '');
            if (imageSize) form.append('size', imageSize);   // OpenAI takes pixels

            for (let i = 0; i < references.length; i++) {
                const buf = await toBuffer(references[i]);
                form.append('image[]', buf, { filename: `reference-${i + 1}.png`, contentType: 'image/png' });
            }

            const response = await axios.post('https://api.openai.com/v1/images/edits', form, {
                headers: { ...form.getHeaders(), Authorization: `Bearer ${OPENAI_API_KEY}` },
                maxBodyLength: Infinity,
            });
            data = response.data;
        } else {
            const payload = { model: imageModel, prompt: prompt || '' };
            if (imageSize) payload.size = imageSize;        // OpenAI takes pixels

            const response = await axios.post('https://api.openai.com/v1/images/generations', payload, {
                headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${OPENAI_API_KEY}` },
            });
            data = response.data;
        }
    } catch (e) {
        const d = e.response && e.response.data;
        const message = (d && d.error && d.error.message) || e.message;
        return res.json({ error: message, status: (e.response && e.response.status) || 0 });
    }

    const entry = data && data.data && data.data[0];
    if (!entry || !entry.b64_json) {
        return res.json({ error: 'The generation returned no image.' });
    }

    try {
        const imageData = Buffer.from(entry.b64_json, 'base64');
        const filename = 'ai-' + Math.random().toString(36).substring(2, 7) + '.jpg';

        if (localSave) {
            const filePath = path.join($path, folderPath, filename);
            await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toFile(filePath);
            return res.json({ url: $urlpath + path.posix.join(folderPath, filename) });
        }

        const buffer = await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toBuffer();
        const key = ($prefix ? $prefix + '/' : '') + path.posix.join(folderPath, filename);
        await s3.send(new PutObjectCommand({
            Bucket: process.env.AWS_BUCKET,
            Key: key,
            Body: buffer,
            ContentType: 'image/jpeg',
        }));
        return res.json({ url: `https://${process.env.AWS_BUCKET}.s3.amazonaws.com/${key}` });
    } catch (e) {
        return res.json({ error: 'Could not save the generated image: ' + e.message });
    }
}

app.post('/sendcommand', async (req, res) => {

    const OPENAI_API_KEY = process.env.OPENAI_API_KEY;
    
    const DEFAULT_MODEL = 'gpt-4o-mini';

    // Image generation — one call, answered from the same route so a host
    // needs no extra endpoint. Neither v1 nor v2 sends `imageModel`.
    if (req.body && req.body.imageModel) {
        try {
            return await serveOpenAIImage(req, res);
        } catch (e) { return res.json({ error: e.message }); }
    }

    // chatEngine 'v2' — conversational shape. No `messages` = v1, unchanged.
    if (req.body && req.body.messages) {
        try {
            return await serveConversation(req, res, {
                url: 'https://api.openai.com/v1/chat/completions',
                apiKey: OPENAI_API_KEY,
                defaultModel: DEFAULT_MODEL,
            });
        } catch (e) { return res.json({ error: e.message }); }
    }

    let { question, context, system, functs, model } = req.body;

    const RESPONSE_ENDPOINT_MODELS = ['gpt-5.1-codex-mini', 'gpt-5.1-codex'];
    const useResponsesEndpoint = RESPONSE_ENDPOINT_MODELS.includes(model);

    const url = useResponsesEndpoint 
        ? 'https://api.openai.com/v1/responses'
        : 'https://api.openai.com/v1/chat/completions';

    const messages = [
        { role: 'system', content: system },
        { role: 'assistant', content: context || '' },
        { role: 'user', content: question }
    ];

    try {
        const requestBody = useResponsesEndpoint
            ? {
                model,
                input: messages,
            }
            : {
                model: model || DEFAULT_MODEL,
                messages,
                functions: functs.length === 0 ? undefined : functs
            };

        const response = await axios.post(url, requestBody, {
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${OPENAI_API_KEY}`,
            },
        });

        let answer, usage;

        if(useResponsesEndpoint) {

            // Normalize v1/responses format to match v1/chat/completions
            const messageOutput = response.data.output.find(item => item.type === 'message');
            answer = {
                choices: [{
                    message: {
                        role: messageOutput.role,
                        content: messageOutput.content[0].text
                    }
                }]
            };
            usage = response.data.usage;

        } else {

            answer = functs.length === 0
            ? response.data
            : (response.data.choices[0].message.function_call ? 
                response.data.choices[0].message.function_call.arguments 
                : response.data.choices[0].message );
            usage = response.data.usage;

        }

        res.json({ answer, usage });
        
    } catch (error) {
        res.json({ error: error.message });
    }
});

/*
════════════════════════════════════════════════════════════════════════════
IMAGE GENERATION VIA OPENROUTER

One call, no queue: POST the prompt and the image comes back in the same
response. That is the whole difference from Fal, which answers with a request
id to poll and a result to fetch afterwards.

The API returns base64. It is saved here and only a URL goes back to the
browser — the same thing /result-fal does with Fal's hosted image — because
base64 in the page would be written into every save of that content.
════════════════════════════════════════════════════════════════════════════
*/
async function serveImageGeneration(req, res) {
    const OPENROUTER_API_KEY = process.env.OPENROUTER_API_KEY;
    const { imageModel, prompt, imageSize, input_references, customData } = req.body;

    const folderPath = (customData && customData.folderPath) || '';
    const imageQuality = 70;

    const payload = { model: imageModel, prompt };
    if (imageSize) payload.aspect_ratio = imageSize;   // OpenRouter takes a ratio
    if (Array.isArray(input_references) && input_references.length) {
        payload.input_references = input_references.map(url => ({
            type: 'image_url',
            image_url: { url },
        }));
    }

    let data;
    try {
        const response = await axios.post('https://openrouter.ai/api/v1/images', payload, {
            headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${OPENROUTER_API_KEY}` },
        });
        data = response.data;
    } catch (e) {
        const d = e.response && e.response.data;
        const message = (d && d.error && d.error.message) || e.message;
        return res.json({ error: message, status: (e.response && e.response.status) || 0 });
    }

    const entry = data && data.data && data.data[0];
    if (!entry || !entry.b64_json) {
        return res.json({ error: 'The generation returned no image.' });
    }

    try {
        const imageData = Buffer.from(entry.b64_json, 'base64');
        const filename = 'ai-' + Math.random().toString(36).substring(2, 7) + '.jpg';

        if (localSave) {
            const filePath = path.join($path, folderPath, filename);
            await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toFile(filePath);
            return res.json({ url: $urlpath + path.posix.join(folderPath, filename) });
        }

        const buffer = await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toBuffer();
        const key = ($prefix ? $prefix + '/' : '') + path.posix.join(folderPath, filename);
        await s3.send(new PutObjectCommand({
            Bucket: process.env.AWS_BUCKET,
            Key: key,
            Body: buffer,
            ContentType: 'image/jpeg',
        }));
        return res.json({ url: `https://${process.env.AWS_BUCKET}.s3.amazonaws.com/${key}` });
    } catch (e) {
        return res.json({ error: 'Could not save the generated image: ' + e.message });
    }
}

app.post('/openrouter', async (req, res) => {

    try {
        const OPENROUTER_API_KEY = process.env.OPENROUTER_API_KEY;
        
        const url = "https://openrouter.ai/api/v1/chat/completions";
        const DEFAULT_MODEL = 'openai/gpt-4o-mini'; // Default model for content generation
        /*
        Models:
        - openai/gpt-4o-mini
        - anthropic/claude-3.5-sonnet
        - qwen/qwen-2.5-72b-instruct
        - deepseek/deepseek-r1-distill-llama-70b
        - meta-llama/llama-3.3-70b-instruct
        For more models: https://openrouter.ai/models
        Note: Different models produce different results. 
        Please test the model to ensure it aligns with your preferences, as responses can vary in style, depth, and accuracy.
        */
        const DEFAULT_TEMPERATURE = 0.6;
        const DEFAULT_TOP_P = 0.9;
        const DEFAULT_NUM = 1;

        // Image generation — one call, answered from the same route so a host
        // needs no extra endpoint. Neither v1 nor v2 sends `imageModel`.
        if (req.body && req.body.imageModel) {
            try {
                return await serveImageGeneration(req, res);
            } catch (e) { return res.json({ error: e.message }); }
        }

        // chatEngine 'v2' — conversational shape. No `messages` = v1, unchanged.
        if (req.body && req.body.messages) {
            try {
                return await serveConversation(req, res, {
                    url,
                    apiKey: OPENROUTER_API_KEY,
                    defaultModel: DEFAULT_MODEL,
                });
            } catch (e) { return res.json({ error: e.message }); }
        }

        let { question, context, system, functs, temperature, topP, num, model } = req.body;

        const messages = [
            { role: 'system', content: system },
            { role: 'assistant', content: context || '' },
            { role: 'user', content: question }
        ];

        try {

            if(functs.length>0) { 

                // --- Function Calling Branch ---
                // For function calling, only certain models are recommended.
                // The model is used solely for request categorization. No impact on content generation.

                const tools = [
                    {
                        type: "function",
                        function: {
                            ...functs[0]
                        }
                    }
                ];

                const response = await axios.post(url, {
                    model: 'anthropic/claude-3.5-sonnet',
                    /*
                    Models for function calling:
                    - anthropic/claude-3.5-sonnet
                    - qwen/qwen-2.5-72b-instruct
                    */
                    messages,
                    temperature: parseFloat(temperature) || DEFAULT_TEMPERATURE,
                    top_p: parseFloat(topP) || DEFAULT_TOP_P,
                    n: parseInt(num) || DEFAULT_NUM,
                    tools,
                }, {
                    headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${OPENROUTER_API_KEY}`,
                    },
                    validateStatus: () => true,
                });

                if(response.data && response.data.error && response.data.error.message) {
                    res.json({ ok: false, status: 404, error: response.data.error.message });
                    return;
                }

                let answer =
                    functs.length === 0
                        ? response.data
                        : response.data.choices[0].message.tool_calls
                        ? response.data.choices[0].message.tool_calls[0].function.arguments
                        : response.data.choices[0].message;
                let usage = response.data.usage;

                res.json({ answer, usage });

            } else {

                // --- Content Generation Branch ---

                const response = await axios.post(url, {
                    model: model || DEFAULT_MODEL,
                    messages,
                    temperature: parseFloat(temperature) || DEFAULT_TEMPERATURE,
                    top_p: parseFloat(topP) || DEFAULT_TOP_P,
                    n: parseInt(num) || DEFAULT_NUM
                }, {
                    headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${OPENROUTER_API_KEY}`,
                    },
                });

                let answer = response.data;
                let usage = response.data.usage;
    
                res.json({ answer, usage });

            }
            
        } catch (error) {
            res.json({ ok: false, status: 500, error: error.message }); 
        }

    } catch(e) {
        res.status(200).json({ ok: true, status: 500, error: 'Something went wrong.' });
    }
});
```

## Using OpenAI

Replace this in your server file:

```javascript
app.post('/sendcommand', async (req, res) => {
    ...
});
```

with this:

```javascript
/*
════════════════════════════════════════════════════════════════════════════
IMAGE GENERATION VIA OPENAI

Answered in one call, like OpenRouter — no queue, so no request id to poll.
The one difference is that OpenAI splits the two operations across endpoints:
/images/generations for a new image, /images/edits for one derived from a
reference. The route picks between them by whether references were sent.

An edit takes multipart form data rather than JSON, so a reference arriving as
a data URL is decoded back to bytes here.

Either way the API answers with base64. It is saved here and only a URL goes
back to the browser — base64 in the page would be written into every save of
that content.
════════════════════════════════════════════════════════════════════════════
*/
async function serveOpenAIImage(req, res) {
    const OPENAI_API_KEY = process.env.OPENAI_API_KEY;
    const { imageModel, prompt, imageSize, input_references, customData } = req.body;

    const folderPath = (customData && customData.folderPath) || '';
    const imageQuality = 70;
    const references = Array.isArray(input_references) ? input_references : [];

    // A data URL or an http(s) URL, either way ending as bytes for the form.
    const toBuffer = async (url) => {
        if (/^data:/i.test(url)) return Buffer.from(url.split(',')[1] || '', 'base64');
        const r = await axios.get(url, { responseType: 'arraybuffer' });
        return Buffer.from(r.data);
    };

    let data;
    try {
        if (references.length) {
            // Required here rather than at the top: an edit is the only path
            // that needs it, and this keeps the helper self-contained.
            const FormData = require('form-data');
            const form = new FormData();
            form.append('model', imageModel);
            form.append('prompt', prompt || '');
            if (imageSize) form.append('size', imageSize);   // OpenAI takes pixels

            for (let i = 0; i < references.length; i++) {
                const buf = await toBuffer(references[i]);
                form.append('image[]', buf, { filename: `reference-${i + 1}.png`, contentType: 'image/png' });
            }

            const response = await axios.post('https://api.openai.com/v1/images/edits', form, {
                headers: { ...form.getHeaders(), Authorization: `Bearer ${OPENAI_API_KEY}` },
                maxBodyLength: Infinity,
            });
            data = response.data;
        } else {
            const payload = { model: imageModel, prompt: prompt || '' };
            if (imageSize) payload.size = imageSize;        // OpenAI takes pixels

            const response = await axios.post('https://api.openai.com/v1/images/generations', payload, {
                headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${OPENAI_API_KEY}` },
            });
            data = response.data;
        }
    } catch (e) {
        const d = e.response && e.response.data;
        const message = (d && d.error && d.error.message) || e.message;
        return res.json({ error: message, status: (e.response && e.response.status) || 0 });
    }

    const entry = data && data.data && data.data[0];
    if (!entry || !entry.b64_json) {
        return res.json({ error: 'The generation returned no image.' });
    }

    try {
        const imageData = Buffer.from(entry.b64_json, 'base64');
        const filename = 'ai-' + Math.random().toString(36).substring(2, 7) + '.jpg';

        if (localSave) {
            const filePath = path.join($path, folderPath, filename);
            await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toFile(filePath);
            return res.json({ url: $urlpath + path.posix.join(folderPath, filename) });
        }

        const buffer = await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toBuffer();
        const key = ($prefix ? $prefix + '/' : '') + path.posix.join(folderPath, filename);
        await s3.send(new PutObjectCommand({
            Bucket: process.env.AWS_BUCKET,
            Key: key,
            Body: buffer,
            ContentType: 'image/jpeg',
        }));
        return res.json({ url: `https://${process.env.AWS_BUCKET}.s3.amazonaws.com/${key}` });
    } catch (e) {
        return res.json({ error: 'Could not save the generated image: ' + e.message });
    }
}

app.post('/sendcommand', async (req, res) => {

    const OPENAI_API_KEY = process.env.OPENAI_API_KEY;
    
    const DEFAULT_MODEL = 'gpt-4o-mini';

    // Image generation — one call, answered from the same route so a host
    // needs no extra endpoint. Neither v1 nor v2 sends `imageModel`.
    if (req.body && req.body.imageModel) {
        try {
            return await serveOpenAIImage(req, res);
        } catch (e) { return res.json({ error: e.message }); }
    }

    // chatEngine 'v2' — conversational shape. No `messages` = v1, unchanged.
    if (req.body && req.body.messages) {
        try {
            return await serveConversation(req, res, {
                url: 'https://api.openai.com/v1/chat/completions',
                apiKey: OPENAI_API_KEY,
                defaultModel: DEFAULT_MODEL,
            });
        } catch (e) { return res.json({ error: e.message }); }
    }

    let { question, context, system, functs, model } = req.body;

    const RESPONSE_ENDPOINT_MODELS = ['gpt-5.1-codex-mini', 'gpt-5.1-codex'];
    const useResponsesEndpoint = RESPONSE_ENDPOINT_MODELS.includes(model);

    const url = useResponsesEndpoint 
        ? 'https://api.openai.com/v1/responses'
        : 'https://api.openai.com/v1/chat/completions';

    const messages = [
        { role: 'system', content: system },
        { role: 'assistant', content: context || '' },
        { role: 'user', content: question }
    ];

    try {
        const requestBody = useResponsesEndpoint
            ? {
                model,
                input: messages,
            }
            : {
                model: model || DEFAULT_MODEL,
                messages,
                functions: functs.length === 0 ? undefined : functs
            };

        const response = await axios.post(url, requestBody, {
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${OPENAI_API_KEY}`,
            },
        });

        let answer, usage;

        if(useResponsesEndpoint) {

            // Normalize v1/responses format to match v1/chat/completions
            const messageOutput = response.data.output.find(item => item.type === 'message');
            answer = {
                choices: [{
                    message: {
                        role: messageOutput.role,
                        content: messageOutput.content[0].text
                    }
                }]
            };
            usage = response.data.usage;

        } else {

            answer = functs.length === 0
            ? response.data
            : (response.data.choices[0].message.function_call ? 
                response.data.choices[0].message.function_call.arguments 
                : response.data.choices[0].message );
            usage = response.data.usage;

        }

        res.json({ answer, usage });
        
    } catch (error) {
        res.json({ error: error.message });
    }
});


/*
════════════════════════════════════════════════════════════════════════════
IMAGE GENERATION VIA OPENROUTER

One call, no queue: POST the prompt and the image comes back in the same
response. That is the whole difference from Fal, which answers with a request
id to poll and a result to fetch afterwards.

The API returns base64. It is saved here and only a URL goes back to the
browser — the same thing /result-fal does with Fal's hosted image — because
base64 in the page would be written into every save of that content.
════════════════════════════════════════════════════════════════════════════
*/
async function serveImageGeneration(req, res) {
    const OPENROUTER_API_KEY = process.env.OPENROUTER_API_KEY;
    const { imageModel, prompt, imageSize, input_references, customData } = req.body;

    const folderPath = (customData && customData.folderPath) || '';
    const imageQuality = 70;

    const payload = { model: imageModel, prompt };
    if (imageSize) payload.aspect_ratio = imageSize;   // OpenRouter takes a ratio
    if (Array.isArray(input_references) && input_references.length) {
        payload.input_references = input_references.map(url => ({
            type: 'image_url',
            image_url: { url },
        }));
    }

    let data;
    try {
        const response = await axios.post('https://openrouter.ai/api/v1/images', payload, {
            headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${OPENROUTER_API_KEY}` },
        });
        data = response.data;
    } catch (e) {
        const d = e.response && e.response.data;
        const message = (d && d.error && d.error.message) || e.message;
        return res.json({ error: message, status: (e.response && e.response.status) || 0 });
    }

    const entry = data && data.data && data.data[0];
    if (!entry || !entry.b64_json) {
        return res.json({ error: 'The generation returned no image.' });
    }

    try {
        const imageData = Buffer.from(entry.b64_json, 'base64');
        const filename = 'ai-' + Math.random().toString(36).substring(2, 7) + '.jpg';

        if (localSave) {
            const filePath = path.join($path, folderPath, filename);
            await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toFile(filePath);
            return res.json({ url: $urlpath + path.posix.join(folderPath, filename) });
        }

        const buffer = await sharp(imageData).jpeg({ quality: imageQuality }).withMetadata(false).toBuffer();
        const key = ($prefix ? $prefix + '/' : '') + path.posix.join(folderPath, filename);
        await s3.send(new PutObjectCommand({
            Bucket: process.env.AWS_BUCKET,
            Key: key,
            Body: buffer,
            ContentType: 'image/jpeg',
        }));
        return res.json({ url: `https://${process.env.AWS_BUCKET}.s3.amazonaws.com/${key}` });
    } catch (e) {
        return res.json({ error: 'Could not save the generated image: ' + e.message });
    }
}
```

Using both? Paste the conversation helpers once, then both image helpers and both routes.

---

# PHP

## 1. Add this file as `api/conversation.php`

```php
<?php
/*
────────────────────────────────────────────────────────────────────────────
Code Chat v2  (chatEngine: 'v2')

Handles a v2 request: the conversation is passed through to your AI provider
and the model's reply is returned as-is.
────────────────────────────────────────────────────────────────────────────
*/

/**
 * Pass the RAW request body here, not a decoded array.
 *
 * json_decode($json, true) turns an empty JSON object into an empty PHP array,
 * which re-encodes as [] and makes the provider reject the request. Decoding
 * to objects here avoids that.
 */
/*
Marks the end of the conversation as cacheable, so the next call in the same
Code Chat request re-uses everything sent so far instead of paying for it again.
*/
function cb_mark_conversation_cacheable($messages) {
    for ($i = count($messages) - 1; $i >= 0; $i--) {
        $m = $messages[$i];
        if (!is_object($m) || !isset($m->content)) continue;

        if (is_array($m->content)) {
            $last = end($m->content);
            if (is_object($last) && isset($last->type) && $last->type === 'text') {
                $last->cache_control = (object) ['type' => 'ephemeral'];
                return $messages;
            }
            continue;
        }

        if (is_string($m->content) && $m->content !== '') {
            $block = new stdClass();
            $block->type = 'text';
            $block->text = $m->content;
            $block->cache_control = (object) ['type' => 'ephemeral'];
            $m->content = [$block];
            return $messages;
        }
    }
    return $messages;
}

/*
A chunk can end mid-line. Hold the remainder until the rest arrives, or the
line is echoed truncated and its tail is dropped — which corrupts long values
such as the HTML in a tool call.
*/
function cb_forward_sse_chunk($chunk, &$pending) {
    $pending .= $chunk;
    $lines = explode("\n", $pending);
    $pending = array_pop($lines);
    foreach ($lines as $line) {
        if (strpos(ltrim($line), 'data:') === 0) {
            echo $line . "\n";
            flush();
        }
    }
    return strlen($chunk);
}

function cb_serve_conversation($rawBody, $url, $apiKey, $defaultModel) {

    $req = json_decode($rawBody);   // NOTE: objects, not associative arrays

    // Streaming is a flag on the same request, not a second endpoint.
    $streaming = isset($req->stream) && $req->stream;

    $messages = isset($req->messages) ? $req->messages : null;

    if (!is_array($messages) || count($messages) === 0) {
        echo json_encode(['error' => 'messages must be a non-empty array']);
        return;
    }

    // The system prompt is sent separately and prepended here, matching the
    // Node route, so the client does not have to rebuild it every turn.
    $system = isset($req->system) ? $req->system : '';
    $cacheable = false;
    if ($system !== '') {
        $sys = new stdClass();
        $sys->role = 'system';

        // Mark the instructions as cacheable. One Code Chat request makes
        // several calls with the same instructions, so this cuts the cost.
        $cacheable = strlen($system) > 2000
            && (!isset($req->cacheContext) || $req->cacheContext !== false);

        if ($cacheable) {
            $block = new stdClass();
            $block->type = 'text';
            $block->text = $system;
            $block->cache_control = (object) ['type' => 'ephemeral'];
            $sys->content = [$block];
        } else {
            $sys->content = $system;
        }
        array_unshift($messages, $sys);
    }

    // Cache what has been sent so far, so later calls in the same request do not
    // pay for the whole conversation again.
    if ($cacheable) $messages = cb_mark_conversation_cacheable($messages);

    $data = [
        'model' => isset($req->model) && $req->model ? $req->model : $defaultModel,
        'messages' => $messages,
        'temperature' => isset($req->temperature) ? floatval($req->temperature) : 0.7,
        'top_p' => isset($req->topP) ? floatval($req->topP) : 0.9,
    ];

    // Code Chat sends plain tool definitions; providers expect each wrapped.
    if (!empty($req->tools) && is_array($req->tools)) {
        $tools = [];
        foreach ($req->tools as $t) {
            $wrapped = new stdClass();
            $wrapped->type = 'function';
            $wrapped->function = $t;
            $tools[] = $wrapped;
        }
        $data['tools'] = $tools;
        $data['tool_choice'] = 'auto';
    }

    $headers = [
        'Content-Type: application/json',
        'Authorization: Bearer ' . $apiKey,
    ];

    // Streams the reply back so the answer appears as it is written.
    // Used automatically when the request has `stream: true`.
    if ($streaming) {
        $data['stream'] = true;
        $data['stream_options'] = (object) ['include_usage' => true];

        header('Content-Type: text/event-stream');
        header('Cache-Control: no-cache');
        header('X-Accel-Buffering: no');    // stop nginx buffering the stream
        while (ob_get_level() > 0) { ob_end_flush(); }

        /*
        One attempt. A failed stream reports its status in the headers only, so
        the body is collected rather than forwarded — it is needed both for the
        error frame and to decide whether a retry is worth making.
        */
        $attempt = function ($payload) use ($url, $headers, &$streamPending) {
            $status = 200; $retryAfter = null; $errBody = '';

            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, $url);
            curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
            curl_setopt($ch, CURLOPT_POST, 1);
            curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
            curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
            curl_setopt($ch, CURLOPT_HEADERFUNCTION, function ($ch, $header) use (&$status, &$retryAfter) {
                if (preg_match('#^HTTP/\S+\s+(\d{3})#i', $header, $m)) $status = (int) $m[1];
                if (stripos($header, 'retry-after:') === 0) {
                    $retryAfter = (int) trim(substr($header, strlen('retry-after:')));
                }
                return strlen($header);
            });
            curl_setopt($ch, CURLOPT_WRITEFUNCTION, function ($ch, $chunk) use (&$status, &$errBody, &$streamPending) {
                if ($status >= 400) { $errBody .= $chunk; return strlen($chunk); }
                return cb_forward_sse_chunk($chunk, $streamPending);
            });
            curl_exec($ch);
            curl_close($ch);

            return ['status' => $status, 'retryAfter' => $retryAfter, 'body' => $errBody];
        };

        $streamPending = '';
        $res = $attempt($data);

        /*
        Some newer OpenAI models apply a default reasoning effort their
        chat/completions endpoint will not combine with function tools, and say
        so. Older models reject the argument outright, so it cannot always be
        sent — the provider's own message decides, once.
        */
        if ($res['status'] >= 400 && stripos($res['body'], 'reasoning_effort') !== false
            && !isset($data['reasoning_effort'])) {
            $data['reasoning_effort'] = 'none';
            $streamPending = '';
            $res = $attempt($data);
        }

        if ($res['status'] >= 400) {
            $parsed = json_decode($res['body'], true);
            $msg = $parsed['error']['message'] ?? ('Request failed with status code ' . $res['status']);
            $frame = ['error' => ['message' => $msg], 'status' => $res['status']];
            if ($res['retryAfter']) $frame['retryAfter'] = $res['retryAfter'];
            echo 'data: ' . json_encode($frame) . "\n";
            flush();
        } elseif (strpos(ltrim($streamPending), 'data:') === 0) {
            echo $streamPending . "\n";
            flush();
        }

        echo "data: [DONE]\n\n";
        flush();
        return;
    }

    $send = function ($payload) use ($url, $headers) {
        $retryAfter = null;

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($ch, CURLOPT_POST, 1);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);

        // Capture Retry-After, so Code Chat waits the right amount of time
        // when you are rate limited.
        curl_setopt($ch, CURLOPT_HEADERFUNCTION, function ($ch, $header) use (&$retryAfter) {
            if (stripos($header, 'retry-after:') === 0) {
                $retryAfter = (int) trim(substr($header, strlen('retry-after:')));
            }
            return strlen($header);
        });

        $body = curl_exec($ch);
        $status = (int) curl_getinfo($ch, CURLINFO_HTTP_CODE);
        $err = $body === false ? curl_error($ch) : null;
        curl_close($ch);

        return ['body' => $body, 'status' => $status, 'retryAfter' => $retryAfter, 'curlError' => $err];
    };

    $res = $send($data);

    /*
    Some newer OpenAI models apply a default reasoning effort their
    chat/completions endpoint will not combine with function tools, and say so.
    Older models reject the argument outright, so it cannot always be sent —
    the provider's own message decides, once.
    */
    if ($res['status'] >= 400 && is_string($res['body'])
        && stripos($res['body'], 'reasoning_effort') !== false
        && !isset($data['reasoning_effort'])) {
        $data['reasoning_effort'] = 'none';
        $res = $send($data);
    }

    $response = $res['body'];
    $status = $res['status'];
    $retryAfter = $res['retryAfter'];

    if ($response === false) {
        echo json_encode(['error' => 'cURL Error: ' . $res['curlError']]);
        return;
    }

    $responseData = json_decode($response, true);

    if ($status >= 400) {
        $msg = isset($responseData['error']['message'])
            ? $responseData['error']['message']
            : 'Request failed with status code ' . $status;
        $out = ['error' => $msg, 'status' => $status];
        if ($retryAfter) $out['retryAfter'] = $retryAfter;
        echo json_encode($out);
        return;
    }

    if (isset($responseData['error'])) {
        $msg = is_array($responseData['error']) && isset($responseData['error']['message'])
            ? $responseData['error']['message']
            : (is_string($responseData['error']) ? $responseData['error'] : 'provider error');
        echo json_encode(['error' => $msg]);
        return;
    }

    $message = isset($responseData['choices'][0]['message'])
        ? $responseData['choices'][0]['message']
        : null;

    if (!$message) {
        echo json_encode(['error' => 'no message in provider response']);
        return;
    }

    // Return the whole message: Code Chat needs `tool_calls` to continue.
    echo json_encode([
        'message' => $message,
        'usage' => isset($responseData['usage']) ? $responseData['usage'] : null,
    ]);
}
?>
```

## 2. Using OpenAI — replace `api/sendcommand.php` with this

```php
<?php
require_once 'config.php';
require_once 'conversation.php';
require_once 'common.php';

$DEFAULT_MODEL = 'gpt-4o-mini';

$RESPONSE_ENDPOINT_MODELS = ['gpt-5.1-codex-mini', 'gpt-5.1-codex'];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // The RAW body is kept: json_decode turns an empty JSON object into [],
    // which breaks a tool definition's "parameters": {}.
    $rawBody = file_get_contents('php://input');
    $body = json_decode($rawBody, true);

    // Image generation — one call, answered from the same route so a host
    // needs no extra endpoint. Neither v1 nor v2 sends `imageModel`.
    if (!empty($body['imageModel'])) {
        cb_serve_openai_image($body, $OPENAI_API_KEY);
        exit;
    }

    // chatEngine 'v2' — conversational shape. No `messages` = v1, unchanged.
    // v2 always uses chat/completions, never the v1/responses endpoint.
    if (!empty($body['messages'])) {
        cb_serve_conversation(
            $rawBody,
            'https://api.openai.com/v1/chat/completions',
            $OPENAI_API_KEY,
            $DEFAULT_MODEL
        );
        exit;
    }
    $question = $body['question'];
    $context = $body['context'];
    $system = $body['system'];
    $functs = $body['functs'];
    $model = $body['model'] ?? $DEFAULT_MODEL;

    // Determine which endpoint to use
    $useResponsesEndpoint = in_array($model, $RESPONSE_ENDPOINT_MODELS);
    
    $url = $useResponsesEndpoint 
        ? 'https://api.openai.com/v1/responses'
        : 'https://api.openai.com/v1/chat/completions';

    $messages = [
        ['role' => 'system', 'content' => $system],
        ['role' => 'assistant', 'content' => $context ?: ''],
        ['role' => 'user', 'content' => $question]
    ];

    $headers = [
        'Content-Type: application/json',
        'Authorization: Bearer ' . $OPENAI_API_KEY,
    ];

    // Build request body based on endpoint
    if ($useResponsesEndpoint) {
        $data = [
            'model' => $model,
            'input' => $messages,
        ];
    } else {
        $data = [
            'model' => $model,
            'messages' => $messages,
        ];

        if (!empty($functs)) {
            $data['functions'] = $functs;
        }
    }

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_POST, 1);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
    
    $response = curl_exec($ch);

    if ($response === false) {
        // Handle the cURL error here
        echo json_encode(['error' => 'cURL Error: ' . curl_error($ch)]);
    } else {
        $responseData = json_decode($response, true);

        if ($useResponsesEndpoint) {
            // Normalize v1/responses format to match v1/chat/completions
            $messageOutput = null;
            foreach ($responseData['output'] as $item) {
                if ($item['type'] === 'message') {
                    $messageOutput = $item;
                    break;
                }
            }

            $answer = [
                'choices' => [[
                    'message' => [
                        'role' => $messageOutput['role'],
                        'content' => $messageOutput['content'][0]['text']
                    ]
                ]]
            ];
            $usage = $responseData['usage'];
        } else {
            // Original v1/chat/completions logic
            $answer = empty($functs)
                ? $responseData
                : ($responseData['choices'][0]['message']['function_call'] ?
                $responseData['choices'][0]['message']['function_call']['arguments'] :
                $responseData['choices'][0]['message']);
            $usage = $responseData['usage'];
        }

        echo json_encode(['answer' => $answer, 'usage' => $usage]);

        curl_close($ch);
    }
}
?>

/*
Image generation via OpenAI — one call, no queue.

The one difference from OpenRouter is that OpenAI splits the two operations:
/images/generations for a new image, /images/edits for one derived from a
reference. An edit takes multipart form data rather than JSON, so a reference
arriving as a data URL is decoded back to bytes here.

Either way the API answers with base64. It is saved here and only a URL goes
back to the browser — base64 in the page would be written into every save of
that content.
*/
function cb_serve_openai_image($body, $apiKey) {
    global $path, $urlpath;

    $references = $body['input_references'] ?? [];
    if (!is_array($references)) $references = [];

    $ch = curl_init();

    if ($references !== []) {
        $fields = [
            'model'  => $body['imageModel'],
            'prompt' => $body['prompt'] ?? '',
        ];
        if (!empty($body['imageSize'])) {
            $fields['size'] = $body['imageSize'];   // OpenAI takes pixels
        }

        $i = 0;
        foreach ($references as $url) {
            $bytes = strpos($url, 'data:') === 0
                ? base64_decode(substr($url, strpos($url, ',') + 1))
                : @file_get_contents($url);

            if ($bytes === false) continue;

            $tmp = tempnam(sys_get_temp_dir(), 'cbimg');
            file_put_contents($tmp, $bytes);
            $fields['image[]'] = new CURLFile($tmp, 'image/png', 'reference-' . (++$i) . '.png');
        }

        curl_setopt($ch, CURLOPT_URL, 'https://api.openai.com/v1/images/edits');
        curl_setopt($ch, CURLOPT_POSTFIELDS, $fields);
        curl_setopt($ch, CURLOPT_HTTPHEADER, ['Authorization: Bearer ' . $apiKey]);
    } else {
        $payload = [
            'model'  => $body['imageModel'],
            'prompt' => $body['prompt'] ?? '',
        ];
        if (!empty($body['imageSize'])) {
            $payload['size'] = $body['imageSize'];   // OpenAI takes pixels
        }

        curl_setopt($ch, CURLOPT_URL, 'https://api.openai.com/v1/images/generations');
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
        curl_setopt($ch, CURLOPT_HTTPHEADER, [
            'Content-Type: application/json',
            'Authorization: Bearer ' . $apiKey,
        ]);
    }

    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_TIMEOUT, 300);

    $response = curl_exec($ch);
    $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);

    if (curl_errno($ch)) {
        $err = curl_error($ch);
        curl_close($ch);
        echo json_encode(['error' => $err]);
        return;
    }
    curl_close($ch);

    $data = json_decode($response, true);

    if ($status < 200 || $status >= 300) {
        echo json_encode([
            'error'  => $data['error']['message'] ?? 'Request failed with status code ' . $status,
            'status' => $status,
        ]);
        return;
    }

    $b64 = $data['data'][0]['b64_json'] ?? '';
    if ($b64 === '') {
        echo json_encode(['error' => 'The generation returned no image.']);
        return;
    }

    $folderPath = $body['customData']['folderPath'] ?? '';
    $folderPathWithSlash = $folderPath ? $folderPath . '/' : '';
    $filename = 'ai-' . generateRandomString(5) . '.png';
    $filePath = $path . $folderPathWithSlash . $filename;

    if (!is_dir(dirname($filePath))) {
        mkdir(dirname($filePath), 0777, true);
    }

    if (file_put_contents($filePath, base64_decode($b64)) === false) {
        echo json_encode(['error' => 'Could not save the generated image.']);
        return;
    }

    echo json_encode(['url' => $urlpath . $folderPathWithSlash . $filename]);
}
```

## 3. Using OpenRouter — replace `api/openrouter.php` with this

```php
<?php
require_once 'config.php';
require_once 'conversation.php';
require_once 'common.php';

set_time_limit(60);

$url = 'https://openrouter.ai/api/v1/chat/completions';
$model = 'openai/gpt-4o-mini';
/*
Models:
- openai/gpt-4o-mini
- anthropic/claude-3.5-sonnet
- qwen/qwen-2.5-72b-instruct
- deepseek/deepseek-r1-distill-llama-70b
- meta-llama/llama-3.3-70b-instruct
For more models: https://openrouter.ai/models
Note: Different models produce different results. 
Please test the model to ensure it aligns with your preferences, as responses can vary in style, depth, and accuracy.
*/

$DEFAULT_TEMPERATURE = 0.6;
$DEFAULT_TOP_P = 0.9;
$DEFAULT_NUM = 1;

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // The RAW body is kept: json_decode turns an empty JSON object into [],
    // which breaks a tool definition's "parameters": {}.
    $rawBody = file_get_contents('php://input');
    $body = json_decode($rawBody, true);

    // Image generation — one call, answered from the same route so a host
    // needs no extra endpoint. Neither v1 nor v2 sends `imageModel`.
    if (!empty($body['imageModel'])) {
        cb_serve_image_generation($body, $OPENROUTER_API_KEY);
        exit;
    }

    // chatEngine 'v2' — conversational shape. No `messages` = v1, unchanged.
    if (!empty($body['messages'])) {
        cb_serve_conversation($rawBody, $url, $OPENROUTER_API_KEY, $model);
        exit;
    }
    $question = $body['question'];
    $context = $body['context'];
    $system = $body['system'];
    $functs = $body['functs'];
    $temperature = $body['temperature'];
    $topP = $body['topP'];
    $num = $body['num'];

    $messages = [
        ['role' => 'system', 'content' => $system],
        ['role' => 'assistant', 'content' => $context ?: ''],
        ['role' => 'user', 'content' => $question]
    ];

    $headers = [
        'Content-Type: application/json',
        'Authorization: Bearer ' . $OPENROUTER_API_KEY,
    ];

    $data = [
        'messages' => $messages,
        'temperature' => floatval($temperature) ?: $DEFAULT_TEMPERATURE,
        'top_p' => floatval($topP) ?: $DEFAULT_TOP_P,
        'n' => intval($num) ?: $DEFAULT_NUM,
    ];

    if (!empty($functs)) {
        // --- Function Calling Branch ---
        // For function calling, only certain models are recommended.
        // The model is used solely for request categorization. No impact on content generation.
        // Models for function calling:
        // - anthropic/claude-3.5-sonnet
        // - qwen/qwen-2.5-72b-instruct
        $data['model'] = 'anthropic/claude-3.5-sonnet';
        $data['tools'] = [
            [
                'type' => 'function',
                'function' => $functs[0]
            ]
        ];
    } else {
        // --- Content Generation Branch ---
        $data['model'] = $model;
    }

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_POST, 1);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
    
    $response = curl_exec($ch);

    if ($response === false) {
        echo 'cURL Error: ' . curl_error($ch);
    } else {
        $responseData = json_decode($response, true);

        if (empty($functs)) {
            $answer = $responseData;
        } else {
            if (isset($responseData['choices']) && isset($responseData['choices'][0]['message'])) {
                if (isset($responseData['choices'][0]['message']['tool_calls'][0]['function']['arguments'])) {
                    $answer = $responseData['choices'][0]['message']['tool_calls'][0]['function']['arguments'];
                } else {
                    $answer = $responseData['choices'][0]['message'];
                }
            } else {
                $answer = null;
            }
        }

        echo json_encode(['answer' => $answer]);
        curl_close($ch);
    }
}
?>

/*
Image generation via OpenRouter — one call, no queue.

Unlike Fal there is no request id to poll and no result to fetch: the image
comes back in the same response. It arrives as base64 and is saved here, so the
browser only ever receives a URL — base64 in the page would be written into
every save of that content.
*/
function cb_serve_image_generation($body, $apiKey) {
    global $path, $urlpath, $localSave, $s3, $bucket, $prefix;

    $payload = [
        'model'  => $body['imageModel'],
        'prompt' => $body['prompt'] ?? '',
    ];
    if (!empty($body['imageSize'])) {
        $payload['aspect_ratio'] = $body['imageSize'];   // OpenRouter takes a ratio
    }
    if (!empty($body['input_references']) && is_array($body['input_references'])) {
        $payload['input_references'] = array_map(function ($url) {
            return ['type' => 'image_url', 'image_url' => ['url' => $url]];
        }, $body['input_references']);
    }

    $ch = curl_init('https://openrouter.ai/api/v1/images');
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'Content-Type: application/json',
            'Authorization: Bearer ' . $apiKey,
        ],
        CURLOPT_POSTFIELDS => json_encode($payload),
        CURLOPT_TIMEOUT => 300,
    ]);
    $response = curl_exec($ch);
    $status = curl_getinfo($ch, CURLINFO_HTTP_CODE);

    if (curl_errno($ch)) {
        $err = curl_error($ch);
        curl_close($ch);
        echo json_encode(['error' => $err]);
        return;
    }
    curl_close($ch);

    $data = json_decode($response, true);

    if ($status < 200 || $status >= 300) {
        echo json_encode([
            'error'  => $data['error']['message'] ?? 'Request failed with status code ' . $status,
            'status' => $status,
        ]);
        return;
    }

    $b64 = $data['data'][0]['b64_json'] ?? '';
    if ($b64 === '') {
        echo json_encode(['error' => 'The generation returned no image.']);
        return;
    }

    $folderPath = $body['customData']['folderPath'] ?? '';
    $folderPathWithSlash = $folderPath ? $folderPath . '/' : '';
    $filename = 'ai-' . generateRandomString(5) . '.jpg';
    $filePath = $path . $folderPathWithSlash . $filename;

    if (!is_dir(dirname($filePath))) {
        mkdir(dirname($filePath), 0777, true);
    }

    if (file_put_contents($filePath, base64_decode($b64)) === false) {
        echo json_encode(['error' => 'Could not save the generated image.']);
        return;
    }

    echo json_encode(['url' => $urlpath . $folderPathWithSlash . $filename]);
}
```

---

# Next.js

## 1. Add this file as `src/lib/serveConversation.ts`

```typescript
// src/lib/serveConversation.ts
/*
────────────────────────────────────────────────────────────────────────────
Code Chat v2  (chatEngine: 'v2')

Handles a v2 request: the conversation is passed through to your AI provider
and the model's reply is returned as-is.
────────────────────────────────────────────────────────────────────────────
*/
import { NextResponse } from 'next/server';

type Body = {
    messages?: unknown;
    tools?: Array<Record<string, unknown>>;
    system?: string;
    model?: string;
    temperature?: number | string;
    topP?: number | string;
    cacheContext?: boolean;
    stream?: boolean;
};

/*
Marks the end of the conversation as cacheable, so the next call in the same
Code Chat request re-uses everything sent so far instead of paying for it again.
*/
function markConversationCacheable(messages: unknown[]): unknown[] {
    const out = messages.slice() as Array<Record<string, unknown>>;
    for (let i = out.length - 1; i >= 0; i--) {
        const m = out[i];
        if (!m || !m.role) continue;

        if (Array.isArray(m.content)) {
            const parts = (m.content as Array<Record<string, unknown>>).map(p => ({ ...p }));
            const last = parts[parts.length - 1];
            if (last && last.type === 'text') {
                parts[parts.length - 1] = { ...last, cache_control: { type: 'ephemeral' } };
                out[i] = { ...m, content: parts };
                return out;
            }
            continue;
        }

        if (typeof m.content === 'string' && m.content.length > 0) {
            out[i] = {
                ...m,
                content: [{ type: 'text', text: m.content, cache_control: { type: 'ephemeral' } }],
            };
            return out;
        }
    }
    return out;
}

export async function serveConversation(
    body: Body,
    { url, apiKey, defaultModel }: { url: string; apiKey?: string; defaultModel: string }
) {
    const messages = body.messages;

    if (!Array.isArray(messages) || messages.length === 0) {
        return NextResponse.json({ error: 'messages must be a non-empty array' });
    }

    // Mark the instructions as cacheable. One Code Chat request makes several
    // calls with the same instructions, so this cuts the cost significantly.
    const cacheable = !!body.system && body.system.length > 2000 && body.cacheContext !== false;
    const systemMessage = cacheable
        ? { role: 'system', content: [{ type: 'text', text: body.system, cache_control: { type: 'ephemeral' } }] }
        : { role: 'system', content: body.system };

    // Cache what has been sent so far, so later calls in the same request do not
    // pay for the whole conversation again.
    const conversation = cacheable ? markConversationCacheable(messages) : messages;

    const withSystem = body.system ? [systemMessage, ...conversation] : conversation;

    const payload: Record<string, unknown> = {
        model: body.model || defaultModel,
        messages: withSystem,
        temperature: body.temperature === undefined ? 0.7 : parseFloat(String(body.temperature)),
        top_p: body.topP === undefined ? 0.9 : parseFloat(String(body.topP)),
    };

    // Code Chat sends plain tool definitions; providers expect each wrapped.
    if (Array.isArray(body.tools) && body.tools.length > 0) {
        payload.tools = body.tools.map(t => ({ type: 'function', function: t }));
        payload.tool_choice = 'auto';
    }

    // Streams the reply back so the answer appears as it is written.
    // Used automatically when the request has `stream: true`.
    if (body.stream) {
        payload.stream = true;
        payload.stream_options = { include_usage: true };
    }

    const send = () => fetch(url, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${apiKey}`,
        },
        body: JSON.stringify(payload),
    });

    let response = await send();
    let errorText = '';

    if (response.status >= 400) {
        errorText = await response.text();

        /*
        Some newer OpenAI models apply a default reasoning effort their
        chat/completions endpoint will not combine with function tools, and say
        so. Older models reject the argument outright, so it cannot always be
        sent — the provider's own message decides, once.
        */
        if (/reasoning_effort/i.test(errorText) && payload.reasoning_effort === undefined) {
            payload.reasoning_effort = 'none';
            response = await send();
            errorText = response.status >= 400 ? await response.text() : '';
        }
    }

    // Pass the provider's status through, so Code Chat waits and retries
    // when you are rate limited instead of giving up.
    if (response.status >= 400) {
        let message = `Request failed with status code ${response.status}`;
        try {
            const body = JSON.parse(errorText);
            if (body?.error?.message) message = body.error.message;
        } catch { /* not JSON */ }
        const retryAfter = response.headers.get('retry-after');
        return NextResponse.json({
            error: message,
            status: response.status,
            ...(retryAfter ? { retryAfter: Number(retryAfter) } : {}),
        });
    }

    if (body.stream && response.body) {
        return new NextResponse(response.body, {
            headers: {
                'Content-Type': 'text/event-stream',
                'Cache-Control': 'no-cache',
                'X-Accel-Buffering': 'no',
            },
        });
    }

    const data = await response.json();

    if (data?.error) {
        return NextResponse.json({ error: data.error.message || String(data.error) });
    }

    const message = data?.choices?.[0]?.message;
    if (!message) {
        return NextResponse.json({ error: 'no message in provider response' });
    }

    // Return the whole message: Code Chat needs `tool_calls` to continue.
    return NextResponse.json({ message, usage: data.usage });
}
```

## 2. Add this file as `src/lib/serveImageGeneration.ts`

```typescript
// src/lib/serveImageGeneration.ts
//
// Image generation via OpenRouter — one call, no queue.
//
// Unlike Fal there is no request id to poll and no result to fetch: the image
// comes back in the same response. It arrives as base64 and is saved here, so
// the browser only ever receives a URL — base64 in the page would be written
// into every save of that content.

import { NextResponse } from 'next/server';
import path from 'path';
import fs from 'fs';

const uploadPath = process.env.UPLOAD_PATH || '';   // Physical path (e.g. "/public/uploads/")
const uploadUrl = process.env.UPLOAD_URL || '';     // URL base (e.g. "/uploads/")

function randomName() {
    return 'ai-' + Math.random().toString(36).substring(2, 7);
}

// eslint-disable-next-line @typescript-eslint/no-explicit-any
export async function serveImageGeneration(body: any, apiKey?: string, provider: 'openrouter' | 'openai' = 'openrouter') {
    const references: string[] = Array.isArray(body.input_references) ? body.input_references : [];
    const isOpenAI = provider === 'openai';

    let response: Response;

    if (isOpenAI && references.length) {
        /*
        OpenAI is the only one that splits the two operations: an image derived
        from a reference goes to /images/edits as multipart form data, not JSON.
        A reference arriving as a data URL is decoded back to bytes for the form.
        */
        const form = new FormData();
        form.append('model', body.imageModel);
        form.append('prompt', body.prompt || '');
        if (body.imageSize) form.append('size', body.imageSize);      // OpenAI takes pixels

        for (let i = 0; i < references.length; i++) {
            const url = references[i];
            const blob = /^data:/i.test(url)
                ? await (await fetch(url)).blob()
                : await (await fetch(url)).blob();
            form.append('image[]', blob, `reference-${i + 1}.png`);
        }

        response = await fetch('https://api.openai.com/v1/images/edits', {
            method: 'POST',
            headers: { Authorization: `Bearer ${apiKey}` },
            body: form,
        });
    } else {
        const payload: Record<string, unknown> = {
            model: body.imageModel,
            prompt: body.prompt || '',
        };
        // OpenRouter takes an aspect ratio, OpenAI takes pixels
        if (body.imageSize) payload[isOpenAI ? 'size' : 'aspect_ratio'] = body.imageSize;
        if (!isOpenAI && references.length) {
            payload.input_references = references.map((url: string) => ({
                type: 'image_url',
                image_url: { url },
            }));
        }

        response = await fetch(
            isOpenAI ? 'https://api.openai.com/v1/images/generations' : 'https://openrouter.ai/api/v1/images',
            {
                method: 'POST',
                headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${apiKey}` },
                body: JSON.stringify(payload),
            });
    }

    const data = await response.json();

    if (!response.ok) {
        return NextResponse.json({
            error: data?.error?.message || `Request failed with status code ${response.status}`,
            status: response.status,
        });
    }

    const b64 = data?.data?.[0]?.b64_json;
    if (!b64) return NextResponse.json({ error: 'The generation returned no image.' });

    try {
        const folderPath = body?.customData?.folderPath || '';
        const filename = `${randomName()}.jpg`;
        const filePath = path.join(process.cwd(), uploadPath, folderPath, filename);

        fs.mkdirSync(path.dirname(filePath), { recursive: true });
        fs.writeFileSync(filePath, Buffer.from(b64, 'base64'));

        return NextResponse.json({ url: uploadUrl + path.posix.join(folderPath, filename) });
    } catch (e) {
        return NextResponse.json({ error: 'Could not save the generated image: ' + (e as Error).message });
    }
}
```

## 3. Using OpenRouter — replace `src/app/api/openrouter/route.ts` with this

```typescript
// src/app/api/openrouter/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { serveConversation } from '@/lib/serveConversation';
import { serveImageGeneration } from '@/lib/serveImageGeneration';

const OPENROUTER_API_KEY = process.env.OPENROUTER_API_KEY;

export async function POST(req: NextRequest) {
    try {
        const url = "https://openrouter.ai/api/v1/chat/completions";
        const DEFAULT_MODEL = 'openai/gpt-4o-mini';
        const DEFAULT_TEMPERATURE = 0.6;
        const DEFAULT_TOP_P = 0.9;
        const DEFAULT_NUM = 1;

        const body = await req.json();

        // Image generation — one call, answered from the same route so a host
        // needs no extra endpoint. Neither v1 nor v2 sends `imageModel`.
        if (body?.imageModel) {
            return await serveImageGeneration(body, OPENROUTER_API_KEY);
        }

        // chatEngine 'v2' — conversational shape. No `messages` = v1, unchanged.
        if (body?.messages) {
            return await serveConversation(body, {
                url,
                apiKey: OPENROUTER_API_KEY,
                defaultModel: DEFAULT_MODEL,
            });
        }

        const { question, context, system, functs = [], temperature, topP, num, model } = body;

        const messages = [
            { role: 'system', content: system },
            { role: 'assistant', content: context || '' },
            { role: 'user', content: question }
        ];

        try {
            if (functs.length > 0) {
                // --- Function Calling Branch ---

                const tools = [
                    {
                        type: "function",
                        function: {
                            ...functs[0]
                        }
                    }
                ];

                const response = await fetch(url, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${OPENROUTER_API_KEY}`,
                    },
                    body: JSON.stringify({
                        model: 'anthropic/claude-3.5-sonnet',
                        /*
                        Models for function calling:
                        - anthropic/claude-3.5-sonnet
                        - qwen/qwen-2.5-72b-instruct
                        */
                        messages,
                        temperature: parseFloat(temperature) || DEFAULT_TEMPERATURE,
                        top_p: parseFloat(topP) || DEFAULT_TOP_P,
                        n: parseInt(num) || DEFAULT_NUM,
                        tools,
                    })
                });

                const data = await response.json();

                if (data && data.error && data.error.message) {
                    return NextResponse.json(
                        { ok: false, status: 404, error: data.error.message },
                        { status: 200 }
                    );
                }

                const answer = functs.length === 0
                    ? data
                    : (data.choices[0].message.tool_calls
                        ? data.choices[0].message.tool_calls[0].function.arguments
                        : data.choices[0].message);
                const usage = data.usage;

                return NextResponse.json({ answer, usage });

            } else {
                // --- Content Generation Branch ---

                const response = await fetch(url, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${OPENROUTER_API_KEY}`,
                    },
                    body: JSON.stringify({
                        model: model || DEFAULT_MODEL,
                        messages,
                        temperature: parseFloat(temperature) || DEFAULT_TEMPERATURE,
                        top_p: parseFloat(topP) || DEFAULT_TOP_P,
                        n: parseInt(num) || DEFAULT_NUM
                    })
                });

                const data = await response.json();
                const answer = data;
                const usage = data.usage;

                return NextResponse.json({ answer, usage });
            }

        } catch (error) {
            return NextResponse.json(
                { ok: false, status: 500, error: error instanceof Error ? error.message : 'Unknown error' },
                { status: 200 }
            );
        }

    } catch {
        return NextResponse.json(
            { ok: true, status: 500, error: 'Something went wrong.' },
            { status: 200 }
        );
    }
}
```

## 4. Using OpenAI — replace `src/app/api/openai/route.ts` with this

```typescript
// src/app/api/openai/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { serveConversation } from '@/lib/serveConversation';
import { serveImageGeneration } from '@/lib/serveImageGeneration';

const OPENAI_API_KEY = process.env.OPENAI_API_KEY;

export async function POST(req: NextRequest) {
    try {
        const DEFAULT_MODEL = 'gpt-4o-mini';

        const body = await req.json();

        // Image generation — one call, answered from the same route so a host
        // needs no extra endpoint. Neither v1 nor v2 sends `imageModel`.
        if (body?.imageModel) {
            return await serveImageGeneration(body, OPENAI_API_KEY, 'openai');
        }

        // chatEngine 'v2' — conversational shape. No `messages` = v1, unchanged.
        // v2 always uses chat/completions, never the v1/responses endpoint.
        if (body?.messages) {
            return await serveConversation(body, {
                url: 'https://api.openai.com/v1/chat/completions',
                apiKey: OPENAI_API_KEY,
                defaultModel: DEFAULT_MODEL,
            });
        }

        const { question, context, system, functs = [], model } = body;

        const RESPONSE_ENDPOINT_MODELS = ['gpt-5.1-codex-mini', 'gpt-5.1-codex'];
        const useResponsesEndpoint = RESPONSE_ENDPOINT_MODELS.includes(model);

        const url = useResponsesEndpoint 
            ? 'https://api.openai.com/v1/responses'
            : 'https://api.openai.com/v1/chat/completions';

        const messages = [
            { role: 'system', content: system },
            { role: 'assistant', content: context || '' },
            { role: 'user', content: question }
        ];

        const requestBody = useResponsesEndpoint
            ? {
                model,
                input: messages,
            }
            : {
                model: model || DEFAULT_MODEL,
                messages,
                functions: functs.length === 0 ? undefined : functs
            };

        const openAiResponse = await fetch(url, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${OPENAI_API_KEY}`
            },
            body: JSON.stringify(requestBody)
        });

        if (!openAiResponse.ok) {
            const errorData = await openAiResponse.json();
            return NextResponse.json({ error: errorData }, { status: openAiResponse.status });
        }

        const data = await openAiResponse.json();

        let answer, usage;

        if (useResponsesEndpoint) {
            // Normalize v1/responses format to match v1/chat/completions
            type ResponseOutputItem = { type: string; role: string; content: { text: string }[] };
            const messageOutput = data.output.find((item: ResponseOutputItem) => item.type === 'message');
            answer = {
                choices: [{
                    message: {
                        role: messageOutput.role,
                        content: messageOutput.content[0].text
                    }
                }]
            };
            usage = data.usage;
        } else {
            const choices = data.choices;
            answer = functs.length === 0
                ? data
                : (choices[0]?.message?.function_call
                    ? choices[0].message.function_call.arguments
                    : choices[0]?.message);
            usage = data.usage;
        }

        return NextResponse.json({ answer, usage });

    } catch (err: unknown) {
        return NextResponse.json(
            { ok: false, error: err instanceof Error ? err.message : "Unknown error" },
            { status: 500 }
        );
    }
}
```

---

# Check it worked

```bash
curl -X POST http://localhost:8081/openrouter \
  -H 'Content-Type: application/json' \
  -d '{"model":"openai/gpt-4o-mini","system":"You edit a page.",
       "messages":[{"role":"user","content":"Say OK"}]}'
```

| You get | Meaning |
|---|---|
| `{"message":{...}}` | Working — set `chatEngine: 'v2'` |
| `{"answer":{...}}` | Not updated yet |

---

# Another language?

Return the model's whole reply as `message`, including `tool_calls`:

```json
{ "message": { "role": "assistant", "content": "...", "tool_calls": [ ... ] } }
```

When the request has `stream: true`, return the provider's `text/event-stream`
unchanged. On an error, include the status so Code Chat can wait and retry:

```json
{ "error": "Request failed with status code 429", "status": 429, "retryAfter": 20 }
```

An image request arrives with `imageModel` and `prompt` instead of `messages`.
Send it to the provider's image API, save the image, and return its URL:

```json
{ "url": "uploads/ai-a1b2c.jpg" }
```
