# Plugin Development Guide

**How to create a plugin for ContentBuilder or ContentBox.**

## 🚀 Getting Started

### Plugin Structure

Every plugin is just a folder with 2 files:

```
plugins/
  my-plugin/
    index.js      ← Your JavaScript code
    style.css     ← Your styles
```

### Your First Plugin: a Click Counter

**Step 1: Create the Plugin File**

File: `plugins/click-counter/index.js`

```javascript
export default {
    name: 'click-counter',      	// Plugin identifier
    displayName: 'Click Counter', // Friendly name shown in UI
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

**Step 3: Let the Runtime Find the Plugin**

Put the plugin in the plugins folder (`/assets/plugins/click-counter/index.js` and `style.css`) and tell the runtime where the folder is. A plugin used in the content is then loaded by its name, with no registration:

```javascript
const runtime = new ContentBoxRuntime({
    pluginFolder: '/assets/plugins/'
});

runtime.init();
```

**Or register it by hand.** Use this for a plugin kept somewhere else (another folder or a CDN), or when you set `autoRegisterPlugins: false` so the runtime loads only the plugins you list:

```javascript
const runtime = new ContentBoxRuntime({
    pluginFolder: '/assets/plugins/',
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

ContentBuilder or ContentBox Runtime handles the loading and initialization of the plugins.

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

- Receives the element and its options
- Must return an object (can be empty `{}` or `null` on error)

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

## 🔧 Configuration Options: Settings and **Custom Editor**

Your plugin can have configuration in two ways: **Simple Settings** (auto-generated UI) and **Custom Editor** (advanced UI).

```javascript
export default {
    name: 'my-plugin',
    
    settings: {
        // 1. Simple Settings (auto-generated UI)
        columns: { type: 'number', label: 'Columns', default: 3 }
    },
    
    editor: {
        // 2. Custom Editor
        openContentEditor: function(element, builder, onChange) {
            // Custom UI for managing list items (add, remove, edit), etc.
        }
    },
    
    mount: function(element, options) {
        // Your plugin logic
    }
}
```

## ⚙️ Simple Settings (Auto-Generated UI)

When using ContentBox or ContentBuilder, the builder will automatically generate UI for the plugin based on the plugin **settings**. With the UI, users can adjust/configure the plugin visually.

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

**HTML:**

```html
<div data-cb-type="logo-loop"
     data-cb-speed="20" 
     data-cb-direction="left"
     data-cb-pause-on-hover="true">
</div>
```

### Supported Field Types

#### text

```javascript
title: {
    type: 'text',
    label: 'Title',
    default: 'My Title',
    placeholder: 'Enter title...'
}
```

#### textarea

```javascript
description: {
    type: 'textarea',
    label: 'Description',
    default: '',
    rows: 4,
    placeholder: 'Enter description...'
}
```

#### number

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

#### boolean

```javascript
enabled: {
    type: 'boolean',
    label: 'Enable Feature',
    default: true
}
```

#### select

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

#### radio

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

#### color

```javascript
backgroundColor: {
    type: 'color',
    label: 'Background Color',
    default: '#ffffff'
}
```

#### range

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

### Conditional Fields (`dependsOn`)

A field can declare `dependsOn` to appear only when another setting has a
particular value — so a setting that doesn't apply isn't shown at all:

```javascript
settings: {
    sizeMode: {
        type: 'select', label: 'Size', default: 'fixed',
        options: [
            { value: 'fixed', label: 'Fixed Height' },
            { value: 'fill', label: 'Fill Container' }
        ]
    },
    // Only shown while Size is "Fixed Height"
    height: {
        type: 'number', label: 'Height', default: 380, unit: 'px',
        dependsOn: { field: 'sizeMode', value: 'fixed' }
    }
}
```

Supported forms:

| Form | Shows the field when |
|---|---|
| `{ field: 'x', value: 'a' }` | `x` is `a` |
| `{ field: 'x', value: ['a', 'b'] }` | `x` is `a` **or** `b` |
| `{ field: 'x', value: true }` | the `x` checkbox is on |
| `{ field: 'x', not: 'a' }` | `x` is anything **but** `a` |
| `{ field: 'x' }` | `x` is set (non-empty / checked) |
| `[{ field: 'x', … }, { field: 'y', … }]` | **all** conditions pass |

Notes:

- A hidden field keeps its value — it's still saved to the element and comes
  back unchanged when the field applies again, so switching back and forth
  never loses what the user typed.
- Hiding uses `display: none`, so a setting that doesn't apply is also out of
  the tab order and the accessibility tree.
- A group whose fields are all hidden hides its title too.
- Naming a field that doesn't exist shows the field rather than hiding it — a
  typo can't make a setting disappear from the panel.
- The depended-on field doesn't have to be in the panel. Values resolve as
  *live form input → element's `data-cb-*` attribute → schema `default`*, so a
  rule can key off an [attribute-only setting](#attribute-only-settings-hidden).

### Attribute-Only Settings (`hidden`)

`hidden: true` keeps a setting out of the panel while it still works as a
`data-cb-*` attribute — for a choice that belongs to whoever authors the block's
HTML (a snippet, a section template) rather than to the person editing the page:

```javascript
sizeMode: {
    type: 'select', label: 'Size', default: 'fixed',
    options: [
        { value: 'fixed', label: 'Fixed Height' },
        { value: 'fill', label: 'Fill Container' }
    ],
    hidden: true            // set via data-cb-size-mode="fill" in the HTML
}
```

- The runtime builds `options` from **every** `data-cb-*` attribute on the
  element, independently of the settings schema, so `mount` still receives
  `options.sizeMode`.
- Editing other fields never touches a hidden setting's attribute — only
  rendered inputs are written back.
- Keeping the field in `settings` (rather than dropping it) is still worth it:
  it documents the option and gives its `default` to `dependsOn`.
- Other fields may `dependsOn` a hidden setting — the rule reads the element's
  attribute, falling back to the schema `default`.

### Settings and HTML Attributes

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

## 🎨 Custom Editor (Advanced UI)

The custom editor allows you to create **custom UI** to build, modify, and manage **HTML structure** of your plugin. 

### The openContentEditor Function

```javascript
export default {
    name: 'my-plugin',
    
    settings: {

    },
    
    editor: {
        // Custom Editor
        openContentEditor: function(element, builder, onChange) {
            // Build your custom UI
            // Custom UI is used for building HTML structure of your plugin element, e.g. add, remove, edit list items, etc.
        		const container = document.createElement('div');
        
            // ... add your custom UI elements ...

            return container; // Must return a DOM element
        }
    },
    
    mount: function(element, options) {
        // Never build HTML structure of your plugin element in mount.
      	// Use mount only for adding style or behavior of your plugin using options
    }
}
```

**Important:** Inside the openContentEditor function, create a container where you build your custom UI and then return the container.

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

Access builder features like file pickers.

**For a URL field, use `createFileButtons`.** It returns the buttons to put next
to the input: *select* (opens the asset manager) and *upload* (from the user's
computer). Selecting is only offered where the app has configured an asset
manager, so this is the way to be sure your plugin never shows a button that
cannot open anything:

```javascript
openContentEditor: function(element, builder, onChange) {
    const input = document.createElement('input');
    input.type = 'text';
    input.className = 'cbx-grow';

    const row = document.createElement('div');
    row.className = 'cbx-row';
    row.append(input, ...builder.createFileButtons('image', (url) => {
        input.value = url;
        imageElement.src = url;
        onChange();
    }));
}
```

Pass a function instead of a type for a field whose kind follows another control
(a Media Type select, say); it is read on each click:

```javascript
builder.createFileButtons(() => (typeSelect.value === 'video' ? 'video' : 'image'), onPick)
```

The two actions are also available on their own:

```javascript
builder.hasFilePicker('image')                  // is an asset manager configured?
builder.openFilePicker('image', (url) => {...}) // open the asset manager
builder.openFileUpload('image', (url) => {...}) // upload from the computer
```

**Available File Picker Types:**

- `'image'` - Image files only
- `'video'` - Video files only
- `'audio'` - Audio files only
- `'file'` - All file types
- `'media'` - Image and Video files

**For a list the user can reorder, use `sortableList`.** The editor handles the
dragging:

```javascript
builder.sortableList(stack, {
    handle: '.cbx-drag',        // optional; omit to drag a row from anywhere
    onSort: () => {
        // stack is already in its new order — apply it to the content
        Array.from(stack.children).forEach((row) => {
            element.appendChild(itemsById.get(row.dataset.itemId));
        });
        onChange();
    }
});
```

Safe to call every time you rebuild the list; repeat calls on the same element
do nothing.

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

### Important:

**You just need to return the container** - the builder handles everything else (the builder will place the returned container into its own panel system).

```javascript
openContentEditor: function(element, builder, onChange) {
    const container = document.createElement('div');
    
    // Build your UI inside container
    container.appendChild(titleInput);
    container.appendChild(descriptionTextarea);
    container.appendChild(addButton);
    
    return container; // CRITICAL: Required!
}
```

Also, **do not style the container**, such as adding: **container.style.cssText = 'padding: ...'** This is not needed since the builder will place the returned container in its own panel.

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

## 🎯 Editable Areas

Editable areas are content regions within your plugin that users can edit directly in the page editor **without needing complex code** in your plugin.

**Requirements:** `is-subblock` class + `edit` class

```html
<div data-cb-type="callout-box">
    <!-- Rich content editing: full formatting -->
    <div class="is-subblock edit">
      	<h3>Important Information</h3>
        <p>This is an important message with <strong>formatting</strong>!</p>
        <p>You can add multiple paragraphs and <a href="#">links</a>.</p>
    </div>
</div>
```
