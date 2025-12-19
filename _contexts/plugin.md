# Context: GLPI 11 Plugin Development

## Environment

- **GLPI Version**: 11.0.x (target)
- **PHP Version**: 8.1+
- **Working directory**: Plugin repository (inside `plugins/pluginname/`)

## Paths

```
src/                    # PHP classes (REQUIRED - never use inc/)
front/                  # Entry points
ajax/                   # AJAX handlers
templates/              # Twig templates
tests/                  # PHPUnit tests (not tests/functional/)
setup.php               # Plugin initialization
hook.php                # Install/uninstall
```

## Plugin-Specific Patterns

See `_knowledge/glpi-plugin-patterns.md` for full details.

### Quick Reference

```php
// Namespace
namespace GlpiPlugin\PluginName;

// Hook constants (not strings)
use Glpi\Plugin\Hooks;
$PLUGIN_HOOKS[Hooks::ITEM_PURGE]['pluginname'] = [...];

// Install pattern
public static function install(Migration $migration): void

// Table naming
glpi_plugin_{pluginname}_{tablename}
```

## Reference Implementation

The `advancedldap` plugin is the validated reference:
- `src/SyncFilter.php` - CommonDBTM with install()
- `src/AuthLdapSyncFilter.php` - CommonDBRelation pattern
- `setup.php` - Modern Hooks::* registration

**Path to GLPI core**: `../../` (e.g., `../../src/CommonDBTM.php`)

## Anti-Patterns to Flag

- Code in `inc/` directory (DEPRECATED)
- String hooks instead of `Hooks::*` constants
- Install logic in hook.php instead of `Class::install()`
- Missing `Safe\` functions for json/file operations

## References

- GLPI core: `../../` relative path
- Tests: `tests/` directory (PHPUnit only, no Cypress)
