# ContentBuilder Plugins

Ready-made blocks you can drop into a page. Scan the list below to find one, then jump to it for the full copy-paste HTML — adjust the `data-cb-*` options to taste.

## Building your own plugin

Want to create a plugin, not just use one? These guides live in this folder:

- **[Plugin Development Guide](plugin-dev-guide.md)** — start here. How a plugin is structured and built: the `mount` function, reading options from `data-cb-*` attributes, cleanup with `unmount`, auto-generated settings panels, and custom editors.
- **[Plugin Example](plugin-example.md)** — a complete, annotated reference plugin (a sortable card list with settings *and* a custom editor). Read it or copy from it when starting a real plugin.
- **[Plugin Best Practices](plugin-best-practices.md)** — the baseline rules every plugin should follow: accessibility, SEO-friendly markup, "editor builds the structure / `mount` only adds behavior", CSS scoping, shipping a README, and translatable labels.
- **[Plugin Patterns](plugin-patterns.md)** — *situational* techniques for specific problems (animated/generated content, translating text that `mount` creates, editor-only helpers, and more), each with a clear "use when / do NOT use when". Opt-in — not defaults.

## All plugins

| Plugin | Type | What it does |
|---|---|---|
| [FAQ](#faq) | `faq` | A collapsible list of questions and answers. |
| [Accordion](#accordion) *(deprecated)* | `accordion` | A vertical list of expand/collapse question–answer items (an FAQ accordion). |
| [More Info](#more-info) | `more-info` | An inline tooltip / annotation: wrap a word or phrase in a trigger that reveals a rich-HTML popover on hover or click. |
| [Callout Box](#callout-box) | `callout-box` | A highlighted callout / admonition box — a heading plus rich-text body with a colored left border, in one of four styles (info, success, warning, error). |
| [CTA Buttons](#cta-buttons) | `cta-buttons` | A pair of call-to-action buttons (primary + secondary), each independently styled — gradient, solid, outline, or ghost — with its own colors, padding, radius, font, shadow, and hover-lift. |
| [Button](#button) | `button` | A single call-to-action button — a styled link with a gradient background. |
| [Tabs](#tabs) | `tabs` | Tabbed panels with rich, **inline-editable** content per tab. |
| [Flip Card](#flip-card) | `flip-card` | A card with a front and a back face that flips on hover or click. |
| [Section Divider](#section-divider) | `section-divider` | A decorative in-flow divider between sections. |
| [Table of Contents](#table-of-contents) | `table-of-contents` | Builds a linked "jump to" list from the page's headings (H2/H3/H4). |
| [Text Ticker / Marquee](#text-ticker--marquee) | `marquee` | A seamless, infinitely-scrolling strip of text items — announcements, headlines, "now playing", and the like. |
| [Typewriter](#typewriter) | `typewriter` | A headline with a rotating word or phrase — typed out character by character, or cross-faded. |
| [Code](#code) | `code` | A minimal placeholder plugin. |
| [Code View](#code-view) | `code-view` | A syntax-highlighted code block you type into directly on the page — themes, optional file name, line numbers and a copy button. |
| [Card List](#card-list) | `card-list` | A responsive grid of cards, each with media (image **or** video), a title, an optional description, and an optional link. |
| [Media Grid](#media-grid) | `media-grid` | A masonry-style grid of images and videos, each with an optional hover caption (title, description, link) and optional lightbox. |
| [Pricing Table](#pricing-table) | `pricing-table` | A responsive pricing/plans block with an optional **Monthly / Annually** toggle, a highlighted "featured" plan, per-plan badges, and a features list (with included / not-included states). |
| [Team Members](#team-members) | `team-members` | A responsive grid of team cards — photo, name, role, optional bio, and social links. |
| [Testimonials](#testimonials) | `testimonials` | A responsive grid of testimonial cards — quote, avatar, name, role, and an optional star rating. |
| [Process Steps](#process-steps) | `process-steps` | A numbered "how it works" sequence — horizontal (a row of steps with a connector line) or vertical (a stacked list). |
| [Progress / Skill Bars](#progress--skill-bars) | `progress-bars` | A list of labelled bars with a percentage each. |
| [Interactive Timeline](#interactive-timeline) | `timeline` | A vertical or horizontal timeline of events, each with a date, title, description, and a status (completed / current / upcoming) that styles its marker. |
| [Star Rating](#star-rating) | `star-rating` | A display star rating out of 5 with smooth fractional fills (4.5 → four-and-a-half stars), an optional numeric value, and an editable caption. |
| [Chart](#chart) | `chart` | A dependency-free chart — **bar**, **line**, or **donut** — rendered with plain SVG + CSS. |
| [Simple Stats](#simple-stats) | `simple-stats` | A clean row of headline numbers, each with a label — no icons, optional vertical dividers between them. |
| [Animated Stats](#animated-stats) | `animated-stats` | A row of statistics that count up from zero when they scroll into view. |
| [Countdown Timer](#countdown-timer) | `countdown-timer` | Counts down to a fixed date/time — or runs an **evergreen** per-visitor countdown (a fixed duration from each visitor's first view, remembered in `localStorage`). |
| [Floating Showcase](#floating-showcase) | `floating-showcase` | A 3D showcase of images or videos (screenshots of your work) floating in perspective — they drift slowly, shift with the cursor (parallax), gently tilt as a group, and the one you hover scales forward while the others recede. |
| [Marquee Wall](#marquee-wall) | `marquee-wall` | A "wall of work": several columns of images (or videos) scrolling continuously, with columns alternating direction at slightly different speeds — a lively portfolio band or hero. |
| [Media Slider](#media-slider) | `media-slider` | A horizontal carousel of cards, each with media (image **or** video), a title, an optional description, and an optional link. |
| [Slider (Swiper)](#slider-swiper) | `swiper-slider` | A full-featured image/video slider built on [Swiper](https://swiperjs.com/): autoplay, loop, transition effects, zoom, navigation, pagination, captions with positioning, and **responsive video sources** (a different video per screen size). |
| [Text Slider](#text-slider) | `text-slider` | A carousel of **editable text cards** (quotes, announcements, feature blurbs) on [Swiper](https://swiperjs.com/): items-per-view, autoplay, loop, fade/slide, text alignment, and optional solid/glass card styling. |
| [Media Slider (Fullscreen)](#media-slider-fullscreen) | `fullscreen-slider` | A fullscreen background slideshow of images **and/or** videos, with a slow Ken-Burns zoom, a darkening overlay, and optional text content per slide. |
| [Before/After Slider](#beforeafter-slider) | `before-after-slider` | An image comparison slider — drag a handle to wipe between a "before" and an "after" image. |
| [Video Embed](#video-embed) | `video-embed` | A self-hosted `<video>` player embedded in the content — with aspect-ratio, fit, and corner-radius controls, plus the usual autoplay/loop/muted/controls/inline options and an optional poster image. |
| [YouTube](#youtube) | `youtube-embed` | Embeds a YouTube video from any YouTube link — `watch`, `youtu.be`, `/embed/`, `/shorts/`, `/live/`, or a raw 11-character ID. |
| [Video Background](#video-background) | `video-background` | A full-cover looping background video for a section, with an optional color overlay and optional play/pause & mute controls. |
| [Audio / Podcast Player](#audio--podcast-player) | `audio-player` | A themeable player around a native `<audio>` — play/pause, draggable seek with a buffered bar, elapsed/total time, skip back/forward, playback speed, volume, and an optional download button. |
| [Logo Loop](#logo-loop) | `logo-loop` | A continuously scrolling marquee of logos (or any small images) — the classic "as seen in / trusted by" strip. |
| [Browser Mockup](#browser-mockup) | `browser-mockup` | Frames a screenshot inside a 3D-tilted browser/app window (with the classic three traffic-light dots) — great for showcasing a product, app, or website. |
| [Map](#map) | `map-embed` | Embeds an interactive map for an address, place name, `lat,lng` coordinates, or a pasted Google Maps link. |
| [Aurora Glow](#aurora-glow) | `aurora-glow` | A flowing **aurora** background on a `<canvas>` — several large, soft color clouds that drift organically and glow where they overlap, finished with a subtle film grain. |
| [Gradient Blobs](#gradient-blobs) | `blobs-bg` | A content block filled with soft, animated blurred-gradient "blobs" that slowly drift and morph — a warm decorative panel. |
| [Hero Animation](#hero-animation) | `hero-animation` | A generative canvas animation for hero sections and section backdrops — pick from 12 algorithmic visuals (neural network, pendulum harmonics, oscillating rings, fractal tree, Fourier series, Lorenz attractor, Bezier curves, circular packing, Lissajous curves, graph network, spline interpolation, circular waves). |
| [Pendulum Harmonics](#pendulum-harmonics) | `pendulum` | A generative canvas animation of many pendulums swinging at slightly different frequencies, tracing evolving harmonic (pendulum-wave) patterns. |
| [Particle Constellation](#particle-constellation) | `particle-constellation` | A generative canvas animation of drifting particles that link up with lines when they're close, forming a shifting "constellation" web that also reacts to the mouse. |
| [Vector Force Field](#vector-force-field) | `vector-force` | A big display-text headline whose letters behave like a **force field**: as the cursor moves near them, each letter is pushed away and springs back on a little physics simulation (spring + damping), with a custom ring cursor and glow. |
| [Form](#form) | `form` | Build a form visually — short and long text, email, phone, number, dropdown, select, multi-select, switch, slider, date, time, url, colour, file upload — style it in the builder's Designer, and drop it into a page. The form is written into the page's HTML, so search engines see it and visitors see it immediately. |
| [Contact / Newsletter Form](#contact--newsletter-form) | `contact-form` | A real, accessible `<form>` block with a configurable field set (text, email, phone, textarea, consent checkbox), client-side validation, and submit-via-`fetch` with inline success/error feedback. |
| [Social Share](#social-share) | `social-share` | A row of share buttons — X, Facebook, LinkedIn, WhatsApp, Copy Link, and a "More" dropdown (Reddit, Telegram, Email). |
| [Site Logo](#site-logo) | `site-logo` | A site logo as text, an image, or both. The text takes the font and colour of the design it sits in. |
| [Site Menu](#site-menu) | `site-menu` | A site navigation menu with optional submenus and a small-screen menu panel, styled by the design it sits in. In a ContentBox navigation bar, the bar scrolls away with the header, comes back at the top, and takes the look of the section under it. |
| [Site Menu (always visible)](#site-menu-always-visible) | `site-menu-sticky` | Site Menu with a bar that stays on screen the whole time, getting its background once the header moves. |
| [Navigation Menu](#navigation-menu) | `nav-menu` | A responsive site navigation bar: a logo (text **or** image), a row of menu items with optional **dropdown submenus**, a mobile hamburger toggle, a subtle shadow on scroll, and a light / dark / "dark-on-start" theme. |
| [Sticky Category Nav](#sticky-category-nav) | `sticky-category-nav` | A horizontal in-page navigation bar that sticks to the top and **highlights the current section** as you scroll (scrollspy). |
| [Click Counter](#click-counter) | `click-counter` | A block that counts how many times it's clicked and updates its own text. |
| [Toggle](#toggle) | `toggle` | A block that shows or hides another element on the page when clicked. |
| [Tooltip](#tooltip) | `tooltip` | A block that shows a small text tooltip near the cursor on hover. |

## Text & content

### FAQ

A collapsible list of questions and answers. Add, remove, reorder, and rename items from the editor; the answer of each item is a rich-text region edited inline. Fully keyboard-accessible (Arrow/Home/End) with correct ARIA wiring.

```html
<div data-cb-type="faq" data-cb-allow-multiple="false" data-cb-open-first="true" data-cb-border-radius="10">
    <div class="accordion-item">
        <div class="accordion-header">
            <span class="accordion-question edit">What is this platform about?</span>
            <span class="accordion-icon">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none" aria-hidden="true">
                    <path d="M5 7.5L10 12.5L15 7.5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
            </span>
        </div>
        <div class="accordion-content">
            <div class="accordion-answer is-subblock edit">
                <p>A short, helpful answer goes here.</p>
            </div>
        </div>
    </div>
    <div class="accordion-item">
        <div class="accordion-header">
            <span class="accordion-question edit">How do I get started?</span>
            <span class="accordion-icon">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none" aria-hidden="true">
                    <path d="M5 7.5L10 12.5L15 7.5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
            </span>
        </div>
        <div class="accordion-content">
            <div class="accordion-answer is-subblock edit">
                <p>Another answer goes here.</p>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](faq/README.md)

### Accordion

> **Deprecated** — kept so existing pages keep working; use **FAQ** for new content.

A vertical list of expand/collapse question–answer items (an FAQ accordion). Clicking a question reveals its answer; by default only one item is open at a time.

```html
<div data-cb-type="accordion" data-cb-allow-multiple="false" data-cb-open-first="true">
    <div class="accordion-item">
        <div class="accordion-header">
            <span class="accordion-question">What is your return policy?</span>
            <span class="accordion-icon">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="none" aria-hidden="true">
                    <path d="M5 7.5L10 12.5L15 7.5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
            </span>
        </div>
        <div class="accordion-content">
            <div class="accordion-answer">You can return any item within 30 days.</div>
        </div>
    </div>
    <!-- more .accordion-item blocks … -->
</div>
```

[Options & details →](accordion/README.md)

### More Info

An inline tooltip / annotation: wrap a word or phrase in a trigger that reveals a rich-HTML popover on hover or click. Three visual styles cover the common needs.

```html
<span data-cb-type="more-info" class="more-info-trigger style-info"
      data-cb-content="&lt;strong&gt;Jupiter&lt;/strong&gt;&lt;br&gt;&lt;br&gt;The largest planet in the Solar System."
      role="button" tabindex="0" aria-expanded="false" aria-haspopup="true">Jupiter</span>
```

[Options & details →](more-info/README.md)

### Callout Box

A highlighted callout / admonition box — a heading plus rich-text body with a colored left border, in one of four styles (info, success, warning, error). Good for notes, tips, warnings, and important messages. Pick the style in the settings; edit the title and body directly in the content.

```html
<div data-cb-type="callout-box" data-cb-style="info" class="callout-box callout-info">
    <h3 class="edit">Important Information</h3>
    <div class="is-subblock edit">
        <p>This is an important message with <strong>formatting</strong>!</p>
        <p>You can add multiple paragraphs and <a href="#">links</a>.</p>
    </div>
</div>
```

[Options & details →](callout-box/README.md)

### CTA Buttons

A pair of call-to-action buttons (primary + secondary), each independently styled — gradient, solid, outline, or ghost — with its own colors, padding, radius, font, shadow, and hover-lift. Set each button's text and link in the content editor; style them and set the row gap/alignment in the settings.

```html
<div data-cb-type="cta-buttons" data-cb-gap="16" data-cb-alignment="center"
     data-cb-primary-text="Start Building" data-cb-primary-link="#signup" data-cb-primary-style="gradient" data-cb-primary-color1="#6366f1" data-cb-primary-color2="#8b5cf6" data-cb-primary-text-color="#ffffff"
     data-cb-secondary-text="View Docs" data-cb-secondary-link="#docs" data-cb-secondary-style="outline" data-cb-secondary-color1="#e5e5e5" data-cb-secondary-color2="#cdcdcd" data-cb-secondary-text-color="#0f172a">
    <a href="#signup" class="cta-btn">Start Building</a>
    <a href="#docs" class="cta-btn">View Docs</a>
</div>
```

[Options & details →](cta-buttons/README.md)

### Button

A single call-to-action button — a styled link with a gradient background. Edit its text and URL in the content editor; set the two gradient colors in the settings.

```html
<div data-cb-type="button">
    <a href="https://example.com">Click me</a>
</div>
```

[Options & details →](button/README.md)

### Tabs

Tabbed panels with rich, **inline-editable** content per tab. Add, remove, reorder, and rename tabs from the editor; edit each tab's content directly on the page. Implements the ARIA tabs pattern (roles, `aria-selected`, `aria-controls`, roving tabindex, arrow-key navigation).

```html
<div data-cb-type="tabs" data-cb-accent-color="#2563eb" data-cb-tab-align="start" role="region" aria-label="Tabbed content">
    <div class="tabs-nav" role="tablist" aria-label="Tabs">
        <button class="tabs-tab is-active" type="button" role="tab">Overview</button>
        <button class="tabs-tab" type="button" role="tab">Features</button>
        <button class="tabs-tab" type="button" role="tab">Pricing</button>
    </div>
    <div class="tabs-panels">
        <div class="tabs-panel is-active" role="tabpanel">
            <div class="is-subblock edit"><h3>Overview</h3><p>Write the overview here. This content is editable directly on the page.</p></div>
        </div>
        <div class="tabs-panel" role="tabpanel">
            <div class="is-subblock edit"><h3>Features</h3><p>List your features here.</p></div>
        </div>
        <div class="tabs-panel" role="tabpanel">
            <div class="is-subblock edit"><h3>Pricing</h3><p>Describe your pricing here.</p></div>
        </div>
    </div>
</div>
```

[Options & details →](tabs/README.md)

### Flip Card

A card with a front and a back face that flips on hover or click. Both faces are inline-editable rich text. Flip horizontally or vertically, set the height, and pick the back-face color.

```html
<div data-cb-type="flip-card" data-cb-trigger="hover" data-cb-axis="horizontal" data-cb-height="280" data-cb-accent-color="#2563eb">
    <div class="fc-inner">
        <div class="fc-face fc-front">
            <div class="is-subblock edit">
                <h3>Hover to flip</h3>
                <p>The front of the card.</p>
            </div>
        </div>
        <div class="fc-face fc-back">
            <div class="is-subblock edit">
                <h3>The back</h3>
                <p>More details revealed on flip.</p>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](flip-card/README.md)

### Section Divider

A decorative in-flow divider between sections. Choose a style — plain line, dashed, dotted, gradient, three dots, a centered icon, or a centered editable label — and set the color, thickness, width, and vertical spacing.

```html
<div data-cb-type="section-divider" data-cb-style="line" data-cb-color="#d1d5db" data-cb-thickness="1" data-cb-width="100" data-cb-space="24">
    <span class="sd-line sd-line-left"></span>
    <span class="sd-center"><span class="sd-label edit">Section</span></span>
    <span class="sd-line sd-line-right"></span>
</div>
```

[Options & details →](section-divider/README.md)

### Table of Contents

Builds a linked "jump to" list from the page's headings (H2/H3/H4). Clicking an item smooth-scrolls to that heading and marks it active (so the block shows the chapter you last jumped to). The list is generated from the surrounding content; the title above it is editable inline.

```html
<div data-cb-type="table-of-contents" data-cb-levels="h2h3" data-cb-accent-color="#2563eb" role="navigation" aria-label="Table of contents">
    <p class="toc-title">On this page</p>
    <ul class="toc-list"></ul>
</div>
```

[Options & details →](table-of-contents/README.md)

### Text Ticker / Marquee

A seamless, infinitely-scrolling strip of text items — announcements, headlines, "now playing", and the like. Manage the items (add/remove/reorder) from the content editor.

```html
<div data-cb-type="marquee" data-cb-direction="left" data-cb-speed="60" data-cb-separator="•" data-cb-pause-on-hover="true">
    <div class="mq-viewport">
        <div class="mq-track" aria-hidden="true"></div>
    </div>
    <ul class="mq-items">
        <li>Breaking news: we just shipped something great</li>
        <li>50% off for a limited time</li>
        <li>Free shipping worldwide</li>
    </ul>
</div>
```

[Options & details →](marquee/README.md)

### Typewriter

A headline with a rotating word or phrase — typed out character by character, or cross-faded. Edit the prefix/suffix inline; manage the rotating words (add/remove/reorder) from the content editor.

```html
<div data-cb-type="typewriter" data-cb-effect="type" data-cb-type-speed="70" data-cb-pause="1400" data-cb-cursor="true" data-cb-loop="true" data-cb-accent-color="#2563eb">
    <h2 class="tw-line">
        <span class="tw-prefix edit">We build </span>
        <span class="tw-rotate" aria-hidden="true">websites</span>
        <span class="tw-suffix edit"></span>
    </h2>
    <ul class="tw-words">
        <li>websites</li>
        <li>brands</li>
        <li>experiences</li>
    </ul>
</div>
```

[Options & details →](typewriter/README.md)

### Code

A minimal placeholder plugin. It marks a block as `data-cb-type="code"` but currently has **no settings, no editor, and no runtime behavior** — its `mount` does nothing.

```html
<div data-cb-type="code">
    <pre><code>your code here</code></pre>
</div>
```

[Options & details →](code/README.md)

### Code View

A syntax-highlighted code block you type into directly on the page — no dialog. Highlighting follows the code as you type (HTML, CSS, JavaScript, TypeScript, JSON, Python, PHP, Shell, SQL, or auto-detected), in a Light, Dark or Midnight theme. Optional file-name bar, line numbers, line wrapping, and a copy button. Editing follows the usual code-editor keys: auto-indent, `Tab`/`Shift`+`Tab` on selected lines, `Cmd`/`Ctrl`+`/` to comment, `Alt`+`↑`/`↓` to move lines.

```html
<div data-cb-type="code-view" data-cb-theme="light" data-cb-language="auto" data-cb-copy-button="true">
    <pre><code class="edit">const greet = (name) =&gt; `Hello, ${name}!`;
console.log(greet('World'));</code></pre>
</div>
```

[Options & details →](code-view/README.md)

## Cards, lists & data

### Card List

A responsive grid of cards, each with media (image **or** video), a title, an optional description, and an optional link. Good for product listings, feature grids, portfolios, or team members. Set the number of columns, gap, and card aspect ratio in the settings; edit each card's content in the content editor (drag to reorder).

```html
<div data-cb-type="card-list" data-cb-items-per-row="3" data-cb-gap="16" data-cb-aspect-ratio="16/9" data-cb-accent-color="#0066cc">
    <div class="card-list-grid">
        <div class="card-item">
            <div class="item-media">
                <img src="assets/sample.jpg" alt="Sample Title">
            </div>
            <div class="item-content">
                <h3 class="item-title">Sample Title</h3>
                <p class="item-description">Description</p>
                <a href="/product/1" class="item-link" target="_blank" rel="noopener noreferrer">Learn More →</a>
            </div>
        </div>
        <!-- more .card-item blocks … -->
    </div>
</div>
```

[Options & details →](card-list/README.md)

### Media Grid

A masonry-style grid of images and videos, each with an optional hover caption (title, description, link) and optional lightbox. Add, remove, reorder, and edit items from the content editor.

```html
<div data-cb-type="media-grid" data-cb-columns="3" data-cb-gap="16" data-cb-padding="0" data-cb-rounded="8" data-cb-content-theme="dark">
    <div class="gallery-item">
        <img src="assets/photo-1.jpg" alt="A description">
        <div class="item-content">
            <h4>Title</h4>
            <div>A short caption.</div>
            <a class="item-link" href="https://example.com">Learn More →</a>
        </div>
    </div>
    <div class="gallery-item">
        <img src="assets/photo-2.jpg" alt="">
    </div>
</div>
```

[Options & details →](media-grid/README.md)

### Pricing Table

A responsive pricing/plans block with an optional **Monthly / Annually** toggle, a highlighted "featured" plan, per-plan badges, and a features list (with included / not-included states).

```html
<div data-cb-type="pricing-table" data-cb-columns="3" data-cb-currency="$" data-cb-accent-color="#2563eb" data-cb-billing-toggle="true" data-cb-default-period="monthly" data-cb-shadow="true" data-cb-rounded="true" role="region" aria-label="Pricing plans">
    <div class="pricing-billing-toggle" role="group" aria-label="Billing period">
        <button type="button" class="billing-option is-active" data-period="monthly">Monthly</button>
        <button type="button" class="billing-option" data-period="annually">Annually <span class="billing-save">Save 20%</span></button>
    </div>
    <div class="pricing-grid">
        <div class="pricing-plan">
            <div class="plan-header">
                <h3 class="plan-name">Starter</h3>
                <p class="plan-description">For individuals getting started</p>
            </div>
            <div class="plan-price">
                <span class="price-currency">$</span><span class="price-amount" data-monthly="9" data-annually="90">9</span><span class="price-period">/mo</span>
            </div>
            <ul class="plan-features">
                <li>1 project</li>
                <li>Up to 5 pages</li>
                <li class="feature-disabled">Priority support</li>
            </ul>
            <a href="#" class="plan-button">Get Started</a>
        </div>
        <div class="pricing-plan is-featured">
            <div class="plan-badge">Most Popular</div>
            <div class="plan-header">
                <h3 class="plan-name">Pro</h3>
                <p class="plan-description">For growing teams</p>
            </div>
            <div class="plan-price">
                <span class="price-currency">$</span><span class="price-amount" data-monthly="29" data-annually="290">29</span><span class="price-period">/mo</span>
            </div>
            <ul class="plan-features">
                <li>Unlimited projects</li>
                <li>Unlimited pages</li>
                <li>Priority support</li>
            </ul>
            <a href="#" class="plan-button">Get Started</a>
        </div>
        <div class="pricing-plan">
            <div class="plan-header">
                <h3 class="plan-name">Enterprise</h3>
                <p class="plan-description">For large organizations</p>
            </div>
            <div class="plan-price">
                <span class="price-currency">$</span><span class="price-amount" data-monthly="99" data-annually="990">99</span><span class="price-period">/mo</span>
            </div>
            <ul class="plan-features">
                <li>Everything in Pro</li>
                <li>Dedicated manager</li>
                <li>Custom integrations</li>
            </ul>
            <a href="#" class="plan-button">Contact Sales</a>
        </div>
    </div>
</div>
```

[Options & details →](pricing-table/README.md)

### Team Members

A responsive grid of team cards — photo, name, role, optional bio, and social links. Add / remove / reorder members and edit each (including per-member social links) from the editor.

```html
<div data-cb-type="team-members" data-cb-columns="3" data-cb-image-shape="circle" data-cb-show-bio="true" data-cb-accent-color="#2563eb" data-cb-shadow="true" data-cb-rounded="true" role="region" aria-label="Team members">
    <div class="team-grid">
        <div class="team-card">
            <img class="team-photo" src="assets/minimalist-blocks/images/ai-cvlfg.jpg" alt="Jane Doe">
            <div class="team-info">
                <h3 class="team-name">Jane Doe</h3>
                <p class="team-role">CEO &amp; Founder</p>
                <p class="team-bio">Leads product and vision.</p>
                <div class="team-social"></div>
            </div>
        </div>
        <div class="team-card">
            <img class="team-photo" src="assets/minimalist-blocks/images/ai-HzKor.jpg" alt="Sara Kim">
            <div class="team-info">
                <h3 class="team-name">Sara Kim</h3>
                <p class="team-role">Design Lead</p>
                <p class="team-bio">Owns the experience.</p>
                <div class="team-social"></div>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](team-members/README.md)

### Testimonials

A responsive grid of testimonial cards — quote, avatar, name, role, and an optional star rating. Built from semantic `figure`/`blockquote`/`figcaption` markup for SEO and accessibility.

```html
<div data-cb-type="testimonials" data-cb-columns="3" data-cb-variant="cards" data-cb-background="solid" data-cb-bg-color="#ffffff" data-cb-border-color="#e5e7eb" data-cb-gap="24" data-cb-show-rating="true" data-cb-accent-color="#2563eb" data-cb-rounded="true" data-cb-hover="lift" role="region" aria-label="Testimonials">
    <div class="testimonials-grid">
        <figure class="testimonial">
            <div class="testimonial-rating" data-rating="5" aria-label="Rated 5 out of 5">
                <span class="star is-filled">★</span><span class="star is-filled">★</span><span class="star is-filled">★</span><span class="star is-filled">★</span><span class="star is-filled">★</span>
            </div>
            <blockquote class="testimonial-quote"><p>This tool completely changed how our team ships pages. Worth every penny.</p></blockquote>
            <figcaption class="testimonial-author">
                <img class="testimonial-avatar" src="assets/minimalist-blocks/images/ai-cvlfg.jpg" alt="Jane Doe">
                <span class="testimonial-meta">
                    <span class="testimonial-name">Jane Doe</span>
                    <span class="testimonial-role">CEO, Acme</span>
                </span>
            </figcaption>
        </figure>
        <figure class="testimonial">
            <div class="testimonial-rating" data-rating="4" aria-label="Rated 4 out of 5">
                <span class="star is-filled">★</span><span class="star is-filled">★</span><span class="star is-filled">★</span><span class="star is-filled">★</span><span class="star">★</span>
            </div>
            <blockquote class="testimonial-quote"><p>A polished experience with just the right amount of flexibility.</p></blockquote>
            <figcaption class="testimonial-author">
                <img class="testimonial-avatar" src="assets/minimalist-blocks/images/ai-HzKor.jpg" alt="Sara Kim">
                <span class="testimonial-meta">
                    <span class="testimonial-name">Sara Kim</span>
                    <span class="testimonial-role">Designer, Studio</span>
                </span>
            </figcaption>
        </figure>
    </div>
</div>
```

[Options & details →](testimonials/README.md)

### Process Steps

A numbered "how it works" sequence — horizontal (a row of steps with a connector line) or vertical (a stacked list). Add, remove, reorder, and edit steps (title + description) from the editor. Step numbers are assigned automatically from position, so reordering just works.

```html
<div data-cb-type="process-steps" data-cb-layout="horizontal" data-cb-connector="true" data-cb-accent-color="#2563eb" role="list" aria-label="Process steps">
    <div class="ps-list">
        <div class="ps-step" role="listitem">
            <div class="ps-num">1</div>
            <div class="ps-body"><h3 class="ps-title">Sign up</h3><p class="ps-desc">Create your free account in seconds.</p></div>
        </div>
        <div class="ps-step" role="listitem">
            <div class="ps-num">2</div>
            <div class="ps-body"><h3 class="ps-title">Customize</h3><p class="ps-desc">Set things up the way you like.</p></div>
        </div>
        <div class="ps-step" role="listitem">
            <div class="ps-num">3</div>
            <div class="ps-body"><h3 class="ps-title">Launch</h3><p class="ps-desc">Go live and share with the world.</p></div>
        </div>
    </div>
</div>
```

[Options & details →](process-steps/README.md)

### Progress / Skill Bars

A list of labelled bars with a percentage each. The fill animates in when the block scrolls into view. Add / remove / reorder bars and set each label + percentage from the editor.

```html
<div data-cb-type="progress-bars" data-cb-accent-color="#2563eb" data-cb-show-percent="true" data-cb-animate="true" data-cb-size="medium" data-cb-rounded="true" role="group" aria-label="Skills">
    <div class="pb-list">
        <div class="pb-item">
            <div class="pb-head"><span class="pb-label">HTML &amp; CSS</span><span class="pb-percent">90%</span></div>
            <div class="pb-track" role="progressbar" aria-valuemin="0" aria-valuemax="100" aria-valuenow="90">
                <div class="pb-fill" data-value="90" style="width:90%"></div>
            </div>
        </div>
        <div class="pb-item">
            <div class="pb-head"><span class="pb-label">JavaScript</span><span class="pb-percent">80%</span></div>
            <div class="pb-track" role="progressbar" aria-valuemin="0" aria-valuemax="100" aria-valuenow="80">
                <div class="pb-fill" data-value="80" style="width:80%"></div>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](progress-bars/README.md)

### Interactive Timeline

A vertical or horizontal timeline of events, each with a date, title, description, and a status (completed / current / upcoming) that styles its marker. Items reveal on scroll. Good for company history, roadmaps, or process steps. Set orientation, animation, and appearance in the settings; edit and reorder events in the content editor (drag to reorder).

```html
<div data-cb-type="timeline" data-cb-orientation="vertical" data-cb-animate="true" data-cb-accent-color="#2563eb" data-cb-line-width="2" data-cb-dot-size="12">
    <div>
        <div class="timeline-item" data-status="completed">
            <div class="timeline-marker" aria-hidden="true"></div>
            <div class="timeline-content">
                <span class="timeline-date">Q1 2023</span>
                <h3 class="timeline-title">Company Founded</h3>
                <p class="timeline-desc">Started with a vision to build something meaningful.</p>
            </div>
        </div>
        <!-- more .timeline-item blocks … -->
    </div>
</div>
```

[Options & details →](timeline/README.md)

### Star Rating

A display star rating out of 5 with smooth fractional fills (4.5 → four-and-a-half stars), an optional numeric value, and an editable caption. Set the rating in the settings; edit the caption inline.

```html
<div data-cb-type="star-rating" data-cb-rating="4.5" data-cb-size="medium" data-cb-show-value="true" data-cb-accent-color="#f59e0b" role="img" aria-label="Rated 4.5 out of 5">
    <span class="sr-stars" aria-hidden="true">
        <span class="sr-stars-bg">★★★★★</span>
        <span class="sr-stars-fill">★★★★★</span>
    </span>
    <span class="sr-value">4.5</span>
    <span class="sr-caption edit">Based on 128 reviews</span>
</div>
```

[Options & details →](star-rating/README.md)

### Chart

A dependency-free chart — **bar**, **line**, or **donut** — rendered with plain SVG + CSS. No Chart.js, no CDN, nothing loaded at runtime. Edit the data (label + value, add/remove/reorder) from the content editor. The chart intentionally has no built-in title — add a heading above the block with the normal editor for full formatting control.

```html
<div data-cb-type="chart" data-cb-chart-type="bar" data-cb-accent-color="#2563eb" data-cb-height="320" data-cb-show-values="true" data-cb-show-grid="true">
    <ul class="chart-data">
        <li data-value="40">Jan</li>
        <li data-value="65">Feb</li>
        <li data-value="52">Mar</li>
        <li data-value="78">Apr</li>
        <li data-value="90">May</li>
    </ul>
    <div class="chart-canvas" aria-hidden="true"></div>
</div>
```

[Options & details →](chart/README.md)

### Simple Stats

A clean row of headline numbers, each with a label — no icons, optional vertical dividers between them. Numbers count up from zero when they scroll into view, and support a prefix/suffix (e.g. `$`, `+`, `%`) and non-numeric values like `24/7`. Good for quick "by the numbers" strips. Set layout, animation, and sizes in the settings; edit each stat in the content editor.

```html
<div data-cb-type="simple-stats" data-cb-alignment="center" data-cb-duration="2" data-cb-delay="0.1" data-cb-easing="ease-out" data-cb-number-size="60" data-cb-label-size="14" data-cb-show-dividers="true">
    <div class="stat-item" data-target="250" data-prefix="" data-suffix="+">
        <div class="stat-number">250</div>
        <div class="stat-label">Projects Delivered</div>
    </div>
    <!-- more .stat-item blocks … -->
</div>
```

[Options & details →](simple-stats/README.md)

### Animated Stats

A row of statistics that count up from zero when they scroll into view. Each stat has an icon (emoji, icon font, or inline SVG), a target number with optional prefix/suffix, and a label. Good for "trusted by", results, or key metrics. Set columns, alignment, animation timing, and sizes in the settings; edit each stat in the content editor.

```html
<div data-cb-type="animated-stats" data-cb-columns="4" data-cb-alignment="center" data-cb-variant="cards" data-cb-background="solid" data-cb-bg-color="#ffffff" data-cb-border-color="#e5e7eb" data-cb-gap="32" data-cb-hover="lift" data-cb-rounded="true" data-cb-duration="2" data-cb-delay="0.1" data-cb-easing="ease-out" data-cb-show-icon="true" data-cb-icon-size="48" data-cb-number-size="42" data-cb-number-weight="500" data-cb-label-size="16">
    <div class="stat-item" data-target="1000" data-prefix="" data-suffix="+">
        <div class="stat-icon" data-icon-type="emoji">📊</div>
        <div class="stat-number">1000</div>
        <div class="stat-label">Happy Clients</div>
    </div>
    <!-- more .stat-item blocks … -->
</div>
```

[Options & details →](animated-stats/README.md)

### Countdown Timer

Counts down to a fixed date/time — or runs an **evergreen** per-visitor countdown (a fixed duration from each visitor's first view, remembered in `localStorage`). Shows days / hours / minutes / seconds with editable labels, and a message when it reaches zero.

```html
<div data-cb-type="countdown-timer" data-cb-target="2026-12-31T23:59" data-cb-mode="date" data-cb-evergreen-hours="24" data-cb-expired-text="This offer has ended." data-cb-size="medium" data-cb-accent-color="#2563eb" role="timer" aria-label="Countdown">
    <div class="cd-grid">
        <div class="cd-unit"><span class="cd-value" data-unit="days">00</span><span class="cd-label">Days</span></div>
        <div class="cd-unit"><span class="cd-value" data-unit="hours">00</span><span class="cd-label">Hours</span></div>
        <div class="cd-unit"><span class="cd-value" data-unit="minutes">00</span><span class="cd-label">Minutes</span></div>
        <div class="cd-unit"><span class="cd-value" data-unit="seconds">00</span><span class="cd-label">Seconds</span></div>
    </div>
    <div class="cd-expired">This offer has ended.</div>
</div>
```

[Options & details →](countdown-timer/README.md)

## Media

### Floating Showcase

A 3D showcase of images or videos (screenshots of your work) floating in perspective — they drift slowly, shift with the cursor (parallax), gently tilt as a group, and the one you hover scales forward while the others recede. Great for a portfolio hero or a "featured work" strip. Images stay real, SEO-friendly HTML, and each item can link out.

```html
<div data-cb-type="floating-showcase" data-cb-transparent-bg="false" data-cb-bg-color="#0b0b12" data-cb-corner-radius="12" data-cb-shadow="true" data-cb-drift="45" data-cb-parallax="55" data-cb-depth="50" data-cb-spread="100" style="min-height: 560px;">
    <div class="fg-stage">
        <div class="fg-item fg-featured">
            <a class="fg-card"><img class="fg-img" src="assets/minimalist-blocks/images/ai-01Opn.jpg" alt="Project one"></a>
        </div>
        <div class="fg-item">
            <a class="fg-card" href="https://example.com" target="_blank" rel="noopener noreferrer"><img class="fg-img" src="assets/minimalist-blocks/images/ai-0Yhoi.jpg" alt="Project two"></a>
        </div>
        <div class="fg-item">
            <a class="fg-card"><img class="fg-img" src="assets/minimalist-blocks/images/ai-1ANxo.jpg" alt="Project three"></a>
        </div>
    </div>
</div>
```

[Options & details →](floating-showcase/README.md)

### Marquee Wall

A "wall of work": several columns of images (or videos) scrolling continuously, with columns alternating direction at slightly different speeds — a lively portfolio band or hero. You add one flat list of media; it splits them across the columns and loops each seamlessly. Images stay real, SEO-friendly HTML, and each item can link out.

```html
<div data-cb-type="marquee-wall" data-cb-columns="3" data-cb-gap="16" data-cb-height="520" data-cb-corner-radius="12" data-cb-shadow="true" data-cb-transparent-bg="true" data-cb-speed="45" data-cb-direction="alternate" data-cb-pause-on-hover="true" data-cb-fade-edges="true" data-cb-tilt-x="0" data-cb-tilt-y="-8" data-cb-perspective="1600">
    <div class="mw-wall">
        <div class="mw-item"><a class="mw-card"><img class="mw-img" src="assets/minimalist-blocks/images/ai-01Opn.jpg" alt="Project one"></a></div>
        <div class="mw-item"><a class="mw-card" href="https://example.com" target="_blank" rel="noopener noreferrer"><img class="mw-img" src="assets/minimalist-blocks/images/ai-0Yhoi.jpg" alt="Project two"></a></div>
        <div class="mw-item"><a class="mw-card"><img class="mw-img" src="assets/minimalist-blocks/images/ai-1ANxo.jpg" alt="Project three"></a></div>
        <div class="mw-item"><a class="mw-card"><img class="mw-img" src="assets/minimalist-blocks/images/ai-1owT9.jpg" alt="Project four"></a></div>
        <div class="mw-item"><a class="mw-card"><img class="mw-img" src="assets/minimalist-blocks/images/ai-2JkRP.jpg" alt="Project five"></a></div>
        <div class="mw-item"><a class="mw-card"><img class="mw-img" src="assets/minimalist-blocks/images/ai-2K7j5.jpg" alt="Project six"></a></div>
    </div>
</div>
```

[Options & details →](marquee-wall/README.md)

### Media Slider

A horizontal carousel of cards, each with media (image **or** video), a title, an optional description, and an optional link. Shows one or more items per view, with optional arrows, dots, counter, autoplay, loop, and touch/keyboard navigation. Good for product carousels, featured content, or galleries. Set layout and behavior in the settings; edit each card in the content editor (drag to reorder).

```html
<div data-cb-type="media-slider" data-cb-items-per-slide="1" data-cb-gap="16" data-cb-aspect-ratio="16/9" data-cb-accent-color="#0066cc" data-cb-autoplay="false" data-cb-autoplay-speed="5000" data-cb-loop="true" data-cb-pause-on-hover="true" data-cb-show-arrows="true" data-cb-show-dots="true" data-cb-show-counter="false">
    <div class="media-slider-track">
        <div class="slider-item">
            <div class="item-media">
                <img src="assets/sample.jpg" alt="Product showcase">
            </div>
            <div class="item-content">
                <h3 class="item-title">Premium Widget</h3>
                <p class="item-description">High-quality craftsmanship</p>
                <a href="/product/1" class="item-link" target="_blank" rel="noopener noreferrer">Learn More →</a>
            </div>
        </div>
        <!-- more .slider-item blocks … -->
    </div>
</div>
```

[Options & details →](media-slider/README.md)

### Slider (Swiper)

A full-featured image/video slider built on [Swiper](https://swiperjs.com/): autoplay, loop, transition effects, zoom, navigation, pagination, captions with positioning, and **responsive video sources** (a different video per screen size).

```html
<div data-cb-type="swiper-slider" data-cb-autoplay="true" data-cb-loop="true" data-cb-effect="slide" data-cb-navigation="true" data-cb-pagination="true">
    <div class="swiper">
        <div class="swiper-wrapper">
            <div class="swiper-slide">
                <div class="slide-image">
                    <img src="assets/slide-1.jpg" alt="A description">
                    <div class="slide-overlay"></div>
                    <div class="slide-caption">
                        <div class="is-subblock edit">
                            <h3 class="slide-caption-title">Title</h3>
                            <p class="slide-caption-description">A short caption.</p>
                        </div>
                    </div>
                </div>
            </div>
            <div class="swiper-slide">
                <div class="slide-image"><img src="assets/slide-2.jpg" alt=""></div>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](swiper-slider/README.md)

### Text Slider

A carousel of **editable text cards** (quotes, announcements, feature blurbs) on [Swiper](https://swiperjs.com/). Text is edited inline on the page; slides are added / removed / reordered from the settings panel. Options for items-per-view, autoplay, loop, fade/slide, text alignment, accent color, and optional solid/glass card styling.

```html
<div data-cb-type="text-slider" data-cb-autoplay="true" data-cb-loop="true" data-cb-effect="slide" data-cb-slides-per-view="1" data-cb-text-align="center" data-cb-navigation="true" data-cb-pagination="true" data-cb-background="transparent" data-cb-border="true">
    <div class="swiper">
        <div class="swiper-wrapper">
            <div class="swiper-slide">
                <div class="text-slide">
                    <div class="is-subblock edit">
                        <h3 class="text-slide-title">Slide title</h3>
                        <p class="text-slide-text">Editable text goes here.</p>
                    </div>
                </div>
            </div>
        </div>
        <div class="swiper-button-prev"></div>
        <div class="swiper-button-next"></div>
        <div class="swiper-pagination"></div>
    </div>
</div>
```

[Options & details →](text-slider/README.md)

### Media Slider (Fullscreen)

A fullscreen background slideshow of images **and/or** videos, with a slow Ken-Burns zoom, a darkening overlay, and optional text content per slide. Built for hero sections — autoplay, keyboard nav, pagination dots, a slide counter, and prev/next buttons are all optional. Designed to sit inside a ContentBox section (`.is-section` / `.is-overlay`).

```html
<div data-cb-type="fullscreen-slider" data-cb-autoplay-duration="6000" data-cb-transition-speed="1200" data-cb-zoom-effect="true" data-cb-zoom-scale="108" data-cb-overlay-opacity="30">
    <div class="media-overlay"></div>
    <div class="media-slide" data-slide-id="slide-1">
        <div class="media-slide-inner">
            <img src="assets/photo-1.jpg" alt="Mountain view">
        </div>
    </div>
    <div class="media-slide" data-slide-id="slide-2">
        <div class="media-slide-inner">
            <video autoplay loop muted playsinline data-src-desktop="assets/clip.mp4"></video>
        </div>
    </div>
    <!-- more .media-slide blocks … -->
</div>
```

[Options & details →](fullscreen-slider/README.md)

### Before/After Slider

An image comparison slider — drag a handle to wipe between a "before" and an "after" image. Works horizontally or vertically, with optional corner labels. Good for edits/retouching, renovations, or product comparisons. Set the start position, orientation, and labels in the settings; set the two images, their alt text, and label text in the content editor.

```html
<div data-cb-type="before-after-slider" data-cb-start-position="50" data-cb-orientation="horizontal" data-cb-show-labels="true">
    <div class="ba-container">
        <div class="ba-after">
            <img src="assets/after.jpg" alt="After">
        </div>
        <div class="ba-before">
            <img src="assets/before.jpg" alt="Before">
        </div>
        <span class="ba-label-before">Before</span>
        <span class="ba-label-after">After</span>
        <div class="ba-slider">
            <div class="ba-handle"></div>
        </div>
    </div>
</div>
```

[Options & details →](before-after-slider/README.md)

### Video Embed

A self-hosted `<video>` player embedded in the content — with aspect-ratio, fit, and corner-radius controls, plus the usual autoplay/loop/muted/controls/inline options and an optional poster image. For a *background* video use the **Video Background** plugin instead; this one is an in-flow player.

```html
<div data-cb-type="video-embed" data-cb-autoplay="false" data-cb-loop="false" data-cb-muted="false" data-cb-controls="true" data-cb-plays-inline="true" data-cb-aspect-ratio="16/9" data-cb-object-fit="cover" data-cb-border-radius="8" data-cb-hover-shadow="true" data-cb-controls-overlay="70">
    <div class="video-wrapper">
        <video>
            <source src="assets/clip.mp4" type="video/mp4">
            Your browser does not support the video tag.
        </video>
    </div>
</div>
```

[Options & details →](video-embed/README.md)

### YouTube

Embeds a YouTube video from any YouTube link — `watch`, `youtu.be`, `/embed/`, `/shorts/`, `/live/`, or a raw 11-character ID. Advanced but easy: a **click-to-load** thumbnail facade (loads the heavy player only when clicked, so the page stays fast), **privacy mode** (`youtube-nocookie`), start time, autoplay/mute/loop/controls, a related-videos toggle, aspect ratio, and corner radius.

```html
<div data-cb-type="youtube-embed" data-cb-video="https://www.youtube.com/watch?v=aqz-KE-bpKQ" data-cb-start-time="0" data-cb-click-to-load="true" data-cb-autoplay="false" data-cb-mute="false" data-cb-loop="false" data-cb-controls="true" data-cb-no-cookie="true" data-cb-related-videos="false" data-cb-aspect-ratio="16/9" data-cb-border-radius="12">
    <div class="yt-embed-frame"></div>
</div>
```

[Options & details →](youtube-embed/README.md)

### Video Background

A full-cover looping background video for a section, with an optional color overlay and optional play/pause & mute controls. Supports **responsive sources** — different video files per screen size, chosen automatically by the browser — plus a poster image for the loading state.

```html
<div style="min-height:450px; position:relative;">
    <div data-cb-type="video-background" data-cb-autoplay="true" data-cb-loop="true" data-cb-muted="true" data-cb-plays-inline="true" data-cb-show-overlay="false" data-cb-overlay-opacity="20" data-cb-overlay-color="#000000" data-cb-show-controls="true" data-cb-controls-position="bottom-right">
        <div class="video-bg-wrapper">
            <video>
                <source src="assets/clip.mp4" type="video/mp4">
            </video>
            <div class="video-bg-overlay"></div>
        </div>
    </div>

    <!-- your foreground content here (sits above the video) -->
</div>
```

[Options & details →](video-background/README.md)

### Audio / Podcast Player

A themeable player around a native `<audio>` — play/pause, draggable seek with a buffered bar, elapsed/total time, skip back/forward, playback speed, volume, and an optional download button. Set the audio file, title, subtitle, and artwork in the editor; the look and which controls appear are options.

```html
<div data-cb-type="audio-player" data-cb-accent-color="#2563eb" data-cb-bg-color="#ffffff" data-cb-text-color="#1a1a1a" data-cb-art-shape="rounded" data-cb-art-size="68" data-cb-corner-radius="14" data-cb-shadow="true" data-cb-border="true" data-cb-skip-buttons="true" data-cb-skip-seconds="15" data-cb-speed-control="true" data-cb-volume-control="true" data-cb-show-time="true" data-cb-download="false" data-cb-loop="false" role="region" aria-label="Audio player">
    <div class="ap-card">
        <img class="ap-art" src="assets/minimalist-blocks/images/ai-cvlfg.jpg" alt="">
        <div class="ap-main">
            <div class="ap-title">Episode 1 — Getting Started</div>
            <div class="ap-subtitle">The Podcast</div>
            <div class="ap-controls"></div>
        </div>
        <audio class="ap-audio" src="assets/minimalist-blocks/example.mp3" preload="metadata"></audio>
    </div>
</div>
```

[Options & details →](audio-player/README.md)

### Logo Loop

A continuously scrolling marquee of logos (or any small images) — the classic "as seen in / trusted by" strip. Add, edit, and remove logos from the content editor.

```html
<div data-cb-type="logo-loop" data-cb-speed="20" data-cb-direction="left" data-cb-gap="40" data-cb-pause-on-hover="true">
    <div class="logo-item"><img src="assets/logo-1.svg" alt="Acme"></div>
    <div class="logo-item"><img src="assets/logo-2.svg" alt="Globex"></div>
    <div class="logo-item"><img src="assets/logo-3.svg" alt="Initech"></div>
</div>
```

[Options & details →](logo-loop/README.md)

### Browser Mockup

Frames a screenshot inside a 3D-tilted browser/app window (with the classic three traffic-light dots) — great for showcasing a product, app, or website. Set the screenshot in the content editor; tune the tilt and size in the settings.

```html
<div data-cb-type="browser-mockup" data-cb-max-width="1100" data-cb-rotate-x="8" data-cb-rotate-y="-8" data-cb-enable-hover="true">
    <div class="app-showcase">
        <div class="app-container">
            <div class="app-window">
                <div class="window-header">
                    <span class="window-dot"></span>
                    <span class="window-dot"></span>
                    <span class="window-dot"></span>
                </div>
                <div class="app-screenshot">
                    <img class="screenshot-image" src="assets/screenshot.jpg" alt="App Mockup">
                </div>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](browser-mockup/README.md)

### Map

Embeds an interactive map for an address, place name, `lat,lng` coordinates, or a pasted Google Maps link. Set the location, zoom, map type, and height in the settings panel. Adding `data-cb-size-mode="fill"` to the block makes it a full-cover map for use as a section background (e.g. in a ContentBox `.is-overlay-content`).

```html
<div data-cb-type="map-embed" data-cb-location="Eiffel Tower, Paris, France" data-cb-zoom="14" data-cb-map-type="road" data-cb-size-mode="fixed" data-cb-height="380" data-cb-border-radius="12">
    <div class="map-embed-frame"></div>
</div>
```

[Options & details →](map-embed/README.md)

## Backgrounds & visual effects

### Aurora Glow

A flowing **aurora** background on a `<canvas>` — several large, soft color clouds that drift organically and glow where they overlap, finished with a subtle film grain. Great as a hero backdrop or a section background. Choose a dark (glowing), light (pastel), or transparent (overlay) treatment. There's no per-block content to edit.

```html
<div data-cb-type="aurora-glow"
     data-cb-background="light"
     data-cb-color-a="#5b8cff"
     data-cb-color-b="#c15bff"
     data-cb-color-c="#38e8c6"
     data-cb-speed="1"
     data-cb-intensity="100"
     data-cb-blur="40"
     data-cb-grain="12"
     style="min-height: 480px;">
</div>
```

[Options & details →](aurora-glow/README.md)

### Gradient Blobs

A content block filled with soft, animated blurred-gradient "blobs" that slowly drift and morph — a warm decorative panel. Pick a color scheme, intensity, blob count, animation speed, opacity, and blur in the settings. There's no per-block content to edit.

```html
<div data-cb-type="blobs-bg"
     data-cb-color-scheme="autumn"
     data-cb-intensity="medium"
     data-cb-blob-count="4"
     data-cb-animation-speed="medium"
     data-cb-blob-opacity="70"
     data-cb-blob-blur="80"
     style="min-height: 50px;">
</div>
```

[Options & details →](blobs-bg/README.md)

### Hero Animation

A generative canvas animation for hero sections and section backdrops — pick from 12 algorithmic visuals (neural network, pendulum harmonics, oscillating rings, fractal tree, Fourier series, Lorenz attractor, Bezier curves, circular packing, Lissajous curves, graph network, spline interpolation, circular waves). The visual is drawn from a single base color, at a chosen speed and canvas size. Everything is configured in the settings — there's no per-block content to edit.

```html
<div data-cb-type="hero-animation" data-cb-animation-type="neural" data-cb-speed="1" data-cb-base-color="#6464ff" data-cb-canvas-width="600" data-cb-canvas-height="600">
    <canvas></canvas>
</div>
```

[Options & details →](hero-animation/README.md)

### Pendulum Harmonics

A generative canvas animation of many pendulums swinging at slightly different frequencies, tracing evolving harmonic (pendulum-wave) patterns. Highly tunable — physics (count, angles, frequency spread), drawing style (lines / bezier curves / dots, trail length and fade, blur), colors, and layout are all settings. There's no per-block content to edit.

```html
<div data-cb-type="pendulum" data-cb-speed="2.1" data-cb-color="#6464ff" data-cb-pendulum-count="10" data-cb-trail-length="100" data-cb-bg-color="#ffffff" data-cb-color-variation="100" data-cb-line-width="3.5" data-cb-trail-opacity="0.1" data-cb-pendulum-opacity="0" data-cb-ball-size="3" data-cb-ball-opacity="0.1" data-cb-blur="0" data-cb-draw-mode="curves" data-cb-fade-curve="linear" data-cb-swing-angle="60" data-cb-start-angle="45" data-cb-length-ratio="0.3" data-cb-base-freq="0.005" data-cb-freq-increment="0.001" data-cb-freq-distribution="linear" data-cb-pivot-y="20" data-cb-canvas-size="600" data-cb-margin="10" data-cb-transparent-bg="true">
</div>
```

[Options & details →](pendulum/README.md)

### Particle Constellation

A generative canvas animation of drifting particles that link up with lines when they're close, forming a shifting "constellation" web that also reacts to the mouse. Highly tunable — particle count/size/shape, connection distance and line style, colors, and glow/blur are all settings. There's no per-block content to edit.

```html
<div data-cb-type="particle-constellation" data-cb-particle-count="80" data-cb-color="#6464ff" data-cb-connection-distance="120" data-cb-canvas-size="600">
</div>
```

[Options & details →](particle-constellation/README.md)

### Vector Force Field

A big display-text headline whose letters behave like a **force field**: as the cursor moves near them, each letter is pushed away and springs back on a little physics simulation (spring + damping), with a custom ring cursor and glow. Optional proximity color-shift, rotation, and scale. The text stays real, selectable HTML (SEO-friendly). Best on a **dark section** (default text color is white).

```html
<div data-cb-type="vector-force"
     data-cb-font-size="6" data-cb-font-weight="900" data-cb-letter-spacing="0.05" data-cb-text-color="#4970ff"
     data-cb-force-radius="200" data-cb-force-strength="0.8" data-cb-spring-strength="0.15" data-cb-damping="0.85" data-cb-max-displacement="100"
     data-cb-cursor-size="30" data-cb-cursor-color="#4970ff" data-cb-show-glow="true" data-cb-glow-color="#6496ff"
     data-cb-enable-color-change="true" data-cb-enable-rotation="false" data-cb-enable-scale="false" data-cb-rotation-intensity="2" data-cb-scale-intensity="0.3">
    <h1 class="vf-text">FORCE FIELD</h1>
</div>
```

[Options & details →](vector-force/README.md)

## Forms & sharing

### Form

Build a form visually — short and long text, email, phone, number, dropdown, select, multi-select, switch, slider, date, time, url, colour, file upload — style it in the builder's Designer, and drop it into a page. The form is written into the page's HTML, so search engines see it and visitors see it immediately.

```html
<div data-cb-type="form" data-cb-loading="none">
    <form class="cb-form" method="POST" action="/formsubmission">
        <div class="cb-form-fields formview-container fb-ui"><div class="form-header"><h1 class="form-title">Get in touch</h1><p class="form-desc">We usually reply within one business day.</p></div><div class="form-wrapper" style="flex-flow: wrap;"><div class="form-field"><label for="short_text_label_id-xih1ow90g" class="field-label">Your Name: <span class="text-red-500">*</span></label><input id="short_text_label_id-xih1ow90g" type="text" name="your_name" class="inp-base" placeholder="Enter your name" required="required"></div><div class="form-field"><label for="email_label_id-1tz92khft" class="field-label">Your Email: <span class="text-red-500">*</span></label><input id="email_label_id-1tz92khft" type="email" name="email" class="inp-base" placeholder="Enter your email" required="required"></div><div class="form-field"><label for="long_text_label_id-92mowv747" class="field-label">Message:</label><textarea id="long_text_label_id-92mowv747" name="message" class="inp-base" placeholder="How can we help?"></textarea></div><div class="form-field reference_code"><label for="short_text_label_id-2nnlpbfdg" class="field-label">Reference Code:</label><input id="short_text_label_id-2nnlpbfdg" type="text" name="reference_code" autocomplete="off" aria-hidden="true" tabindex="-1" class="inp-base"></div><div class="submit-container"><button class="btn-submitform">Send</button></div></div></div>
        <div class="cb-form-status" role="status" aria-live="polite"></div>
    </form>
    <div class="cb-form-result"></div>
    <script type="application/json" class="cb-form-def">{
  "form": {
    "title": "Get in touch",
    "description": "We usually reply within one business day.",
    "elements": [
      {
        "title": "Your Name:",
        "name": "your_name",
        "type": "short-text",
        "isRequired": true,
        "placeholder": "Enter your name"
      },
      {
        "title": "Your Email:",
        "name": "email",
        "type": "email",
        "isRequired": true,
        "placeholder": "Enter your email"
      },
      {
        "title": "Message:",
        "name": "message",
        "type": "long-text",
        "placeholder": "How can we help?"
      }
    ],
    "useSubmitButton": true,
    "submitText": "Send"
  },
  "settings": {
    "action": "/formsubmission",
    "submissionType": "json",
    "successMessage": "Thanks for your submission!",
    "errorMessage": "Oops! There was a problem submitting your form.",
    "recaptchaKey": "",
    "resultSelector": ""
  }
}</script>
</div>
```

[Options & details →](form/README.md)

### Contact / Newsletter Form

A real, accessible `<form>` block with a configurable field set (text, email, phone, textarea, consent checkbox), client-side validation, and submit-via-`fetch` with inline success/error feedback.

```html
<div data-cb-type="contact-form" data-cb-action="/submit-form" data-cb-method="POST" data-cb-button-text="Send Message" data-cb-success-message="Thanks! We'll be in touch soon." data-cb-error-message="Something went wrong. Please try again." data-cb-accent-color="#2563eb" role="form" aria-label="Contact form">
    <form class="cf-form" novalidate>
        <div class="cf-fields">
            <label class="cf-field">
                <span class="cf-label">Name <span class="cf-req" aria-hidden="true">*</span></span>
                <input type="text" name="name" placeholder="Your name" required>
            </label>
            <label class="cf-field">
                <span class="cf-label">Email <span class="cf-req" aria-hidden="true">*</span></span>
                <input type="email" name="email" placeholder="you@example.com" required>
            </label>
            <label class="cf-field">
                <span class="cf-label">Message</span>
                <textarea name="message" placeholder="How can we help?" rows="4"></textarea>
            </label>
        </div>
        <button type="submit" class="cf-submit">Send Message</button>
        <p class="cf-status" role="status" aria-live="polite"></p>
    </form>
</div>
```

[Options & details →](contact-form/README.md)

### Social Share

A row of share buttons — X, Facebook, LinkedIn, WhatsApp, Copy Link, and a "More" dropdown (Reddit, Telegram, Email). Buttons open the platform's share dialog in a popup; Copy Link writes the current URL to the clipboard. Choose which platforms show, the alignment, and an optional label in the settings.

```html
<div data-cb-type="social-share" data-cb-alignment="left" data-cb-show-label="true" data-cb-label-text="Share" data-cb-show-x="true" data-cb-show-facebook="true" data-cb-show-linked-in="true" data-cb-show-whats-app="true" data-cb-show-copy-link="true" data-cb-show-more="true"></div>
```

[Options & details →](social-share/README.md)

## Navigation

### Site Logo

A site logo that can be text, an image, or both, switchable in the settings panel. The text takes its font, size and colour from where the logo is placed, so it fits any header design.

```html
<div data-cb-type="site-logo" data-cb-loading="fade" data-cb-image-height="32" data-cb-gap="10">
    <a class="sl-link" href="/"><span class="sl-text">My Website</span></a>
</div>
```

[Options & details →](site-logo/README.md)

### Site Menu

A site navigation menu that takes the look of the design it is placed in. Links (with optional submenus) are managed in the settings panel; on small screens the same links open from a menu button. In a ContentBox navigation bar section, the bar stays at the top and takes the look of the section under it (set per section in the editor).

```html
<div data-cb-type="site-menu" data-cb-loading="fade" data-cb-collapse="960" data-cb-align="right" data-cb-toggle="icon">
    <nav class="sm-nav" aria-label="Main">
        <button type="button" class="sm-toggle" aria-expanded="false" aria-label="Menu"><span class="sm-bars" aria-hidden="true"></span><span class="sm-toggle-text">Menu</span></button>
        <ul class="sm-list">
            <li class="sm-item"><a class="sm-link" href="#">About</a></li>
            <li class="sm-item"><a class="sm-link" href="#">Work</a></li>
            <li class="sm-item"><a class="sm-link" href="#">Contact</a></li>
        </ul>
    </nav>
</div>
```

[Options & details →](site-menu/README.md)

### Site Menu (always visible)

Site Menu with a bar that stays on screen the whole time: see-through while the header stays in place, with its background once the header moves.

```html
<div data-cb-type="site-menu-sticky" data-cb-loading="fade" data-cb-collapse="960" data-cb-align="right" data-cb-toggle="icon">
    <nav class="sm-nav" aria-label="Main">
        <button type="button" class="sm-toggle" aria-expanded="false" aria-label="Menu"><span class="sm-bars" aria-hidden="true"></span><span class="sm-toggle-text">Menu</span></button>
        <ul class="sm-list">
            <li class="sm-item"><a class="sm-link" href="#">About</a></li>
            <li class="sm-item"><a class="sm-link" href="#">Contact</a></li>
        </ul>
    </nav>
</div>
```

[Options & details →](site-menu-sticky/README.md)

### Navigation Menu

> Navigation Menu and Sticky Category Nav sit in a `div.is-section` wrapper, so they're built for ContentBox pages, not for editing inside ContentBuilder.

A responsive site navigation bar: a logo (text **or** image), a row of menu items with optional **dropdown submenus**, a mobile hamburger toggle, a subtle shadow on scroll, and a light / dark / "dark-on-start" theme.

```html
<div class="is-section is-box nav-menu">
    <div class="is-overlay">
        <div class="is-overlay-content">
            <div data-cb-type="nav-menu" data-cb-theme="light" data-cb-menu-item-border-radius="6" data-cb-dropdown-border-radius="8">
                <div class="nav-wrapper">
                    <a href="/" class="logo">My Website</a>
                    <button class="menu-toggle" aria-label="Toggle menu" aria-expanded="false"></button>
                    <ul class="nav-list" role="menubar">
                        <li class="nav-item" role="none">
                            <a href="#" class="nav-link" role="menuitem">Home</a>
                        </li>
                        <li class="nav-item has-submenu" role="none">
                            <a href="#" class="nav-link" role="menuitem" aria-haspopup="true" aria-expanded="false">
                                Products <span class="nav-arrow" aria-hidden="true"></span>
                            </a>
                            <button class="submenu-toggle" aria-label="Toggle Products submenu" aria-expanded="false"><!-- chevron --></button>
                            <ul class="submenu" role="menu" aria-label="Products submenu">
                                <li role="none"><a href="#" class="submenu-link" role="menuitem">Item</a></li>
                            </ul>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](nav-menu/README.md)

### Sticky Category Nav

A horizontal in-page navigation bar that sticks to the top and **highlights the current section** as you scroll (scrollspy). Each link points at a section `id` on the page (`#hero`, `#articles`, …). A visually-hidden "Skip navigation" link is added for keyboard users.

```html
<div class="is-section is-box is-section-auto" style="position: sticky; height: 70px; top: 0; z-index: 1;">
    <div class="is-overlay">
        <div class="is-overlay-content">
            <div data-cb-type="sticky-category-nav" role="navigation" aria-label="Category navigation">
                <div class="category-nav-container" role="list">
                    <ul class="category-nav-list">
                        <li role="listitem"><a href="#hero">Hero</a></li>
                        <li role="listitem"><a href="#articles">Articles</a></li>
                        <li role="listitem"><a href="#contact">Contact</a></li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</div>
```

[Options & details →](sticky-category-nav/README.md)

## Learning examples

> Deliberately tiny plugins that show the simplest possible cases — read their code to learn how a plugin is built.

### Click Counter

A block that counts how many times it's clicked and updates its own text. It's the smallest kind of ContentBuilder plugin — a good starting point for learning how a plugin is put together.

```html
<div data-cb-type="click-counter">Clicks: 0</div>
```

[Options & details →](click-counter/README.md)

### Toggle

A block that shows or hides another element on the page when clicked. It's a small example of a plugin that reads an option and acts on a different element.

```html
<button data-cb-type="toggle" data-cb-target="details">Show / hide details</button>

<div id="details">
    <p>This panel is toggled by the button above.</p>
</div>
```

[Options & details →](toggle/README.md)

### Tooltip

A block that shows a small text tooltip near the cursor on hover. It's a short example of a plugin that creates its own element and cleans it up afterwards.

```html
<span data-cb-type="tooltip" data-cb-text="Hello from the tooltip!">Hover me</span>
```

[Options & details →](tooltip/README.md)
