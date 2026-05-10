# Plugin Structure & Naming

## Directory Structure

```
plugins/pluginname/
├── src/                        # PHP classes (REQUIRED - not inc/)
│   ├── MyClass.php
│   └── Service/                # Optional service subdirectory
├── front/                      # Entry points
│   ├── myclass.php
│   └── myclass.form.php
├── templates/                  # Twig templates
├── ajax/                       # AJAX handlers
├── tests/                      # PHPUnit tests (not tests/functional/)
├── setup.php                   # Plugin initialization
├── hook.php                    # Install/uninstall hooks
└── composer.json               # Autoload configuration
```

**DEPRECATED**: `inc/` directory - use `src/` only.

## Namespace Convention

```php
namespace GlpiPlugin\PluginName;

use CommonDBTM;
use Migration;
```

## Table Naming

```php
// Plugin table naming: glpi_plugin_{pluginname}_{tablename}
public static function getTable($classname = null): string
{
    return 'glpi_plugin_pluginname_myclasses';
}
```
