# Plugin Development Best Practices

> These are the **baseline rules** that apply to every plugin. For *situational*
> solutions (used only when a specific problem shows up — e.g. animated content,
> localizing `mount`-generated text, editor-only hints), see the opt-in companion
> [`plugin-patterns.md`](./plugin-patterns.md). Don't apply those by default.

## Essential Guidelines for Professional Plugins

### 1. Accessibility is Required

Accessibility attributes should be in the HTML structure from the start.

**HTML with Accessibility:**

```html
<div data-cb-type="sticky-category-nav" role="navigation" aria-label="Category navigation">
    <div class="category-nav-container">
        <ul class="category-nav-list">
            <li><a href="#hero">Hero</a></li>
            <li><a href="#articles">Articles</a></li>
            <li><a href="#contact">Contact</a></li>
        </ul>
    </div>
</div>
```

**Don't add a role an element already has.** A `<ul>` is a list and an `<li>` is a
list item; wrapping them in `role="list"` makes a list whose children are not
list items, which is a failure, not an improvement. The same goes for
`role="button"` on a `<button>`. Reach for a role when there is no element that
says it.

**Editor - Maintain Accessibility When Adding Items:**

```javascript
editor: {
    openContentEditor: function(element, builder, onChange) {

        //...

        const addButton = document.createElement('button');
        addButton.textContent = '+ Add Link';
        addButton.onclick = () => {
            const newLi = document.createElement('li');
            
            const newLink = document.createElement('a');
            newLink.href = '#new';
            newLink.textContent = 'New Link';
            
            newLi.appendChild(newLink);
            list.appendChild(newLi);
            onChange?.();
        };
    }
}
```

#### A widget role needs a name

`role="progressbar"`, `slider`, `tab`, `dialog` and the rest announce as their
role and nothing else unless you name them. The visible label is usually the
name, so take it from there:

```javascript
const label = item.querySelector('.pb-label');
if (label) track.setAttribute('aria-label', label.textContent.trim());
```

#### A copy must not keep its tab stops

Blocks that clone content for a seamless loop, marquees and logo walls, hide the
copies with `aria-hidden="true"`. That hides them from a screen reader but not
from the Tab key, so a keyboard user lands on invisible duplicates. Take the
copy's controls out of the tab order as you make it:

```javascript
clone.setAttribute('aria-hidden', 'true');
clone.querySelectorAll('a[href], button, [tabindex]').forEach(el => el.setAttribute('tabindex', '-1'));
```

**Checklist:**
- ✅ Semantic HTML (`<nav>`, `<button>`, `<a>`), and no role the element already has
- ✅ ARIA roles and labels, including a name on every widget role
- ✅ Keyboard navigation (Tab, Enter, Space)
- ✅ Focus indicators
- ✅ Hidden copies out of the tab order

### 2. SEO-Friendly Structure

Content must exist in HTML before JavaScript executes.

**❌ Wrong - JavaScript-Generated Content:**

```javascript
mount: function(element) {
    // Bad: Content invisible to search engines
    element.innerHTML = `
        <nav>
            <ul>
                <li><a href="#home">Home</a></li>
            </ul>
        </nav>
    `;
}
```

**✅ Correct - HTML Structure with JavaScript Behavior:**

```html
<!-- Content visible in HTML -->
<div data-cb-type="my-nav" role="navigation">
    <nav>
        <ul>
            <li><a href="#home">Home</a></li>
            <li><a href="#about">About</a></li>
        </ul>
    </nav>
</div>
```

```javascript
mount: function(element) {
    // Good: Only add behavior to existing content
    const links = element.querySelectorAll('a');
    links.forEach(link => {
        link.addEventListener('click', handleClick);
    });
}
```

### 3. Editor Builds Structure

Use the custom editor to create, modify, and manage HTML elements.

```javascript
editor: {
    openContentEditor: function(element, builder, onChange) {
        const container = document.createElement('div');
        const list = element.querySelector('.nav-list');
        
        // Create UI to add new items
        const addButton = document.createElement('button');
        addButton.textContent = '+ Add Link';
        addButton.onclick = () => {
            // Build new HTML structure
            const newItem = document.createElement('li');
            const newLink = document.createElement('a');
            newLink.href = '#new';
            newLink.textContent = 'New Link';
            newLink.setAttribute('role', 'listitem');
            
            newItem.appendChild(newLink);
            list.appendChild(newItem);
            onChange?.();
        };
        
        container.appendChild(addButton);

        return container;
    }
}
```

### 4. Mount Adds Behavior Only

