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

## Alternative: Without Runtime Library

If you prefer not to use the runtime library, you can still use the basic content stylesheet:

```javascript
// For viewing only basic content
import '@innovastudio/contentbuilder/public/contentbuilder/content.css'
```

Or in HTML:

```html
<link href="contentbuilder/content.css" rel="stylesheet">
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

## AI Assistant Integration

ContentBuilder includes an **AI Assistant panel** that allows users to generate and edit content using AI. This feature requires a server-side endpoint to communicate with AI providers like OpenAI or OpenRouter.

**Important:** ContentBuilder.js is a client-side library and works with any server platform. The AI Assistant feature simply requires you to create an API endpoint on your server. Examples are provided in **Node.js, Next.js, and PHP** to cover common scenarios, but you can implement the endpoint in any server-side language or framework.

### Requirements

You'll need an API key from one of these services:

- **OpenAI** – https://openai.com
- **OpenRouter** – https://openrouter.ai (provides access to multiple AI models)

### Configuration

**Store your API key:**

**Node.js/Next.js** (`.env` file):

```
OPENAI_API_KEY=your_openai_api_key
```

**PHP** (`config.php` file):

```php
$OPENAI_API_KEY = 'your_openai_api_key';
```

**Configure ContentBuilder:**

**Node.js:**

```javascript
const builder = new ContentBuilder({
    container: '.container',
    sendCommandUrl: '/sendcommand' // Your server endpoint for handling AI requests
});
```

**Next.js:**

```javascript
const builder = new ContentBuilder({
    container: '.container',
    sendCommandUrl: '/api/sendcommand' // Next.js API route
});
```

**PHP:**

```javascript
const builder = new ContentBuilder({
    container: '.container',
    sendCommandUrl: 'api/sendcommand.php' // PHP endpoint
});
```

### Server-Side Implementation

Your endpoint receives AI requests from ContentBuilder and forwards them to the AI provider. Server-side endpoint examples are provided in **Node.js, Next.js, and PHP** within the ContentBuilder package and on GitHub.

The examples included in the ContentBuilder package have this pre-configured, so you can test the AI Assistant immediately after adding your API key.

## AI Image Generation

ContentBuilder supports AI-powered image generation directly from the AI Assistant panel. Users can type a text prompt to generate images that are automatically embedded into their content.

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

Fal.ai provides access to multiple AI image models, from Google's latest models to FLUX variants. Users can select their preferred model and image size/orientation from the AI Assistant settings panel.

### Server-Side Implementation

Server-side endpoint examples for handling image generation requests are provided in **Node.js, Next.js, and PHP** within the ContentBuilder package and on GitHub.

### 📢 Upgrade Note

**For existing users:** If you're currently using Getimg.ai for image generation (`textToImageUrl`, `upscaleImageUrl` parameters), your implementation will continue to work without any changes. The Fal.ai integration is the current recommended approach for accessing a wider range of AI models, but you can continue using your existing setup or upgrade at your own pace.

## Additional Resources

- **Product Website** - https://innovastudio.com/contentbuilder
- **GitHub Repository** - https://github.com/innovastudio-dev/contentbuilder
