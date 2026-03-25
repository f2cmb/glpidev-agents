---
globs: "**/*.twig"
---

# Twig Rules for GLPI

## Output
- Never output raw HTML via `echo` in PHP — use Twig templates
- Use Twig's escaping mechanisms (`{{ variable }}` auto-escapes)
- Use `{{ variable|raw }}` only when explicitly needed and safe

## Templates
- Location: `templates/` directory
- Render via `TemplateRenderer::getInstance()->display()`
- Pass data from PHP controller, never fetch data inside templates

## Conventions
- Use GLPI's existing Twig macros and helpers
- Follow existing template structure patterns in the codebase
- Avoid inline `<script>` blocks — use dedicated JS/TS files instead
