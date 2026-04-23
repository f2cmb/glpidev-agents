# RGAA Topic 12 — Navigation

Priority criteria for keyboard navigation and GLPI page structure.

## Criteria 12.3 / 12.4 — Skip links

Every page must provide a skip link to the main content, visible on focus.

**Anti-pattern:**
```twig
{# ❌ no skip link #}
<body>
  <header>...</header>
  <nav>...</nav>
  <main id="main-content">...</main>
```

**Fix:**
```twig
{# ✅ skip link as first element of body #}
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>
  <header>...</header>
  <nav>...</nav>
  <main id="main-content" tabindex="-1">...</main>
```

```css
/* ✅ visible on focus only */
.skip-link {
    position: absolute;
    left: -9999px;
}
.skip-link:focus {
    left: 0;
    top: 0;
    z-index: 9999;
    padding: 0.5rem 1rem;
    background: #000;
    color: #fff;
}
```

## Criterion 12.6 — Identification of grouping regions

The main regions of the page must be identified by HTML5 landmarks or ARIA.

| Region | HTML5 element | Equivalent ARIA role |
|---|---|---|
| Header | `<header>` | `role="banner"` |
| Main navigation | `<nav>` | `role="navigation"` |
| Main content | `<main>` | `role="main"` |
| Footer | `<footer>` | `role="contentinfo"` |
| Search form | `<search>` or `<form>` | `role="search"` |

**Rule:** if a page contains multiple `<nav>` elements, each must have a distinct `aria-label`.

```twig
{# ✅ labelled navigation #}
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Secondary navigation">...</nav>
```

## Criterion 12.8 — Consistent tab order

The tab order must follow the logical reading order (top → bottom, left → right).

**Rules:**
- Never use `tabindex` with a value > 0 — it breaks the natural order
- `tabindex="0"`: makes a non-interactive element focusable (use sparingly)
- `tabindex="-1"`: focusable via JS only (used for focus management in components)

**Anti-pattern:**
```twig
{# ❌ positive tabindex #}
<div tabindex="3">...</div>
<button tabindex="1">Submit</button>
```

**Fix:**
```twig
{# ✅ no positive tabindex, DOM order = focus order #}
<button type="submit">Submit</button>
```

## Criterion 12.9 — No keyboard trap

The user must always be able to leave an interactive component using Tab or Escape.

**Anti-pattern:**
```javascript
// ❌ focus trapped with no way out
element.addEventListener('keydown', function(e) {
    e.preventDefault(); // blocks EVERYTHING
});
```

**Fix:**
```javascript
// ✅ intentional trap inside a modal (acceptable)
// but always let Escape close it
modal.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') closeModal(modal);
    if (e.key === 'Tab') trapFocus(e, modal); // cycle Tab within the modal
});
```
