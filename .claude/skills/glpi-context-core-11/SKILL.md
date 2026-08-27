---
name: glpi-context-core-11
description: Environment overlay for GLPI 11.0.x core development — PHP 8.1+ features available (match, constructor promotion, union types, null-safe operator), directory layout, `update_11.0.x_to_11.0.y/` migration paths, full test surface (PHPUnit + Playwright E2E + legacy Cypress). Use when working in a GLPI core checkout whose `version.php` reports 11.0, when the branch is `main`, or when the user says the target is GLPI 11.
when_to_use: Load at the start of any GLPI 11 core task — this is the default context for work on the `main` branch.
---

# Context — GLPI 11 core

## Environment

- **GLPI version**: 11.0.x
- **PHP**: 8.1+
- **Working directory**: the GLPI core repository
- **Branch**: `main`

## Layout

```
src/                    # PHP classes
front/                  # Entry points
ajax/                   # AJAX handlers
templates/              # Twig templates
install/migrations/     # update_11.0.x_to_11.0.y/
tests/functional/       # PHPUnit tests
tests/e2e/              # Playwright E2E tests
tests/cypress/e2e/      # Cypress tests (legacy)
```

## PHP 8 patterns available

```php
// Match expression
$label = match ($status) {
    'new'    => __('New'),
    'closed' => __('Closed'),
    default  => __('Unknown'),
};

// Constructor promotion
public function __construct(
    private readonly int $id,
    private string $name,
) {}

// Null-safe operator
$name = $item?->getField('name');
```

Typed properties are encouraged. Migration directory format: `update_11.0.x_to_11.0.y/`.

## Security model

GLPI 11 removed the global `$_GET`/`$_POST` sanitisation pipeline and authenticates entry points by default (opt out via `Glpi\Http\Firewall`). `$DB->request()` escapes automatically, `doQuery()` does not. See the `glpi-plugin-security` skill for the full GLPI 10 vs 11 table.

## Reference

- Upstream: https://github.com/glpi-project/glpi (branch `main`)
