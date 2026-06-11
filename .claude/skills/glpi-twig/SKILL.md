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

## Anonymous templates
- For templates rendered to unauthenticated visitors, pass an explicit field allowlist — never the full PHP object
- `{'item': item}` lets the template introspect `{{ item.fields.* }}` and leak every internal field
- Prefer `{'title': item.fields.name, 'content': item.fields.answer}` — only what the view needs

## Conventions
- Use GLPI's existing Twig macros and helpers
- Follow existing template structure patterns in the codebase
- Avoid inline `<script>` blocks — use dedicated JS/TS files instead
- Don't write HTML attributes already at their browser default (e.g. `referrerpolicy="strict-origin-when-cross-origin"`) — omit them; they add markup noise without changing behaviour

## Examples

### ✅ Render from PHP
```php
TemplateRenderer::getInstance()->display('foo/bar.html.twig', [
    'user' => $user,
    'message' => $message,
]);
```

### ✅ Display with auto-escape
```twig
<p>Welcome {{ user.name }}</p>
```

### ❌ Fetch inside the template
```twig
{% set computers = call('Computer::getAll') %}  {# DO NOT do this — fetch in the controller #}
```

### ❌ Inline <script> in the template
```twig
<script>console.log({{ user.id }})</script>  {# Move into js/ #}
```

### ⚠️ |raw only when the data is already sanitised
```twig
{{ trusted_html|raw }}  {# OK only if trusted_html has been sanitised upstream #}
```

### ❌ Full PHP object exposed to an anonymous template
```php
// Controller rendering a publicly shared view
return $this->render('shared_article.html.twig', [
    'title' => $item->fields['name'],
    'item'  => $item,   // ❌ template can read every field via {{ item.fields.* }}
]);
```

### ✅ Explicit field allowlist
```php
return $this->render('shared_article.html.twig', [
    'title'   => $item->fields['name'],
    'content' => $item->fields['answer'],
]);
```
