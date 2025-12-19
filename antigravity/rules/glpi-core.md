# GLPI Core Development

You are working on GLPI core. Follow these rules strictly.

## Architecture

All items inherit from `CommonDBTM`:
- `prepareInputForAdd/Update($input)` - Validate before save, return `$input` or `false`
- `post_addItem()` / `post_updateItem()` - Side effects after save
- `pre_deleteItem()` / `post_deleteItem()` - Cleanup on delete

## File Structure

- `src/` - PHP classes (PSR-4)
- `front/` - Entry points
- `ajax/` - AJAX handlers
- `templates/` - Twig templates
- `tests/functional/` - PHPUnit tests
- `tests/cypress/e2e/` - Cypress tests
- `install/migrations/` - DB migrations

## Database - Always use abstraction

```php
// Query
$DB->request(['FROM' => 'glpi_computers', 'WHERE' => ['is_deleted' => 0]]);

// Insert/Update/Delete
$DB->insert('glpi_tablename', ['field' => 'value']);
$DB->update('glpi_tablename', ['field' => 'new'], ['id' => $id]);
$DB->delete('glpi_tablename', ['id' => $id]);
```

NEVER use raw SQL queries.

## Naming Conventions

- Tables: `glpi_*`, plural, snake_case
- Fields: snake_case
- Foreign keys: `{itemtype}s_id`
- Classes: PascalCase
- Methods: camelCase

## Rights

```php
Session::haveRight('computer', READ);
Session::haveRight('ticket', CREATE);
```

## Templates

```php
TemplateRenderer::getInstance()->display('path/template.html.twig', [
    'item' => $this,
]);
```

## Debugging

Use `Toolbox::logDebug()`, NEVER `var_dump` or `print_r`.

## Testing

```php
class MyTest extends DbTestCase {
    public function testSomething(): void {
        $id = $this->createItem('Computer', ['name' => 'Test', 'entities_id' => 0]);
        $this->assertTrue($result);
    }
}
```

## Anti-Patterns to REJECT

- Service classes, dependency injection, repositories
- Raw SQL
- Hardcoded IDs
- Bypassing GLPI hooks
- `var_dump`, `print_r`, `echo` for debugging

## Before Committing

Run `make lint` to verify code quality.
