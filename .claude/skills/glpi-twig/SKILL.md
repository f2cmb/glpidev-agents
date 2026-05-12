---
name: glpi-twig
description: GLPI Twig template conventions to apply when reading, writing or editing any *.twig file in a GLPI core or plugin codebase. Enforces TemplateRenderer::getInstance()->display() for rendering from PHP, Twig auto-escaping ({{ variable }}) by default, |raw only when explicitly safe, templates location under templates/, no data fetching inside templates (pass data from controller), reuse of existing GLPI macros and helpers, and prohibits inline <script> blocks (use dedicated JS/TS files). Activates whenever the model is about to read or modify GLPI Twig templates.
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

## Exemples

### ✅ Rendu depuis PHP
```php
TemplateRenderer::getInstance()->display('foo/bar.html.twig', [
    'user' => $user,
    'message' => $message,
]);
```

### ✅ Affichage avec auto-escape
```twig
<p>Bienvenue {{ user.name }}</p>
```

### ❌ Fetch dans le template
```twig
{% set computers = call('Computer::getAll') %}  {# NE PAS faire — fetch côté contrôleur #}
```

### ❌ <script> inline dans le template
```twig
<script>console.log({{ user.id }})</script>  {# Déplacer dans js/ #}
```

### ⚠️ |raw uniquement si la donnée est déjà sanitizée
```twig
{{ trusted_html|raw }}  {# OK seulement si trusted_html a été assaini en amont #}
```
