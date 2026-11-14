# Configuration Reference

This document provides detailed configuration options and advanced features for ContentBuilder.js.

If you're new to ContentBuilder, start with the [Getting Started Guide](https://github.com/innovastudio-dev/contentbuilder) which covers installation, basic setup, and core features. This reference is designed for when you need specific customization options beyond the basics.

## Snippets Management

Snippets are pre-designed content blocks that users can drag and drop into their content. When adding a content block via the **(+) icon**, users can select the **MORE** button to open the snippets dialog.

### Default Snippet Location

Snippet files are located in:

```
assets/minimalist-blocks/
```

This folder contains:

- **`content.js`** – The snippets definition file
- **`images/`** – Image assets used in the snippets

### Loading Snippets

Load snippets dynamically using the `loadSnippets()` method:

```javascript
builder.loadSnippets('assets/minimalist-blocks/content.js');
```

The parameter is the path to your `content.js` file.

### Configuring Snippet Paths

If hosting assets remotely (e.g., on a CDN), configure the paths during initialization:

```javascript
const builder = new ContentBuilder({
    container: '.container',

    // Full URL to snippet file
    snippetUrl: 'https://cdn.example.com/assets/minimalist-blocks/content.js',
    
    // Base path for the remotely hosted snippets
    snippetPath: 'https://cdn.example.com/assets/minimalist-blocks/',
    
    // Replace the default base path with the remotely hosted path
    snippetPathReplace: [
        'assets/minimalist-blocks/images/',
        'https://cdn.example.com/assets/minimalist-blocks/images/'
    ]
});
```

These options ensure all relative paths (especially images) resolve correctly when hosted externally.

### Snippet Modal Position

Control where the snippets panel appears:

```javascript
const builder = new ContentBuilder({
    container: '.container',
    snippetModalLeft: true // Position on the left side
});
```

- **Default (not set)**: Snippets modal appears on the **right** side
- **`snippetModalLeft: true`**: Displays on the **left** side

### Creating Custom Snippets

Add or edit snippets by modifying the `content.js` file, which contains snippets in JSON format.

**Example snippet object:**

```javascript
{
    'thumbnail': 'preview/preview-01.png',
    'category': '120',
    'html': `
        <div class="row">
            <div class="column">
                <h1>Heading Text Goes Here</h1>
                <p>Lorem Ipsum is simply dummy text...</p>
            </div>
        </div>
    `
}
```

Each snippet includes:

- **`thumbnail`** – Path to the preview image
- **`category`** – Category ID this snippet belongs to
- **`html`** – The HTML markup for the block

### Snippet Categories

Organize snippets into categories using the `snippetCategories` option:

```javascript
const builder = new ContentBuilder({
    container: '.container',

    snippetCategories: [
        [120, 'Basic'],
        [118, 'Article'],
        [101, 'Headline'],
        [119, 'Buttons'],
        [102, 'Photos'],
        [103, 'Profile'],
        [116, 'Contact'],
        [104, 'Products'],
        [105, 'Features'],
        [106, 'Process'],
        [107, 'Pricing'],
        [108, 'Skills'],
        [109, 'Achievements'],
        [110, 'Quotes'],
        [111, 'Partners'],
        [112, 'As Featured On'],
        [113, 'Page Not Found'],
        [114, 'Coming Soon'],
        [115, 'Help, FAQ']
    ],

    defaultSnippetCategory: 120 // Category shown first when opening snippets dialog
});
```

The `defaultSnippetCategory` defines which category displays first when the snippets dialog opens.

### Controlling Element Behavior in Snippets

Add special behaviors to elements inside snippets using data attributes:

**1. Prevent Image Replacement**

Make an image non-replaceable by adding `data-fixed`:

```html
<img src="image.png" data-fixed />
```

**2. Make Content Non-editable**

Prevent content editing inside a column using `data-noedit`:

```html
<div class="row">
    <div class="column" data-noedit>
        This content cannot be edited.
    </div>
</div>
```

**3. Fully Protected Column**

Protect a column completely using `data-protected`:

```html
<div class="row">
    <div class="column" data-protected>
        This column is fully protected from any modifications.
    </div>
</div>
```

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

