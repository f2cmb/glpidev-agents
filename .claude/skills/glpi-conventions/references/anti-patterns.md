# Anti-Patterns & Bug Patterns

## Architecture Anti-Patterns

| Anti-Pattern | Why It's Wrong | GLPI Way |
|--------------|----------------|----------|
| Service classes | Not in GLPI architecture | Use static methods or CommonDBTM hooks |
| Dependency injection | Foreign to GLPI | Use `global $DB`, `Session::*` |
| Repository pattern | Over-abstraction | Use `$item->getFromDB()`, `$DB->request()` |
| DTOs | Unnecessary complexity | Use arrays |
| Event dispatchers | Bypasses GLPI hooks | Use `post_addItem()`, `post_updateItem()` |

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
