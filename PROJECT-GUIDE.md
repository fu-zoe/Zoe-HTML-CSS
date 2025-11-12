
# Zoe · HTML/CSS Review — Project Guide

This guide explains the project structure, CSS architecture, naming conventions, and how to add new pages that match the global look & feel.

---

## 1) Repository layout (typical)
```
/
├─ index.html
├─ pages/
│  ├─ html/
│  │  ├─ 01-start-here.html
│  │  ├─ 02-head.html
│  │  └─ …
│  └─ css/
│     ├─ 00-labs.html
│     ├─ 01-basics-cascade.html
│     └─ …
└─ styles/
   ├─ app.css           ← Global bundle that imports all layers
   ├─ tokens.css        ← Design tokens (colors, type scale, spacing, motion…)
   ├─ base.css          ← Base elements, resets, layout utilities
   ├─ components.css    ← Global UI components (nav, buttons, cards, forms, pager)
   └─ pages/
      ├─ html/01-start-here.css   ← Page-only styles (HTML chapter 1)
      ├─ html/02-head.css
      ├─ html/…
      └─ css/..
```

> **Rule of thumb**
>
> - **Global** things live in `tokens.css`, `base.css`, `components.css`.
> - **Per-page** tuning lives in `styles/pages/**` (under the `pages` layer only).
> - Each page HTML loads `styles/app.css` **plus** exactly one page-only CSS file.

---

## 2) CSS architecture (layered)

### 2.1 `tokens.css` — Design tokens
- Theme-agnostic scales: font sizes, spacing, radii, shadows, motion easings.
- Color system & gradients for light/dark, semantic colors, and page/section backgrounds.
- Global constants: container width, sticky nav height (`--nav-h`), unified page padding (`--page-pad`).

**You consume tokens, you do not redefine them in page files.**

### 2.2 `base.css` — Base & layout
- Normalize/soft reset, base elements (`html`, `body`, headings, lists, code/pre).
- Layout primitives (`.l-container`, `.l-stack`, `.l-grid`, `.l-cluster`) and utility helpers (`.u-mt-*`, `.u-text-center`, etc.).
- Guard rails for overflow and responsive type scale.
- Sticky anchor offsets: `section[id] { scroll-margin-top: calc(var(--nav-h) + 12px); }`

### 2.3 `components.css` — Global UI components
- **Nav** (`.c-nav`, `.c-nav__inner`, `.c-nav__link`): glassy sticky header that darkens on scroll.
- **Buttons** (`.c-button`, variants `--primary`, `--outline`, `--subtle`, `--sm/--lg`). Reuse these everywhere.
- **Cards** (`.c-card`), **Breadcrumbs** (`.c-breadcrumbs`), **Pager** (`.c-pager`), **Badges** (`.c-badge--a…e`).
- **Forms** (`.c-field`, `.c-input`, `.c-select`, `.c-textarea`, `.c-help`, `.c-checkrow`).
- **Tables** (`.c-table`).

**Do not restyle these in page CSS.** Compose them with page wrappers and utilities.

### 2.4 `app.css` — Global bundle
- Imports tokens → base → components in the right order (layers).
- Adds any **very small** global glue (if needed). All substantial styling goes either in components or page files.
- Every HTML page should include:
  ```html
  <!-- Global bundle -->
  <link rel="stylesheet" href="../../styles/app.css">
  <!-- Page-only CSS -->
  <link rel="stylesheet" href="../../styles/pages/<area>/<file>.css">
  ```

---

## 3) Page structure & conventions

### 3.1 HTML skeleton (per page)
```html
<body class="page--AREA-chN">
  <!-- Skip link -->
  <a class="c-skiplink" href="#main">Skip to main content</a>

  <!-- Glassy sticky nav (global) -->
  <header class="c-nav" role="banner">
    <div class="c-nav__inner">
      <a class="c-nav__brand" href="../../index.html">Zoe · HTML/CSS Review</a>
      <nav class="c-nav__links" aria-label="Primary">
        <a class="c-nav__link" href="../../index.html">Home</a>
        <a class="c-nav__link is-active" href="./index.html" aria-current="page">HTML</a>
        <a class="c-nav__link" href="../css/index.html">CSS</a>
      </nav>
    </div>
  </header>

  <main id="main" class="l-container chapter">
    <!-- Breadcrumbs (global component) -->
    <nav class="c-breadcrumbs u-mt-4" aria-label="Breadcrumb">…</nav>

    <!-- Hero -->
    <header class="c-card chapter__hero">…</header>

    <!-- Content -->
    <article class="l-stack">…</article>

    <!-- Pager -->
    <nav class="c-pager u-mt-5" aria-label="Page navigation">…</nav>
  </main>

  <footer class="l-container u-text-center u-mt-5 u-mb-4">…</footer>
</body>
```

### 3.2 Page-only CSS
- Create a file under `styles/pages/...` and use **only** the `pages` layer:
  ```css
  /* Page-only styles */
  @layer pages {
    body.page--html-ch1 { 
      background: var(--gradient-bg-mist) fixed;
      background-color: var(--color-bg);
    }
    /* Page wrapper, hero, code overflow guards, media queries… */
  }
  ```
- Keep selectors scoped to the page (`.page--area-chN …` or `.chapter …`) to avoid bleeding styles.
- Do **not** redefine token values or component styles here.

### 3.3 Naming
- **Layout**: `l-*` (e.g., `l-container`, `l-grid`, `l-stack`, `l-cluster`).
- **Components**: `c-*` (e.g., `c-nav`, `c-card`, `c-button`, `c-breadcrumbs`, `c-pager`). Variants via `--modifier`.
- **Utilities**: `u-*` (e.g., `u-mt-4`, `u-text-center`, `u-visually-hidden`).
- **Page wrapper**: `body.page--html-chN` / `body.page--css-chN`.
- **Page sections**: `chapter__hero`, etc. (double underscore for subparts).