### **Using with React/Vue/Modern Frameworks**

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
*Integrating the grid system from a CSS framework like Bootstrap or Tailwind is optional. ContentBuilder already includes its own built-in grid system, which offers flexible cell resizing and configurable screen size targeting. This means users can still take advantage of a CSS framework’s utilities and components, without having to integrate the grid system or configure the **`row`** and **`cols`** options.* 

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
        ['#ffffff','',''],
        ['#282828','dark',''],  
        ['#0088dc','colored','contentbuilder/themes/colored-blue.css'], // color preview, class name & css file
        ['#006add','colored','contentbuilder/themes/colored-blue6.css'],
        ['#0a4d92','colored','contentbuilder/themes/colored-darkblue.css'],
        ['#96af16','colored','contentbuilder/themes/colored-green.css'],
        ['#f3522b','colored','contentbuilder/themes/colored-orange.css'],

        ['#b92ea6','colored','contentbuilder/themes/colored-magenta.css'],
        ['#e73171','colored','contentbuilder/themes/colored-pink.css'],
        ['#782ec5','colored','contentbuilder/themes/colored-purple.css'],
        ['#ed2828','colored','contentbuilder/themes/colored-red.css'],
        ['#f9930f','colored','contentbuilder/themes/colored-yellow.css'],
        ['#13b34b','colored','contentbuilder/themes/colored-green4.css'],
        ['#333333','colored-dark','contentbuilder/themes/colored-dark.css'], 
        
        ['#dbe5f5','light','contentbuilder/themes/light-blue.css'],
        ['#fbe6f2','light','contentbuilder/themes/light-pink.css'],
        ['#dcdaf3','light','contentbuilder/themes/light-purple.css'],
        ['#ffe9e0','light','contentbuilder/themes/light-red.css'],
        ['#fffae5','light','contentbuilder/themes/light-yellow.css'],
        ['#ddf3dc','light','contentbuilder/themes/light-green.css'],
        ['#c7ebfd','light','contentbuilder/themes/light-blue2.css'],

        ['#ffd5f2','light','contentbuilder/themes/light-pink2.css'],
        ['#eadafb','light','contentbuilder/themes/light-purple2.css'],
        ['#c5d4ff','light','contentbuilder/themes/light-blue3.css'],
        ['#ffefb1','light','contentbuilder/themes/light-yellow2.css'],
        ['#fefefe','light','contentbuilder/themes/light-gray3.css'],
        ['#e5e5e5','light','contentbuilder/themes/light-gray2.css'],
        ['#dadada','light','contentbuilder/themes/light-gray.css'],

        ['#3f4ec9','colored','contentbuilder/themes/colored-blue2.css'],
        ['#6779d9','colored','contentbuilder/themes/colored-blue4.css'],
        ['#10b9d7','colored','contentbuilder/themes/colored-blue3.css'], 
        ['#006add','colored','contentbuilder/themes/colored-blue5.css'],
        ['#e92f94','colored','contentbuilder/themes/colored-pink3.css'],
        ['#a761d9','colored','contentbuilder/themes/colored-purple2.css'],
        ['#f9930f','colored','contentbuilder/themes/colored-yellow2.css'],

        ['#f3522b','colored','contentbuilder/themes/colored-red3.css'],
        ['#36b741','colored','contentbuilder/themes/colored-green2.css'],
        ['#00c17c','colored','contentbuilder/themes/colored-green3.css'],
        ['#fb3279','colored','contentbuilder/themes/colored-pink2.css'],
        ['#ff6d13','colored','contentbuilder/themes/colored-orange2.css'], 
        ['#f13535','colored','contentbuilder/themes/colored-red2.css'],
        ['#646464','colored','contentbuilder/themes/colored-gray.css'],

        ['#3f4ec9','dark','contentbuilder/themes/dark-blue.css'],
        ['#0b4d92','dark','contentbuilder/themes/dark-blue2.css'],
        ['#006add','dark','contentbuilder/themes/dark-blue3.css'],
        ['#5f3ebf','dark','contentbuilder/themes/dark-purple.css'],
        ['#e92f69','dark','contentbuilder/themes/dark-pink.css'],
        ['#4c4c4c','dark','contentbuilder/themes/dark-gray.css'],
        ['#ed2828','dark','contentbuilder/themes/dark-red.css'],

        ['#006add','colored','contentbuilder/themes/colored-blue8.css'],
        ['#ff7723','colored','contentbuilder/themes/colored-orange3.css'],
        ['#ff5722','colored','contentbuilder/themes/colored-red5.css'],
        ['#f13535','colored','contentbuilder/themes/colored-red4.css'],
        ['#00bd79','colored','contentbuilder/themes/colored-green5.css'],
        ['#557ae9','colored','contentbuilder/themes/colored-blue7.css'],
        ['#fb3279','colored','contentbuilder/themes/colored-pink4.css'],
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

