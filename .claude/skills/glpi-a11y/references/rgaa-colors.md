# RGAA Topic 3 — Colors

Priority criteria for colors and contrasts in GLPI CSS/templates.

## Criterion 3.1 — Information not conveyed by color alone

Information must not be conveyed by color alone. It must be supplemented by another means (text, icon, pattern, border, etc.).

**Anti-pattern:**
```twig
{# ❌ status indicated by color only #}
<span class="status-dot {{ ticket.is_open ? 'green' : 'red' }}"></span>
```

**Fix:**
```twig
{# ✅ color + text #}
<span class="status-badge {{ ticket.is_open ? 'status-open' : 'status-closed' }}">
  {{ ticket.is_open ? 'Open' : 'Closed' }}
</span>

{# ✅ color + aria-label #}
<span class="status-dot {{ ticket.is_open ? 'green' : 'red' }}"
      aria-label="{{ ticket.is_open ? 'Open' : 'Closed' }}"></span>
```

## Criterion 3.2 — Contrast for normal text (< 18pt or < 14pt bold)

Minimum ratio: **4.5:1** between the text color and its background.

| Case | Required ratio |
|---|---|
| Normal text | 4.5:1 |
| Large text (≥ 18pt or ≥ 14pt bold) | 3:1 |
| Text in UI components (buttons, inputs) | 4.5:1 |

**Verification tool:** WebAIM Contrast Checker — https://webaim.org/resources/contrastchecker/

**Common anti-patterns to flag:**
- Light grey text on a white background (e.g. `color: #999` on `background: #fff` → ratio 2.85:1 ❌)
- Very light grey placeholder text (placeholders are not exempt)
- Status badges with colored text on a colored background

**Note:** Static auditing cannot calculate ratios with certainty when values are stored in CSS or SCSS variables. Flag these as `To verify` in that case.

## Criterion 3.3 — Contrast for UI components

Non-text interface elements (field borders, informative icons, focus cursors) must have a ratio ≥ **3:1** relative to their adjacent color.

**Anti-pattern:**
```css
/* ❌ field border with insufficient contrast */
input {
    border: 1px solid #ccc; /* ratio vs white background: ~1.6:1 */
}
```

**Fix:**
```css
/* ✅ sufficiently contrasted border */
input {
    border: 1px solid #767676; /* ratio vs white background: 4.54:1 */
}
```

## Focus visible

Keyboard focus must be visible and sufficiently contrasted.

**Anti-pattern:**
```css
/* ❌ focus removed with no alternative */
:focus {
    outline: none;
}
```

**Fix:**
```css
/* ✅ custom visible focus */
:focus-visible {
    outline: 2px solid #005fcc;
    outline-offset: 2px;
}
```
