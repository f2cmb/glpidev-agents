# Anti-Patterns & Bug Patterns

## Architecture Anti-Patterns

| Anti-Pattern | Why It's Wrong | GLPI Way |
|--------------|----------------|----------|
| Service classes (`TokenService`, `UserService`) | Not in GLPI architecture | `CommonDBTM` hooks on the data class itself, **or** a method added to an existing helper (`Toolbox`/`Html`). Do **not** create a new full-static class — a genuinely new class uses instance methods (see below) |
| Dependency injection | Foreign to GLPI | Use `global $DB`, `Session::*` |
| Repository pattern | Over-abstraction | Use `$item->getFromDB()`, `$DB->request()` |
| DTOs | Unnecessary complexity | Use arrays |
| Event dispatchers | Bypasses GLPI hooks | Use `post_addItem()`, `post_updateItem()` |
| **A new class with only `static` methods** (renderer, processor, helper — e.g. `VideoEmbedRenderer::renderAll()`, `FooManager::createX()`) | A full-static class **cannot enter the GLPI 11 dependency-injection container later without changing every call site**. Surfaced twice in core review (PR #23544, PR #24268) — treat as a hard rule, not a preference | **NEVER full-static on a new class.** Instance methods only, even for a stateless utility: `(new VideoEmbedRenderer())->renderAll($html)`. Instantiating locally is trivial and testable; the class stays injectable later with zero signature churn |
| Static factory methods on new `CommonDBTM` subclasses (`public static function createX()`, `toggleX()`, `regenerateX()`) — legacy core still has some (e.g. `PendingReason_Item::createForItem`) | Symmetry with the rest of the codebase, testability, and a single customization point (`prepareInputForAdd` / `post_addItem`) favor the standard lifecycle | **NEVER** on new code: `$obj = new X(); $obj->add($input);` and customize via hooks. When touching legacy factories, refactor toward this lifecycle if feasible |

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
| Adding a positional parameter (esp. a boolean) to an existing public method that already takes `array $params` | Add a key to the array — `$params['allow_video_embeds']` — backward-compatible; a positional change breaks callers, overrides, and plugins |
| Documenting a generic parameter by its current caller (`// KB-only`) | Document by technical effect — a cross-cutting flag stays reusable; an `X-only` comment invents a constraint the code doesn't enforce |

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
