# GLPI-Specific Engineering Blind Spots

Categories of issues that engineers — and AI — consistently miss when developing for GLPI. Each category includes what gets missed, why, and the question that surfaces it.

---

## 1. Entity Scoping (Multi-Tenancy)

### Why It's Missed

Entity filtering is GLPI's multi-tenancy mechanism. It's invisible when a developer tests with a single entity. AI adds queries that work correctly in isolation but silently return cross-entity data in production environments with multiple entities.

### What Gets Missed

- New `$DB->request()` without `'WHERE' => ['entities_id' => $_SESSION['glpiactive_entity']]`
- Joins that traverse entity boundaries without filtering the joined table
- Aggregate queries (counts, sums) that don't scope by entity
- Search options (`rawSearchOptions()`) that expose cross-entity results
- Background jobs that process items without setting entity context

### The Question That Catches It

"If two entities have data with the same name, does this query return only the right entity's data?"

"Does every `$DB->request()` in this feature filter by `entities_id` — including subqueries and joins?"

### GLPI Pattern

```php
// Always scope to active entity
$iterator = $DB->request([
    'FROM'  => 'glpi_computers',
    'WHERE' => [
        'entities_id' => $_SESSION['glpiactive_entity'],
        'is_deleted'  => 0,
    ]
]);

// Or use getEntitiesRestrictCriteria() for recursive visibility
$criteria = getEntitiesRestrictCriteria('glpi_computers', '', '', true);
```

---

## 2. Rights & Authorization

### Why It's Missed

AI checks authentication (is the user logged in?) but consistently misses authorization (does this user have the right to do this specific thing to this specific item?). GLPI has a two-level rights system — profile-level rights AND item-level rights — that must both be checked.

### What Gets Missed

- Using `canUpdateItem()` / `canViewItem()` / `canDeleteItem()` instead of `$item->can($id, RIGHT)` — the `can*Item()` methods skip global profile rights checks
- Missing `Session::haveRight()` checks at entry points (`front/`, `ajax/`)
- Authorization checked once at the list level but not at the item level (user can list but then access any item by ID)
- Missing `Session::checkRight()` for destructive actions (POST handlers)
- Profile interface not checked — helpdesk users accessing central-only features

### The Question That Catches It

"If I remove this user's profile rights and replay the request with a direct ID, do I still get data?"

"Is `$item->can($id, UPDATE)` called or `$item->canUpdateItem()`? Only the former checks global profile rights."

### GLPI Pattern

```php
// WRONG — skips profile-level check
if (!$item->canUpdateItem()) {
    throw new AccessDeniedHttpException();
}

// CORRECT — full check: profile rights + item-level rights
if (!$item->can($id, UPDATE)) {
    throw new AccessDeniedHttpException();
}

// Entry point check
Session::checkRight('computer', READ);        // global right
Session::checkRight('computer', UPDATE);      // before POST handler
```

---

## 3. Migration Safety

### Why It's Missed

Migrations are irreversible. AI generates `$migration->dropField()` or `$migration->changeField()` without considering: what is currently using that field? Can the old code version work with the new schema during a rolling update? Is there a data backfill needed?

### What Gets Missed

- Dropping a field still referenced in code, views, search options, or reports
- Renaming a field without a transition period (old + new column during rollout)
- Adding a NOT NULL field without a DEFAULT value — fails on non-empty tables
- Missing backfill for new fields with calculated data
- `changeField()` on a field that has an index — index must be rebuilt
- Data migration for enum/status field changes (existing values become invalid)
- No `$migration->displayWarning()` when the migration requires manual admin action

### The Question That Catches It

"If I run this migration on a production DB with 500,000 rows, does it lock the table? For how long?"

"After this migration, can the previous code version still start and run? Or is rollback impossible?"

"Does any existing code, search option, report, or plugin reference the dropped/renamed field?"

### GLPI Pattern

```php
// Safe: add field with default
$migration->addField('glpi_computers', 'new_field', 'integer', ['value' => 0]);

// Risky: dropping a field — always search references first
// grep -r "old_field" src/ front/ ajax/ templates/
$migration->dropField('glpi_computers', 'old_field');

// Risky: renaming — requires two migrations (add new, backfill, drop old)
$migration->addField('glpi_computers', 'new_name', 'string');
// UPDATE glpi_computers SET new_name = old_name
$migration->dropField('glpi_computers', 'old_name');
```