---

## 4) JavaScript glue (per page)

Add this at the bottom of each page (already present in most files):

```html
<script>
  // Year
  (function () {
    var y = document.getElementById("y");
    if (y) y.textContent = new Date().getFullYear();
  })();

  // Glassy nav darken on scroll (works everywhere)
  (function () {
    var nav = document.querySelector(".c-nav");
    if (!nav) return;
    var onScroll = function () {
      if (window.scrollY > 6) nav.classList.add("is-scrolled");
      else nav.classList.remove("is-scrolled");
    };
    onScroll();
    window.addEventListener("scroll", onScroll, { passive: true });
  })();
</script>
```

---

## 5) Adding a new chapter page (checklist)

1. **Duplicate** a similar page from `pages/html` or `pages/css`.
2. Update:
   - `<title>` and meta description
   - Body class (`page--html-chN` / `page--css-chN`)
   - Breadcrumbs labels
   - Hero badges & text
3. Create a **page-only CSS** file under `styles/pages/...` with:
   - Unified background: `background: var(--gradient-bg-mist) fixed; background-color: var(--color-bg);`
   - Card/code overflow guards (see other chapters)
   - Ultra-narrow safety, sticky anchor offset, responsive type scale
4. Use **global components**: breadcrumbs, buttons (`.c-button` variants), pager, forms, etc.
5. Test:
   - No horizontal scroll at any width
   - Nav glass darkens on scroll
   - Code blocks scroll **inside** the card
   - Reduced-motion preferences respected (if page has animations)

---

## 6) Common pitfalls & fixes

- **“Glow band” or segmented background**  
  Always set both the soft gradient **and** a solid page color on the body:
  ```css
  body.page--… {
    background: var(--gradient-bg-mist) fixed;
    background-color: var(--color-bg);
  }
  ```

- **Glassy nav not spanning full width**  
  Use the provided header markup (`.c-nav` + `.c-nav__inner`) and never add extra wrappers around it. The `.c-nav__inner` already handles max container width and side padding.

- **Buttons not matching the global style**  
  Wrap interactive actions with `.c-button` and an appropriate modifier (`--primary`, `--outline`, `--subtle`). Avoid raw `<button>` without class unless the page demo requires it.

- **Horizontal overflow from long code**  
  Keep pre/code inside cards with `overflow: auto` and `max-width: 100%`. Page-only CSS already sets this; just use `<pre><code>…</code></pre>` blocks.

- **Anchor jumps hidden by sticky nav**  
  Ensure `section[id] { scroll-margin-top: calc(var(--nav-h) + 12px); }` is present in the page CSS.

- **Form fields inconsistent**  
  Prefer the **global form controls** (`.c-field`, `.c-input`, `.c-select`, `.c-textarea`, `.c-help`) over ad‑hoc styles.

---

## 7) Quick reference

- **Tokens you will use often**
  - `--container`, `--page-pad`, `--nav-h`
  - Colors: `--color-bg`, `--color-surface`, `--color-text`, `--color-muted`, `--color-border`, `--color-primary`, `--color-accent`
  - Gradients: `--gradient-bg-mist`, `--gradient-surface`, badge gradients
  - Motion: `--ease`, `--dur-1/2/3`

- **Key components**
  - `.c-nav` sticky header, `.c-breadcrumbs`, `.c-card`, `.c-button(--primary|--outline|--subtle)`, `.c-pager`
  - Form building blocks: `.c-field`, `.c-label`, `.c-input`, `.c-select`, `.c-textarea`, `.c-help`, `.c-checkrow`

---

## 8) Example: minimal page + page CSS

**HTML**
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Area · Chapter N · Title</title>
    <link rel="stylesheet" href="../../styles/app.css" />
    <link rel="stylesheet" href="../../styles/pages/area/NN-title.css" />
  </head>
  <body class="page--area-chN">
    <a class="c-skiplink" href="#main">Skip to main content</a>
    <header class="c-nav" role="banner">…</header>
    <main id="main" class="l-container chapter">
      <nav class="c-breadcrumbs u-mt-4" aria-label="Breadcrumb">…</nav>
      <header class="c-card chapter__hero">…</header>
      <article class="l-stack">…</article>
      <nav class="c-pager u-mt-5" aria-label="Page navigation">…</nav>
    </main>
    <footer class="l-container u-text-center u-mt-5 u-mb-4">
      <small>© <span id="y"></span> Zoe · Area · Chapter N</small>
      <!-- Year + nav scroll script here -->
    </footer>
  </body>
</html>
```

**Page CSS**
```css
/* Page-only */
@layer pages {
  body.page--area-chN {
    background: var(--gradient-bg-mist) fixed;
    background-color: var(--color-bg);
  }
  .chapter .c-card pre { max-width:100%; box-sizing:border-box; overflow:auto; }
  .chapter .c-card code { word-break: normal; }
  @media (max-width: 360px) {
    .chapter .c-card pre { white-space: pre-wrap; word-break: break-word; }
    .c-button { flex-wrap: wrap; max-width: 100%; }
  }
  .chapter section[id] { scroll-margin-top: calc(var(--nav-h) + 12px); }
  .chapter h1 { font-size: clamp(1.7rem, 2.8vw, 2.2rem); }
  .chapter h2 { font-size: clamp(1.25rem, 2vw, 1.6rem); }
  .chapter p, .chapter li { font-size: clamp(0.95rem, 1.8vw, 1rem); }
  .page--area-chN main { overflow-x: hidden; }
}
```

---

Happy building!
