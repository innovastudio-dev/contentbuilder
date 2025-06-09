# ContentBuilder.js Documentation

**ContentBuilder.js is a JavaScript library for creating elegant, editorial-style web content with ease. It brings the feel of magazine-style design to responsive web pages with simple drag & drop.**

Website: [https://innovastudio.com/contentbuilder](https://innovastudio.com/contentbuilder)

Demo: [https://demo.innovastudio.com/contentbuilder/](https://demo.innovastudio.com/contentbuilder/)

Other Resources:

[Using ContentBuilder.js in a React Project](https://innovastudio.com/using-contentbuilder-in-a-react-project)

[Using ContentBuilder.js in a Next.js 14 Project](https://innovastudio.com/using-contentbuilder-in-a-nextjs-project)

[Using ContentBuilder.js in a Laravel Project](https://innovastudio.com/using-contentbuilder-in-a-laravel-project)

## Table of Contents
- [**Getting Started**](#-getting-started)
- [**Usage**](#-usage)
- [**API Methods**](#-api-methods)
- [**Example Page**](#-example-page)
- [**Media Upload Handling**](#-media-upload-handling)
- [**Integrating a Custom File/Asset Picker**](#-integrating-a-custom-fileasset-picker)
- [**Snippets Management**](#-snippets-management)
- [**Lightbox, Form & Slider Blocks**](#%EF%B8%8F-lightbox-form--slider-blocks)
- [**AI Assistant Integration**](#-ai-assistant-integration)
- [**Language File Support**](#-language-file-support)
- [**Plugins**](#-plugins)
- [**Modules**](#-modules)
- [**CSS Framework Support**](#-css-framework-support)
- [**Canvas Mode**](#-canvas-mode)
- [**Themes**](#-themes)
- [**Multiple Editable Area Support**](#-multiple-editable-area-support)
- [**Programmatically Add Editable Area**](#-programmatically-add-editable-area)
- [**Other Configuration Options**](#-other-configuration-options)
- [**Methods**](#%EF%B8%8F-methods)
- [**Event Handling**](#-event-handling)
- [**Toolbar**](#-toolbar)
- [**Customize Editing Controls**](#-customize-editing-controls)
- [**Path Configuration**](#-path-configuration)

## **🚀 Getting Started**

### **Step 1: Installation**

### **Using NPM:**

```jsx
npm install @innovastudio/contentbuilder
```

Then import into your project:

```jsx
import ContentBuilder from '@innovastudio/contentbuilder';
import '@innovastudio/contentbuilder/public/contentbuilder/contentbuilder.css';
```

### **Or include in your HTML:**

```jsx
<!-- CSS -->
<link rel="stylesheet" href="contentbuilder/contentbuilder.css" />
<!-- JS -->
<script src="contentbuilder/contentbuilder.min.js"></script>
```

### **Step 2: Rendering Output**

To display content created or exported by the builder, include the following stylesheet in your output page:

```jsx
<link rel="stylesheet" href="assets/minimalist-blocks/content.css" />
```

This ensures that the generated HTML renders correctly with appropriate styling.

### **Step 3: Assets**

ContentBuilder requires various assets such as **snippets**, icons, images, and stylesheets. These are located inside the **`public/assets/`** folder of the package.

**Action Required:**

Copy the **`assets`** folder into the root of your public project directory or host it via a CDN.

## **🛠 Usage**

### **HTML Structure**

Add a container element where the builder will be initialized:

```jsx
<div class="container"></div>
```

### **JavaScript Initialization**

Initialize the ContentBuilder instance:

```jsx
const builder = new ContentBuilder({
	container: '.container' // Selector for the container element
});
```

## **💡 API Methods**

### **Get Generated HTML**

Retrieve the current HTML content built inside the editor:

```jsx
const html = builder.html();
```

You can then send this HTML to a server for saving.

### **Load HTML into Builder**

Load existing HTML content into the builder for editing:

```jsx
builder.loadHtml(`
<div class="row">
	<div class="column">
		<h1 class="leading-none font-medium size-64">Hi There..</h1>
	</div>
</div>
`);
```

### **Load Snippet File**

Load predefined block snippets (e.g., from **`content.js`**):

```jsx
builder.loadSnippets('assets/minimalist-blocks/content.js', true);
```

The second parameter is required and controls whether the snippets panel should auto-open after loading: true means auto-open.

## **🧪 Example Page**

Here’s a complete example showing how to integrate ContentBuilder.js into an HTML file:

```jsx
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <title>Example</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <link rel="shortcut icon" href="#"> 

    <link href="assets/minimalist-blocks/content.css" rel="stylesheet">
    
    <link href="contentbuilder/contentbuilder.css" rel="stylesheet">

    <style>
        .container {
            margin: 150px auto;
            max-width: 960px;
            width: 100%;
            padding: 0 20px;
            box-sizing: border-box;
        }
    </style>
</head>
<body>

<div class="container"></div>

<script src="contentbuilder/contentbuilder.min.js"></script>
<script>
const builder = new ContentBuilder({
    container: '.container'
});
    
builder.loadSnippets('assets/minimalist-blocks/content.js', true);

html = `
    <div class="row">
        <div class="column">
            <h1>Title here</h1>
        </div>
    </div>`;
builder.loadHTML(html);
</script>

</body>
</html>
```

## **📷 Media Upload Handling**

ContentBuilder.js provides built-in support for media uploads such as images, videos, audio files, and general documents. These are handled through customizable event hooks that allow you to integrate your own upload logic.

### **Enabling Image Uploads**

To enable image uploads in the editor, use the **`onImageUpload`** option when initializing the ContentBuilder instance:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    onImageUpload: async (e) => {
        const data = await uploadFile(e);
        const uploadedFileUrl = data.url;
        builder.returnUrl(uploadedFileUrl);
    }
});
```

The **`onImageUpload`** function receives a file input event. You can process the selected file using your preferred upload method and return the resulting URL back to the editor using **`builder.returnUrl(url)`**.

### **Example Upload Helper Function**

This helper function uploads the selected file to the server and returns the JSON response:

```jsx
async function uploadFile(e) {
    const selectedFile = e.target.files[0];
    const formData = new FormData();
    formData.append('file', selectedFile);

    const response = await fetch('/upload', {
        method: 'POST',
        body: formData,
    });

    const data = await response.json();
    return data;
}
```

### **Other Media Upload Handlers**

In addition to image uploads, ContentBuilder supports several other media types via similar hooks:

- **`onVideoUpload`** – For video files.
- **`onAudioUpload`** – For audio files.
- **`onMediaUpload`** – Used for galleries where both images and videos are allowed.
- **`onFileUpload`** – For general document uploads (PDFs, Word docs, etc.).

Each of these follows the same structure as **`onImageUpload`**. Here's how you can implement them:

```jsx
const builder = new ContentBuilder({
    container: '.container',

    onImageUpload: async (e) => {
        const data = await uploadFile(e);
        builder.returnUrl(data.url);
    },

    onVideoUpload: async (e) => {
        const data = await uploadFile(e);
        builder.returnUrl(data.url);
    },

    onAudioUpload: async (e) => {
        const data = await uploadFile(e);
        builder.returnUrl(data.url);
    },

    onMediaUpload: async (e) => {
        const data = await uploadFile(e);
        builder.returnUrl(data.url);
    },

    onFileUpload: async (e) => {
        const data = await uploadFile(e);
        builder.returnUrl(data.url);
    }
});
```

With these upload handlers in place, ContentBuilder becomes a fully functional editorial tool that allows content creators to embed rich media directly into their layouts.

## **📁 Integrating a Custom File/Asset Picker**

ContentBuilder.js allows you to integrate your own custom file picker or asset manager for media selection through the following configuration options:

- **`imageSelect`** – URL to your image picker.
- **`videoSelect`** – URL to your video picker.
- **`audioSelect`** – URL to your audio picker.
- **`fileSelect`** – URL to your general file picker.
- **`mediaSelect`** – URL to a combined media picker (for both images and videos).

These options are typically used to open an external asset/file picker in a dialog, where users can browse and select existing files from your server or CMS.

### **Basic Integration Example**

```jsx
let builder = new ContentBuilder({
    container: '.container',

    imageSelect: 'assets.html',
    videoSelect: 'assets.html',
    audioSelect: 'assets.html',
    fileSelect: 'assets.html',
    mediaSelect: 'assets.html',
});
```

In the above code, we're using **`assets.html`** as the file picker. This is a simple example provided in the package that lets users select media files. When a user selects a file, it returns the selected file's URL back to the editor using:

```jsx
parent.selectAsset(url);
```

Where **`url`** is the path to the selected file (e.g., **`'uploads/image.png'`**).

This minimal integration enables basic media insertion functionality. You can extend this file picker or replace it with your own custom implementation — just make sure to call **`parent.selectAsset(url)`** with the correct file URL when a file is selected.

### **Recommendation: Use Files.js Asset Manager**

For a more powerful and ready-to-use solution, you can use [Files.js – Asset Manager](https://innovastudio.com/asset-manager) , which integrates seamlessly with ContentBuilder.js and replaces the simple **`assets.html`** file picker with a full-featured media browser.

## **🧩 Snippets Management**

Snippets are pre-designed content blocks that you can add or drag & drop into your content.

When you add a content block by clicking the **(+) icon** , you can also select the **MORE** button to open the snippets dialog.

Snippet files are located in the folder:

```jsx
assets/minimalist-blocks/
```

This folder contains:

- **`content.js`** – The file containing the available snippets
- **`content.css`** – Styles used by the snippets
- **`images/`** – Image assets used in the snippets

### **Configuring Snippet Paths**

If you host your assets remotely (e.g., on a CDN), you can configure the paths during initialization:

```jsx
const builder = new ContentBuilder({
    container: '.container',

    snippetUrl: 'https://path.to/assets/minimalist-blocks/content.js',  // Full URL to snippet file
    snippetPath: 'https://path.to/assets/minimalist-blocks/',  // Base path for images and resources
    snippetPathReplace: [
        'assets/minimalist-blocks/images/',
        'https://path.to/assets/minimalist-blocks/images/' 
    ] // Replace local image paths with remote URLs
});
```

These options ensure that all relative paths inside the snippets (especially image URLs) resolve correctly when hosted externally.

### **Positioning the Snippets Modal**

You can control where the snippets panel appears using the **`snippetModalLeft`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    
    snippetModalLeft: true // Position snippets modal on the left side
});
```

- By default, if **not set** , the snippets modal appears on the **right** side of the workspace.
- Set **`snippetModalLeft: true`** to display it on the **left** instead.

### **Loading Snippets**

To load the snippets panel dynamically, use the **`loadSnippets()`** method:

```jsx
builder.loadSnippets('https://path.to/assets/minimalist-blocks/content.js', true);
```

The first argument is the URL to the **`content.js`** file. You can optionally set the second parameter to **`true`** to auto-open the snippets panel after loading.

### **Customizing Snippets**

You can add or edit snippets by modifying the **`content.js`** file. It contains the list of available snippets in JSON format.

Example of a snippet object:

```jsx
{
    'thumbnail': 'preview/preview-01.png',
    'category': '120',
    'html': `
        <h1>Heading 1 Text Goes Here.</h1>
        <p>Lorem Ipsum is simply dummy text...</p>
    `
}
```

Each snippet includes:

- **`thumbnail`** – Relative path to the preview image.
- **`category`** – The category ID this snippet belongs to.
- **`html`** – The HTML markup for the block.

### **Control Editable Behavior in Snippets**

You can add **special behaviors** to elements inside your snippets using data attributes.

**1. Prevent Image Replacement (`data-fixed`)**

To make an image **not replaceable** , add the **`data-fixed`** attribute to the **`<img>`** tag:

```jsx
<img src="image.png" data-fixed />
```

**2. Make a Column Non-editable (`data-noedit`)**

To prevent content editing inside a column, use the **`data-noedit`** attribute:

```jsx
<div class="row">
    <div class="column" data-noedit>
        This content cannot be edited.
    </div>
</div>
```

**3. Protect a Column Fully (`data-protected`)**

```jsx
<div class="row">
    <div class="column" data-protected>
        This column is fully protected.
    </div>
</div>
```

### **Snippet Categories**

Snippets are organized into categories. You can define custom categories or use the built-in ones using the **`snippetCategories`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',

    snippetCategories: [
        [120,'Basic'],
        [118,'Article'],
        [101,'Headline'],
        [119,'Buttons'],
        [102,'Photos'],
        [103,'Profile'],
        [116,'Contact'],
        [104,'Products'],
        [105,'Features'],
        [106,'Process'],
        [107,'Pricing'],
        [108,'Skills'],
        [109,'Achievements'],
        [110,'Quotes'],
        [111,'Partners'],
        [112,'As Featured On'],
        [113,'Page Not Found'],
        [114,'Coming Soon'],
        [115,'Help, FAQ']
    ],

    defaultSnippetCategory: 120 // Default category shown when opening the snippets dialog
});
```

The **`defaultSnippetCategory`** defines which category is displayed first when the snippets dialog opens.

## **🖼️ Lightbox, Form & Slider Blocks**

ContentBuilder.js supports interactive content blocks like **Forms** , **Sliders** , and **Lightbox-enabled images** .

However, to ensure these blocks work correctly when viewed on a live page, you must include the required **JavaScript and CSS files** in your output HTML. 

Here are the full script and stylesheet includes needed to enable these features:

```jsx
<!-- Glide Slider (for Slider block) -->
<link href="assets/scripts/glide/css/glide.core.css" rel="stylesheet">
<link href="assets/scripts/glide/css/glide.theme.css" rel="stylesheet">
<script src="assets/scripts/glide/glide.js"></script>

<!-- Form Viewer (for Form block) -->
<link href="assets/scripts/formviewer/formviewer.css" rel="stylesheet">
<script src="assets/scripts/formviewer/formviewer.min.js"></script>

<!-- Lightbox (for image or link lightbox effect) -->
<link href="assets/scripts/lightbox/lightbox.css" rel="stylesheet">
<script src="assets/scripts/lightbox/lightbox.min.js"></script>
<script>
    lightbox.init(); // Initialize lightbox
</script>
```

⚠️ **Note:** The lightbox script is only required on the **output/viewing page** — not in the editor itself.

These scripts and styles are included in the ContentBuilder package under:

```jsx
assets/scripts/
```

## **🤖 AI Assistant Integration**

ContentBuilder.js includes an **AI Assistant panel** that allows users to generate editable content using AI.

To use the AI Assistant, you need a **server-side endpoint** that handles communication with an AI  provider (like OpenAI or OpenRouter).

You can configure this endpoint using the **`sendCommandUrl`** option:

```jsx
const builder = new ContentBuilder({
    // ...
    sendCommandUrl: '/sendcommand' // Server endpoint for handling AI commands
});
```

The package includes ready-to-use examples in **Node.js** and **PHP** , making it easy to set up and test the AI Assistant.

### **Steps to Use the AI Assistant**

**1. Get an API Key**

You'll need an API key from one of the following services:

- **OpenAI** – [https://openai.com](https://openai.com/)
- **OpenRouter** – [https://openrouter.ai](https://openrouter.ai/)
    
    > Use OpenRouter if you want access to multiple AI models beyond just OpenAI.
    > 

**2. Configure Your API Key**

Update your environment file with the appropriate key:

**For Node.js:**

Edit **`.env`**:

```jsx
OPENAI_API_KEY=your_openai_api_key
# or
OPENROUTER_API_KEY=your_openrouter_api_key
```

**For PHP:**

Edit **`api/config.php`**:

```jsx
$OPENAI_API_KEY = 'your_openai_api_key';
// or
$OPENROUTER_API_KEY = 'your_openrouter_api_key';
```

**3. Set the Correct Endpoint**

Make sure the **`sendCommandUrl`** points to your server-side endpoint URL:

```jsx
const builder = new ContentBuilder({
    // ...
    sendCommandUrl: '/sendcommand', // default endpoint for AI commands
    // sendCommandUrl: '/openrouter', // alternative for OpenRouter
});
```

**For PHP:**

```jsx
const builder = new ContentBuilder({
    // ...
    sendCommandUrl: 'api/sendcommand.php', // PHP backend
    // sendCommandUrl: 'api/openrouter.php', // OpenRouter support
});
```

Most examples in the package already have this configured, so you can run and test them right away.

💡 The JavaScript example uses **Node.js** (**`server.js`**) as the backend for the **`/sendcommand`** endpoint. 

### **Optional: AI Image Generation**

ContentBuilder also supports AI-generated images via external API.

**1. Get an API Key**

🔗 [https://getimg.ai/tools/api](https://getimg.ai/tools/api)

**2. Configure Your API Key**

**For Node.js:**

Edit **`.env`**:

```jsx
GETIMG_API_KEY=your_getimg_api_key
```

**For PHP:**

Edit **`api/config.php`**:

```jsx
$GETIMG_API_KEY = 'your_getimg_api_key';
```

**3. Set the Endpoints and Model**

```jsx
const builder = new ContentBuilder({
    // ...
    textToImageUrl: '/texttoimage', 
    upscaleImageUrl: '/upscaleimage',
    imageModel: 'flux-schnell',
});
```

**For PHP:**

```jsx
const builder = new ContentBuilder({
    // ...
    textToImageUrl: 'api/texttoimage.php', 
    upscaleImageUrl: 'api/upscaleimage.php',
    imageModel: 'flux-schnell',
});
```

Once configured, users can generate images directly from the AI Assistant panel.

## **🔤 Language File Support**

ContentBuilder.js supports interface translation through language files. This allows you to display the editor UI in different languages.

### **Provided Language Files**

The package includes the following language files:

```jsx
contentbuilder/lang/en.js
contentbuilder/lang/fr.js
```

You can create your own language file by copying and modifying an existing one, then translating the values accordingly.

### **Enabling a Language**

To enable a language, include the corresponding language file **before** loading **`contentbuilder.min.js`**:

```jsx
<script src="contentbuilder/lang/fr.js" type="text/javascript"></script>
```

### **Using with React, Vue or JavaScript Projects**

If you're using a frontend framework, you can import and pass the language object directly during initialization.
First, create your language file (e.g. **`locales/en.js`**) like this:

```java
const _txt = new Array();
_txt["Bold"] = "Bold";
_txt["Italic"] = "Italic";
// ...
export default _txt;
```

Then import it into your project and assign it when creating the ContentBuilder instance:

```jsx
import texts from '../Locales/fr';
```

```jsx
builder = new ContentBuilder({
    container: '.container',
    lang: texts
});
```

With this system, you can easily localize the entire ContentBuilder.js interface to match your users' preferred language.

## **🔌 Plugins**

With plugins, you can extend the functionality of ContentBuilder.js. A plugin can add a custom button to the editor toolbar and define its behavior.

For example, the built-in **`wordcount`** plugin adds a button to the toolbar that, when clicked, displays a dialog showing the word count of the current content.

### **Plugin Location**

Plugin scripts are stored in:

```jsx
contentbuilder/plugins/
```

Each plugin resides in its own folder. The package includes several sample plugins:

- **`contentbuilder/plugins/helloworld`**
- **`contentbuilder/plugins/preview`**
- **`contentbuilder/plugins/symbols`**
- **`contentbuilder/plugins/wordcount`**

### **Loading Plugins**

To load plugins, include them in the **`plugins`** array during initialization:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    plugins: [
        { name: 'preview', showInMainToolbar: true, showInElementToolbar: true },
        { name: 'wordcount', showInMainToolbar: true, showInElementToolbar: true },
        { name: 'symbols', showInMainToolbar: true, showInElementToolbar: false }
    ],
    pluginPath: 'contentbuilder/' // Path to plugin folder
});
```

### **Plugin Options:**

- **`name`**: Name of the plugin (must match the folder name)
- **`showInMainToolbar`**: If **`true`**, the plugin button appears on the **main toolbar** (when text is selected)
- **`showInElementToolbar`**: If **`true`**, the plugin button appears on the **element toolbar** (when a non-text element is selected)

### **Plugin Development**

You can create your own plugins by following the [ContentBuilder.js Plugin Guide](https://innovastudio.com/contentbuilder-plugins) . Each plugin must be placed in a folder under **`contentbuilder/plugins/[plugin-name]`** and must contain a **`plugin.js`** file.

The **`plugin.js`** file typically uses **`_cb.addButton()`** or **`_cb.addButton2()`** to register buttons in the editor interface.

Example:

```jsx
(function () {
    if (typeof _cb === 'undefined') return;

    var button_html = `<button id="my_plugin_button" title="My Plugin">
        <svg>...</svg>
    </button>`;

    // Add a button to the main toolbar
    _cb.addButton('myplugin', button_html, '#my_plugin_button', function () {
        alert('Plugin clicked!');
    });

    // Optionally add the same button to the element toolbar
    _cb.addButton2('myplugin', button_html, '#my_plugin_button', function () {
        alert('Plugin clicked on element toolbar!');
    });
})();
```

This code:

- Checks if **`_cb`** (the ContentBuilder instance) exists
- Defines the HTML for a button
- Uses **`_cb.addButton()`** to place the button in the main toolbar
- Uses **`_cb.addButton2()`** to optionally place it in the element toolbar
- Defines what happens when the button is clicked

### **Creating Your Own Plugin**

If you want to build a custom plugin, you'll need to:

1. Create a new folder under **`contentbuilder/plugins/your-plugin-name/`**
2. Add a **`plugin.js`** file inside it
3. Use **`_cb.addButton()`** or **`_cb.addButton2()`** to register a button
4. Define the behavior (what happens when the user clicks it)

For full guidance, refer to the official guide:
🔗 [https://innovastudio.com/contentbuilder-plugins](https://innovastudio.com/contentbuilder-plugins)

## **🧩 Modules**

With modules, you can extend **snippets** — the reusable content blocks that you drag and drop into your content.

Normally, snippets are defined as **static HTML** . With a **module snippet** , however, you can create **dynamic** and **interactive** content blocks for ContentBuilder.js.

### **How to Use a Module in a Snippet**

You define a module-based snippet inside your **`content.js`** file like this:

```jsx
{
    'thumbnail': 'preview/element-code.png',
    'category': '120',
    'html':
        `<div class="row">
            <div class="column" 
                 data-html="${encodeURIComponent(`
                    <h2>My Custom Module</h2>
                    <div id="number{id}">10</div>
                    <button id="button{id}">Increment ++</button>
                    <script>
                        var docReady = function(fn) {
                            var stateCheck = setInterval(function() {
                                if (document.readyState !== "complete") return;
                                clearInterval(stateCheck);
                                fn();
                            }, 1);
                        };

                        docReady(function () {
                            var btn = document.querySelector('#button{id}');
                            btn.addEventListener('click', function(e) {
                                var div = document.querySelector('#number{id}');
                                div.innerHTML = parseInt(div.innerText) + 1;
                            });
                        });
                    </script>
                `)}">
            </div>
        </div>`
}
```

In this example:

- The **`data-html`** attribute contains the actual HTML and script of the module.
- **`encodeURIComponent()`** is used to safely encode the HTML content.
- **`{id}`** is a placeholder that will be replaced with a unique ID each time the block is added to the page — ensuring multiple instances don’t conflict.

<aside>
💡

**With custom Javascript code, you can implement, for example, a data fetching from the server (database) and then display the result in an element in your block.**

</aside>

### **Using Custom Tags with onAdd**

Sometimes you need to inject values at runtime — such as server-generated IDs, or dynamic content or values.

You can use **custom tags** like **`{recaptchakey}`** inside your module and replace them using the **`onAdd`** event:

```jsx
<script>
     const builder = new ContentBuilder({
          container: '.container',
          onAdd: (html) => {
               html = html.replace(new RegExp(encodeURIComponent('{recaptchakey}'), 'g'), 
               '<your-recaptcha-site-key>');
               return html;
          },
     });
</script>
```

This ensures that variable values are not hard-coded into your snippets.

### **Module Structure Overview**

A module is made up of two main parts:

1. **The Snippet Definition** – Defined in your **`content.js`** file.
2. **The Settings Dialog** – A standalone **`.html`** file located in **`assets/modules/`**.

Let’s break down each part using a working example.

### **Example: Render Dynamic Content Using `fetch()`**

This example shows how to create a content block that loads data from an external API and displays it. It also includes a settings dialog so users can change the URL or customize the description — making the block dynamic and reusable.

**1. Snippet Definition in `content.js`**

```jsx
{
'thumbnail': 'preview/dynamic-content.png',
'category': '120',
'html':
  `<div class="row">
	    <div class="column" data-module="my-module" data-dialog-width="500px" data-module-desc="My Module" data-html="${encodeURIComponent(`
	
	      <div><b>Dynamic Content From Fetch</b></div>
	
	      <h3 id="post_title_{id}"></h3>
	      <div id="post_body_{id}"></div>
	
	      <script>
	          var docReady = function(fn) {
	              var stateCheck = setInterval(function () {
	                  if (document.readyState !== "complete") return;
	                  clearInterval(stateCheck);
	                  fn()
	              }, 1);
	          };
	          docReady(async function() {
	              const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
	              const data = await response.json();
	              document.getElementById("post_title_{id}").innerHTML = data.title;
	              document.getElementById("post_body_{id}").innerHTML= data.body;
	          });
	      </script>
	      `)}" data-settings="${encodeURIComponent(`
	          {
	          "description": "Dynamic Content From Fetch", 
	          "requestUrl": "https://jsonplaceholder.typicode.com/posts/1"
	          }
	      `)}"
	      >
	
	    </div>
  </div>`
}
```

**Important Things to Note:**

- **`data-module="my-module"`** tells ContentBuilder to use the **`my-module.html`** file from the **`assets/modules/`** folder for settings dialog.
- **`data-dialog-width`** sets the width of the settings dialog.
- **`data-module-desc`** sets the description at the top of the dialog.
- **`data-html`** contains the HTML and script that will be rendered on the page. This must be encoded using **`encodeURIComponent()`**.
- **`data-settings`** stores the configuration values for the block, also encoded.

When this block is added to the editor, it will immediately fetch data from the given URL and display it.

**2. Settings Dialog (`assets/modules/my-module.html`)**

```jsx
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <title></title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <style>
        /* style here */
    </style>
</head>
<body>

    <label for="inpDesc">Description:</label>
    <input id="inpDesc" type="text" value="" />

    <label for="inpRequestUrl">Request URL:</label>
    <input id="inpRequestUrl" type="text" value="" />
  
    <script>
        const activeModule = parent._cb.getActiveModule();

        const settings = parent._cb.getModuleSettings();
        if (settings != 'undefined') {
            try {
                const json = JSON.parse(settings);
                document.querySelector("#inpDesc").value = json.description; 
                document.querySelector("#inpRequestUrl").value = json.requestUrl; 
            } catch (e) { }
        }

        function construct() {

            const desc = document.querySelector("#inpDesc").value;
            const requestUrl = document.querySelector("#inpRequestUrl").value;
            
            const html = `
                <div><b>${desc}</b></div>

                <h3 id="post_title_{id}"></h3>
                <div id="post_body_{id}"></div>

                <scr` + `ipt>
                    var docReady = function(fn) {
                        var stateCheck = setInterval(function () {
                            if (document.readyState !== "complete") return;
                            clearInterval(stateCheck);
                            fn()
                        }, 1);
                    };
                    docReady(async function() {
                        const response = await fetch('${requestUrl}');
                        const data = await response.json();
                        document.getElementById("post_title_{id}").innerHTML = data.title;
                        document.getElementById("post_body_{id}").innerHTML= data.body;
                    });
                </scr` + `ipt>`;

            parent._cb.setModuleHtml(html); 

            const settings = {};
            settings.description = desc;
            settings.requestUrl = requestUrl;

            parent._cb.setModuleSettings(JSON.stringify(settings));
        }
    </script>

</body>
</html>

```

**Important Things to Note:**

- When you click the **gear icon** on the block in the editor, this file opens as a dialog.
- **`getModuleSettings()`** retrieves the custom module settings.
- The **`construct()`** function is required — it builds the updated HTML based on user input.
- **`setModuleHtml()`** updates the block with the new HTML.
- **`setModuleSettings()`** saves the new values so they persist when the block is reloaded or edited again.

This means users can open the dialog, change the settings, and see their changes reflected instantly in the embedded content block.

With this structure, you can build powerful, interactive, and customizable content blocks that bring your web pages to life — all within the ContentBuilder.js interface.

## **🎨 CSS Framework Support**

ContentBuilder.js works seamlessly with popular CSS frameworks like Bootstrap, Tailwind, Foundation, and more. It allows you to define grid classes so the editor generates markup compatible with your chosen framework — without modifying individual snippets.

You can specify the CSS classes used for rows and columns using the **`row`** and **`cols`** options when initializing ContentBuilder.

**Bootstrap Example**

```jsx
const builder = new ContentBuilder({
    container: '.container',
    row: 'row',
    cols: [
        'col-md-1', 'col-md-2', 'col-md-3', 'col-md-4', 'col-md-5', 'col-md-6',
        'col-md-7', 'col-md-8', 'col-md-9', 'col-md-10', 'col-md-11', 'col-md-12'
    ]
});
```

This tells ContentBuilder to use Bootstrap's grid system when building content.

**Tailwind Example**

```jsx
const builder = new ContentBuilder({
    container: '.container',
    row: 'flex flex-col md:flex-row',
    cols: [
        'w-full md:w-1/12 px-4',
        'w-full md:w-2/12 px-4',
        'w-full md:w-3/12 px-4',
        'w-full md:w-4/12 px-4',
        'w-full md:w-5/12 px-4',
        'w-full md:w-6/12 px-4',
        'w-full md:w-7/12 px-4',
        'w-full md:w-8/12 px-4',
        'w-full md:w-9/12 px-4',
        'w-full md:w-10/12 px-4',
        'w-full md:w-11/12 px-4',
        'w-full px-4'
    ]
});
```

Here, we're using Tailwind’s utility classes to create a responsive flex-based grid layout.

**Benefits of This Approach**

- Snippets automatically adapt to your framework’s grid system.
- No need to modify **`content.js`** or other snippet files when switching frameworks.
- Works with any CSS framework that supports a grid or flexbox system.

**Notes**

While ContentBuilder supports most CSS grid frameworks, some may require additional CSS adjustments to ensure full compatibility.

<aside>
💡

Integrating the grid system from a CSS framework like Bootstrap or Tailwind is optional. ContentBuilder already includes its own built-in grid system, which offers flexible cell resizing and configurable screen size targeting. This means users can still take advantage of a CSS framework’s utilities and components, without having to integrate the grid system or configure the **`row`** and **`cols`** options. 

</aside>

## **🎨 Canvas Mode**

Canvas mode is an option in ContentBuilder.js that allows users to **freely position and arrange content blocks** anywhere on the page.

To activate Canvas mode, set the **`canvas`** option to **`true`** and provide a **`previewURL`** when initializing ContentBuilder:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    canvas: true,
    previewURL: 'preview-canvas.html'
});
```

- **`canvas: true`** – Enables free positioning of blocks.
- **`previewURL`** – The URL to a standalone HTML file used to preview the canvas-based content (provided in the package). This file includes all necessary styles and scripts to render the output correctly.

To support Canvas mode, you must include the following CSS and JavaScript files in your page:

```jsx
<link href="block/block.css" rel="stylesheet">
<script src="block/block.js"></script>
```

The package includes a working example to help you get started:
• **`example-canvas.html`** – Demonstrates how to use Canvas mode.

## **🎨 Themes**

ContentBuilder.js supports **UI theming** , allowing you to customize the editor's appearance — such as switching between **light** , **dark** , or branded color themes.

You can define multiple themes using the **`themes`** option when initializing ContentBuilder:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    themes: [
        ['#ffffff', '', ''], // Light theme (default)
        ['#282828', 'dark', 'contentbuilder/themes/dark.css'], // Dark theme
        ['#0088dc', 'colored', 'contentbuilder/themes/colored-blue.css'] // Custom colored theme
    ]
});
```

Each theme entry follows this format:

```jsx
[ preview_color_hex, theme_category_name, theme_css_file ]
```

These themes will appear in the **Preferences dialog** , letting users switch between them visually.

### **Applying a Theme Programmatically**

You can also set the UI theme dynamically using the **`setUIColor()`** method:

**To apply a light theme:**

```jsx
builder.setUIColor('');
```

**To apply a dark theme:**

```jsx
builder.setUIColor('dark', 'contentbuilder/themes/dark.css');
```

**To apply a custom colored theme:**

```jsx
builder.setUIColor('colored', 'contentbuilder/themes/colored-blue.css');
```

## **📄 Multiple Editable Area Support**

ContentBuilder.js supports editing content across **multiple editable areas** on the same page.

You can define multiple editable areas in your HTML like this:

```jsx
<div class="section">
    <div class="container area1">
        <!-- Content for area 1 -->
    </div>
</div>

<div class="section">
    <div class="container area2">
        <!-- Content for area 2 -->
    </div>
</div>
```

Then initialize ContentBuilder once using the shared class (e.g., **`.container`**):

```jsx
const builder = new ContentBuilder({
    container: '.container'
});
```

This tells ContentBuilder to make all elements matching **`.container`** editable.

**Retrieving HTML from Specific Areas**

To get the generated HTML from a specific area, use the **`html()`** method and pass the target container element:

```jsx
let html1 = builder.html(document.querySelector('.area1')); // Get content of first area
let html2 = builder.html(document.querySelector('.area2')); // Get content of second area
```

**Try the Example**

You can test multiple editable areas by opening the provided example in the package:

- **`example3.html`**

## **🔧 Programmatically Add Editable Area**

ContentBuilder.js allows you to dynamically add new editable containers to the page **without re-initializing** the entire editor.

This is useful in applications where content areas are added or loaded dynamically.

**How to Make a New Container Editable**

When you insert a new container into the DOM, simply call:

```jsx
builder.applyBehavior();
```

## **🎨 Other Configuration Options**

### **🌈 Color Picker**

By default, ContentBuilder includes a color picker with a set of predefined colors. You can customize this list using the **`colors`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    colors: [
        "#ff8f00", "#ef6c00", "#d84315", "#c62828", "#58362f", "#37474f", "#353535",
        "#f9a825", "#9e9d24", "#558b2f", "#ad1457", "#6a1b9a", "#4527a0", "#616161",
        "#00b8c9", "#009666", "#2e7d32", "#0277bd", "#1565c0", "#283593", "#9e9e9e"
    ]
});
```

This defines the color palette available in the editor’s color picker dropdown.

### **🏷️ Custom Tags (for CMS Integration)**

Custom tags allow you to insert dynamic placeholders into content, which can be replaced with custom values at runtime

You can define custom tags like this:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    customTags: [
        ["Site Name", "{%SITENAME%}"],
        ["Logo", "{%LOGO%}"],
        ["My Plugin", "{%MY_PLUGIN%}"]
    ]
});
```

These tags appear in the tags menu (on the editing toolbar) and let users add placeholders directly into their content.

### **📋 Paste Result Behavior**

When users paste content from external sources (like Word or another website), you can control how it is handled using the **`paste`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    paste: 'text' // or 'html', 'html-without-styles'
});
```

**Values:**

- **`'text'`** – Only plain text is pasted (default)
- **`'html'`** – Full HTML content including styles
- **`'html-without-styles'`** – HTML content without inline style attributes

This helps maintain clean formatting based on your use case.

### **🗑 Clear Saved Preferences**

By default, user preferences (such as theme selection) are saved in the browser's local storage and reused the next time the editor is opened.

To prevent loading previously saved preferences, use:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    clearPreferences: true
});
```

This ensures the editor starts fresh each time, ignoring any stored settings.

### **⚙️ Set Custom Headers for Server Requests**

When ContentBuilder.js interacts with a server — such as when using AI features via the **`sendCommandUrl`** or other backend integrations — it uses the **`fetch()`** API to send requests.

You can customize the HTTP headers sent with these requests using the **`headers`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    headers: {
        'Authorization': 'Bearer YOUR_TOKEN_HERE',
        'X-Custom-Header': 'CustomValue'
    }
});
```

🔒 **Note:** The **`'Content-Type': 'application/json'`** header is automatically added by default. You do not need to include it.

These custom headers will be included in all internal **`fetch()`** requests made by the editor.

## **⚙️ Methods**

### **📥 Load HTML Content Programmatically**

To completely replace the current content in the editor with new HTML, use the **`loadHtml()`** method:

```jsx
builder.loadHtml(html);
```

This is ideal for loading saved content from a database or pre-defined template.

Example:

```jsx
const myContent = `
    <div class="row">
        <div class="column">
            <h1>Welcome</h1>
            <p>This is your saved content.</p>
        </div>
    </div>
`;

builder.loadHtml(myContent);
```

### **📎 Paste HTML Content at Cursor Position**

To insert HTML content at the current cursor position, use:

```jsx
builder.pasteHtmlAtCaret(html);
```

Example:

```jsx
const signature = `<p><strong>Best regards,<br>John Doe</strong></p>`;
builder.pasteHtmlAtCaret(signature);
```

This inserts the given HTML exactly where the cursor is located.

### **🧩 Insert Custom HTML Snippet Programmatically**

You can also insert block snippets using the **`addSnippet()`** method:

```jsx
builder.addSnippet(`
    <div class="row">
        <div class="column">
            <h3>Hello World!</h3>
        </div>
    </div>
`);
```

### **⚙️ Programmatically Open Preference Dialog**

You can open the Preferences dialog from code using:

```jsx
builder.viewConfig();
```

### **⚙️ Programmatically Open Snippets Dialog**

You can open the Snippets dialog from code using:

```jsx
builder.viewSnippets();
```

### **⚙️ Programmatically Open HTML Code Editor**

You can open the HTML code editor from code using:

```jsx
builder.viewHtml();
```

### **⚙️ Disable Column HTML Code Editor**

By default, ContentBuilder.js shows an **HTML editor button** in the column tool popup. This allows users to view and edit the raw HTML of a column.

If you want to **hide this button** , you can disable it using the **`columnHtmlEditor`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    columnHtmlEditor: false,
});
```

### **⚙️ Disable Row HTML Code Editor**

By default, ContentBuilder.js shows an **HTML editor button** in the row tool popup. This allows users to view and edit the raw HTML of a row.

If you want to **hide this button** , you can disable it using the **`rowHtmlEditor`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    rowHtmlEditor: false,
});
```

### **⚙️ Change Element Selection Behavior**

By default, when a user presses **CMD+A (Mac)** or **CTRL+A (Windows)** inside an editable area, ContentBuilder selects only the current element (such as a paragraph or heading) — not the entire text in a column.

You can control this behavior using the **`elementSelection`** option:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    elementSelection: true // Default behavior
});
```

- When **`elementSelection: true`** – Selects only the current element inside the column.
- When **`elementSelection: false`** – Selects the entire column content.

### **↩️ Programmatically Trigger Undo or Redo**

You can also trigger undo or redo actions from your code using the following methods:

```jsx
builder.undo();  // Reverts the last action

builder.redo();  // Re-applies the last undone action
```

These are useful when integrating with custom UI elements.

### **🛑 Destroying the Editor Instance**

To completely remove a ContentBuilder instance, use:

```jsx
builder.destroy();
```

## **⚡ Event Handling**

ContentBuilder.js provides several built-in event hooks that allow you to respond to user actions and content changes in real time. These events are useful for integrating with external systems or applying custom logic.

### **🔄 `onRender` – Triggered When Content Is Updated or Re-rendered**

This event is fired whenever the content structure changes — such as when a new snippet is added, a block is moved, or layout changes occur.

Use it to re-apply custom scripts or behaviors after content updates.

**Example:**

```jsx
const builder = new ContentBuilder({
    container: '.container',
    onRender: function() {
        console.log('Content structure updated');
    }
});
```

### **📝 `onChange` – Triggered When Content Is Edited**

This event fires whenever the actual content is modified — for example, when text is typed, formatted, or deleted.

Useful for auto-saving content.

**Example:**

```jsx
const builder = new ContentBuilder({
    container: '.container',
    onChange: function() {
        console.log('Content was edited');
    }
});
```

### **🖱️ `onContentClick(event)` – Triggered When Content Is Clicked**

This event is triggered every time a user clicks inside the editable area.

You can use this to detect where the user clicked and apply custom logic accordingly.

```jsx
const builder = new ContentBuilder({
    container: '.container',
    onContentClick: function(event) {
        console.log('Content clicked', event.target);
    }
});
```

## **📁 Toolbar**

ContentBuilder.js provides a flexible toolbar system with different toolbars for different editing contexts. You can customize which buttons appear in each toolbar to match your application’s needs.

### **🔤 Main Toolbar**

The **main toolbar** appears when a **text element** is selected.

It supports two rows of buttons:

- The first row is always visible.
- The second row appears when the user clicks the **More (⋯)** button.

**Configuration Options:**

- **`buttons`** – First row of buttons
- **`buttonsMore`** – Second row of buttons

```jsx
const builder = new ContentBuilder({
    container: '.container',
    buttons: ['bold', 'italic', 'underline', 'formatting', 'color', 'align', 'textsettings', 'createLink', 'tags', '|', 'undo', 'redo', 'zoom', 'livepreview', 'aiassistant', 'snippets', 'more'],  
    buttonsMore: ['icon', 'svg', 'image', '|', 'list', 'font', 'formatPara', '|', 'html', 'preferences'], 
});
```

Plugin buttons (like **`preview`**, **`wordcount`**, **`symbols`**) can be used here.

### **🖼️ Element Toolbar**

The **element toolbar** appears when a **non-text element** is selected — such as an image, video, or spacer.

Like the main toolbar, it supports two rows.

**Configuration Options:**

- **`elementButtons`** – First row of buttons
- **`elementButtonsMore`** – Second row of buttons

```jsx
const builder = new ContentBuilder({
    container: '.container',
    elementButtons: ['left', 'center', 'right', 'full' , 'undo', 'redo', 'zoom', 'livepreview', 'aiassistant', 'snippets', 'more'],  
    elementButtonsMore: ['|', 'html', 'preferences'], 
});
```

Plugin buttons are also supported here.

### **🎨 Icon Toolbar**

The **icon toolbar** appears when an **icon** element is selected.

You can configure both rows like this:

**Configuration Options:**

- **`iconButtons`** – First row of buttons
- **`iconButtonsMore`** – Second row of buttons

```jsx
const builder = new ContentBuilder({
    container: '.container',
    iconButtons: ['icon', 'color','textsettings', 'createLink','|', 'undo', 'redo', 'zoom', 'livepreview', 'aiassistant', 'snippets', 'more'],  
    iconButtonsMore: ['html', 'preferences'], 
});
```

## **🛠 Customize Editing Controls**

ContentBuilder.js gives you full control over the editing interface. This is especially useful when building a focused content editor.

Here’s an example that demonstrates how to tailor the editing experience:

```jsx
const builder = new ContentBuilder({
    container: '.container',

    columnTool: false, // Hide column tool
    // Configure buttons in the column menu (when columnTool is true)
    // columnMoreButtons: ['moveleft', 'moveright', 'moveup', 'movedown', 'increase', 'decrease', 'duplicate'],
      
    // Configure buttons in the element menu
    elementMoreButtons: ['moveup', 'movedown', 'duplicate'],

    // Configure buttons in the row menu
    rowMoreButtons: ['moveup', 'movedown', 'duplicate'],

    // Configure buttons in the "+" quick add popover
    quickAddButtons: [
        'paragraph', 'headline', 'image', 'list',
        'heading1', 'heading2', 'heading3', 'heading4',
        'quote', 'preformatted', 'map', 'youtube',
        'video', 'audio', 'icon', 'svg', 'table',
        'social', 'code', 'spacer', 'line'
    ],

    // Hide the Quick Add "More" button that opens the snippets dialog
    noSnippets: true,
    
    // Optional: Customize main toolbar buttons
    buttons: ['bold', 'italic', 'underline', 'formatting', 'color', 'align', 'textsettings', 'createLink', 'tags', '|', 'undo', 'redo', 'aiassistant', 'more'],
    buttonsMore: ['icon', 'svg', 'image', '|', 'list', 'font', 'formatPara', '|', 'html'],
    elementButtons: ['left', 'center', 'right', 'full', 'undo', 'redo', 'aiassistant', 'more'],
    elementButtonsMore: ['|', 'html'],
    iconButtons: ['icon', 'color','textsettings', 'createLink','|', 'undo', 'redo', 'aiassistant', 'more'],
    iconButtonsMore: ['html']
});
```

### **Key Parameters Explained**

| **Parameter** | **Description** |
| --- | --- |
| **`columnTool`** | If**`false`**, hides the column tool. |
| **`columnMoreButtons`** | Sets the buttons shown in the column menu (only if**`columnTool`**is enabled). |
| **`elementMoreButtons`** | Sets the buttons shown in the element menu. |
| **`rowMoreButtons`** | Sets the buttons shown in the row menu. |
| **`quickAddButtons`** | Sets the buttons shown in the "+" quick add popover. |
| **`noSnippets`** | If**`true`**, hides the Quick Add "More" button that opens the snippets dialog. |

## **📁 Path Configuration**

When hosting ContentBuilder.js or its associated asset files in a different location — such as a CDN, cloud storage, or separate directory — you can use the following configuration options to ensure correct file loading and path resolution.

Here’s an example of how to set up custom paths:

```jsx
const builder = new ContentBuilder({
    container: '.container',

    // Base path for general assets
    assetPath: 'https://path.to/assets/', 

    // Font assets path
    fontAssetPath: 'https://path.to/assets/fonts/', 

    // Snippet file and related paths
    snippetUrl: 'https://path.to/assets/minimalist-blocks/content.js',  // Full URL to snippet file
    snippetPath: 'https://path.to/assets/minimalist-blocks/',  // Base path for images and resources
    snippetPathReplace: [
        'assets/minimalist-blocks/images/',
        'https://path.to/assets/minimalist-blocks/images/' 
    ], // Replace local image paths with remote URLs

    // Module and plugin paths
    modulePath: 'https://path.to/assets/modules/',  // module files
    pluginPath: 'https://path.to/contentbuilder/',  // plugin files

    // Media path for AI-generated content (e.g., random images)
    mediaPath: 'https://path.to/assets/gallery/' 
});
```

These settings allow full control over where ContentBuilder looks for external files — making it easy to host assets remotely or serve them from optimized delivery networks.
