---
name: glpi-plugin-patterns
description: >-
  GLPI 11 plugin development reference — modern directory layout (src/ instead of deprecated inc/, front/, ajax/, templates/, tests/, setup.php, hook.php, composer.json), PSR-4 namespace convention (GlpiPlugin\\PluginName), plugin table naming (glpi_plugin_{name}_{table}), setup.php with plugin_init_* and plugin_version_*, Glpi\\Plugin\\Hooks constants (CSRF_COMPLIANT, CONFIG_PAGE, ITEM_ADD/UPDATE/PURGE, PRE_ITEM_*), hook.php install/uninstall delegating to Class::install(Migration), tab integration via getTabNameForItem/displayTabContentForItem, PHP 8 patterns (Safe\\ functions, match expressions, typed static properties), and plugin anti-patterns. Reference plugin: advancedldap. Use when scaffolding, reviewing or migrating a GLPI 11 plugin.
user-invocable: false
---

# GLPI 11 Plugin Patterns

Modern plugin development patterns for GLPI 11. Sections live in `references/`.

## Sections

| Topic | Reference |
|---|---|
| Directory structure, namespace, table naming | [`references/structure.md`](references/structure.md) |
| `setup.php` — plugin_init, plugin_version, hook constants | [`references/setup-php.md`](references/setup-php.md) |
| `hook.php` install/uninstall + `Class::install(Migration)` pattern | [`references/install.md`](references/install.md) |
| Tab integration, PHP 8 patterns, plugin anti-patterns | [`references/integration.md`](references/integration.md) |

## Reference Implementation

The `advancedldap` plugin serves as a validated reference:
- `src/SyncFilter.php` — CommonDBTM with `install()`
- `src/AuthLdapSyncFilter.php` — CommonDBRelation
- `setup.php` — Modern hook registration
