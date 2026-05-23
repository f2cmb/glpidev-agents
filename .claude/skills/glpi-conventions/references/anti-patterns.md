# Anti-Patterns & Bug Patterns

## Architecture Anti-Patterns

| Anti-Pattern | Why It's Wrong | GLPI Way |
|--------------|----------------|----------|
| Service classes | Not in GLPI architecture | Use static methods or CommonDBTM hooks |
| Dependency injection | Foreign to GLPI | Use `global $DB`, `Session::*` |
| Repository pattern | Over-abstraction | Use `$item->getFromDB()`, `$DB->request()` |
| DTOs | Unnecessary complexity | Use arrays |
| Event dispatchers | Bypasses GLPI hooks | Use `post_addItem()`, `post_updateItem()` |
| Static factory methods on `CommonDBTM` subclasses (`public static function createX()`, `toggleX()`, `regenerateX()`) | Short-circuits `prepareInputForAdd/Update()`, `post_*Item()` hooks, historisation and rights orchestrated by `add()`/`update()`/`delete()` | Use the standard lifecycle: `$obj = new X(); $obj->add($input);` — customize via hooks, never via static factories on the data class |
| Static mutable state (`private static $cache`, `private static $instance`) on classes that touch `$_SESSION` or compute access decisions | Global state leaks across requests, breaks test isolation (manual reset required), hides dependencies | Instance methods, instantiated at the call site: `(new TokenManager())->hasSessionAccess(...)` |

## Code Anti-Patterns

| Anti-Pattern | Correct Approach |
|--------------|------------------|
| Raw SQL queries | Use `$DB->request()`, `$DB->insert()`, etc. |
| `var_dump()`, `print_r()` | Use `Toolbox::logDebug()` |
| Hardcoded IDs | Use constants or config |
| Magic numbers | Define constants |
| Direct `$_POST`/`$_GET` | Use GLPI's input handling |
| `echo` in classes | Return data, use TemplateRenderer |
| `canUpdateItem()` / `canViewItem()` / `canDeleteItem()` | Use `$item->can($id, UPDATE)` / `can($id, READ)` / `can($id, DELETE)` — the `can*Item()` methods skip global profile rights checks |
| String literal itemtypes (`'Computer'`) | Use `Computer::class` — compile-time error detection, IDE refactoring support, codebase consistency |
| camelCase variables or array keys (`$oldName`, `'titleDiff'`) | Use snake_case: `$old_name`, `'title_diff'` — GLPI uses snake_case globally for variables and array keys, camelCase is reserved for method names only |
| `if (empty($input['token'])) { $input['token'] = generate(); }` for server-generated sensitive fields (tokens, hashes, secrets) in `prepareInputForAdd/Update()` | Always overwrite unconditionally — `$input['token'] = $this->generate();` — a caller-supplied value is a controlled-value injection vector |
| `'item' => $this` (or any full DB object) in params passed to a template rendered for unauthenticated visitors | Pass an explicit field allowlist — `['title' => $this->fields['name'], 'content' => $this->fields['answer']]` — the template must not be able to introspect the model |

## Common Bug Patterns

Quick reference for investigation:

| Symptom | Likely Cause | Where to Look |
|---------|--------------|---------------|
| Data corruption | Missing validation | `prepareInputForAdd/Update()` |
| Unauthorized access | Permission bypass, or using `canUpdateItem()`/`canViewItem()` instead of `can($id, RIGHT)` | `Session::haveRight()` checks, `can()` vs `canUpdateItem()` usage |
| Twig errors | Undefined variable | Controller data passing |
| DB errors after upgrade | Schema mismatch | Migration files |
| Side effects missing | Hook not triggered | `post_*Item()` registration |
| Blank page | PHP fatal error | `files/_log/php-errors.log` |
| AJAX failure | Wrong response format | `Ajax::returnJson()` |
| Front controller fix not testable at class level | Input normalization in wrong layer | Move to `prepareInputForAdd()` / `prepareInputForUpdate()` |
| Same fix works for Ticket but not Problem/Change | False analogy — different internal architecture | Compare hooks and code paths, not just surface patterns |
