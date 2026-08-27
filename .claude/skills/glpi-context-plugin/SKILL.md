---
name: glpi-context-plugin
description: Environment overlay for GLPI 11 plugin development — plugin directory layout (`src/` required, never `inc/`), `GlpiPlugin\PluginName` namespace, `glpi_plugin_{name}_{table}` tables, `Hooks::*` constants over string hooks, `Class::install(Migration)`, PHPUnit-only tests under `tests/`, and `../../` as the path to GLPI core. Use when working inside `plugins/<name>/` or `marketplace/<name>/`.
when_to_use: Load when the working directory is a plugin checkout, or when the user says the target is a GLPI plugin rather than core.
---

# Context — GLPI 11 plugin

## Environment

- **GLPI target**: 11.0.x
- **PHP**: 8.1+
- **Working directory**: the plugin repository, inside `plugins/<pluginname>/`
- **Path to GLPI core**: `../../` (e.g. `../../src/CommonDBTM.php`)

## Layout

```
src/                    # PHP classes — REQUIRED, never inc/
front/                  # Entry points
ajax/                   # AJAX handlers
templates/              # Twig templates
tests/                  # PHPUnit tests (not tests/functional/)
setup.php               # Plugin initialization
hook.php                # Install/uninstall
```

## Quick reference

```php
// Namespace
namespace GlpiPlugin\PluginName;

// Hook constants, never string literals
use Glpi\Plugin\Hooks;
$PLUGIN_HOOKS[Hooks::ITEM_PURGE]['pluginname'] = [...];

// Install pattern
public static function install(Migration $migration): void

// Table naming
glpi_plugin_{pluginname}_{tablename}
```

## Test surface

PHPUnit only, under `tests/`. No Playwright, no Cypress — those live in GLPI 11 core, not in plugins.

## Where the detail lives

Structure, `setup.php`, `hook.php` and integration patterns are covered by the `glpi-plugin-patterns` skill; the security checks by `glpi-plugin-security`. This overlay only fixes the environment facts. The `advancedldap` plugin is the validated reference implementation.
