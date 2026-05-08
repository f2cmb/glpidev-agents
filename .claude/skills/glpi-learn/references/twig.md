# Twig — Skeleton & GLPI gotchas

## Section skeleton (used in the body of the produced doc)

```
## Le contexte GLPI
## La mécanique Twig (héritage, macros, escaping)
## L'intégration TemplateRenderer
## Pièges courants
## Pour aller plus loin
```

## What goes in each section

**Le contexte GLPI** — where this template lives (`templates/components/...`, `templates/pages/...`), what controller renders it, what data it receives.

**La mécanique Twig** — the Twig features in play: `{% extends %}`, `{% block %}`, `{% import %}`, `{% include %}`, `{% macro %}`, filters, escape strategies, `{{ var|raw }}` semantics.

**L'intégration TemplateRenderer** — how the template is rendered from PHP:

```php
use Glpi\Application\View\TemplateRenderer;
TemplateRenderer::getInstance()->display('path/to/template.html.twig', [...]);
```

Note that the controller passes data — the template never fetches data itself.

**Pièges courants** — pick the relevant ones from the gotchas list below.

**Pour aller plus loin** — related macros, parent templates, components used together.

## GLPI gotchas to surface (when relevant)

- **Escaping** — `{{ var }}` auto-escapes. `{{ var|raw }}` only when the source is trusted *and* explicitly markup.
- **No data fetching in templates** — pass it from the PHP controller. A template should never call `getFromDB`, `Session::*`, or `$DB`.
- **No inline `<script>`** — JS goes in dedicated `.ts`/`.js` files loaded via the asset pipeline.
- **No raw HTML via `echo`** — render via `TemplateRenderer`. Classes that build HTML are an anti-pattern.
- **Use existing macros** — GLPI ships macros for forms, fields, buttons. Don't roll your own when one exists.
- **Translation** — strings via `__()` / `_n()` in PHP, or Twig's `|trans` filter when set up. Don't hardcode user-facing strings.

## Anti-patterns to flag in debrief mode

- `{{ var|raw }}` introduced on user-controlled input → XSS risk.
- New `<script>` tag inline in a `.twig` file → must be a JS module.
- Template calling `class.method()` on a fresh DB query → fetch in controller, pass to template.
- Hardcoded English/French string in template instead of `|trans` / `__()`.
- New `echo "<div>..."` in a PHP class → must move to a Twig template.
