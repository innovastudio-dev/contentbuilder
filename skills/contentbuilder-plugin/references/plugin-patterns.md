# Plugin Patterns (Situational)

> **Read this first.** These are **not** rules and **not** defaults. They are
> solutions to *specific* problems that came up in real plugins. Each one has a
> **Use when** and a **Do NOT use when**. If your plugin's situation doesn't
> match the trigger, **ignore the pattern** — reaching for it anyway adds
> complexity and rigidity where none is needed. The baseline rules every plugin
> follows live in [`plugin-best-practices.md`](./plugin-best-practices.md); this
> file is the opposite of that — a menu you pick from only when it applies.
>
> When in doubt, prefer the simplest thing (plain `.edit` content, plain
> settings). Only escalate to a pattern here when the simple thing can't do the job.

---

## 1. Data as a hidden, accessible list that `mount` renders

**Use when:** the visible output is *generated or animated* and therefore can't be
normal inline-editable text — e.g. an SVG chart, a scrolling ticker, a typed
headline. You have a set of data items (label/value, or phrases).

**Do NOT use when:** the content can just be inline-editable HTML. If a user can
edit the text in place with `class="edit"`, do that — it's simpler and directly
WYSIWYG. Never hide content that the user could have edited in place.

**How:** keep the data as a `<ul>` that is the single source of truth. Hide it
visually **but keep it in the accessibility tree** (an sr-only clip — *not*
`display:none`), so the block still has a real text alternative. Edit it through
the content editor (panel rows). `mount` reads the list and paints an
`aria-hidden` canvas/track from it.

```css
.plugin-data {              /* source of truth, screen-reader readable */
    position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
    overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; border: 0;
}
```

**Examples:** `chart`, `marquee`, `typewriter`.

---

## 2. Localizing text that `mount` generates

**Use when:** you build **editor-facing** text inside `mount` (a badge, a hint)
and want it translated. `mount` only receives `(element, options)` — it has no
`builder`, so `builder.out()` isn't available there.

**Do NOT use when:** the string lives in settings (uigenerator translates those
automatically) or in a custom editor (`openContentEditor` *does* get `builder` →
use `builder.out(...)`). Only reach for this for strings created in `mount`.

**How:** read the same map `out()` uses. When a language file is loaded,
ContentBuilder sets `opts.lang = window._txt`, and `out(s)` returns
`opts.lang[s] || s`. So look the English key up in `window._txt` yourself, and
hand the result to CSS as a quoted string via a variable.

```js
// inside mount, editor-only
if (inEditor) {
    const dict = element.ownerDocument.defaultView && element.ownerDocument.defaultView._txt;
    const label = (dict && dict['Click to flip']) || 'Click to flip';
    element.style.setProperty('--x-label', JSON.stringify(label)); // JSON.stringify => valid CSS string
}
```
```css
content: var(--x-label, "Click to flip");   /* English fallback baked in */
```

Translators then add `_txt["Click to flip"] = "…"` to their language file.

**Example:** `flip-card` ("Click to flip" badge).

---

## 3. Editor-only affordances (hints, shields) that never ship

**Use when:** the user needs help *while editing* — a hint badge, or a
transparent shield over an interactive embed (map/iframe/canvas) so the block
stays selectable — that must **not** appear on, or be saved into, the live page.

**Do NOT use when:** the guidance belongs to the live content. And never put an
editor hint *inside* an `.edit` region — it gets saved to the published page and
users can accidentally delete it.

**How:** scope under `body.data-editor`. For a passive hint use a CSS `::before`
(or a `mount`-injected non-content node) with `pointer-events: none` so it can't
block interaction. For a shield use an overlay that *does* capture pointer
events. Because it's `body.data-editor`-only and not part of content, it's
absent from both the saved HTML and the live page.

```css
body.data-editor [data-cb-type="thing"] .frame::after {  /* click-shield */
    content: ""; position: absolute; inset: 0; z-index: 1;
}
```

**Examples:** `flip-card` (badge), `map-embed` (click-shield).

---

## 4. Measure with `ResizeObserver`, not a one-shot `getBoundingClientRect`

**Use when:** `mount` needs the *rendered size* to compute something — e.g. an
animation duration for a constant px/second speed.

**Do NOT use when:** you don't need a measured size. Don't add an observer "just
in case."

**Why:** at `mount` time the width is unreliable — layout, web fonts, or even the
plugin's own (lazy-loaded) CSS may not be applied yet, especially on a fresh page
load or in Preview. A one-shot measurement often reads `0` and any fixed fallback
then ignores the user's setting. This was a real bug.

