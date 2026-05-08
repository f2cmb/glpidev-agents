# PHP — Skeleton & GLPI gotchas

## Section skeleton (used in the body of the produced doc)

```
## Le contexte GLPI
## La mécanique PHP / OOP en jeu
## Le pattern GLPI (CommonDBTM, hooks, rights, DB)
## Pièges courants
## Pour aller plus loin
```

## What goes in each section

**Le contexte GLPI** — 2–3 sentences placing the subject in GLPI's architecture (which class hierarchy, which entry point, which lifecycle).

**La mécanique PHP / OOP en jeu** — language-level features at play: type hints, late static binding, null-safe (`?->`), null coalescing (`??`), closures, Template Method, Observer. PHP 8.x features when relevant (named args, readonly props, enums).

**Le pattern GLPI** — how GLPI implements the concept. Reference real files: `src/CommonDBTM.php`, hooks (`prepareInputForAdd`, `post_addItem`, `pre_deleteItem`, `post_updateItem`), `Session::*`, `$DB->request()`. Cite real lines.

**Pièges courants** — pick the relevant ones from the gotchas list below.

**Pour aller plus loin** — 2–3 concrete pointers to related classes or code paths.

## GLPI gotchas to surface (when relevant)

- **Entity scoping** — does every `$DB->request()` filter by `entities_id`? Cross-entity leaks are silent.
- **Rights checking** — `$item->can($id, RIGHT)` checks profile + item rights. `canUpdateItem()` skips profile rights and is for override only. Always cite the rule.
- **ITIL divergence** — does the change affect Ticket / Problem / Change identically? `CommonITILObject` children diverge.
- **Hook coverage** — does the API path trigger the same hooks as the web form? `prepareInputForAdd` is the canonical place.
- **Migration safety** — reversible? safe on large tables? backwards-compatible with previous code version?
- **Search options** — every new DB field needs a `rawSearchOptions()` entry.
- **Naming** — variables snake_case, methods camelCase, classes PascalCase, itemtype refs always `ClassName::class` (never `'Computer'`).
- **Architecture rule** — no service classes, no DI, no repositories, no DTOs, no event dispatchers. Use hooks.
- **Logging** — `Toolbox::logDebug/Info/Warning/Error`. Never `var_dump`, `print_r`, `echo` for debugging.

## Anti-patterns to flag in debrief mode

- New raw SQL (`$DB->query("SELECT ...")`) → must use `$DB->request()`.
- `canUpdateItem()` / `canViewItem()` / `canDeleteItem()` used as access control → must be `$item->can($id, RIGHT)`.
- Front controller doing input normalization → must move to `prepareInputForAdd/Update`.
- Service / Repository / DI / event dispatcher introduced → not the GLPI way.
- `echo` / `print_r` / `var_dump` outside a logger.
- Hardcoded itemtype string (`'Computer'`) instead of `Computer::class`.
- Direct HTML output via `echo` in classes → must use `TemplateRenderer`.
