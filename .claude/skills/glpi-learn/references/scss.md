# SCSS — Skeleton & GLPI gotchas

## Section skeleton (used in the body of the produced doc)

```
## Le contexte GLPI
## L'architecture SCSS et le theming
## Les variables et surcharges
## Pièges courants
## Pour aller plus loin
```

## What goes in each section

**Le contexte GLPI** — where this SCSS lives (`css/`, theme directory, plugin assets), what compiles it (Webpack/equivalent), what consumes the output.

**L'architecture SCSS et le theming** — partials structure, theme files, dark/light variants if relevant. Cite the actual files in play.

**Les variables et surcharges** — GLPI variables (Bootstrap-derived where applicable), how to override in a theme without touching core files.

**Pièges courants** — pick from the list.

**Pour aller plus loin** — related theme files, the build config that compiles them.

## GLPI gotchas to surface (when relevant)

- **Specificity wars** — adding styles deep in selectors causes maintenance pain. Prefer minimal specificity + clear scoping.
- **Theme overrides** — override via theme partials, not by editing core SCSS.
- **Bootstrap heritage** — GLPI builds on Bootstrap. Reuse its variables and utilities before inventing new ones.
- **Dark/light** — if relevant, both themes must be tested.
- **No `!important`** — except in narrowly scoped utility classes, and even then sparingly.
- **No raw color hex** — use theme variables; otherwise dark/light variants break.

## Anti-patterns to flag in debrief mode

- New `!important` in core SCSS.
- Hex color in a partial that has access to a theme variable.
- Selector chain longer than 3–4 levels for layout/structural rules.
- Override of Bootstrap utilities reinventing what already exists.
- Inline `style="..."` in a new Twig template instead of a class.
