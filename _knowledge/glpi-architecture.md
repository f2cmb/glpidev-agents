# GLPI Architecture

Reference documentation for GLPI's core architecture patterns.

## CommonDBTM - The Foundation

All GLPI items inherit from `CommonDBTM`. Key hook methods:

| Hook | Purpose | Return |
|------|---------|--------|
| `prepareInputForAdd($input)` | Validate/transform before INSERT | `$input` or `false` to abort |
| `prepareInputForUpdate($input)` | Validate/transform before UPDATE | `$input` or `false` to abort |
| `post_addItem()` | Side effects after INSERT | void |
| `post_updateItem($history)` | Side effects after UPDATE | void |
| `pre_deleteItem()` | Checks before DELETE | `true` to proceed |
| `post_deleteItem()` | Cleanup after DELETE | void |

### Inheritance Hierarchy

```
CommonDBTM
├── CommonDropdown          # Simple dropdowns (Location, Category...)
│   └── CommonTreeDropdown  # Hierarchical dropdowns
├── CommonDBRelation        # M:N relations
├── CommonDBChild           # 1:N child items
├── CommonITILObject        # Ticket, Problem, Change
└── Asset                   # Computer, Monitor, NetworkEquipment...
```

## Database Layer

### DBmysql Abstraction

```php
global $DB;

// Query with iterator (preferred)
$iterator = $DB->request([
    'SELECT' => ['id', 'name'],
    'FROM'   => 'glpi_computers',
    'WHERE'  => ['is_deleted' => 0],
    'ORDER'  => 'name ASC',
    'LIMIT'  => 10
]);
foreach ($iterator as $row) {
    // process $row
}

// Insert
$DB->insert('glpi_tablename', ['field' => 'value']);

// Update
$DB->update('glpi_tablename', ['field' => 'newvalue'], ['id' => $id]);

// Delete
$DB->delete('glpi_tablename', ['id' => $id]);
```

### Migration Class

For schema changes in `install/migrations/`:

```php
$migration->addField('glpi_tablename', 'new_field', 'string');
$migration->addKey('glpi_tablename', 'new_field');
$migration->dropField('glpi_tablename', 'old_field');
$migration->changeField('glpi_tablename', 'field', 'field', 'integer');
```

## Session & Rights

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

### Right Constants

```php
READ    = 1
UPDATE  = 2
CREATE  = 4
DELETE  = 8
PURGE   = 16
```

### Item-Level Rights Checking

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
## Template Rendering (Twig)

```php
use Glpi\Application\View\TemplateRenderer;

TemplateRenderer::getInstance()->display('path/to/template.html.twig', [
    'item'    => $this,
    'params'  => $options,
    'candel'  => $candel,
]);
```

Templates location: `templates/` directory.

## AJAX Handling

```php
// In ajax/*.php controller
use Glpi\Http\Response;

Ajax::returnJson([
    'success' => true,
    'message' => __('Done'),
    'data'    => $result
]);
```

## Helper Classes

| Class | Purpose |
|-------|---------|
| `Toolbox` | Utilities: logging, strings, arrays, files |
| `Html` | HTML generation: forms, buttons, scripts |
| `Dropdown` | Dropdown rendering and AJAX |
| `Session` | User session, rights, preferences |
| `DBConnection` | Database connection management |
| `Plugin` | Plugin management |

## Logging

```php
// Debug (dev only)
Toolbox::logDebug('Message', $variable);

// Info (always logged)
Toolbox::logInfo('Important event');

// Warning
Toolbox::logWarning('Potential issue');

// Error
Toolbox::logError('Error occurred', $exception);
```

**Never use**: `var_dump()`, `print_r()`, `echo` for debugging.