The mount function applies runtime behavior and styling. Never create HTML structure here.

```javascript
mount: function(element, options) {
    // Apply dynamic styles from settings
    element.style.setProperty('--bg-color', options.backgroundColor);
    element.style.setProperty('--text-color', options.textColor);
    
    // Add event listeners
    const links = element.querySelectorAll('a');
    links.forEach(link => {
        link.addEventListener('click', (e) => {
            e.preventDefault();
            scrollToSection(link.href);
        });
    });
    
    // Track scroll position
    window.addEventListener('scroll', () => {
        updateActiveState(links);
    });
    
    return {};
}
```

**What mount should do:**
- ✅ Apply CSS custom properties
- ✅ Add event listeners
- ✅ Start animations/timers
- ❌ **Never** create HTML elements
- ❌ **Never** build structure

### 5. Styling: Two Surfaces, Two Rules

A plugin has two distinct styling surfaces. Keep them separate.

#### Surface A — the rendered block (`style.css`)

This is the block your visitors see. Style it however you like — **the only rule is that every selector must be scoped to your plugin's block** so it can't leak into other plugins, the host page, or the editor UI. Prefix the type attribute onto every rule:

```css
/* ✅ scoped — safe */
[data-cb-type="pricing-table"] .plan-name { font-weight: 600; }

/* ❌ global — collides with other plugins and the page */
.plan-name { font-weight: 600; }
```

There are **no shared tokens or base classes to learn** for the rendered block. Scoping is enough.

##### 💡 Suggestion (not a rule): design for flexibility, not one fixed look

*Use judgement — this is a nudge, not a requirement, and many plugins won't need any of it.*

When a block has a strong visual style, consider exposing the **stylistic choices** as options so it adapts to different site designs instead of locking into one aesthetic. A block that only looks right when it's rounded and softly shadowed will feel out of place on a flat/minimal or sharp/editorial site. This matters most for AI-generated plugins, which otherwise tend to bake in a single "nice" default.

Good candidates **when they apply to the block**:
- **Card shadow** on/off (flat vs. elevated)
- **Rounded vs. square** corners
- **Border** on/off, and the accent color
- sometimes density/size or alignment

It's cheap with the patterns you're already using — a `boolean` setting → `data-cb-*` attribute → a scoped CSS override, plus keeping colors as CSS variables (which also makes dark-mode adoption trivial):

```css
[data-cb-type="my-block"][data-cb-shadow="false"] .card { box-shadow: none; }
[data-cb-type="my-block"][data-cb-rounded="false"] .card { border-radius: 0; }
```

Default to your intended look so existing embeds are unchanged, and **only add options that give real design latitude** — skip ones that would just be noise.

**Don't force these onto blocks that are already neutral.** A background-effect or a divider has no "card" to soften. And a block that's already visually restrained — a plain form, an underline tab bar — adapts across designs on its own; leave it alone rather than bolting on flat/square toggles it doesn't need. These options earn their place on strongly-styled blocks (cards with shadows and big radii, like `pricing-table` / `testimonials` — see `data-cb-shadow`, `data-cb-rounded`), not on everything.

##### A `color` setting's `default` must equal the CSS variable it feeds

When a color setting drives a CSS variable (`element.style.setProperty('--x', options.x)`), the setting's `default` and the variable's default in `style.css` **must be the exact same value**. If they differ, the panel's color swatch shows one color while the block renders another — the picker lies about the real color.

```css
[data-cb-type="my-block"] { --card-border: #e5e7eb; }   /* stylesheet default */
```
```js
borderColor: { type: 'color', default: '#e5e7eb' }      /* ✅ same value — swatch matches the block */
borderColor: { type: 'color', default: '' }             /* ❌ swatch shows empty; block still renders #e5e7eb */
```

Two consequences of how the color field works — it stores `toHex(color)`, so it is **hex-only, no alpha**:

- **Keep the variable's default a plain hex** — no `rgb(255 255 255 / 85%)`, `rgba()`, or any alpha. The picker can't represent it, so it would round to the opaque hex and drift out of sync. If you want a translucent fill, that's what a **Glass / opacity** option is for; keep the base solid fill a solid hex.
- **The "inherit the site theme" case is the one place an empty default is right** — e.g. a text color you want to inherit rather than pin (see `animated-stats` Number Color). There, pair `default: ''` with a CSS variable default of `inherit` (not a concrete color) and `setProperty('--x', options.x || '')`. Both sides then say "unset." Never mix an empty default with a *concrete* variable default — that's the mismatch above.

