# Templates, AJAX, Helpers & Logging

## Template Rendering (Twig)

```php
use Glpi\Application\View\TemplateRenderer;

TemplateRenderer::getInstance()->display('path/to/template.html.twig', [
    'item'    => $this,
    'params'  => $options,
    'candel'  => $candel,
]);
```

Templates location: `templates/` directory.

## AJAX Handling

```php
// In ajax/*.php controller
use Glpi\Http\Response;

Ajax::returnJson([
    'success' => true,
    'message' => __('Done'),
    'data'    => $result
]);
```

## Helper Classes

| Class | Purpose |
|-------|---------|
| `Toolbox` | Utilities: logging, strings, arrays, files |
| `Html` | HTML generation: forms, buttons, scripts |
| `Dropdown` | Dropdown rendering and AJAX |
| `Session` | User session, rights, preferences |
| `DBConnection` | Database connection management |
| `Plugin` | Plugin management |

## Logging

```php
// Debug (dev only)
Toolbox::logDebug('Message', $variable);

// Info (always logged)
Toolbox::logInfo('Important event');

// Warning
Toolbox::logWarning('Potential issue');

// Error
Toolbox::logError('Error occurred', $exception);
```

**Never use**: `var_dump()`, `print_r()`, `echo` for debugging.