---

## 4. Hook Coverage

### Why It's Missed

GLPI's plugin extensibility depends entirely on hooks being triggered at the right moments. AI implements the happy path but forgets that `post_addItem()`, `post_updateItem()`, `pre_deleteItem()` must fire for all code paths — not just the web form path.

### What Gets Missed

- Logic placed in `front/` controllers instead of `prepareInputForAdd()` / `post_addItem()` — hooks not triggered via API or tests
- Bypassing `$item->add()` / `$item->update()` with direct `$DB->insert()` / `$DB->update()` — no hooks fire
- `Plugin::doHook()` calls missing after state changes that plugins need to react to
- Business logic in `post_addItem()` that should be in `prepareInputForAdd()` — validation skipped on API path
- Side effects (related record creation, notifications) duplicated in both hook and front controller

### The Question That Catches It

"If this action is triggered via the REST API instead of the web form, do all hooks still fire?"

"Would a plugin listening to `post_addItem()` / `post_updateItem()` know about this change?"

"Is there any logic in `front/` that should be in a `prepareInput*` or `post_*` hook?"

### GLPI Pattern

```php
// WRONG — logic in front controller, skipped by API
// front/computer.form.php
if ($_POST['action'] === 'add') {
    $input = $_POST;
    $input['computed_field'] = compute($input); // never runs via API
    $computer->add($input);
}

// CORRECT — logic in hook, always runs
// src/Computer.php
public function prepareInputForAdd($input) {
    $input['computed_field'] = $this->compute($input);
    return $input;
}
```

---

## 5. Session & Profile Context

### Why It's Missed

GLPI's session carries the active entity, active profile, and interface type (central vs helpdesk). Features tested by one developer with full rights and a single entity miss failures that occur when: profile changes, entities switch, or the helpdesk interface is active.

### What Gets Missed

- Hardcoded interface assumptions — feature only works in `central`, breaks in `helpdesk`
- `Session::getActiveEntity()` returns the active entity, not all allowed entities — batch operations miss data
- `Session::isMultiEntitiesMode()` not checked — UI assumes single-entity always
- Profile rights assumed (developer has full rights) — regular user profile hits permission walls
- `$_SESSION['glpiactive_entity_recursive']` not considered — recursive visibility ignored
- Session data mutated directly instead of using `Session::*` methods

### The Question That Catches It

"Does this feature work correctly when the active profile is a limited helpdesk technician instead of super-admin?"

"If the user switches their active entity mid-session, does this feature use the new entity or the stale one?"

"Does this feature respect `is_recursive` — do items visible in a parent entity appear in child entities?"

---

## 6. CommonITILObject Divergence

### Why It's Missed

Ticket, Problem, and Change all inherit from `CommonITILObject` and appear similar. AI applies a fix to `Ticket` and assumes it works for `Problem` and `Change`. But the three classes override hooks differently, have different status machines, and different actor/task/validation structures.

### What Gets Missed

- Fix applied only to `Ticket` — `Problem` and `Change` have the same bug but different code paths
- Status constants are different per class — `Ticket::INCOMING` ≠ `Problem::INCOMING` ≠ `Change::INCOMING`
- Actor roles differ — `Problem` has `requester`/`observer`/`assigned`, `Change` has additional `approval` roles
- Validation flow is only on `Ticket` and `Change` — not `Problem`
- Timeline items (tasks, followups, solutions) have class-specific hooks
- Solution/validation approval loops work differently in Change vs Ticket

### The Question That Catches It

"This fix is on Ticket. Does Problem and Change have the same issue? Do they share the same code path or override differently?"

"Are status constants referenced by integer value? They differ between ITIL types."

---

## 7. Search Options Consistency

### Why It's Missed

`rawSearchOptions()` drives GLPI's search engine, reports, and list views. AI adds a new field to the DB but forgets to add the corresponding search option — or adds one with incorrect `table`/`field` references, wrong `datatype`, or missing `joinparams`. The feature "works" but the field is invisible in searches.

### What Gets Missed