**How:** measure inside a function and drive it with a `ResizeObserver` — it
fires as soon as the true size is known *and* on every resize (keeping the result
correct and stable). Optionally recompute on `document.fonts.ready`. Disconnect
in `unmount`. Never fall back to a fixed value that ignores the setting.

```js
const apply = () => {
    const w = track.firstElementChild.getBoundingClientRect().width;
    if (w > 0) track.style.animationDuration = (w / speed).toFixed(2) + 's';
};
apply();
let ro = null;
if (view.ResizeObserver) { ro = new view.ResizeObserver(apply); ro.observe(track.firstElementChild); }
return { destroy() { if (ro) ro.disconnect(); } };
```

**Example:** `marquee` (speed).

---

## 5. Opt-in color to preserve a neutral / inherited look

**Use when:** a plugin is intentionally neutral — its text **inherits `color`**
from the page, so it adapts to light/dark automatically — and you want to *add*
an optional color override without losing that default.

**Do NOT use when:** the plugin already owns a palette (accent colors, etc.). A
plain `color` setting is fine there; don't force this toggle everywhere.

**How:** a native `<input type="color">` can't be "empty" — it always holds a
value — so a plain color setting would *always* override and kill the
inheritance. Gate it with a boolean (default off → inherit). In `mount`,
`removeProperty` when off, `setProperty` when on; CSS falls back to `inherit`.

```js
if (options.useCustomColor && options.textColor) element.style.setProperty('--x-color', options.textColor);
else element.style.removeProperty('--x-color');
```
```css
color: var(--x-color, inherit);
```

**Example:** `marquee` (text color).

---

## 6. Don't own a title/heading

**Use when:** the plugin renders structured content and any heading above it would
be better authored with the normal editor, where the user gets full text
formatting.

**Do NOT use when:** a label is intrinsic to the component and is editable in
place (e.g. a card's own heading as an `.edit` region). Keep those.

**How:** simply omit a built-in title element; note in the README that users add
a heading above the block with the editor. One less thing to style, and the user
gets more flexibility.

**Example:** `chart` (no built-in title).

---

## 7. In the editor, flip/reveal on click when hover would trap editing

**Use when:** a hover-driven interaction (hover-to-flip, hover-to-reveal) hides
part of the content, so in the editor the user can't reach that part to edit it —
hovering to get there re-triggers the effect.

**Do NOT use when:** the hover effect doesn't cover editable content (e.g. a
tooltip beside the text). Keep hover live in the editor then — plugins should
behave like the real thing while editing whenever they can.

**How:** gate the hover behavior to `body:not(.data-editor)` so it's live on the
page but off in the editor; in the editor drive the same effect with **click**
(`mount` wires it), and guard clicks that land inside an `.edit` region or on real
controls so text editing still works. Add an editor-only hint (see pattern 3) so
the click behavior is discoverable.

**Example:** `flip-card`.

---

## 8. Inline plugin applied to a text selection

**Use when:** your plugin is an **inline** span that wraps text — a tooltip, a
footnote/citation, a "define this term", a highlight-with-note — and users should
be able to select existing text and turn it into an instance (not just insert a
fresh snippet).

**Do NOT use when:** the plugin is a block/section, or is always inserted whole.
This is only for wrapping a *selection*.

**Why not auto-wire it:** plugin scripts are lazy-loaded by the **ContentBuilder
Runtime** — a plugin's code isn't present until an instance appears in content —
so the editor can't discover an "inline" declaration at init. Instead the editor
exposes a public method and the developer wires their own toolbar button to it.
The developer owns the **button** (icon/title/placement) and the **wrapper**
(class, ARIA, `data-cb-*`); the framework owns the mechanics.

