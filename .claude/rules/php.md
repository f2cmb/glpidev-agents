---
paths:
  - "**/*.php"
---

# PHP Rules for GLPI

## Naming
- Variables and array keys: `snake_case` (`$old_name`, `'title_diff'`)
- Methods: `camelCase` (`getFromDB()`, `prepareInputForAdd()`)
- Classes: `PascalCase` (`Computer`, `TicketValidation`)
- Constants: `UPPER_SNAKE` (`READ`, `CREATE`)
- Itemtype references: always `ClassName::class`, never string literals (`'Computer'`)

## Architecture
- No service classes, no DI, no repository pattern, no DTOs, no event dispatchers
- Use CommonDBTM hooks (`prepareInputForAdd()`, `post_addItem()`, etc.)
- Use `global $DB` and `Session::*` — no abstraction layers
- Input normalization belongs in `prepareInputForAdd()`/`prepareInputForUpdate()`, never in front controllers

## Database
- Never raw SQL — use `$DB->request()`, `$DB->insert()`, `$DB->update()`, `$DB->delete()`
- Tables: `glpi_` prefix, plural, snake_case
- Foreign keys: `{itemtype}s_id`

## Rights
- Always `$item->can($id, RIGHT)` for permission checks
- Never `canUpdateItem()`/`canViewItem()`/`canDeleteItem()` for access control (they skip global rights)

## Code Quality
- `Toolbox::logDebug()` for debugging, never `var_dump()`/`print_r()`/`echo`
- `_s()` for translatable strings
- No hardcoded IDs or magic numbers
- Use `TemplateRenderer` for HTML output, never `echo` in classes