- New field added to DB but no entry in `rawSearchOptions()` — field not searchable/sortable
- `table` or `field` in search option doesn't match actual DB column name
- Wrong `datatype` — `'string'` for a boolean causes incorrect comparisons
- `joinparams` missing for linked tables — query fails or returns wrong results
- `nosearch`, `nodisplay`, `nosort` not set when appropriate
- Search option ID conflicts with existing IDs (must be unique per class)
- Translation string missing for `'name'` key in the option

### The Question That Catches It

"For every new DB field in this feature, is there a corresponding `rawSearchOptions()` entry?"

"Does the search option's `table`/`field` exactly match the DB schema? What is the `joinparams` for linked tables?"

---

## 8. Asset Inventory Conflicts

### Why It's Missed

GLPI's automated inventory (via Glpi Inventory plugin or agent) can overwrite fields set manually. AI implements manual field management without considering that the inventory process may reset those fields on the next agent run.

### What Gets Missed

- Field added to `glpi_computers` (or other asset tables) — will inventory overwrite it?
- Lock mechanism (`is_dynamic`) not considered — inventory respects locks, but lock logic must be maintained
- `is_dynamic` flag set incorrectly — manual entry treated as inventory-managed
- Import rules not updated to handle new field — inventory imports pass, but field is ignored
- Network port / software inventory interaction not considered for network-related features

### The Question That Catches It

"When the GLPI agent runs its next inventory cycle, will it overwrite the data this feature manages?"

"Does this field need to be locked (`is_dynamic = 0`) to survive inventory updates?"

---

## 9. Security Least-Privilege & Proven Necessity

### Why It's Missed

AI grants capabilities defensively — an extra `sandbox` token, a broader scope, a second sanitizer
config — and then writes a comment to justify it ("needed for cross-origin playback", "KB-only").
A narrative justification feels like diligence, but it is not proof. The capability was never tested
*without*, and security state gets duplicated to guard something that is already inert.

### What Gets Missed

- iframe `sandbox` tokens added "to be safe" — `allow-same-origin` + `allow-scripts` together let a
  same-origin frame escape the sandbox; `allow-popups` rarely needed. None were tested by removal.
- A second `HtmlSanitizerConfig`/sanitizer variant created just to allow inert `data-*` attributes in
  one context — duplicated static state, no security gain (the real defense is the render-point
  allowlist + regex, not the sanitizer).
- A permission/right/scope widened beyond the one call site that needs it, justified by a comment.
- Re-emitting a value that is already the default (e.g. `referrerpolicy="strict-origin-when-cross-origin"`)
  — noise mistaken for hardening.

### The Question That Catches It

"For each permission granted (sandbox token, scope, right, allowed attribute): has anyone verified it
*breaks* without it? A justifying comment is not a test."

"Does this conditional security branch protect anything real, or duplicate state for an inert attribute
(`data-*`) that is safe everywhere?"

### GLPI Pattern

```php
// ❌ capability + narrative justification, never tested without
sandbox="allow-scripts allow-same-origin allow-popups"   // "cross-origin player needs same-origin"

// ✅ least-privilege; each token proven necessary by removal
sandbox="allow-scripts allow-presentation"

// ❌ second sanitizer just to allow inert data-* in one context
self::$kb_sanitizer ??= self::buildConfig(allow_data: true);

// ✅ allow the inert data-* once, defend dynamic content at render (allowlist + ID regex)
self::$sanitizer ??= self::buildConfig();
```

---

## Quick Reference

| Blind Spot | Single Most Revealing Question |
|-----------|-------------------------------|
| Entity scoping | "Does every query filter by `entities_id`?" |
| Rights & authorization | "Does `$item->can($id, RIGHT)` check both profile and item rights?" |
| Migration safety | "Can the previous code version still run after this migration?" |
| Hook coverage | "Does this work identically via API as via web form?" |
| Session context | "Does this work with a limited helpdesk profile?" |
| ITIL divergence | "Does the fix apply to Problem and Change, or only Ticket?" |
| Search options | "Is every new DB field in `rawSearchOptions()`?" |
| Inventory conflicts | "Will the next agent run overwrite this data?" |
| Least-privilege proven | "Was each granted permission tested by removal, or just justified by a comment?" |