**How:** add a text-editing button, and in its click handler call
`builder.convertSelectionToPlugin(pluginName, { create })`. The method takes the
selected text (the live selection, falling back to the one the editor saved — so
you don't snapshot it yourself), wraps it, stamps `data-cb-type`, mounts via the
runtime, and opens the plugin's settings/content editor.

Where the button goes depends on the host:

**ContentBox** — `builder.addTextButton(html, selector, exec)` adds it to the
Text panel of the control panel, under the Link/Image/Icon/SVG row. (Careful:
ContentBox's `builder.addButton()` is a different API — it adds a *sidebar*
button.) The settings panel opens in the control panel.

```js
// host init (e.g. src/index.js)
builder.addTextButton(buttonHtml, '.moreinfo-button', () => {
    builder.convertSelectionToPlugin('more-info', {
        create(selectedText, doc) {
            const span = doc.createElement('span');
            span.className = 'more-info-trigger style-info';
            span.setAttribute('role', 'button');
            span.setAttribute('tabindex', '0');
            span.setAttribute('data-cb-content', 'Add tooltip content…');
            return span;            // data-cb-type is stamped by the framework
        }
        // openEditor: true  (default) — opens the content editor afterward
    });
});
```

**ContentBuilder** — a custom name in `buttons` renders a `data-plugin`
placeholder that `builder.addButton()` replaces, on the floating RTE:

```js
options.buttons = [/* …defaults… */, 'createLink', 'moreinfo', 'tags', /* … */];

builder.addButton('moreinfo', buttonHtml, '[data-plugin="moreinfo"]', () => {
    builder.convertSelectionToPlugin('more-info', { create /* same as above */ });
});
```

`config` also accepts `attributes` / `className` / `tag` (convenience when you
don't need a `create` function) and `openEditor: false` (skip the editor).

**Example:** `more-info` (wired in the JS demo, `src/index.js`).

---

## 9. A rich-text field in a settings panel (Inscribe)

**Use when:** a plugin's settings panel needs real **rich-text editing** (bold /
italic / links / lists) for a value — e.g. a tooltip's popover HTML, a card's body.

**Do NOT use when:** the value is plain text (use a normal `<input>`/`<textarea>`),
or the content is edited **inline on the block** via `class="edit"` (that already
gets the full editor — see the baseline rules). This is only for rich text that
lives in the *settings panel*.

**Why not raw `contenteditable` + `execCommand`:** `execCommand` is deprecated and
gives inconsistent output and no active-state feedback. The builder ships a modern,
execCommand-free engine (Inscribe); `builder.createPanelEditor()` hands you an
instance and manages its lifecycle for you.

**How:** build your editable element (styled with `.cbx-rte`) and your toolbar
(`.cbx-iconbtn` buttons), then create the editor and wire the buttons to its
command API. Subscribe to `probe` for active-button states and pass `onChange`
to sync. **Don't destroy it yourself** — the framework tears every panel editor
down when the settings panel rebuilds or closes (multiple editors per panel are
fine).

```js
const editor = document.createElement('div');
editor.className = 'cbx-rte';
editor.innerHTML = element.dataset.cbContent || '';

const ed = builder.createPanelEditor(editor, {
    onChange: () => { element.dataset.cbContent = ed.getHTML(); onChange && onChange(); }
});

boldBtn.addEventListener('click', () => ed.toggle('bold'));   // toggle/link/list/setBlock/…
ed.on('probe', (s) => boldBtn.classList.toggle('is-active', !!s.bold));
```

Commands: `toggle('bold'|'italic'|'underline')`, `link(url)` / `unlink()`,
`list('ul'|'ol')`, `setBlock(tag)`, `insertHTML`, `getHTML()` / `setHTML()`.
Panel CSS helpers: `.cbx-rte`, `.cbx-rte-toolbar`, `.cbx-rte.cbx-rte-code`,
`.cbx-hint`, `.cbx-iconbtn.is-active`.

**Example:** `more-info` (Popover Content field).

---

## 10. Make an embedded `<iframe>` selectable in the editor

**Use when:** your plugin renders an interactive `<iframe>` (a map, a video embed, an external widget). In the editor an iframe swallows clicks, so the block can't be selected/dragged.

**Do NOT use when:** the plugin has no iframe (regular DOM handles clicks fine), or the embed is a plain `<img>`/`<video>` (those don't trap selection).

**Why not a plugin-managed shield:** block selection is the *editor's* job, not the plugin's — and a shield the plugin adds/removes in `mount` is fragile (it can miss editor-init timing, and it duplicates logic every plugin would have to get right). The editor exposes a one-class **contract** instead.

**How:** in `mount`, add the class `iframe-overlay` to the element that wraps the `<iframe>`:

```js
mount(element) {
    const frame = element.querySelector('.map-embed-frame');
    frame.classList.add('iframe-overlay');   // opt in — that's it
    // …build the iframe inside `frame`…
}
```

The editor does the rest (a CSS `::after` shield lets a click select the block; the selected one gets `.iframe-active`, which drops the shield so the iframe is interactive; clicking elsewhere re-shields). It's **editor-only** — the shield CSS ships in the editor stylesheet, not the runtime, so the live page is unaffected and the iframe is always interactive there. No `unmount` cleanup needed.

**Example:** `map-embed` (`.map-embed-frame`).

---

*Add to this file when a genuinely reusable, situational solution proves itself in
a real plugin — always with a **Do NOT use when** so it stays a menu, not a mandate.*
