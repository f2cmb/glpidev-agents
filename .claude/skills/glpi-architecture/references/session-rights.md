# Session & Rights

```php
// Check permission
Session::haveRight('computer', READ);
Session::haveRight('ticket', CREATE);
Session::haveRightsOr('computer', [READ, UPDATE]);

// Current user info
Session::getLoginUserID();
Session::getCurrentInterface(); // 'central' or 'helpdesk'

// Active entity
Session::getActiveEntity();
Session::isMultiEntitiesMode();
```

## Right Constants

```php
READ    = 1
UPDATE  = 2
CREATE  = 4
DELETE  = 8
PURGE   = 16
```

## Item-Level Rights Checking

**CRITICAL**: Always use `$item->can($id, RIGHT)` instead of `canUpdateItem()` / `canViewItem()` / `canDeleteItem()`.

| Method | Checks global rights (profile) | Checks item-level rights | Use it? |
|--------|-------------------------------|--------------------------|---------|
| `$item->can($id, UPDATE)` | **Yes** | **Yes** | **Yes** |
| `$item->can($id, READ)` | **Yes** | **Yes** | **Yes** |
| `$item->can($id, DELETE)` | **Yes** | **Yes** | **Yes** |
| `$item->can($id, PURGE)` | **Yes** | **Yes** | **Yes** |
| `$item->canUpdateItem()` | **No** | Yes | **No** — internal override hook only |
| `$item->canViewItem()` | **No** | Yes | **No** — internal override hook only |
| `$item->canDeleteItem()` | **No** | Yes | **No** — internal override hook only |

`can($id, RIGHT)` performs the full permission check: global rights (via `Session::haveRight()`) AND item-level conditions (via `canUpdateItem()`/`canViewItem()` internally).

`canUpdateItem()` and similar methods only check item-specific conditions (ownership, entity, etc.) but **skip global profile rights verification**. This means a user without the UPDATE right in their profile could still pass the check. In most GLPI classes, `canUpdateItem()` returns `true` by default. These methods are meant to be overridden in subclasses to add extra item-level conditions, not to perform complete permission checks.

```php
// WRONG - does not check global rights
if (!$item->canUpdateItem()) {
    throw new AccessDeniedHttpException();
}

// CORRECT - checks both global rights AND item-level rights
if (!$item->can($id, UPDATE)) {
    throw new AccessDeniedHttpException();
}
```
