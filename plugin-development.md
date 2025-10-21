# Plugin Development Guide

**Build powerful, configurable plugins for ContentBuilder and ContentBox**

------

## 📚 Table of Contents

1. [Introduction](#-introduction)
2. [Getting Started](#-getting-started)
3. [Basic Plugin Development](#-basic-plugin-development)
4. [Configuration Options](#-configuration-options)
   - [Simple Settings (Auto-Generated UI)](#%EF%B8%8F-simple-settings-auto-generated-ui)
   - [Custom Editors (Advanced UI)](#-custom-editors-advanced-ui)
5. [Complete Example](#-complete-example)

------

## 🎯 Introduction

### What is a Plugin?

A plugin adds interactive features to your website. Instead of manually writing JavaScript for each page, you create a plugin once and use it anywhere with simple HTML attributes. ContentBuilder or ContentBox Runtime handles loading and initialization automatically.

### What You'll Learn

This guide covers everything you need to know about plugin development:

- **Basic plugins** - Simple interactive components
- **Settings system** - Auto-generated configuration UI
- **Custom editors** - Advanced, interactive configuration interfaces
- **Best practices** - Patterns for maintainable, scalable plugins

------

## 🚀 Getting Started

### Plugin Structure

Every plugin is just a folder with 2 files:

```
plugins/
  my-plugin/
    index.js      ← Your JavaScript code
    style.css     ← Your styles (optional)
```

### Your First Plugin (5 Minutes)

Let's create a Click Counter plugin. When users click an element, it shows how many times it was clicked.

**Step 1: Create the Plugin File**

File: `plugins/click-counter/index.js`

```javascript
export default {
    name: 'click-counter',
    version: '1.0.0',
    
    mount: function(element, options) {
        var count = 0;
        
        element.addEventListener('click', function() {
            count = count + 1;
            element.textContent = 'Clicks: ' + count;
        });
        
        return { count: count };
    }
};
```

**Step 2: Add Styling (Optional)**

File: `plugins/click-counter/style.css`

```css
[data-cb-type="click-counter"] {
    cursor: pointer;
    padding: 10px 20px;
    background: #f0f0f0;
    border-radius: 5px;
    display: inline-block;
}

[data-cb-type="click-counter"]:hover {
    background: #e0e0e0;
}
```

**Step 3: Register the Plugin**

```javascript
const runtime = new ContentBoxRuntime({
    plugins: {
        'click-counter': {
            url: '/assets/plugins/click-counter/index.js',
            css: '/assets/plugins/click-counter/style.css'
        }
    }
});

runtime.init();
```

**Step 4: Use in HTML**

```html
<div data-cb-type="click-counter">
    Click me!
</div>
```

Done! 🎉

------

## 📖 Basic Plugin Development

### The Basic Structure

Every plugin exports an object with a mount function:

```javascript
export default {
    name: 'plugin-name',      // Plugin identifier
    displayName: 'My Plugin', // Optional: friendly name shown in UI
    version: '1.0.0',         // Version number
    
    mount: function(element, options) {
        // Your code here
        return {};
    }
};
```

### The mount Function

This is the heart of your plugin. It runs for every element that uses your plugin.

```javascript
mount: function(element, options) {
    // element = the HTML element (<div>, <button>, etc.)
    // options = settings from data-cb-* attributes or settings UI
    
    // Do your magic here...
    
    // Always return an object (or null on error)
    return { /* anything you want to track */ };
}
```

**Key Points:**

- Gets called once per element
- Receives the element and its options
- Must return an object (can be empty `{}` or `null` on error)
- Can be async if needed for complex initialization

### Getting Options from HTML

Options come from `data-cb-*` attributes:

**HTML:**

```html
<div data-cb-type="tooltip" 
     data-cb-text="Hello" 
     data-cb-position="top">
</div>
```

**JavaScript:**

```javascript
mount: function(element, options) {
    var text = options.text;         // "Hello"
    var position = options.position; // "top"
}
```

**Rules:**

- Attributes must start with `data-cb-`
- Use lowercase with hyphens: `data-cb-my-option`
- Access in code with camelCase: `options.myOption`

### More Examples

**Example 1: Simple Tooltip**

Shows a tooltip when hovering over an element.

javascript

```javascript
export default {
    name: 'tooltip',
    version: '1.0.0',
    
    mount: function(element, options) {
        var tooltipText = options.text || 'Tooltip';
        var tooltip = document.createElement('div');
        
        // Style the tooltip
        tooltip.textContent = tooltipText;
        tooltip.style.cssText = 'position:fixed; background:#333; color:#fff; ' +
                                'padding:5px 10px; border-radius:4px; ' +
                                'font-size:12px; display:none; z-index:9999;';
        document.body.appendChild(tooltip);
        
        // Show on hover
        element.addEventListener('mouseenter', function(e) {
            tooltip.style.display = 'block';
            tooltip.style.left = e.clientX + 10 + 'px';
            tooltip.style.top = e.clientY + 10 + 'px';
        });
        
        // Hide when mouse leaves
        element.addEventListener('mouseleave', function() {
            tooltip.style.display = 'none';
        });
        
        return { tooltip: tooltip };
    }
};
```

**Usage:**

html

```html
<span data-cb-type="tooltip" data-cb-text="Helpful information">
    Hover me
</span>
```

**Example 2: Toggle Visibility**

Shows/hides another element when clicked.

javascript

```javascript
export default {
    name: 'toggle',
    version: '1.0.0',
    
    mount: function(element, options) {
        var targetId = options.target;
        var target = document.getElementById(targetId);
        
        if (!target) {
            console.error('Target element not found:', targetId);
            return null;
        }
        
        element.addEventListener('click', function() {
            if (target.style.display === 'none') {
                target.style.display = 'block';
            } else {
                target.style.display = 'none';
            }
        });
        
        return { target: target };
    }
};
```

**Usage:**

html

```html
<button data-cb-type="toggle" data-cb-target="myContent">
    Toggle Content
</button>

<div id="myContent">
    This content can be shown/hidden
</div>
```

### Cleanup with unmount

If your plugin needs cleanup (remove event listeners, timers, etc.), add an unmount function:

```javascript
export default {
    name: 'my-plugin',
    version: '1.0.0',
    
    mount: function(element, options) {
        var timer = setInterval(function() {
            console.log('Running...');
        }, 1000);
        
        return { 
            timer: timer,
            stop: function() {
                clearInterval(timer);
            }
        };
    },
    
    unmount: function(element, instance) {
        if (instance && instance.stop) {
            instance.stop();
        }
    }
};
```

### Async Mount for Complex Plugins

For plugins that need complex initialization:

```javascript
export default {
    name: 'complex-plugin',
    version: '1.0.0',
    
    async mount(element, options) {
        try {
            await someAsyncOperation();
            const instance = createComponent(element, options);
            return instance;
        } catch (error) {
            console.error('Plugin mount error:', error);
            return null;
        }
    }
};
```

### Class-Based Pattern

For sophisticated plugins, use a class-based approach:

```javascript
class MyComponent {
    constructor(element, options) {
        this.element = element;
        this.options = options;
        this.init();
    }
    
    init() {
        // Setup your component
    }
    
    destroy() {
        // Cleanup
    }
}

export default {
    name: 'my-plugin',
    version: '1.0.0',
    
    async mount(element, options) {
        try {
            const instance = new MyComponent(element, options);
            return instance;
        } catch (error) {
            console.error('Mount error:', error);
            return null;
        }
    },
    
    unmount(element, instance) {
        if (instance && typeof instance.destroy === 'function') {
            instance.destroy();
        }
    }
};
```

------

## 🔧 Configuration Options

Your plugin can have configuration in two ways: **Simple Settings** (auto-generated UI) and **Custom Editors** (advanced UI). You can use both in the same plugin!

### When to Use Each Approach

#### Use Simple Settings (Auto-Generated UI)

**Best for:** Simple configuration options like numbers, dropdowns, and checkboxes.

```javascript
settings: {
    columns: { type: 'number', label: 'Columns', default: 3 },
    showTitles: { type: 'boolean', label: 'Show Titles', default: true },
    theme: { type: 'select', label: 'Theme', options: [...] }
}
```

**Pros:**

- Zero UI code needed
- Consistent look and feel
- Quick to implement

#### Use Custom Editor

**Best for:** Complex, interactive configuration like sortable lists, dynamic forms, or live previews.

```javascript
editor: {
    openContentEditor: function(element, builder, onChange) {
        // Build your custom UI here
        const container = document.createElement('div');
        // Add sortable lists, dynamic forms, etc.
        return container;
    }
}
```

**Pros:**

- Full control over UI/UX
- Support for complex interactions
- Can integrate third-party libraries

#### Use Both Together! 🎉

```javascript
export default {
    name: 'my-plugin',
    
    settings: {
        // Simple options (auto-generated UI)
        columns: { type: 'number', label: 'Columns', default: 3 }
    },
    
    editor: {
        // Complex editor for advanced features
        openContentEditor: function(element, builder, onChange) {
            // Custom UI for managing list items, etc.
        }
    },
    
    mount: function(element, options) {
        // Your plugin logic
    }
}
```

Users will see **both** the auto-generated settings form AND your custom editor.

------

## ⚙️ Simple Settings (Auto-Generated UI)

The settings system automatically creates a visual configuration panel for your plugin options.

### Basic Implementation

Add a `settings` property to your plugin:

```javascript
export default {
    name: 'logo-loop',
    displayName: 'Logo Loop',
    version: '1.0.0',
    
    settings: {
        speed: {
            type: 'number',
            label: 'Animation Speed',
            default: 20,
            min: 5,
            max: 60,
            step: 1,
            unit: 's'
        },
        direction: {
            type: 'select',
            label: 'Scroll Direction',
            default: 'left',
            options: [
                { value: 'left', label: 'Left to Right' },
                { value: 'right', label: 'Right to Left' }
            ]
        },
        pauseOnHover: {
            type: 'boolean',
            label: 'Pause on Hover',
            default: true
        }
    },
    
    mount: function(element, options) {
        // Access settings via options.speed, options.direction, etc.
    }
};
```

### Supported Field Types

#### Text Input

```javascript
title: {
    type: 'text',
    label: 'Title',
    default: 'My Title',
    placeholder: 'Enter title...'
}
```

#### Textarea

```javascript
description: {
    type: 'textarea',
    label: 'Description',
    default: '',
    rows: 4,
    placeholder: 'Enter description...'
}
```

#### Number

```javascript
count: {
    type: 'number',
    label: 'Item Count',
    default: 5,
    min: 1,
    max: 20,
    step: 1,
    unit: 'items'
}
```

#### Boolean

```javascript
enabled: {
    type: 'boolean',
    label: 'Enable Feature',
    default: true
}
```

#### Select (Dropdown)

```javascript
size: {
    type: 'select',
    label: 'Size',
    default: 'medium',
    options: [
        { value: 'small', label: 'Small' },
        { value: 'medium', label: 'Medium' },
        { value: 'large', label: 'Large' }
    ]
}
```

#### Radio Buttons

```javascript
alignment: {
    type: 'radio',
    label: 'Alignment',
    default: 'center',
    options: [
        { value: 'left', label: 'Left' },
        { value: 'center', label: 'Center' },
        { value: 'right', label: 'Right' }
    ]
}
```

#### Color Picker

```javascript
backgroundColor: {
    type: 'color',
    label: 'Background Color',
    default: '#ffffff'
}
```

#### Range Slider

```javascript
opacity: {
    type: 'range',
    label: 'Opacity',
    default: 100,
    min: 0,
    max: 100,
    step: 5,
    unit: '%'
}
```

### Grouped Settings

For complex plugins, organize settings into collapsible groups:

```javascript
export default {
    name: 'advanced-carousel',
    displayName: 'Advanced Carousel',
    version: '1.0.0',
    
    settings: {
        _groups: [
            {
                id: 'general',
                label: 'General Settings',
                fields: ['speed', 'autoplay', 'loop']
            },
            {
                id: 'appearance',
                label: 'Appearance',
                fields: ['showArrows', 'showDots', 'gap']
            }
        ],
        
        speed: {
            type: 'number',
            label: 'Transition Speed',
            default: 500,
            unit: 'ms',
            group: 'general'
        },
        autoplay: {
            type: 'boolean',
            label: 'Autoplay',
            default: true,
            group: 'general'
        },
        showArrows: {
            type: 'boolean',
            label: 'Show Navigation Arrows',
            default: true,
            group: 'appearance'
        }
    },
    
    mount: function(element, options) {
        // Access all settings via options
    }
};
```

### Settings to HTML Attributes

The settings system automatically converts between:

**Settings Definition** → **HTML Attributes** → **Options Object**

```javascript
// Setting key: camelCase
settings: {
    pauseOnHover: { type: 'boolean', default: true }
}

// HTML attribute: kebab-case with data-cb- prefix
data-cb-pause-on-hover="true"

// Options object: camelCase
mount: function(element, options) {
    console.log(options.pauseOnHover); // true
}
```

------

## 🎨 Custom Editors (Advanced UI)

When you need more than simple form fields—like sortable lists, dynamic item management, or live previews—use custom editors.

### What Custom Editors Enable

- **Sortable lists** (drag & drop reordering)
- **Dynamic item management** (add/remove/duplicate)
- **Collapsible sections** (accordion-style editors)
- **Live previews** (image thumbnails, color swatches)
- **Complex layouts** (multi-column forms, tabs)
- **Custom interactions** (anything you can build with JavaScript!)

### The openContentEditor Function

```javascript
editor: {
    openContentEditor: function(element, builder, onChange) {
        // Build your custom UI
        const container = document.createElement('div');
        
        // ... add your custom UI elements ...
        
        return container; // Must return a DOM element
    }
}
```

### Parameters

#### 1. `element` - The Plugin Element

The DOM element containing your plugin content.

```javascript
openContentEditor: function(element, builder, onChange) {
    // Access plugin content
    const cards = element.querySelectorAll('.card');
    
    // Modify plugin content
    element.appendChild(newElement);
    existingElement.remove();
}
```

#### 2. `builder` - ContentBuilder/ContentBox Instance

Access builder features like file pickers:

```javascript
openContentEditor: function(element, builder, onChange) {
    // Open file picker for images
    builder.openFilePicker('image', (url) => {
        imageElement.src = url;
        onChange();
    });
    
    // Open file picker for videos
    builder.openFilePicker('video', (url) => {
        videoElement.src = url;
        onChange();
    });
    
    // Available types: 'image', 'video', 'audio', 'file'
}
```

**Available File Picker Types:**

- `'image'` - Image files only
- `'video'` - Video files only
- `'audio'` - Audio files only
- `'file'` - All file types
- `'media'` - Image and Video files

#### 3. `onChange` - Change Notification Callback

Call this whenever plugin content changes:

```javascript
openContentEditor: function(element, builder, onChange) {
    input.addEventListener('input', () => {
        element.querySelector('h1').textContent = input.value;
        onChange(); // Required!
    });
}
```

**When to call onChange:**

- After modifying element content
- After adding/removing elements
- After reordering elements
- After any change that should be saved

**Note:** You can also call `onChange` with a delay for better performance with frequent updates:

```javascript
let timer;
input.addEventListener('input', () => {
    clearTimeout(timer);
    timer = setTimeout(() => {
        element.textContent = input.value;
        onChange();
    }, 300); // Wait 300ms after user stops typing
});
```

### Return Value

**Must return a DOM element** containing your custom UI:

```javascript
openContentEditor: function(element, builder, onChange) {
    const container = document.createElement('div');
    
    // Build your UI inside container
    container.appendChild(titleInput);
    container.appendChild(descriptionTextarea);
    container.appendChild(addButton);
    
    return container; // Required!
}
```

### Quick Start Example

Let's build a Card List editor where users can add, remove, and edit cards:

```javascript
export default {
    name: 'card-list',
    displayName: 'Card List',
    version: '1.0.0',
    
    settings: {
        columns: {
            type: 'number',
            label: 'Columns',
            default: 3,
            min: 1,
            max: 6
        }
    },
    
    editor: {
        openContentEditor: function(element, builder, onChange) {
            const container = document.createElement('div');
            const cards = element.querySelectorAll('.card');

            // Create editor for a single card
            const createCardEditor = (card, index) => {
                const editor = document.createElement('div');
                editor.style.cssText = 'border: 1px solid #ddd; padding: 12px; margin-bottom: 10px; border-radius: 4px;';

                // Title input
                const titleInput = document.createElement('input');
                titleInput.type = 'text';
                titleInput.value = card.querySelector('h3')?.textContent || '';
                titleInput.placeholder = 'Card title';
                titleInput.style.cssText = 'margin-bottom: 8px;';

                titleInput.addEventListener('input', () => {
                    card.querySelector('h3').textContent = titleInput.value;
                    onChange();
                });

                // Description textarea
                const descInput = document.createElement('textarea');
                descInput.value = card.querySelector('p')?.textContent || '';
                descInput.placeholder = 'Card description';
                descInput.style.cssText = 'height: 140px; margin-bottom: 8px;';

                descInput.addEventListener('input', () => {
                    card.querySelector('p').textContent = descInput.value;
                    onChange();
                });

                // Delete button
                const deleteBtn = document.createElement('button');
                deleteBtn.textContent = 'Delete';
                deleteBtn.onclick = (e) => {
                    e.preventDefault();
                    card.remove();
                    editor.remove();
                    onChange();
                };

                editor.appendChild(titleInput);
                editor.appendChild(descInput);
                editor.appendChild(deleteBtn);

                return editor;
            };

            // Create editor for each card
            cards.forEach((card, index) => {
                container.appendChild(createCardEditor(card, index));
            });

            // Add "New Card" button
            const addBtn = document.createElement('button');
            addBtn.textContent = '+ Add Card';
            addBtn.onclick = (e) => {
                e.preventDefault();

                const newCard = document.createElement('div');
                newCard.className = 'card';
                newCard.innerHTML = `
                    <img src="https://placehold.co/600x400?text=New+Card" alt="">
                    <h3>New Card</h3>
                    <p>Card description</p>
                `;

                element.appendChild(newCard);
                
                const editor = createCardEditor(newCard, container.children.length);
                container.insertBefore(editor, addBtn);

                onChange();
            };

            container.appendChild(addBtn);

            return container;
        }
    },
    
    mount: function(element, options) {
        element.style.display = 'grid';
        element.style.gridTemplateColumns = `repeat(${options.columns || 3}, 1fr)`;
        element.style.gap = '16px';
        return {};
    }
};
```

> 💡 **Note:** Standard HTML elements (input, textarea, button, select) are automatically styled by ContentBuilder/ContentBox. You only need custom styles for special elements like image previews or custom layouts.

## 📦 Complete Example

### Sortable Card List with All Features

A full-featured card editor with drag-and-drop sorting, collapsible sections, image picker, and more:

```javascript
export default {
    name: 'card-list',
    displayName: 'Card List',
    version: '1.4.0',

    settings: {
        columns: {
            type: 'number',
            label: 'Columns',
            default: 3,
            min: 1,
            max: 6
        }
    },

    editor: {
        openContentEditor: function (element, builder, onChange) { // builder is COntentBuilder/ContentBox object
            // Load SortableJS dynamically
            const loadSortable = () => {
                return new Promise((resolve, reject) => {
                    if (window.Sortable) {
                        resolve(window.Sortable);
                        return;
                    }
                    const script = document.createElement('script');
                    script.src = 'https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/Sortable.min.js';
                    script.onload = () => resolve(window.Sortable);
                    script.onerror = reject;
                    document.head.appendChild(script);
                });
            };

            const container = document.createElement('div');
            const cards = element.querySelectorAll('.card');

            const getCardContainer = (el) =>
                el.matches('.card') ? el : el.querySelector('.card');

            // Collapsible Card Editor
            const createCardEditor = (cardWrapper, index) => {
                const card = getCardContainer(cardWrapper);

                const section = document.createElement('div');
                section.dataset.cardId = cardWrapper.dataset.cardId || `card-${Date.now()}-${index}`;
                cardWrapper.dataset.cardId = section.dataset.cardId;

                // Header
                const header = document.createElement('div');

                const titleText =
                    card.querySelector('h3')?.textContent || `New Card ${index + 1}`;

                const headerTitle = document.createElement('span');
                headerTitle.textContent = titleText;

                const thumb = document.createElement('img');
                thumb.src = card.querySelector('img')?.src || 'https://placehold.co/40x40?text=No+Img';
                thumb.style.cssText = `
                    width: 32px; height: 32px; object-fit: cover;
                    border-radius: 4px; margin-right: 8px;
                `;

                const left = document.createElement('div');
                left.style.display = 'flex';
                left.style.alignItems = 'center';
                
                // Drag handle icon
                const dragHandle = document.createElement('span');
                dragHandle.innerHTML = '⠿';
                dragHandle.style.cssText = `
                    margin-right: 8px;
                    font-size: 18px;
                    color: #999;
                    cursor: move;
                `;
                dragHandle.className = 'drag-handle';
                
                left.appendChild(dragHandle);
                left.appendChild(thumb);
                left.appendChild(headerTitle);

                const toggle = document.createElement('span');
                toggle.innerHTML = '<svg style="width: 20px; height: 20px;transform: rotate(360deg);transition: all 0.2s ease"><use xlink:href="#icon-chevron-down"></use></svg>';

                header.appendChild(left);
                header.appendChild(toggle);

                // Body
                const body = document.createElement('div');
                body.style.cssText = `
                    padding: 12px;
                    display: none;
                `;

                // --- Fields ---
                // Image URL + Preview
                const imgPreview = document.createElement('img');
                imgPreview.src = card.querySelector('img')?.src || 'https://placehold.co/80x80?text=No+Img';
                imgPreview.style.cssText = `
                    width: 80px; height: 80px; object-fit: cover;
                    display: block; border-radius: 6px; margin-bottom: 8px;
                `;

                // Delete button
                const deleteBtn = document.createElement('button');
                deleteBtn.innerHTML = '<svg><use xlink:href="#icon-trash2"></use></svg>';
                deleteBtn.style.cssText = 'width:45px';
                deleteBtn.onclick = (e) => {
                    e.preventDefault();
                    const target = cardWrapper.tagName === 'A' ? cardWrapper : card;
                    target.remove();
                    section.remove();
                    onChange?.();
                };
                
                const group1 = document.createElement('div');
                group1.style.cssText = 'display: flex; justify-content: space-between;';
                group1.appendChild(imgPreview);
                group1.appendChild(deleteBtn);

                const imgInput = document.createElement('input');
                imgInput.type = 'text';
                imgInput.style.cssText = 'flex: 1;';
                imgInput.placeholder = 'Image URL';
                imgInput.value = card.querySelector('img')?.src || '';

                imgInput.addEventListener('input', () => {
                    let img = card.querySelector('img');
                    if (!img) {
                        img = document.createElement('img');
                        card.prepend(img);
                    }
                    img.src = imgInput.value;
                    imgPreview.src = imgInput.value || 'https://placehold.co/80x80?text=No+Img';
                    thumb.src = imgPreview.src;
                    onChange?.();
                });

                const browseBtn = document.createElement('button');
                browseBtn.innerHTML = '<svg><use xlink:href="#icon-folder"></use></svg>';
                browseBtn.style.cssText = 'width: 45px;';
                browseBtn.onclick = (e) => {
                    e.preventDefault();
                    builder.openFilePicker('image', (url) => {
                        imgInput.value = url;
                        let img = card.querySelector('img');
                        img.src = url;
                        imgPreview.src = url;
                        thumb.src = url;

                        if (typeof onChange === 'function') {
                            onChange();
                        }
                    }, browseBtn);
                };

                const group2 = document.createElement('div');
                group2.style.cssText = 'display: flex; gap: 4px; margin-bottom: 8px;';
                group2.appendChild(imgInput);
                group2.appendChild(browseBtn);

                // Title
                const titleInput = document.createElement('input');
                titleInput.type = 'text';
                titleInput.placeholder = 'Title';
                titleInput.value = card.querySelector('h3')?.textContent || '';
                titleInput.style.cssText = 'width: 100%; margin-bottom: 8px;';

                titleInput.addEventListener('input', () => {
                    card.querySelector('h3').textContent = titleInput.value;
                    headerTitle.textContent = titleInput.value || 'Untitled Card';
                    onChange?.();
                });

                // Description
                const descInput = document.createElement('textarea');
                descInput.placeholder = 'Description';
                descInput.value = card.querySelector('p')?.textContent || '';
                descInput.rows = 3;
                descInput.style.cssText = 'width: 100%; height: 180px; margin-bottom: 8px;';

                let descTimer;
                descInput.addEventListener('input', () => {
                    clearTimeout(descTimer);
                    descTimer = setTimeout(() => {
                        card.querySelector('p').textContent = descInput.value;
                        onChange?.();
                    }, 400);
                });

                // Link URL
                const linkInput = document.createElement('input');
                linkInput.type = 'text';
                linkInput.placeholder = 'Link URL (optional)';
                linkInput.value =
                    cardWrapper.tagName === 'A' ? cardWrapper.getAttribute('href') || '' : '';
                linkInput.style.cssText = 'width: 100%; margin-bottom: 8px;';

                linkInput.addEventListener('input', () => {
                    const link = linkInput.value.trim();
                    const isWrapped = cardWrapper.tagName === 'A';
                    const isMedia = /\.(jpe?g|png|gif|webp|bmp|svg|mp4|webm|ogg)$/i.test(link);
                    if (link && !isWrapped) {
                        const a = document.createElement('a');
                        a.href = link;
                        a.target = '_blank';
                        a.rel = 'noopener noreferrer';
                        a.dataset.cardId = cardWrapper.dataset.cardId;
                        if (isMedia) a.classList.add('is-lightbox');
                        a.appendChild(cardWrapper.cloneNode(true));
                        cardWrapper.replaceWith(a);
                        onChange?.();
                    } else if (!link && isWrapped) {
                        const a = cardWrapper;
                        const newCard = a.querySelector('.card');
                        newCard.dataset.cardId = a.dataset.cardId;
                        a.replaceWith(newCard);
                        onChange?.();
                    } else if (link && isWrapped) {
                        cardWrapper.setAttribute('href', link);
                        if (isMedia) cardWrapper.classList.add('is-lightbox');
                        else cardWrapper.classList.remove('is-lightbox');
                        onChange?.();
                    }
                });

                // Add after description input
                const altLabel = document.createElement('label');
                altLabel.textContent = 'Image Alt Text';
                altLabel.style.cssText = 'display: block; font-size: 13px; font-weight: 500; margin-bottom: 4px; color: #333;';

                const altHint = document.createElement('small');
                altHint.textContent = 'For accessibility. Defaults to card title.';
                altHint.style.cssText = 'display: block; font-size: 11px; color: #666; margin-top: -6px; margin-bottom: 8px;';

                // Add after description input
                const altInput = document.createElement('input');
                altInput.type = 'text';
                altInput.placeholder = 'Alt text (leave empty for decorative images)';
                altInput.value = card.querySelector('img')?.getAttribute('alt') || titleInput.value;
                altInput.style.cssText = 'width: 100%; margin-bottom: 8px; font-size: 13px;';

                altInput.addEventListener('input', () => {
                    const img = card.querySelector('img');
                    if (img) {
                        img.alt = altInput.value; // Empty string is valid for decorative
                    }
                    onChange?.();
                });

                // When title changes, update alt input IF it matches the old title (keep sync)
                let lastTitle = titleInput.value;
                titleInput.addEventListener('input', () => {
                    card.querySelector('h3').textContent = titleInput.value;
                    headerTitle.textContent = titleInput.value || 'Untitled Card';
                    
                    // Auto-sync alt if it was using the title
                    if (altInput.value === lastTitle) {
                        altInput.value = titleInput.value;
                        const img = card.querySelector('img');
                        if (img) img.alt = titleInput.value;
                    }
                    lastTitle = titleInput.value;
                    
                    onChange?.();
                });

                // Append fields
                body.appendChild(group1);
                body.appendChild(group2);
                body.appendChild(titleInput);
                body.appendChild(descInput);
                body.appendChild(altLabel);
                body.appendChild(altInput);
                body.appendChild(altHint);
                body.appendChild(linkInput);

                // Toggle logic
                header.addEventListener('click', (e) => {
                    if (e.target.closest('.drag-handle')) return;
                    header.style.cursor = 'pointer';
                    const open = body.style.display === 'block';
                    body.style.display = open ? 'none' : 'block';
                    const icon = toggle.querySelector('svg');
                    if(open) {
                        icon.style.transform = 'rotate(360deg)';
                    } else {
                        icon.style.transform = 'rotate(180deg)';
                    }
                });
                
                // Show pointer cursor on hover over header (but not drag handle)
                header.addEventListener('mouseenter', (e) => {
                    if (!e.target.closest('.drag-handle')) {
                        header.style.cursor = 'pointer';
                    }
                });
                
                header.addEventListener('mouseleave', () => {
                    header.style.cursor = '';
                });

                section.appendChild(header);
                section.appendChild(body);

                return section;
            };

            // Sortable container wrapper
            const sortableContainer = document.createElement('div');
            sortableContainer.className = 'sortable-cards-container is-accordion';

            // Existing cards
            cards.forEach((card, i) => {
                const wrapper = card.parentElement.tagName === 'A' ? card.parentElement : card;
                sortableContainer.appendChild(createCardEditor(wrapper, i));
            });

            container.appendChild(sortableContainer);

            // Initialize SortableJS
            loadSortable().then((Sortable) => {
                new Sortable(sortableContainer, {
                    animation: 150,
                    handle: '.drag-handle',
                    ghostClass: 'sortable-ghost',
                    chosenClass: 'sortable-chosen',
                    dragClass: 'sortable-drag',
                    onEnd: function(evt) {
                        // Reorder actual cards in the element
                        const editorCards = Array.from(sortableContainer.children).filter(c => c.dataset.cardId);
                        const elementCards = Array.from(element.children);
                        
                        // Create a map of cardId to actual element
                        const cardMap = new Map();
                        elementCards.forEach(card => {
                            const cardId = card.dataset.cardId;
                            if (cardId) cardMap.set(cardId, card);
                        });

                        // Reorder actual elements based on editor order
                        editorCards.forEach((editorCard, index) => {
                            const cardId = editorCard.dataset.cardId;
                            const actualCard = cardMap.get(cardId);
                            if (actualCard && actualCard.parentElement === element) {
                                element.appendChild(actualCard);
                            }
                        });

                        onChange?.();
                    }
                });

                // Add CSS for sortable states
                const style = document.createElement('style');
                style.textContent = `
                    .sortable-ghost {
                        opacity: 0.4;
                        background: #f0f0f0;
                    }
                    .sortable-chosen {
                        background: #f9f9f9;
                    }
                    .sortable-drag {
                        opacity: 0.8;
                    }
                `;
                container.appendChild(style);
            }).catch((error) => {
                console.error('Failed to load SortableJS:', error);
            });

            // Add Card button
            const addBtn = document.createElement('button');
            addBtn.textContent = '+ Add Card';
            addBtn.onclick = (e) => {
                e.preventDefault();
                const newCard = document.createElement('div');
                newCard.className = 'card';
                newCard.dataset.cardId = `card-${Date.now()}`;
                newCard.innerHTML = `
                    <img src="https://placehold.co/600x600?text=No+Img" alt="">
                    <h3>New Card</h3>
                    <p>Description</p>
                `;
                element.appendChild(newCard);
                const editor = createCardEditor(newCard, sortableContainer.children.length);
                sortableContainer.appendChild(editor);
                onChange?.();
            };

            container.appendChild(addBtn);
            return container; // Must returns element to update content inside the plugin element
        }
    },

    mount: function (element, options) {
        element.style.display = 'grid';
        element.style.gridTemplateColumns = `repeat(${options.columns || 3}, 1fr)`;
        element.style.gap = '16px';
        return {};
    }
};
```

**The Style:**

```css
/* Card List Container */
[data-cb-type="card-list"] {
    display: grid;
    gap: 24px;
    padding: 20px 0;
}

/* Individual Card */
[data-cb-type="card-list"] .card {
    position: relative;
    display: flex;
    flex-direction: column;
    background-color: #ffffff;
    border: 1px solid #e5e7eb;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04);
    transition: box-shadow 0.25s ease, border-color 0.25s ease, transform 0.25s ease;
}

/* Hover Effect */
[data-cb-type="card-list"] .card:hover {
    transform: translateY(-2px);
    border-color: #d1d5db;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

/* Linked Card (entire card clickable) */
[data-cb-type="card-list"] a {
    display: block;
    border-radius: 12px;
    overflow: hidden;
    text-decoration: none;
    color: inherit;
    transition: transform 0.25s ease;
}
[data-cb-type="card-list"] a:hover {
    transform: translateY(-2px);
}
[data-cb-type="card-list"] a .card {
    cursor: pointer;
}

/* Card Image */
[data-cb-type="card-list"] .card img {
    width: 100%;
    height: 200px;
    object-fit: cover;
    display: block;
    transition: transform 0.3s ease, opacity 0.3s ease;
}
[data-cb-type="card-list"] .card:hover img {
    transform: scale(1.03);
    opacity: 0.96;
}

/* Card Content */
[data-cb-type="card-list"] .card h3,
[data-cb-type="card-list"] .card p {
    padding: 0 20px;
}

[data-cb-type="card-list"] .card h3 {
    margin: 16px 0 8px;
    font-size: 18px;
    font-weight: 500;
    color: #111827;
    line-height: 1.4;
}

[data-cb-type="card-list"] .card p {
    margin: 0 0 20px;
    font-size: 14.5px;
    color: #4b5563;
    line-height: 1.6;
}

/* Responsive */
@media (max-width: 1024px) {
    [data-cb-type="card-list"] .card img {
        height: 180px;
    }
}
@media (max-width: 768px) {
    [data-cb-type="card-list"] {
        grid-template-columns: 1fr !important;
        gap: 16px;
    }
    [data-cb-type="card-list"] .card img {
        height: 160px;
    }
}

```

**HTML Usage:**

```html
<div data-cb-type="card-list" data-cb-columns="3">
    <div class="card">
        <img src="image1.jpg" alt="Card 1">
        <h3>Card 1</h3>
        <p>Description 1</p>
    </div>
    <div class="card">
        <img src="image2.jpg" alt="Card 2">
        <h3>Card 2</h3>
        <p>Description 2</p>
    </div>
</div>
```

