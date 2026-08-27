---
name: glpi-context-core-10
description: Environment overlay for GLPI 10.0.x core development — PHP 7.4-8.x constraints (no match expressions, limited type hints), directory layout, `update_10.0.x_to_10.0.y/` migration paths, PHPUnit-only test surface (no Playwright, no Cypress). Use when working in a GLPI core checkout whose `version.php` reports 10.0, when the branch is `10.0/bugfixes`, or when the user says the target is GLPI 10.
when_to_use: Load at the start of any GLPI 10 core task, and whenever a fix must stay PHP 7.4-compatible.
---

# Context — GLPI 10 core

## Environment

- **GLPI version**: 10.0.x
- **PHP**: 7.4 → 8.x — the code must run on 7.4
- **Working directory**: the GLPI core repository
- **Branch**: `10.0/bugfixes`

## Layout

```
src/                    # PHP classes
front/                  # Entry points
ajax/                   # AJAX handlers
templates/              # Twig templates
install/migrations/     # update_10.0.x_to_10.0.y/
tests/functional/       # PHPUnit tests
```

## Version constraints

- **PHP 7.4 syntax only** — no `match` expressions, no constructor promotion, no union types, no `?->`.
- Migration directory format: `update_10.0.x_to_10.0.y/`.
- Test surface is PHPUnit only — `tests/e2e/` and `tests/cypress/` do not exist in GLPI 10.

## Security model

GLPI 10 sanitises `$_GET`/`$_POST` globally via `includes.php`, and entry points are **not** authenticated by default. See the `glpi-plugin-security` skill for the full GLPI 10 vs 11 table — a missing `Session::checkLoginUser()` on `front/`/`ajax/` is always critical here.

## Reference

- Upstream: https://github.com/glpi-project/glpi (branch `10.0/bugfixes`)
