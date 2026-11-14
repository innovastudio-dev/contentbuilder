**Plugin Example**

### Sortable Card List with All Features

This example follows the conventions in `plugin-best-practices.md` — the editor panel uses the shared **`.cbx-*` helper classes** (no `<style>` block, no hand-set margins), and every front-end selector is **scoped under `[data-cb-type="card-list"]`**.

```javascript
export default {
    name: 'card-list',
    displayName: 'Card List',
    version: '1.0.0',

    settings: {
        _groups: [
            {
                id: 'layout',
                label: 'Layout Settings',
                fields: ['itemsPerRow', 'gap', 'aspectRatio']
            },
            {
                id: 'appearance',
                label: 'Appearance',
                fields: ['accentColor']
            }
        ],
        itemsPerRow: {
            type: 'number',
            label: 'Items Per Row',
            default: 3,
            min: 1,
            max: 6,
            group: 'layout'
        },
        gap: {
            type: 'number',
            label: 'Gap Between Items',
            default: 16,
            min: 0,
            max: 48,
            unit: 'px',
            group: 'layout'
        },
        aspectRatio: {
            type: 'select',
            label: 'Card Aspect Ratio',
            default: '16/9',
            options: [
                { value: '16/9', label: '16:9 (Landscape)' },
                { value: '4/3', label: '4:3 (Standard)' },
                { value: '1/1', label: '1:1 (Square)' },
                { value: '3/4', label: '3:4 (Portrait)' },
                { value: 'auto', label: 'Auto (Content Height)' }
            ],
            group: 'layout'
        },
        accentColor: {
            type: 'color',
            label: 'Accent Color',
            default: '#0066cc',
            group: 'appearance'
        }
    },

    editor: {
        openContentEditor: function(element, builder, onChange) {
            // PATTERN: Helper function to detect if URL is video
            const isVideoUrl = (url) => /\.(mp4|webm|ogg|mov)$/i.test(url);

            // PATTERN: Helper to extract media source from various element structures
            const getMediaSrc = (item) => {
                const media = item.querySelector('img, video, video source');
                let src = '';
                if (media) {
                    if (media.tagName === 'IMG') src = media.getAttribute('src') || '';
                    else if (media.tagName === 'VIDEO') src = media.getAttribute('src') || media.querySelector('source')?.getAttribute('src') || '';
                    else if (media.tagName === 'SOURCE') src = media.getAttribute('src') || '';
                }
                return src;
            };

            // PATTERN: Helper to switch media type based on URL
            const switchMediaType = (item, url) => {
                const mediaWrapper = item.querySelector('.item-media');
                const isVideo = isVideoUrl(url);
                const currentMedia = mediaWrapper.querySelector('img, video');
                const currentIsVideo = currentMedia?.tagName === 'VIDEO';

                if (isVideo && !currentIsVideo) {
                    const video = document.createElement('video');
                    video.muted = true;
                    video.playsInline = true;
                    const source = document.createElement('source');
                    source.src = url;
                    source.type = 'video/' + url.split('.').pop().toLowerCase();
                    video.appendChild(source);
                    mediaWrapper.innerHTML = '';
                    mediaWrapper.appendChild(video);
                } else if (!isVideo && currentIsVideo) {
                    const img = document.createElement('img');
                    img.src = url;
                    img.alt = item.querySelector('.item-title')?.textContent || '';
                    mediaWrapper.innerHTML = '';
                    mediaWrapper.appendChild(img);
                } else if (currentMedia) {
                    if (currentMedia.tagName === 'IMG') {
                        currentMedia.src = url;
                    } else {
                        const source = currentMedia.querySelector('source');
                        if (source) {
                            source.src = url;
                            source.type = 'video/' + url.split('.').pop().toLowerCase();
                            currentMedia.load();
                        }
                    }
                }
                onChange?.();
            };

            // PATTERN: The returned root is a .cbx-stack, so the item list and the
            // "Add Item" button space themselves — no margins to set.
            const container = document.createElement('div');
            container.className = 'cbx-stack';

            const grid = element.querySelector('.card-list-grid');
            const items = grid ? grid.querySelectorAll('.card-item') : [];

            // PATTERN: Factory function to create editor UI for each card item
            const createItemEditor = (item, itemIndex) => {
                const itemEditor = document.createElement('div');
                itemEditor.className = 'cbx-item';                    // collapsible item card
                itemEditor.dataset.itemId = item.dataset.itemId || `item-${Date.now()}-${itemIndex}`; // PATTERN: always use itemId
                item.dataset.itemId = itemEditor.dataset.itemId;

                // PATTERN: Header (drag handle + thumbnail + title + delete)
                const itemHeader = document.createElement('div');
                itemHeader.className = 'cbx-item-head';

                const dragHandle = document.createElement('span');
                dragHandle.className = 'cbx-drag';
                dragHandle.innerHTML = '⠿';

                // PATTERN: Thumbnail is a special element → inline VISUAL styles only
                // (size/shape, not spacing; the header's gap handles spacing).
                const thumb = document.createElement('img');
                thumb.src = getMediaSrc(item) || 'https://placehold.co/40x40?text=No+Media';
                thumb.style.cssText = 'width: 32px; height: 32px; object-fit: cover; border-radius: 4px; flex: 0 0 auto;';
                thumb.onerror = () => { thumb.src = 'https://placehold.co/40x40?text=Media'; };

                const itemTitle = document.createElement('span');
                itemTitle.className = 'cbx-title';
                itemTitle.textContent = item.querySelector('.item-title')?.textContent || `Item ${itemIndex + 1}`;

                const deleteBtn = document.createElement('button');
                deleteBtn.type = 'button';
                deleteBtn.className = 'cbx-iconbtn is-danger';        // compact red icon button
                deleteBtn.setAttribute('aria-label', 'Delete item');
                deleteBtn.innerHTML = '<svg><use xlink:href="#icon-trash2"></use></svg>';
                deleteBtn.onclick = (e) => {
                    e.preventDefault();
                    item.remove();
                    itemEditor.remove();
                    onChange?.();
                };

                itemHeader.append(dragHandle, thumb, itemTitle, deleteBtn);

                // PATTERN: Collapsible body — a .cbx-item-body is itself a stack, so
                // every field spaces itself. Hidden by default.
                const itemBody = document.createElement('div');
                itemBody.className = 'cbx-item-body';
                itemBody.style.display = 'none';

                // PATTERN: Helper to add a simple labelled field (label on top of its control)
                const addField = (labelText, control) => {
                    const field = document.createElement('div');
                    field.className = 'cbx-field';
                    const label = document.createElement('label');
                    label.textContent = labelText;
                    field.append(label, control);
                    itemBody.appendChild(field);
                    return field;
                };

                // PATTERN: Input with a Browse button — label on top, then a .cbx-row
                // holding [input, button] so the button aligns with the input.
                const mediaInput = document.createElement('input');
                mediaInput.type = 'text';
                mediaInput.className = 'cbx-grow';
                mediaInput.placeholder = 'Image or video URL';
                mediaInput.value = getMediaSrc(item) || '';

                const refreshThumb = (url) => {
                    thumb.src = isVideoUrl(url) ? 'https://placehold.co/40x40?text=Video' : url;
                    thumb.onerror = () => { thumb.src = 'https://placehold.co/40x40?text=Media'; };
                };

                mediaInput.addEventListener('input', () => {
                    const url = mediaInput.value.trim();
                    if (url) { switchMediaType(item, url); refreshThumb(url); }
                });

                const mediaField = document.createElement('div');
                mediaField.className = 'cbx-field';
                const mediaLabel = document.createElement('label');
                mediaLabel.textContent = 'Media URL';
                const mediaRow = document.createElement('div');
                mediaRow.className = 'cbx-row';
                // The editor's file buttons: select (asset manager) and upload.
                // 'media' allows both image and video. Select is included only
                // where the app has configured an asset manager.
                mediaRow.append(mediaInput, ...builder.createFileButtons('media', (url) => {
                    mediaInput.value = url;
                    switchMediaType(item, url);
                    refreshThumb(url);
                }));
                mediaField.append(mediaLabel, mediaRow);
                itemBody.appendChild(mediaField);

                // PATTERN: Title input with real-time DOM updates
                const titleInput = document.createElement('input');
                titleInput.type = 'text';
                titleInput.placeholder = 'Item title';
                titleInput.value = item.querySelector('.item-title')?.textContent || '';
                titleInput.addEventListener('input', () => {
                    let titleEl = item.querySelector('.item-title');
                    if (!titleEl && titleInput.value) {
                        titleEl = document.createElement('h3');
                        titleEl.className = 'item-title';
                        const content = item.querySelector('.item-content');
                        if (!content) {
                            const newContent = document.createElement('div');
                            newContent.className = 'item-content';
                            newContent.appendChild(titleEl);
                            item.appendChild(newContent);
                        } else {
                            content.prepend(titleEl);
                        }
                    }
                    if (titleEl) titleEl.textContent = titleInput.value;
                    itemTitle.textContent = titleInput.value || `Item ${itemIndex + 1}`;
                    const imgEl = item.querySelector('img');
                    if (imgEl) imgEl.alt = titleInput.value;
                    onChange?.();
                });
                addField('Title', titleInput);

                // PATTERN: Textarea with debounced updates
                const descInput = document.createElement('textarea');
                descInput.placeholder = 'Item description (optional)';
                descInput.rows = 3;
                descInput.value = item.querySelector('.item-description')?.textContent || '';
                let descTimer;
                descInput.addEventListener('input', () => {
                    clearTimeout(descTimer);
                    descTimer = setTimeout(() => {
                        let descEl = item.querySelector('.item-description');
                        if (!descEl && descInput.value) {
                            descEl = document.createElement('p');
                            descEl.className = 'item-description';
                            const content = item.querySelector('.item-content');
                            if (!content) {
                                const newContent = document.createElement('div');
                                newContent.className = 'item-content';
                                newContent.appendChild(descEl);
                                item.appendChild(newContent);
                            } else {
                                content.appendChild(descEl);
                            }
                        }
                        if (descEl) descEl.textContent = descInput.value;
                        onChange?.();
                    }, 400);
                });
                addField('Description', descInput);

                // PATTERN: Alt text (images only)
                const currentMedia = item.querySelector('img, video');
                if (currentMedia?.tagName === 'IMG') {
                    const altInput = document.createElement('input');
                    altInput.type = 'text';
                    altInput.placeholder = 'Describe the image for screen readers';
                    altInput.value = item.querySelector('img')?.alt || titleInput.value;
                    altInput.addEventListener('input', () => {
                        const img = item.querySelector('img');
                        if (img) img.alt = altInput.value;
                        onChange?.();
                    });
                    addField('Alt Text (Accessibility)', altInput);
                }

                // PATTERN: Optional link URL + link text
                const linkEl = item.querySelector('.item-content a.item-link');

                const linkInput = document.createElement('input');
                linkInput.type = 'text';
                linkInput.placeholder = 'https://example.com';
                linkInput.value = linkEl?.getAttribute('href') || '';

                const linkTextInput = document.createElement('input');
                linkTextInput.type = 'text';
                linkTextInput.placeholder = 'Learn More';
                linkTextInput.value = linkEl?.textContent || 'Learn More →';

                linkInput.addEventListener('input', () => {
                    const url = linkInput.value.trim();
                    const content = item.querySelector('.item-content');
                    if (!content) return;
                    const existingLink = content.querySelector('a.item-link');
                    if (url && !existingLink) {
                        const a = document.createElement('a');
                        a.href = url;
                        a.className = 'item-link';
                        a.target = '_blank';
                        a.rel = 'noopener noreferrer';
                        a.textContent = linkTextInput.value || 'Learn More →';
                        content.appendChild(a);
                    } else if (url && existingLink) {
                        existingLink.href = url;
                    } else if (!url && existingLink) {
                        existingLink.remove();
                    }
                    onChange?.();
                });
                addField('Link URL (Optional)', linkInput);

                linkTextInput.addEventListener('input', () => {
                    const content = item.querySelector('.item-content');
                    const existingLink = content?.querySelector('a.item-link');
                    if (existingLink) {
                        existingLink.textContent = linkTextInput.value || 'Learn More →';
                        onChange?.();
                    }
                });
                addField('Link Text', linkTextInput);

                // PATTERN: Click header to toggle body. Toggle between 'none' and ''
                // (never 'block' — that would override .cbx-item-body's flex layout).
                itemHeader.addEventListener('click', (e) => {
                    if (e.target.closest('.cbx-drag') || e.target.closest('button')) return;
                    itemBody.style.display = itemBody.style.display === 'none' ? '' : 'none';
                });

                itemEditor.append(itemHeader, itemBody);
                return itemEditor;
            };

            // PATTERN: Item list is a .cbx-stack (items space themselves)
            const itemsContainer = document.createElement('div');
            itemsContainer.className = 'cbx-stack';
            items.forEach((item, i) => itemsContainer.appendChild(createItemEditor(item, i)));
            container.appendChild(itemsContainer);

            // PATTERN: Drag-and-drop sorting. In onSort the panel list is already
            // reordered, so the content just follows it.
            builder.sortableList(itemsContainer, {
                handle: '.cbx-drag',
                onSort: () => {
                    const itemMap = new Map();
                    Array.from(grid.children).forEach(item => {
                        if (item.dataset.itemId) itemMap.set(item.dataset.itemId, item);
                    });
                    Array.from(itemsContainer.children).forEach(editorItem => {
                        const actualItem = itemMap.get(editorItem.dataset.itemId);
                        if (actualItem) grid.appendChild(actualItem);
                    });
                    onChange?.();
                }
            });

            // PATTERN: Add button — a plain full-width button (the panel styles it);
            // the container stack spaces it from the list.
            const addBtn = document.createElement('button');
            addBtn.type = 'button';
            addBtn.textContent = '+ Add Item';
            addBtn.onclick = (e) => {
                e.preventDefault();
                const newItem = document.createElement('div');
                newItem.className = 'card-item';
                newItem.dataset.itemId = `item-${Date.now()}`;
                newItem.innerHTML = `
                    <div class="item-media">
                        <img src="https://placehold.co/800x450?text=New+Item" alt="New item">
                    </div>
                    <div class="item-content">
                        <h3 class="item-title">New Item</h3>
                        <p class="item-description">Item description</p>
                    </div>
                `;
                grid.appendChild(newItem);
                itemsContainer.appendChild(createItemEditor(newItem, itemsContainer.children.length));
                onChange?.();
            };
            container.appendChild(addBtn);

            // NOTE: No <style> block — all panel layout comes from the shared .cbx-*
            // helper classes, and no margins are set in editor code.
            return container;
        }
    },

    // PATTERN: mount applies runtime behavior/styling from options — never builds structure
    mount: function(element, options) {
        const opts = {
            itemsPerRow: parseInt(options.itemsPerRow) || 3,
            gap: parseInt(options.gap) || 16,
            aspectRatio: options.aspectRatio || '16/9',
            accentColor: options.accentColor || '#0066cc'
        };

        const grid = element.querySelector('.card-list-grid');
        const items = grid ? grid.querySelectorAll('.card-item') : [];
        if (items.length === 0) return {};

        // Apply grid layout from options
        grid.style.display = 'grid';
        grid.style.gridTemplateColumns = `repeat(${opts.itemsPerRow}, 1fr)`;
        grid.style.gap = `${opts.gap}px`;

        // Apply aspect ratio to each item's media
        items.forEach(item => {
            const media = item.querySelector('.item-media');
            if (media && opts.aspectRatio !== 'auto') media.style.aspectRatio = opts.aspectRatio;
        });

        // Helper: convert any color to RGB
        const parseColor = (color) => {
            const temp = document.createElement('div');
            temp.style.color = color;
            document.body.appendChild(temp);
            const computed = window.getComputedStyle(temp).color;
            document.body.removeChild(temp);
            const match = computed.match(/rgba?\((\d+),\s*(\d+),\s*(\d+)/);
            return match ? { r: +match[1], g: +match[2], b: +match[3] } : { r: 0, g: 102, b: 204 };
        };
        const rgbToHsl = (r, g, b) => {
            r /= 255; g /= 255; b /= 255;
            const max = Math.max(r, g, b), min = Math.min(r, g, b);
            let h, s, l = (max + min) / 2;
            if (max === min) { h = s = 0; }
            else {
                const d = max - min;
                s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
                switch (max) {
                    case r: h = ((g - b) / d + (g < b ? 6 : 0)) / 6; break;
                    case g: h = ((b - r) / d + 2) / 6; break;
                    case b: h = ((r - g) / d + 4) / 6; break;
                }
            }
            return { h: h * 360, s: s * 100, l: l * 100 };
        };
        const hslToRgb = (h, s, l) => {
            h /= 360; s /= 100; l /= 100;
            let r, g, b;
            if (s === 0) { r = g = b = l; }
            else {
                const hue2rgb = (p, q, t) => {
                    if (t < 0) t += 1;
                    if (t > 1) t -= 1;
                    if (t < 1/6) return p + (q - p) * 6 * t;
                    if (t < 1/2) return q;
                    if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
                    return p;
                };
                const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
                const p = 2 * l - q;
                r = hue2rgb(p, q, h + 1/3); g = hue2rgb(p, q, h); b = hue2rgb(p, q, h - 1/3);
            }
            return { r: Math.round(r * 255), g: Math.round(g * 255), b: Math.round(b * 255) };
        };
        const lightenColor = (color, amount = 25) => {
            const rgb = parseColor(color);
            const hsl = rgbToHsl(rgb.r, rgb.g, rgb.b);
            hsl.l = Math.min(90, hsl.l + amount);
            const c = hslToRgb(hsl.h, hsl.s, hsl.l);
            return `rgb(${c.r}, ${c.g}, ${c.b})`;
        };

        // PATTERN: expose accent color via CSS custom properties (the scoped CSS reads them)
        element.style.setProperty('--accent-color', opts.accentColor);
        element.style.setProperty('--accent-color-light', lightenColor(opts.accentColor, 25));

        // Accessibility
        element.setAttribute('role', 'region');
        if (!element.getAttribute('aria-label')) element.setAttribute('aria-label', 'Card list');

        return {};
    },

    unmount: function(element, instance) {
        // No cleanup needed for static grid
    }
};
```