#### Surface B — the settings panel (`openContentEditor` UI)

Your custom editor renders inside the builder's settings panel (`.is-modal.pluginsettings`), which **already styles your controls** — do not fight it:

- `input`, `select`, `textarea`, `button` are **automatically full-width and styled**. Don't set widths/backgrounds on them.
- `label` is block, 13px, medium weight — just use `<label>`.
- `<label class="checkbox">` gives you an inline *checkbox + text* row for free.
- The builder ships an icon sprite — reuse it: `<svg><use xlink:href="#icon-trash2"></use></svg>`.

**Do not ship a `<style>` block for panel layout.** For any layout the plain vertical stack can't express, use the shared **`.cbx-*` helper classes** (`cbx` = *ContentBuilder eXtension*), which are baked into the builder and already win specificity over the panel defaults:

| Class | Use |
|---|---|
| `.cbx-stack` | Vertical stack with even spacing — use for **every** vertical list: the whole form, fields, rows, *and* lists of `.cbx-item`s (item margins are auto-handled, so no doubling) |
| `.cbx-field` | One labelled control: a `<label>` + its input/select/textarea. Owns the label→control gap |
| `.cbx-item` | A collapsible card for a repeatable item (a plan, a slide, a card…). Self-spacing when standalone; inside a `.cbx-stack` the stack's gap takes over automatically |
| `.cbx-item-head` | Its clickable header row (holds drag handle + title + delete) |
| `.cbx-title` | The item's title text inside the header (truncates) |
| `.cbx-item-body` | The collapsible body — already a stack, so its fields space themselves |
| `.cbx-row` | Horizontal group; children keep natural width |
| `.cbx-grow` | Put on the `.cbx-row` child that should expand (input, column) |
| `.cbx-drag` | Drag handle; pass as `handle` to `builder.sortableList()` |
| `.cbx-iconbtn` | Compact square icon button (undoes `button { width:100% }`); add `.is-danger` for red |

#### The one hard rule: never set a margin or padding in editor code

> **All spacing is owned by the `.cbx-*` primitives.** Do not write `el.style.margin*`, `padding`, or a `cssText` with spacing in your `openContentEditor`. If two things need space between them, make their parent a `.cbx-stack`; if a label needs space above its input, wrap them in a `.cbx-field`.

This is what prevents the two most common panel bugs: labels sitting too tight against their inputs, and hand-added margins that *double up* with spacing a container already provides. The spacing scale is defined once on `.is-modal.pluginsettings`, so every plugin shares the same rhythm automatically:

- `--cbx-space` (14px) — between fields/rows inside a form or card body
- `--cbx-card-gap` (9px) — between cards (`.cbx-item`) in a list; auto-applied to any stack that holds cards
- `--cbx-row-gap` (8px) — between rows in a sub-stack made **only** of `.cbx-row`s (e.g. a feature list); auto-applied. A single side-by-side `.cbx-row` sitting among normal fields keeps the field spacing, not this
- `--cbx-label-gap` (6px) — between a label and the control right under it

Want a different gap? Change the token — don't sprinkle margins.

**Structure a form as a stack of fields; wrap the label with its control:**

```javascript
// The returned root is a stack → its children space themselves.
const container = document.createElement('div');
container.className = 'cbx-stack';

// Each field is a label + control unit.
const nameField = document.createElement('div');
nameField.className = 'cbx-field';
nameField.innerHTML = '<label>Name</label><input type="text">';

// Two fields side by side: a row of .cbx-field.cbx-grow — spacing stays identical.
const row = document.createElement('div');
row.className = 'cbx-row';
row.innerHTML = `
    <div class="cbx-field cbx-grow"><label>Min</label><input type="text"></div>
    <div class="cbx-field cbx-grow"><label>Max</label><input type="text"></div>
`;

const addBtn = document.createElement('button');   // no margin — the stack gaps it
addBtn.type = 'button';
addBtn.textContent = '+ Add';

container.append(nameField, row, addBtn);
```

**A repeatable item editor with zero custom CSS and zero margins:**

```javascript
const list = document.createElement('div');
list.className = 'cbx-stack';                       // every vertical list is a stack

const item = document.createElement('div');
item.className = 'cbx-item';
item.innerHTML = `
    <div class="cbx-item-head">
        <span class="cbx-drag">⠿</span>
        <span class="cbx-title">Item name</span>
        <button type="button" class="cbx-iconbtn is-danger" aria-label="Delete">
            <svg><use xlink:href="#icon-trash2"></use></svg>
        </button>
    </div>
    <div class="cbx-item-body">
        <div class="cbx-field"><label>Name</label><input type="text"></div>
        <div class="cbx-row">
            <div class="cbx-field cbx-grow"><label>Min</label><input type="text"></div>
            <div class="cbx-field cbx-grow"><label>Max</label><input type="text"></div>
        </div>
    </div>