### **⌨️ Keyboard Access**

Everything in a row, a column, an element or a piece of text can be done
without a mouse. With the cursor in the content, `Esc` (or `Alt+F10`) moves to
the tools for whatever you are in, and each `Esc` after that steps out one
level: the text tools, the tools of the picture or table you are in, the
element, the column and the row, and back to your text. `Shift+Esc` steps back
in.

Each strip is a toolbar in its own right: the arrows move along it, `Home` and
`End` jump to its ends, and `Enter` or `Space` uses a button. A panel that opens
from a button takes the focus and hands it back when `Esc` closes it.

See `docs/keyboard-access.md` for every key, with worked examples.

Below 970px the tools are hidden, since laying out a page on a narrow screen is
awkward. To keep them at every width, set **`simpleEditingBreakpoint`** to an
empty string:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    simpleEditingBreakpoint: ''
});
```

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
    paste: 'auto' // or 'html', 'html-without-styles', 'text'
});
```

**Values:**

- **`'auto'`** – Pasted content adopts the destination block (for example, pasting a heading into a paragraph keeps it a paragraph) while preserving text formatting such as bold, italic, links, and colors (default)
- **`'html'`** – Full HTML content including styles
- **`'html-without-styles'`** – HTML content without inline style attributes
- **`'text'`** – Only plain text is pasted

Users can also change this behavior at any time from the editor's **Preferences** dialog.

### **🧹 Unwanted Markup from Browser Extensions**

Some browser extensions add their own elements and attributes to any text area they find, including the editor. That markup is not part of the content, so it is removed when the content is saved: the injected elements and attributes go, and the text they wrapped stays.

This happens automatically. No configuration is needed.

If your users have an extension that isn't covered, add its markup to the cleanup list:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    extensionArtifacts: {
        elements: ['my-extension-overlay'],     // removed entirely
        wrappers: ['span[data-myext-id]'],      // removed, the text inside is kept
        attributePrefixes: ['data-myext-'],     // attributes removed by prefix
        stylePrefixes: ['--myext-']             // style properties removed by prefix
    }
});
```

Every list is added to the built-in ones.

Extensions keep working while your users edit. If you would rather they didn't run inside the editing areas at all, you can turn them off there:

```jsx
const builder = new ContentBuilder({
    container: '.container',
    disableEditorExtensions: true
});
```

Saved content is cleaned either way.

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

### **🖼️ Programmatically Open the Image Editor**

The Edit button on an image opens the image editor (crop, straighten, rotate, white balance, tone, film looks, grain). You can open the same editor on any image from your own UI, for example a background image in your own panel:

```jsx
builder.openImageEditor(url, (newUrl, filename) => {
    // apply newUrl wherever the image is used
    element.style.backgroundImage = `url("${newUrl}")`;
});
```

The edited image is rendered at full resolution and saved through your upload handler under a new name, so the original file is never overwritten. If no upload handler is configured, `newUrl` is a data URL.

A third argument is optional: the button that opened the editor, so keyboard focus returns to it when the dialog closes.

```jsx
builder.openImageEditor(url, callback, button);
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

### **🪟 Build Your Own Dialog**

`builder.modal()` builds a dialog around your content. You supply what goes
inside; the shell, title bar, close button, accessibility attributes, padding and
dragging are handled for you:

```jsx
const wordCount = builder.modal({
    title: builder.out('Word Count'),
    width: 320,
    className: 'wordcount',   // optional hook for your own CSS
    html: `
        <div>
            <div style="line-height:1">
                <span class="wc-words" style="font-size:56px;font-weight:700;">0</span>
                <span style="font-size:15px;margin-left:6px;">${builder.out('words')}</span>
            </div>
        </div>`
});
```

