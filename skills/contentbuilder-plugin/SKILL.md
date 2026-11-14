---
name: contentbuilder-plugin
description: Build plugins for ContentBuilder.js and ContentBox.js, the content blocks placed with data-cb-type that the editor and the runtime load (FAQ, tabs, sliders, forms, maps and the like). Use when the user asks to create, change or fix a plugin, add settings or a custom editor panel to one, or make a plugin accessible, translatable or editable in the editor.
---

# ContentBuilder / ContentBox plugins

A plugin is a folder with an `index.js` (a `mount` function, optional settings and
custom editor) and a `style.css`. Content uses it with
`<div data-cb-type="plugin-name" data-cb-*="...">`. The same plugin works in
ContentBuilder and ContentBox, in the editor and on the live site.

## Read the references first

| File | Read it when |
|------|--------------|
| `references/plugin-dev-guide.md` | Always. Structure, `mount` / `unmount`, options from `data-cb-*`, settings panels, custom editors, editable areas. |
| `references/plugin-best-practices.md` | Always. The baseline rules every plugin follows: accessibility, markup, CSS scoping, the README, translatable labels. |
| `references/plugin-example.md` | Building a real plugin with settings or a custom editor. A complete annotated plugin to copy from. |
| `references/plugin-patterns.md` | Only when the plugin has one of the specific problems it lists. Each pattern says when to use it and when not to. Never apply them by default. |
| `references/plugin-catalog.md` | To check whether a plugin already exists, or to see how an existing one is used. |

The plugin folders the references link to (`faq/`, `pricing-table/` and so on) ship
with ContentBuilder and ContentBox in `assets/plugins/`. When the user's project has
that folder, read a similar plugin's `index.js` there before writing a new one.

## What to deliver

A plugin folder, named after the plugin in lowercase with hyphens:

```
my-plugin/
  index.js
  style.css
  README.md
```

- Hand it over as one zip, `my-plugin.zip`, holding the `my-plugin/` folder with
  the three files inside, so the user unzips it straight into `assets/plugins/`.
  Write the folder and zip it with a script; do not hand over loose files. If you
  cannot create files, say the plugin name and folder name first, then give the
  three files.
- The `README.md` follows the structure in `plugin-best-practices.md`, with the
  copy-paste HTML that uses the plugin.
- Every front-end style is scoped under `[data-cb-type="my-plugin"]`.
- Build in-page content blocks, not page-level widgets fixed to the window
  (cookie bars, back-to-top buttons, reading-progress bars).

## Tell the user how to try it

1. Unzip it into the plugins folder, next to the existing plugins
   (usually `assets/plugins/`). A runtime with `pluginFolder` set loads it by name.
   If the page lists its plugins by hand (a `plugins: { ... }` map), add an entry
   for the new one at the end of the map.
2. Paste the HTML from the plugin's `README.md` into the page, then open it in the
   editor and in the published page.
