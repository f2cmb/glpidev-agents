---
applyTo:
  - "src/**/*.php"
  - "front/**/*.php"
  - "ajax/**/*.php"
  - "templates/**/*.twig"
  - "tests/functional/**/*.php"
  - "install/migrations/**/*.php"
---

# GLPI Core Development

You are working on GLPI core (version 10 or 11).

## File Structure

- `src/` - PHP classes (PSR-4)
- `front/` - Entry points
- `ajax/` - AJAX handlers
- `templates/` - Twig templates
- `tests/functional/` - PHPUnit tests
- `tests/cypress/e2e/` - Cypress tests
- `install/migrations/` - DB migrations

## CommonDBTM Hooks

```php
// Validation before save
public function prepareInputForAdd($input) {
    if (empty($input['name'])) {
        Session::addMessageAfterRedirect(__('Name required'), false, ERROR);
        return false;
    }
    return $input;
}

// Side effects after save
public function post_addItem() {
    // Notifications, logging, relations...
}
```

## Template Rendering

```php
TemplateRenderer::getInstance()->display('path/to/template.html.twig', [
    'item' => $this,
    'params' => $options,
]);
```

## Testing

PHPUnit tests extend `DbTestCase`:
```php
$id = $this->createItem('Computer', ['name' => 'Test', 'entities_id' => 0]);
$this->updateItem('Computer', $id, ['name' => 'Updated']);
$this->login('glpi', 'glpi');
```

## Key Rules

1. Follow existing GLPI patterns - search codebase for similar implementations
2. Use `Toolbox::logDebug()` for debugging, never `var_dump`
3. No raw SQL - use `$DB->request()`, `$DB->insert()`, etc.
4. Run `make lint` before committing
