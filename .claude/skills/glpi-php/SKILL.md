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
- A new class MUST use instance methods, **never** be full-static — even a stateless utility (renderer, processor). `(new VideoEmbedRenderer())->renderAll($html)`, not `VideoEmbedRenderer::renderAll($html)`. A full-static class cannot enter the GLPI 11 DI container later without breaking every call site (surfaced twice in core review)
- For new `CommonDBTM` subclasses: MUST use `$obj->add($input)` / `update($input)` / `delete($input)`, **never** static factories (`createX()`, `toggleX()`). Legacy core still has some (e.g. `PendingReason_Item::createForItem`); refactor toward the standard lifecycle when feasible
- Use CommonDBTM hooks (`prepareInputForAdd()`, `post_addItem()`, etc.)
- Use `global $DB` and `Session::*` — no abstraction layers
- Input normalization belongs in `prepareInputForAdd()`/`prepareInputForUpdate()`, never in front controllers
- Server-generated sensitive fields (tokens, hashes, secrets) are overwritten unconditionally in `prepareInputForAdd/Update()` — never `if (empty($input[...]))`; the caller must not be able to control them
- When extending a public method, add a key to an existing `array $params` rather than a positional parameter. A new positional argument (especially a boolean) changes the public signature and breaks callers, overrides, and plugins; an array key is backward-compatible. Any public signature change must be justified
- Document a parameter by its technical **effect**, not its current caller. A generic flag on a cross-cutting class (e.g. `RichText::getEnhancedHtml()`) stays reusable — avoid `// X-only` comments that invent a constraint the code does not enforce and that deter the next legitimate consumer

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
- Don't re-emit a value that is already the browser/framework default in generated markup (e.g. `referrerpolicy="strict-origin-when-cross-origin"`) — it's noise, not hardening; omit it
- Comments are minimal: clear, direct, only the non-obvious *why*. Never reference PR numbers, commit hashes, or issue IDs in code — that metadata belongs in the commit/PR, not the source. A verbose comment adds reviewer cognitive load and reads as AI padding

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

### ✅ New utility class — instance methods, never full-static
```php
// A stateless renderer/helper is still written with instance methods.
// Callers instantiate locally; the class stays injectable into the GLPI 11
// DI container later without changing a single call site.
class VideoEmbedRenderer {
    public function renderAll(string $html): string {
        // ...
    }
}

// Call site:
$html = (new VideoEmbedRenderer())->renderAll($content);
```

### ❌ Same class as full-static (cannot be dependency-injected later)
```php
class VideoEmbedRenderer {
    public static function renderAll(string $html): string { /* ... */ }   // ❌
}
VideoEmbedRenderer::renderAll($content);   // ❌ every call site breaks the day DI is needed
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
