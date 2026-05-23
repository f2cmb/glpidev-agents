---
name: glpi-php
description: GLPI PHP conventions to apply when reading, writing or editing any *.php file in a GLPI core or plugin codebase. Enforces snake_case variables and array keys, PascalCase classes, ClassName::class itemtype references (never string literals), CommonDBTM hooks (prepareInputForAdd/Update, post_addItem, post_updateItem, pre/post_deleteItem), no service classes / DI / repositories / DTOs / event dispatchers / static factories on CommonDBTM subclasses, no raw SQL (use $DB->request/insert/update/delete), $item->can($id, RIGHT) for access control instead of canUpdateItem/canViewItem/canDeleteItem, Toolbox::logDebug instead of var_dump/print_r/echo, _s() for translatable strings, TemplateRenderer for HTML output. Activates whenever the model is about to read or modify GLPI PHP code.
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
- No `public static function createX()` / `toggleX()` / `regenerateX()` on `CommonDBTM` subclasses — call `$obj->add($input)` / `update($input)` / `delete($input)` and customize via hooks
- No `private static` mutable state (cache, singleton) on classes that touch `$_SESSION` or compute access decisions — use instance methods, instantiated at the call site
- Use CommonDBTM hooks (`prepareInputForAdd()`, `post_addItem()`, etc.)
- Use `global $DB` and `Session::*` — no abstraction layers
- Input normalization belongs in `prepareInputForAdd()`/`prepareInputForUpdate()`, never in front controllers
- Server-generated sensitive fields (tokens, hashes, secrets) are overwritten unconditionally in `prepareInputForAdd/Update()` — never `if (empty($input[...]))`; the caller must not be able to control them

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

## Examples

### ✅ CommonDBTM hook with input normalisation
```php
public function prepareInputForAdd($input) {
    if (isset($input['items_id']) && is_string($input['items_id'])) {
        $input['items_id'] = (int) $input['items_id'];
    }
    return $input;
}
```

### ❌ Business logic in front/
```php
// front/computer.form.php — DO NOT do this
$_POST['name'] = strtolower($_POST['name']);
$computer->add($_POST);
```

### ❌ canUpdateItem as access control
```php
if ($computer->canUpdateItem()) { /* ... */ }   // ❌ skip global rights
if ($computer->can($id, UPDATE)) { /* ... */ }  // ✅
```

### ✅ Query via $DB->request()
```php
$iterator = $DB->request([
    'FROM'  => Computer::getTable(),
    'WHERE' => ['entities_id' => $_SESSION['glpiactive_entity']],
]);
```

### ❌ Raw SQL
```php
$DB->doQuery("SELECT * FROM glpi_computers WHERE entities_id = " . $eid);
```

### ❌ Static factory on a CommonDBTM subclass
```php
// ShareToken.php — bypasses prepareInputForAdd, post_addItem, historisation, rights
class ShareToken extends CommonDBTM {
    public static function createToken(string $itemtype, int $items_id): self|false {
        $t = new self();
        $t->add(['itemtype' => $itemtype, 'items_id' => $items_id]);
        return $t;
    }
}
```

### ✅ Standard CommonDBTM lifecycle (customize via hooks)
```php
$token = new ShareToken();
$token->add([
    'itemtype' => Computer::class,
    'items_id' => $items_id,
]);
// Customization belongs in ShareToken::prepareInputForAdd() / post_addItem()
```

### ❌ Conditional override of a server-controlled sensitive field
```php
public function prepareInputForAdd($input) {
    if (empty($input['token'])) {            // ❌ caller can supply $input['token']
        $input['token'] = self::generate();
    }
    return $input;
}
```

### ✅ Unconditional server-side generation
```php
public function prepareInputForAdd($input) {
    $input['token'] = $this->generate();    // ✅ always overwritten, no injection vector
    return $input;
}
```

### ❌ Static mutable state on a class touching $_SESSION
```php
class TokenManager {
    private static array $validation_cache = [];  // ❌ leaks across requests, test isolation pain

    public static function hasSessionAccess(string $itemtype, int $id): bool { /* ... */ }
}
```

### ✅ Instance methods, instantiated at the call site
```php
class TokenManager {
    public function hasSessionAccess(string $itemtype, int $id): bool { /* ... */ }
}

// Caller
if ((new TokenManager())->hasSessionAccess(Computer::class, $id)) { /* ... */ }
```
