---
name: glpi-php
description: GLPI PHP conventions to apply when reading, writing or editing any *.php file in a GLPI core or plugin codebase. Enforces snake_case variables and array keys, PascalCase classes, ClassName::class itemtype references (never string literals), CommonDBTM hooks (prepareInputForAdd/Update, post_addItem, post_updateItem, pre/post_deleteItem), no service classes / DI / repositories / DTOs / event dispatchers, no raw SQL (use $DB->request/insert/update/delete), $item->can($id, RIGHT) for access control instead of canUpdateItem/canViewItem/canDeleteItem, Toolbox::logDebug instead of var_dump/print_r/echo, _s() for translatable strings, TemplateRenderer for HTML output. Activates whenever the model is about to read or modify GLPI PHP code.
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

## Exemples

### ✅ Hook CommonDBTM avec normalisation d'input
```php
public function prepareInputForAdd($input) {
    if (isset($input['items_id']) && is_string($input['items_id'])) {
        $input['items_id'] = (int) $input['items_id'];
    }
    return $input;
}
```

### ❌ Logique métier dans front/
```php
// front/computer.form.php — NE PAS faire ça
$_POST['name'] = strtolower($_POST['name']);
$computer->add($_POST);
```

### ❌ canUpdateItem comme contrôle d'accès
```php
if ($computer->canUpdateItem()) { /* ... */ }   // ❌ skip global rights
if ($computer->can($id, UPDATE)) { /* ... */ }  // ✅
```

### ✅ Requête via $DB->request()
```php
$iterator = $DB->request([
    'FROM'  => Computer::getTable(),
    'WHERE' => ['entities_id' => $_SESSION['glpiactive_entity']],
]);
```

### ❌ SQL brut
```php
$DB->doQuery("SELECT * FROM glpi_computers WHERE entities_id = " . $eid);
```