**The Style:**

Every selector is scoped under `[data-cb-type="card-list"]` so the rendered block's styles can never leak into other plugins, the host page, or the editor UI.

```css
/* Card List Container */
[data-cb-type="card-list"] {
    position: relative;
    width: 100%;
    --accent-color: #0066cc;
    --accent-color-light: #4d9aff;
}

/* Card Grid */
[data-cb-type="card-list"] .card-list-grid {
    display: grid;
    gap: 16px;
}

/* Individual Card Item */
[data-cb-type="card-list"] .card-item {
    display: flex;
    flex-direction: column;
    background: rgb(255 255 255 / 85%);
    border-radius: 8px;
    overflow: hidden;
    border: 1px solid #e5e7eb;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
    transition: box-shadow 0.3s ease, border-color 0.3s ease;
}

[data-cb-type="card-list"] .card-item:hover {
    border-color: #d1d5db;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
}

/* Item Media */
[data-cb-type="card-list"] .card-item .item-media {
    position: relative;
    width: 100%;
    overflow: hidden;
    background: #e9ecef;
}

[data-cb-type="card-list"] .card-item .item-media img,
[data-cb-type="card-list"] .card-item .item-media video {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
    transition: transform 0.4s ease;
}

[data-cb-type="card-list"] .card-item:hover .item-media img,
[data-cb-type="card-list"] .card-item:hover .item-media video {
    transform: scale(1.05);
}

/* Item Content */
[data-cb-type="card-list"] .card-item .item-content {
    padding: 16px;
    display: flex;
    flex-direction: column;
    gap: 8px;
    flex: 1;
}

[data-cb-type="card-list"] .card-item .item-content .item-title {
    font-size: 18px;
    font-weight: 500;
    line-height: 1.4;
    color: #1a1a1a;
    margin: 0;
    transition: color 0.2s ease;
}

[data-cb-type="card-list"] .card-item:hover .item-content .item-title {
    color: var(--accent-color);
}

[data-cb-type="card-list"] .card-item .item-content .item-description {
    font-size: 16px;
    line-height: 1.6;
    color: #666;
    margin: 0;
}

[data-cb-type="card-list"] .card-item .item-content .item-link {
    display: inline-flex;
    align-items: center;
    align-self: flex-start;
    gap: 4px;
    font-size: 14px;
    font-weight: 500;
    color: var(--accent-color-light);
    text-decoration: none;
    margin-top: auto;
    padding-top: 8px;
    transition: color 0.2s ease;
}

[data-cb-type="card-list"] .card-item .item-content .item-link:hover {
    color: var(--accent-color);
    text-decoration: underline;
}

/* Focus Styles (WCAG Compliance) */
[data-cb-type="card-list"] .card-item .item-content .item-link:focus-visible {
    outline: 2px solid var(--accent-color);
    outline-offset: 2px;
    border-radius: 2px;
}

/* Responsive Design */
@media (max-width: 1024px) {
    [data-cb-type="card-list"] { padding: 16px; }
    [data-cb-type="card-list"] .card-list-grid { padding: 16px; }
    [data-cb-type="card-list"] .item-content { padding: 14px; }
    [data-cb-type="card-list"] .item-title { font-size: 16px; }
    [data-cb-type="card-list"] .item-description { font-size: 13px; }
}

@media (max-width: 768px) {
    [data-cb-type="card-list"] { padding: 12px; }
    [data-cb-type="card-list"] .card-list-grid {
        padding: 12px;
        grid-template-columns: repeat(2, 1fr) !important;
        gap: 12px;
    }
    [data-cb-type="card-list"] .item-content { padding: 12px; }
    [data-cb-type="card-list"] .item-title { font-size: 15px; }
    [data-cb-type="card-list"] .item-description { font-size: 12.5px; }
}

@media (max-width: 480px) {
    [data-cb-type="card-list"] { border-radius: 8px; }
    [data-cb-type="card-list"] .card-list-grid { grid-template-columns: 1fr !important; }
    [data-cb-type="card-list"] .card-item { border-radius: 6px; }
}

/* Loading State */
[data-cb-type="card-list"][data-loading="true"] {
    opacity: 0.6;
    pointer-events: none;
}

/* Video Specific Styles */
[data-cb-type="card-list"] .item-media video {
    background: #000;
}

[data-cb-type="card-list"] .item-media video::-webkit-media-controls {
    opacity: 0;
    transition: opacity 0.3s ease;
}

[data-cb-type="card-list"] .item-media:hover video::-webkit-media-controls {
    opacity: 1;
}

/* Reduced motion preference */
@media (prefers-reduced-motion: reduce) {
    [data-cb-type="card-list"] .card-item,
    [data-cb-type="card-list"] .item-media img,
    [data-cb-type="card-list"] .item-media video,
    [data-cb-type="card-list"] .item-title,
    [data-cb-type="card-list"] .item-link {
        transition: none !important;
        animation: none !important;
    }
}

/* High contrast mode support */
@media (prefers-contrast: high) {
    [data-cb-type="card-list"] .card-item {
        border: 2px solid currentColor;
    }
}

/* Print styles */
@media print {
    [data-cb-type="card-list"] .card-item { page-break-inside: avoid; margin-bottom: 20px; }
    [data-cb-type="card-list"] .card-list-grid { display: block; }
}

/* Empty state (no content) */
[data-cb-type="card-list"] .card-item:empty::before {
    content: 'Add content to this item';
    display: block;
    padding: 40px 20px;
    text-align: center;
    color: #999;
    font-size: 14px;
}
```

