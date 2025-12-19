# GLPI Development Instructions

You are assisting with GLPI development. Follow these guidelines strictly.

## GLPI Architecture

All items inherit from `CommonDBTM`. Key hooks:
- `prepareInputForAdd/Update($input)` - Validate/transform before INSERT/UPDATE, return `$input` or `false`
- `post_addItem()` / `post_updateItem()` - Side effects after save
- `pre_deleteItem()` / `post_deleteItem()` - Cleanup on delete

## Database Layer

Use GLPI's DB abstraction, never raw SQL:
```php
$DB->request(['FROM' => 'glpi_computers', 'WHERE' => ['is_deleted' => 0]]);
$DB->insert('glpi_tablename', ['field' => 'value']);
$DB->update('glpi_tablename', ['field' => 'new'], ['id' => $id]);
```

## Naming Conventions

- Tables: `glpi_*` prefix, plural, snake_case
- Fields: snake_case (`is_deleted`, `date_creation`)
- Foreign keys: `{itemtype}s_id` (`computers_id`)
- Classes: PascalCase
- Methods: camelCase

## Session & Rights

```php
Session::haveRight('computer', READ);
Session::getLoginUserID();
Session::getActiveEntity();
```

## Anti-Patterns to Avoid

- Service classes, DI, repositories (not in GLPI)
- Raw SQL queries
- `var_dump`/`print_r` (use `Toolbox::logDebug()`)
- Hardcoded IDs
- Bypassing hook system

## Code Quality

Run `make lint` before committing.

## References

See `_knowledge/` folder for detailed documentation:
- `glpi-architecture.md` - Full architecture reference
- `glpi-conventions.md` - Coding standards
- `glpi-testing.md` - Test patterns
