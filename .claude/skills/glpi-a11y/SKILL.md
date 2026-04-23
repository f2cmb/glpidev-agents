---
name: glpi-a11y
description: RGAA 4.1 / WCAG AA criteria applied to the GLPI frontend — Twig, JS, CSS, PHP legacy. Activates only on frontend files.
user-invocable: false
---

# GLPI A11y — Frontend Accessibility

Reference: **RGAA 4.1** (primary) + **WCAG 2.2 AA** (baseline).
Target stack: Twig templates, jQuery/vanilla JS, CSS, PHP legacy with inline HTML.

## Activation Rule

This skill activates **only** if the examined file is:
- `.twig`
- `.js` or `.ts`
- `.scss` or `.css`
- `.php` containing at least one structural HTML tag among: `<form>`, `<table>`, `<input>`, `<select>`, `<button>`, `<a href`

For any other file (pure PHP, migrations, tests): **complete silence**.

## Loading References

Load only the relevant files based on detected content:

| Detected content | Reference to load |
|---|---|
| `<img>`, `<svg>`, `<figure>`, `<picture>` | `references/rgaa-images.md` |
| `<form>`, `<input>`, `<select>`, `<textarea>`, `<button>` | `references/rgaa-forms.md` |
| `<table>`, `<th>`, `<td>`, `<caption>` | `references/rgaa-tables.md` |
| JS event listeners, `aria-*`, interactive components (modal, dropdown, tabs) | `references/rgaa-scripts.md` |
| CSS color properties, `color:`, `background`, `border-color` | `references/rgaa-colors.md` |
| `<nav>`, `<header>`, `<main>`, `<footer>`, `role="navigation"`, skip links | `references/rgaa-navigation.md` |

Do not load all files if the content does not justify it.

## Output Format

For each violation found:

```
[RGAA X.X] Criterion title
Severity: Critical | Major | Minor | To verify
File: path/to/file.html.twig:42
Issue: short description of the problem
Fix:
  [corrected code excerpt]
```

If the context is ambiguous (e.g. impossible to determine whether an image is decorative or informative without business context): report as `To verify` rather than a firm violation.

## Severities

- **Critical**: blocking for assistive technologies — silent screen reader, form inaccessible via keyboard, keyboard trap.
- **Major**: definite RGAA non-compliance but can be worked around — label absent but aria-label present, insufficient contrast on secondary text.
- **Minor**: best practice not followed — caption missing on a simple table, scope absent on an obvious th.
- **To verify**: contextual ambiguity — impossible to decide without business information (e.g. decorative vs informative image). Describe the ambiguity without producing a fix.

## Final Note (closing the review)

Always append at the end of the a11y report:

> Static auditing does not replace testing with a real screen reader (NVDA, VoiceOver).
> Recommended bookmarklets to complement: **Focus Order**, **Structure Revealer** (a11y-tools.com/bookmarklets/).