It returns a handle:

```jsx
wordCount.open();     // show it
wordCount.close();    // hide it
wordCount.body        // the content wrapper - query your own markup from here
wordCount.el          // the dialog element
```

Pair it with a toolbar button and there is nothing else to wire:

```jsx
options.buttons = ['bold','italic','underline','wordcount','|','undo','redo','more'];

const html = `<button class="wordcount-button" title="${builder.out('Word Count')}"><!-- icon --></button>`;

builder.addButton('wordcount', html, '.wordcount-button', () => {
    wordCount.body.querySelector('.wc-words').textContent = countWords();
    wordCount.open();
});
```

**Options**

| Option | Description |
| --- | --- |
| `html` | The content. Required. |
| `title` | Title bar text. Omit for a bare dialog with a floating close button. |
| `width` | `320` for pixels, or any CSS length such as `'40rem'`. |
| `padding` | Overrides the body padding. Skipped automatically if your outermost element already sets padding inline. |
| `closeButton` | Default `true`. |
| `draggable` | Default `true`. Applies to the title bar. |
| `className` | Extra class on the dialog, for your own CSS. |
| `onOpen` / `onClose` | Callbacks, each receiving the handle. |

See also `docs/others/TIPS.md` (recipe 7) for the full worked example.

### **🖥️ Page Preview**

A fullscreen preview of the page with a width ruler, for checking the layout at
any size without leaving the editor.

```jsx
builder.openPagePreview();
```

The frame loads your `previewURL`, and the content reaches that page through the
existing `onPreviewOpen` hook:

```jsx
const builder = new ContentBuilder({
    previewURL: 'preview.html',
    onPreviewOpen: () => {
        // default: saves builder.html() to localStorage
    },
});
```

Nothing happens if `previewURL` is not set. In ContentBuilder the toolbar button is on by
default — `'preview'` is part of `buttonsMore`, `elementButtonsMore` and `iconButtonsMore` —
and it only appears once `previewURL` is set, since that is the page it opens.

> Not to be confused with `openPreview()`, which opens Live Preview, the small
> floating window. They are separate tools and both remain available.

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
    buttons: ['bold', 'italic', 'underline', 'formatting', 'color', 'align', 'textsettings', 'createLink', 'tags', '|', 'undo', 'redo', 'zoom', 'preview', 'aiassistant', 'snippets', 'more'],  
    buttonsMore: ['icon', 'svg', 'image', '|', 'list', 'font', 'formatPara', '|', 'html', 'preferences'], 
});
```

Built-in names you can add here include **`preview`**, **`searchreplace`** and **`symbols`**.
`preview` and `searchreplace` are already in the default lists; `symbols` is not, so add it if
you want it. Your own buttons work here too — give the name in this list and fill it in with
`builder.addButton()` (see `docs/others/TIPS.md`).

### **🖼️ Element Toolbar**

The **element toolbar** appears when a **non-text element** is selected — such as an image, video, or spacer.

Like the main toolbar, it supports two rows.

**Configuration Options:**

- **`elementButtons`** – First row of buttons
- **`elementButtonsMore`** – Second row of buttons

```jsx
const builder = new ContentBuilder({
    container: '.container',
    elementButtons: ['left', 'center', 'right', 'full' , 'undo', 'redo', 'zoom', 'preview', 'aiassistant', 'snippets', 'more'],  
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
    iconButtons: ['icon', 'color','textsettings', 'createLink','|', 'undo', 'redo', 'zoom', 'preview', 'aiassistant', 'snippets', 'more'],  
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
  
    // Form Builder module paths
    assetsFolder: 'https://path.to/assets/formfiles/',
    exampleImageUrl: 'https://path.to/assets/formfiles/ai-V8Khk.jpg',
    exampleVideoUrl: 'https://path.to/assets/formfiles/ai-thhyR.mp4',
    exampleAudioUrl: 'https://path.to/assets/formfiles/ai-3EVMb.wav',
});
```

These settings allow full control over where ContentBuilder looks for external files — making it easy to host assets remotely or serve them from optimized delivery networks.