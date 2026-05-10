---
name: glpi-architecture
description: GLPI core architecture reference — CommonDBTM hooks (prepareInputForAdd/Update, post_addItem, post_updateItem, pre/post_deleteItem) and inheritance hierarchy (CommonDropdown, CommonDBRelation, CommonDBChild, CommonITILObject, Asset), database abstraction layer ($DB->request/insert/update/delete + Migration class), Session and rights management ($item->can() vs canUpdateItem(), READ/UPDATE/CREATE/DELETE/PURGE constants), thin front controllers (routing only, business logic belongs in hooks), TemplateRenderer for Twig output, AJAX handlers, helper classes (Toolbox, Html, Dropdown, Plugin) and logging conventions (Toolbox::logDebug/Info/Warning/Error, never var_dump). Use when reasoning about how GLPI core works under the hood.
user-invocable: false
disable-model-invocation: true
---

# GLPI Architecture

Reference documentation for GLPI's core architecture patterns. Each section lives in `references/` and can be loaded on demand.

## Sections

| Topic | Reference |
|---|---|
| CommonDBTM hooks & class hierarchy | [`references/commondbtm.md`](references/commondbtm.md) |
| Database layer (`$DB->request()`, Migration) | [`references/db-layer.md`](references/db-layer.md) |
| Session, rights, `can()` vs `canUpdateItem()` | [`references/session-rights.md`](references/session-rights.md) |
| Front controllers — thin routing layer | [`references/front-controllers.md`](references/front-controllers.md) |
| Templates, AJAX, helper classes, logging | [`references/templates.md`](references/templates.md) |

## Quick rules of thumb

- **Permission checks**: always `$item->can($id, RIGHT)` — never `canUpdateItem()`/`canViewItem()`/`canDeleteItem()` for access control.
- **Business logic placement**: in `prepareInputForAdd()`/`prepareInputForUpdate()` and `post_*Item()` hooks — never in `front/*.php`.
- **Database**: never raw SQL. Use `$DB->request()`, `$DB->insert()`, `$DB->update()`, `$DB->delete()`.
- **Output**: never `echo`/`var_dump`/`print_r`. Use `TemplateRenderer` for HTML, `Toolbox::log*` for debugging.
- **Itemtype references**: always `ClassName::class`, never string literals.