`;
list.appendChild(item);
```

> Collapsing an item? Toggle `body.style.display` between `'none'` and `''` (empty) — never `'block'`, which would override `.cbx-item-body`'s flex layout and kill the gaps.

**Pattern: an input with an adjacent action button** (e.g. a "browse" button for a file/image picker). Keep the label on top, and put the **input + button in the row** — not the whole field. The button then aligns with the input; if you instead wrap the label+input in the row alongside the button, the button aligns against the taller label+input block and sits too high.

```javascript
// ✅ label on top; the row holds just the input + button
const field = document.createElement('div');
field.className = 'cbx-field';
field.innerHTML = `
    <label>Avatar</label>
    <div class="cbx-row">
        <input type="text" class="cbx-grow" placeholder="Image URL">
    </div>
`;
// then put the file buttons in the row — select is included only where the app
// has an asset manager, so nothing is shown that cannot open:
// row.append(...builder.createFileButtons('image', url => { … }))
```

```html
<!-- ❌ button is a sibling of the whole field, so it centers against label+input -->
<div class="cbx-row">
    <div class="cbx-field cbx-grow"><label>Avatar</label><input type="text"></div>
    <button class="cbx-iconbtn">…</button>
</div>
```

See `plugins/pricing-table/index.js` for a full reference implementation.

> The `.cbx-*` classes and spacing tokens live in `src/scss/contentbuilder.scss` under `.is-modal.pluginsettings`. If you add or retune one, rebuild with `npm run scss2`. They are editor-only (settings panels don't exist on the live site), so they never affect the rendered block.

### 6. Ship a `README.md` in the plugin folder

Every plugin should include a `README.md` next to its `index.js`, so anyone can adopt it without reading the source. Use a consistent structure (see `plugins/pricing-table/README.md` and `plugins/contact-form/README.md`):

- **One-line summary** of what the block does.
- **Server-side required: Yes/No** and **external dependencies** up front.
- **Embed code** — a complete, paste-ready example.
- **Options table** — every `data-cb-*` attribute with values + defaults.
- **Structure notes** — anything hand-editable (special classes, data attributes).

**If the plugin talks to a server, document the contract explicitly.** ContentBuilder is a client-side product: a plugin may *send* data, but standing up the endpoint (email, storage, spam protection, auth) is the **integrating developer's responsibility**. Any `server.js` route or `public/api/*.php` we ship is an **example for guidance only**, not a production backend. Spell out: the request format the plugin sends, what response it expects (e.g. 2xx = success, non-2xx = error), and give minimal Node + PHP reference snippets plus a note about third-party form/services.

### 7. Make settings labels translatable

ContentBuilder translates UI strings through `out(str)`, which looks the string up in the active language file (`contentbuilder/lang/en.js`, `fr.js`, …). The **key is the English source string** — untranslated strings pass through unchanged, so you always just write English.

- **Auto-generated settings** (from the `settings` object) are translated **automatically** — the settings-form generator runs every label, group title, select/radio option, and placeholder through `out()`. Just write English labels; nothing else to do.
- **Custom editor** (`openContentEditor`) is hand-built, so wrap your own user-facing strings with the builder's translator. Grab a short helper at the top and use it for labels, button text, placeholders, and `aria-label`s:

  ```javascript
  openContentEditor: function (element, builder, onChange) {
      const out = (s) => (builder && typeof builder.out === 'function') ? builder.out(s) : s;
      // …
      label.textContent = out('Tab Label');
      addBtn.textContent = out('+ Add Tab');
      input.placeholder = out('Field label');
      del.setAttribute('aria-label', out('Delete tab'));
  }
  ```

- **Don't translate rendered-block text** (the content site visitors see) via `out()` — that's the site's language, owned by the site, and `mount` has no access to the editor's translator. Only translate **editor UI** strings.
- **Translations live in the language files**, keyed by the English string (many common ones like `Delete`, `Add`, `Color` already exist). To find every string your plugin needs translated, run the editor with `checkLang: true` — `out()` logs each untranslated string to the console; add those to `fr.js` etc.

See the four reference plugins (`pricing-table`, `testimonials`, `contact-form`, `tabs`) for the `out()` pattern in a custom editor.