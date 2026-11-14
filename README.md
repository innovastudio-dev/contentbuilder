# Getting Started with ContentBuilder.js

ContentBuilder.js is a Javascript library for creating elegant, editorial-style web content with ease. It brings the feel of magazine-style design to responsive web pages with simple drag & drop.

Website: [https://innovastudio.com/contentbuilder](https://innovastudio.com/contentbuilder)

## Installation

### Via NPM

```bash
npm install @innovastudio/contentbuilder @innovastudio/contentbuilder-runtime
```

### Via CDN

Include the files directly in your HTML:

```html
<link href="contentbuilder/contentbuilder.css" rel="stylesheet">
<link href="runtime/contentbuilder-runtime.css" rel="stylesheet">

<script src="contentbuilder/contentbuilder.min.js"></script>
<script src="runtime/contentbuilder-runtime.min.js"></script>
```

## Quick Start

### Basic Usage

**Using CDN (HTML/JavaScript):**

```html
<!-- Include stylesheets -->
<link href="contentbuilder/contentbuilder.css" rel="stylesheet">
<link href="runtime/contentbuilder-runtime.css" rel="stylesheet">

<!-- Your content container -->
<div class="container"></div>

<!-- Include scripts -->
<script src="contentbuilder/contentbuilder.min.js"></script>
<script>
// Initialize ContentBuilder
const builder = new ContentBuilder({
   container: '.container'
});

// Load snippets (drag & drop blocks)
builder.loadSnippets('assets/minimalist-blocks/content.js');

// Load initial HTML content
builder.loadHtml(`
<div class="row">
    <div class="column">
        <p>Lorem Ipsum is simply dummy text.</p>
    </div>
</div>
`);

// Get the HTML content
const html = builder.html();
</script>

<!-- Initialize Runtime -->
<script src="runtime/contentbuilder-runtime.min.js"></script>
<script>
    const runtime = new ContentBuilderRuntime();
    runtime.init();
</script>
```

**Using NPM (React/Vue/Modern Frameworks):**

```javascript
// Import ContentBuilder library
import ContentBuilder from '@innovastudio/contentbuilder'
import '@innovastudio/contentbuilder/public/contentbuilder/contentbuilder.css'

// Import Runtime library
import ContentBuilderRuntime from '@innovastudio/contentbuilder-runtime'
import '@innovastudio/contentbuilder-runtime/dist/contentbuilder-runtime.css'

// Initialize ContentBuilder
const builder = new ContentBuilder({
   container: '.container'
});

// Load snippets (drag & drop blocks)
builder.loadSnippets('assets/minimalist-blocks/content.js');

// Load initial HTML content
builder.loadHtml(`
<div class="row">
    <div class="column">
        <p>Lorem Ipsum is simply dummy text.</p>
    </div>
</div>
`);

// Get the HTML content
const html = builder.html();

// Initialize Runtime
const runtime = new ContentBuilderRuntime();
runtime.init();
```

## Setting Up Assets

ContentBuilder requires certain assets (snippets, icons, etc.) to be accessible in your project. These assets are available in the ContentBuilder package (downloaded from the official website or from the examples provided in GitHub).

- Copy `/contentbuilder`, `/runtime`, and `/assets` folders to your public directory

**For advanced setup:** These assets can also be hosted on a CDN or external storage by adjusting the paths in the configuration (see full documentation for details).

## 📢 Upgrade Note for Existing Users

**Important:** If you're upgrading from a previous version, your existing implementation will continue to work without any changes.

The installation now includes two parts: the **ContentBuilder library** (for editing/building content) and the **Runtime library** (for displaying/rendering the content you create). The new runtime library is **optional but recommended** for better performance and enhanced rendering capabilities. You can upgrade at your own pace.

## Examples

### Example 1: HTML - Editing Mode

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ContentBuilder Example</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    
    <link href="contentbuilder/contentbuilder.css" rel="stylesheet">
    <link href="runtime/contentbuilder-runtime.css" rel="stylesheet">
</head>
<body>

<div class="container"></div>

<button onclick="save()">Save</button>

<script src="contentbuilder/contentbuilder.min.js"></script>
<script>
// Initialize ContentBuilder
const builder = new ContentBuilder({
    container: '.container'
});

// Load snippets
builder.loadSnippets('assets/minimalist-blocks/content.js');

// Load content
const html = localStorage.getItem('content') || `
<div class="row">
    <div class="column">
        <h1>Hello World</h1>
        <p>Start editing your content here.</p>
    </div>
</div>`;
builder.loadHtml(html);

// Save function
function save() {
    const html = builder.html();
    localStorage.setItem('content', html);
    alert('Content saved!');
}
</script>

<!-- Initialize Runtime -->
<script src="runtime/contentbuilder-runtime.min.js"></script>
<script>
    const runtime = new ContentBuilderRuntime();
    runtime.init();
</script>

</body>
</html>
```

### Example 2: HTML - Viewing Mode

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>View Content</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    
    <link href="runtime/contentbuilder-runtime.css" rel="stylesheet">
</head>
<body>

    <!-- Important: Use 'is-container' class when viewing -->
    <div class="container is-container">
        <!-- Your saved content here -->
    </div>
    
    <script src="runtime/contentbuilder-runtime.min.js"></script>
    <script>
        const runtime = new ContentBuilderRuntime();
        runtime.init();
    </script>
    
</body>
</html>
```

**Important:** When using Runtime for viewing, always add the `is-container` class to your content wrapper.

### Example 3: React - Editing Mode

```tsx
import { useEffect, useRef } from 'react'
import ContentBuilder from '@innovastudio/contentbuilder'
import '@innovastudio/contentbuilder/public/contentbuilder/contentbuilder.css'

// Runtime library
import ContentBuilderRuntime from '@innovastudio/contentbuilder-runtime'
import '@innovastudio/contentbuilder-runtime/dist/contentbuilder-runtime.css'

function App() {
    const builderRef = useRef<ContentBuilder | null>(null);

    useEffect(() => {
        // Initialize ContentBuilder
        builderRef.current = new ContentBuilder({
            container: '.container'
        });

        // Load snippets
        builderRef.current.loadSnippets('/assets/minimalist-blocks/content.js')

        // Load initial content
        const savedHtml = localStorage.getItem('content') || `
        <div class="row">
            <div class="column">
                <h1>Hello World</h1>
                <p>Start editing your content here.</p>
            </div>
        </div>`;
        builderRef.current.loadHtml(savedHtml)

        // Initialize runtime
        let runtime: ContentBuilderRuntime | null = new ContentBuilderRuntime();
        runtime.init();

        // Cleanup
        return () => {
            builderRef.current?.destroy()
            runtime?.destroy();
            runtime = null;
        };
    }, []);

    const handleSave = () => {
        if (!builderRef.current) return;
        const html = builderRef.current.html();
        localStorage.setItem('content', html)
        alert('Content saved!')
    };

    return (
        <>
            <div className="container" style={{ maxWidth: '980px', margin: '0 auto' }}></div>
            <button onClick={handleSave}>Save</button>
        </>
    );
}

export default App
```

### Example 4: React - Viewing Mode

```tsx
import { useEffect, useRef } from 'react'
import ContentBuilderRuntime from '@innovastudio/contentbuilder-runtime'
import '@innovastudio/contentbuilder-runtime/dist/contentbuilder-runtime.css'

export default function ViewPage() {
    const containerRef = useRef<HTMLDivElement | null>(null)

    useEffect(() => {
        const savedHtml = localStorage.getItem('content') || '<p>No content available.</p>'
        
        if (containerRef.current) {
            // Render content
            containerRef.current.innerHTML = ''
            const range = document.createRange()
            const fragment = range.createContextualFragment(savedHtml)
            containerRef.current.appendChild(fragment)

            // Initialize runtime
            const runtime = new ContentBuilderRuntime();
            runtime.init();

            // Cleanup
            return () => {
                runtime?.destroy();
            };
        }
    }, []);

    return (
        <div
            className="is-container"
            ref={containerRef}
            style={{ maxWidth: '980px', margin: '0 auto' }}
        ></div>
    );
}
```

## Media Upload Handling

ContentBuilder allows users to embed images, videos, audio files, or add links to uploaded documents directly within the editor. You can enable this feature by providing your own upload logic through the `upload` parameter.

### Enabling Media Upload

To enable media uploads, pass an `upload` function when initializing ContentBuilder:

```javascript
const builder = new ContentBuilder({
    container: '.container',

    upload: async (file) => {
        // Your custom file upload logic
        const formData = new FormData();
        formData.append('file', file);

        const response = await fetch('/upload', {
            method: 'POST',
            body: formData,
        });

        const data = await response.json();
        return data; // must return a JSON object containing at least { url }
    },
});
```

The `upload` function receives the selected file as a parameter. Your function should:

1. Upload the file to your server or storage service
2. Return a JSON object containing at least `{ url: 'uploaded-file-url' }`

ContentBuilder will automatically use the returned URL to embed the media in the editor.

**Note:** Your upload function can save files to your local server, AWS S3, or any other storage service of your choice. The implementation is entirely up to you.

### 📢 Upgrade Note

**For existing users:** If you're currently using `onImageUpload`, `onVideoUpload`, `onAudioUpload`, or `onFileUpload` parameters, they will continue to work without any changes. The new `upload` parameter is a simplified alternative that handles all media types with a single function. You can upgrade to the new approach at your own pace.

## Integrating a File Picker / Asset Manager

In addition to uploading new files, ContentBuilder allows users to select existing files from your server or storage. This is done by integrating a file picker or asset manager application that opens in a modal dialog.

### Enabling File Picker

```javascript
const builder = new ContentBuilder({
    container: '.container',

    // Opens the file picker for inserting images, videos, audio, or document links
    filePicker: 'assets.html',
    // You can replace this with your own file manager application,
    // or use the Files.js Asset Manager: https://innovastudio.com/asset-manager

    filePickerSize: 'large', // medium (default), large, fullscreen
});
```

The file picker opens in a modal (loaded as an iframe). This is a separate application where you can build your own custom file browser or use an existing asset manager. Users can browse and select files from your server or storage service.

### How It Works

When a user selects a file in the file picker, the picker application must return the selected file URL back to ContentBuilder by calling:

```javascript
builder.selectAsset(url); // url is the selected file URL
```

Where `url` is the path to the selected file (e.g., `'uploads/image.png'` or `'https://yourdomain.com/media/video.mp4'`).

**Example File Picker:**

A simple example (`assets.html`) is included in the ContentBuilder package. This demonstrates how file selection works and shows the proper use of the `selectAsset()` method. Note that this is just a basic demonstration with selectable images, not a full file browsing system.

**Recommended Asset Manager:**

For a production-ready file management solution, you can use [Files.js Asset Manager](https://innovastudio.com/asset-manager), which integrates seamlessly with ContentBuilder.

**Custom Implementation:**

You can build your own file picker application. The only requirement is that it must call `builder.selectAsset(url)` with the selected file URL when a file is chosen.

### 📢 Upgrade Note

**For existing users:** If you're currently using `imageSelect`, `videoSelect`, `audioSelect`, `fileSelect`, or `mediaSelect` parameters, they will continue to work without any changes. The new `filePicker` parameter is a simplified alternative that handles all media types with a single configuration. You can upgrade to the new approach at your own pace.

## AI Assistant (Code Chat)

ContentBuilder includes a **Code Chat panel** that lets users build and edit content using AI. Users can describe what they want, ask questions, and attach a screenshot or document to work from.

Code Chat needs a server-side endpoint to talk to an AI provider. ContentBuilder.js is a client-side library and works with any server platform — examples are provided in **Node.js, PHP and Next.js**, and you can implement the endpoint in any language.

### Requirements

An API key from one of these:

- **OpenRouter** – https://openrouter.ai (access to models from several providers)
- **OpenAI** – https://openai.com

### Store your API key

**Node.js / Next.js** (`.env`):

```
OPENROUTER_API_KEY=YOUR_API_KEY
OPENAI_API_KEY=YOUR_API_KEY
```

**PHP** (`config.php`):

```php
$OPENROUTER_API_KEY = 'YOUR_API_KEY';
$OPENAI_API_KEY = 'YOUR_API_KEY';
```

### Configure ContentBuilder

**Using OpenRouter:**

```javascript
const builder = new ContentBuilder({
    container: '.container',

    chatEngine: 'v2',
    chatAttachments: true,          // let users attach images, PDFs and documents

    sendCommandUrl: '/openrouter',  // your endpoint

    codeModels: [                   // models offered in Chat Settings
        { id: '~anthropic/claude-opus-latest', label: 'Claude Opus (Latest)' },
        { id: '~anthropic/claude-sonnet-latest', label: 'Claude Sonnet (Latest)' },
        { id: '~google/gemini-pro-latest', label: 'Gemini Pro (Latest)' },
        { id: '~openai/gpt-latest', label: 'GPT (Latest)' },
    ],

    defaultChatSettings: {
        codeModel: '~anthropic/claude-opus-latest',
        imageModel: 'fal-ai/nano-banana',
        imageSize: '',
    },
});
```

**Using OpenAI:**

```javascript
const builder = new ContentBuilder({
    container: '.container',

    chatEngine: 'v2',
    chatAttachments: true,

    sendCommandUrl: '/sendcommand',

    codeModels: [
        { id: 'gpt-5.6-sol', label: 'GPT-5.6 Sol' },
        { id: 'gpt-5.6-terra', label: 'GPT-5.6 Terra' },
        { id: 'gpt-5.5', label: 'GPT-5.5' },
        { id: 'gpt-5.4-mini', label: 'GPT-5.4 Mini' },
    ],

    defaultChatSettings: {
        codeModel: 'gpt-5.6-terra',
        imageModel: 'fal-ai/nano-banana',
        imageSize: '',
    },
});
```

One model does the work, and users pick it from `codeModels` in Chat Settings. The first entry is also the fallback if a saved choice is no longer in the list, so put a dependable model there.

Image generation is configured separately — see **AI Image Generation** below.

### Server-Side Implementation

Your endpoint receives requests from Code Chat and forwards them to the AI provider. Working examples for **Node.js, PHP and Next.js** are included in this package, pre-configured — add your API key and Code Chat works immediately.

Writing your own endpoint, or adding Code Chat to a server you already have? The code to paste is in
[Enabling Chat Engine v2 on your server](chat-engine-v2-server.md).

### What users get

- **Precise edits.** Asking to change one thing changes that row only — the rest of the content is left exactly as it was.
- **Questions are answered in the chat**, without touching the content.
- **Attachments.** Users can attach a screenshot to match, a photo to place, or a document to build from — see **File Attachments** below.
- **Progress as it works** — "Looking at the page", "Reading row 3", "Updating row 3" — and answers stream in as they are written.

### File Attachments

Set `chatAttachments: true` and users can attach files to a message. It works
without further configuration:

- **Images** on any model that reads images.
- **Text files** — `.txt`, `.md`, `.csv`, `.js` and similar — on **every** model.
  They are read in the browser and sent as part of the message, so they do not
  depend on the model accepting file uploads.
- **PDFs** on models that read them directly: Anthropic, Google, OpenAI and Grok.

A file type that is not offered for the selected model is refused in the
browser, before any request is sent.

#### Choosing what a model accepts

Add `accepts` to a model entry to change what may be attached for it:

```javascript
codeModels: [
    { id: 'vendor/some-model', label: 'Some Model', accepts: ['image'] },
],
```

The categories are `'image'`, `'pdf'`, `'text'` and `'office'`.

`accepts` is a policy, not a description of the model. You can allow less than
a model supports — to keep users on file types you have tested, or to hold down
the size of requests — and you never need to know a model's full capability to
use it. Declare what you are sure of; leave it out and the defaults above apply.

To offer PDFs on a model that is not one of the four above, add `'pdf'` yourself:

```javascript
    { id: 'vendor/some-model', label: 'Some Model', accepts: ['image', 'text', 'pdf'] },
```

A model with no file input has two sensible declarations, and they mean
different things:

```javascript
    { id: 'vendor/text-only-model', label: 'Text Only', accepts: [] },        // no attachments at all
    { id: 'vendor/text-only-model', label: 'Text Only', accepts: ['text'] },  // text files still work
```

`accepts: []` turns the attach button off for that model.

#### Office documents

Word, Excel and PowerPoint files are off by default. Turn them on with:

```javascript
    convertAttachments: true,
```

They are then read in the browser and sent as text, which works on any model.
Only the text comes through — tables, layout and images inside the document do
not — which is why it is opt-in rather than the default.

An `accepts` you have declared still applies: a model declared as
`accepts: ['image']` takes images only, whether or not this option is on.

If the model reads Office files itself, declare `'office'` instead of turning
conversion on. The file is then sent as it is, for the model to read directly,
and nothing is flattened:

```javascript
    { id: 'vendor/some-model', label: 'Some Model', accepts: ['image', 'text', 'office'] },
```

Use conversion when you want Office files to work everywhere; declare
`'office'` when you know the model reads them and want the full document.

#### What users see

When a file is not accepted, the user is told in the chat panel:

> notes.md: file type not accepted. Please attach images or PDF files instead.

The suggestion at the end names what is enabled for the model they have
selected, so it changes with the model.

### Already using the previous engine?

Code Chat's original engine is still there, and setting `chatEngine: 'v1'` (or omitting the option) keeps it. It uses two extra options that `'v2'` ignores:

```javascript
    systemModel: 'openai/gpt-4o-mini',   // v1 only — analyses each request
    chatModels: [ /* ... */ ],           // v1 only — model for answering questions
```

`'v2'` produces better results and costs less, and it is the engine to use for anything new. Switching needs one update to your endpoint — see the link above. Until you make that change, keep `'v1'` and everything works as before; if you switch without it, Code Chat tells you so on the first message.

**Canvas mode** stays on `'v1'`. A freeform layout has no rows for v2's row-based editing to work with, so `chatEngine: 'v2'` is ignored when `canvas: true`.

## AI Image Generation

Users can generate an image from the Code Chat panel, or ask for a change to a
photo already on the page. Pick one provider: OpenRouter, OpenAI or Fal.ai.

### Using OpenRouter or OpenAI

Set the provider and list the models to offer. Image requests go to the endpoint
Code Chat already uses, so there is nothing else to set up.

**OpenRouter:**

```javascript
const builder = new ContentBuilder({
    // ...
    sendCommandUrl: '/openrouter',

    defaultImageGenerationProvider: 'openrouter',
    imageModels: [
        { id: 'google/gemini-3-pro-image', label: 'Gemini 3 Pro Image', image_input: true,
          sizes: ['auto', '1:1', '4:3', '3:4', '16:9', '9:16'] },
        { id: 'google/gemini-3.1-flash-image', label: 'Gemini 3.1 Flash Image', image_input: true,
          sizes: ['auto', '1:1', '4:3', '3:4', '16:9', '9:16'] },
    ],
});
```

**OpenAI:**

```javascript
const builder = new ContentBuilder({
    // ...
    sendCommandUrl: '/sendcommand',

    defaultImageGenerationProvider: 'openai',
    imageModels: [
        { id: 'gpt-image-2', label: 'GPT Image 2', image_input: true,
          sizes: ['auto', '1024x1024', '1536x1024', '1024x1536', '2048x2048'] },
        { id: 'gpt-image-1-mini', label: 'GPT Image 1 Mini', image_input: true,
          sizes: ['auto', '1024x1024', '1536x1024', '1024x1536'] },
    ],
});
```

In each model entry:

- **`id`**, **`label`** — the provider's model id, and the name users see in Chat Settings.
- **`image_input: true`** — this model can work from an existing photo, so users
  can ask for a change to an image on the page. Without it the model only makes
  new images.
- **`sizes`** — the choices in the size button. Use values the provider accepts:
  aspect ratios such as `16:9` for OpenRouter, pixel dimensions such as
  `1536x1024` for OpenAI. Include `'auto'` to let the model choose — on an edit
  that keeps the original photo's shape.

Uses the API key you already set for Code Chat:

```
OPENROUTER_API_KEY=your_key
OPENAI_API_KEY=your_key
```

### Using Fal.ai

Fal.ai needs three URLs configured.

### Requirements

To enable AI image generation, you need an API key from:

**Fal.ai** – https://fal.ai

### Configuration

**Store your API key:**

**Node.js/Next.js** (`.env` file):

```
FAL_API_KEY=your_fal_api_key
```

**PHP** (`config.php` file):

```php
$FAL_API_KEY = 'your_fal_api_key';
```

**Configure ContentBuilder:**

**Node.js:**

```javascript
const builder = new ContentBuilder({
    container: '.container',
    
    defaultImageGenerationProvider: 'fal',
    generateMediaUrl_Fal: '/request-fal',
    checkRequestStatusUrl_Fal: '/status-fal',
    getResultUrl_Fal: '/result-fal',
});
```

**Next.js:**

```javascript
const builder = new ContentBuilder({
    container: '.container',
    
    defaultImageGenerationProvider: 'fal',
    generateMediaUrl_Fal: '/api/request-fal',
    checkRequestStatusUrl_Fal: '/api/status-fal',
    getResultUrl_Fal: '/api/result-fal',
});
```

**PHP:**

```javascript
const builder = new ContentBuilder({
    container: '.container',
    
    defaultImageGenerationProvider: 'fal',
    generateMediaUrl_Fal: 'api/request-fal.php',
    checkRequestStatusUrl_Fal: 'api/status-fal.php',
    getResultUrl_Fal: 'api/result-fal.php',
});
```

### Choosing Image Models

Whichever provider you use, the models you list in `imageModels` are what users
pick from in Chat Settings, along with the size choices you gave that model.

`imageModels` is optional: each provider ships with a built-in list, so leaving
it out gives users a ready set of models. Set it when you want to offer fewer
models, different ones, or your own size choices.

### Server-Side Implementation

Working examples for **Node.js, PHP and Next.js** are included in this package,
pre-configured. OpenRouter and OpenAI are served by the Code Chat route you
already have; Fal.ai uses its own three endpoints.

### 📢 Upgrade Note

**For existing users:** If you're currently using Getimg.ai for image generation (`textToImageUrl`, `upscaleImageUrl` parameters), your implementation will continue to work without any changes. The Fal.ai integration is the current recommended approach for accessing a wider range of AI models, but you can continue using your existing setup or upgrade at your own pace.

## AI Skill for Claude and ChatGPT

The `contentbuilder-plugin` skill teaches Claude (and ChatGPT) to build plugins for ContentBuilder and ContentBox. Ask for one, for example *"Create a pricing table plugin with monthly and yearly prices"*, and you get the plugin folder (`index.js`, `style.css`, `README.md`), ready to drop into `assets/plugins/`.

### Install

**Claude app (claude.ai):** download [contentbuilder-plugin.zip](skills/contentbuilder-plugin.zip), then go to Settings > Capabilities > Skills and upload it.

**Claude Code:** copy the [skills/contentbuilder-plugin](skills/contentbuilder-plugin) folder into `~/.claude/skills/` (for all projects) or into your project's `.claude/skills/`.

**ChatGPT:** open Skills, choose Create, then Upload from your computer, and select the zip.

### Use

Ask for a plugin. Put the generated folder in `assets/plugins/` and paste the HTML from its `README.md` into a page.

## Resources

- **[Plugin Development Guide](plugin-development.md)** - Learn how to create custom plugins for ContentBuilder
- **[Configuration Reference](reference.md)**
- **[Keyboard Access](keyboard-access.md)** - Using the editor without a mouse
- **Product Website** - https://innovastudio.com/contentbuilder
- **GitHub Repository** - https://github.com/innovastudio-dev/contentbuilder