**HTML Usage:**

```html
<div data-cb-type="card-list" data-cb-items-per-row="2" data-cb-gap="26" data-cb-aspect-ratio="16/9" data-cb-accent-color="#000000" role="region" aria-label="Card list">
    <div class="card-list-grid">
        <div class="card-item">
            <div class="item-media">
                <img src="assets/minimalist-blocks/images/ai-XUe84.jpg" alt="Premium Widget">
            </div>
            <div class="item-content">
                <h3 class="item-title">Premium Widget</h3>
                <p class="item-description">High-quality craftsmanship</p>
                <a href="/product/1" class="item-link">Learn More →</a>
            </div>
        </div>
        <div class="card-item">
            <div class="item-media">
                <img src="assets/minimalist-blocks/images/ai-cvlfg.jpg" alt="Premium Widget">
            </div>
            <div class="item-content">
                <h3 class="item-title">Premium Widget</h3>
                <p class="item-description">High-quality craftsmanship</p>
                <a href="/product/2" class="item-link">Learn More →</a>
            </div>
        </div>
    </div>
</div>
```

**Follow all the PATTERNS** shown above to create a similar plugin that adds, removes, edits, and sorts items. The important PATTERNS:

- **Sorting with `builder.sortableList(el, { handle, onSort })`** and a `.cbx-drag`
  handle on each row.
- **Adding/removing items** — build the front-end structure and its editor row together.
- **Thumbnail preview** for each item (a special element → inline *visual* styles only, never spacing).
- **Use `itemId`** for the unique id (not `slideId` or other names):
  `itemEditor.dataset.itemId = item.dataset.itemId || \`item-${Date.now()}-${itemIndex}\`;`
- **Input with file buttons** from `builder.createFileButtons(type, onPick)`: label on top, then a `.cbx-row` holding `[input.cbx-grow, ...buttons]` — so the buttons align with the input. Never build the buttons by hand: the editor decides which ones the app can actually offer.
- **Editor panel UI uses the shared `.cbx-*` helper classes** (`.cbx-stack`, `.cbx-item`, `.cbx-item-head`, `.cbx-drag`, `.cbx-title`, `.cbx-iconbtn` `is-danger`, `.cbx-item-body`, `.cbx-field`, `.cbx-row`, `.cbx-grow`). **Do not ship a `<style>` block for panel layout, and never set margins/padding in editor code** — spacing is owned by the helpers.
- **Front-end CSS is scoped** under `[data-cb-type="<name>"]` on every selector.

See `plugin-best-practices.md` for the full styling standard and the `.cbx-*` reference.
