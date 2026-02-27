# GLPI 11 Plugin Development

You are working on a GLPI 11 plugin.

## File Structure

- `src/` - PHP classes (REQUIRED - never use `inc/`)
- `front/` - Entry points
- `templates/` - Twig templates
- `tests/` - PHPUnit tests
- `setup.php` - Plugin initialization
- `hook.php` - Install/uninstall

## Namespace

```php
namespace GlpiPlugin\PluginName;
```

## Hook Registration (setup.php)

Use `Glpi\Plugin\Hooks::*` constants, not strings:
```php
use Glpi\Plugin\Hooks;

$PLUGIN_HOOKS[Hooks::CSRF_COMPLIANT]['pluginname'] = true;
$PLUGIN_HOOKS[Hooks::ITEM_PURGE]['pluginname'] = [
    'AuthLDAP' => 'plugin_pluginname_item_purge',
];

Plugin::registerClass(MyClass::class, ['addtabon' => ['Computer']]);
```

## Install Pattern

Each class implements `install(Migration)`:
```php
public static function install(Migration $migration): void {
    global $DB;
    $default_charset = DBConnection::getDefaultCharset();
    $default_collation = DBConnection::getDefaultCollation();
    $default_key_sign = DBConnection::getDefaultPrimaryKeySignOption();

    $table = self::getTable(); // glpi_plugin_{name}_{table}
    if (!$DB->tableExists($table)) {
        // CREATE TABLE...
    }
}
```

## Table Naming

Tables must be named: `glpi_plugin_{pluginname}_{tablename}`

## PHP 8 Patterns

Use Safe\ functions:
```php
use function Safe\json_encode;
use function Safe\json_decode;
```

## Anti-Patterns

- Code in `inc/` (DEPRECATED)
- String hooks instead of `Hooks::*`
- Install logic in hook.php instead of `Class::install()`
- Missing Safe\ functions

## Key Rules

1. Reference GLPI core via `../../src/`
2. Follow patterns from validated plugins (advancedldap)
3. Run `make lint` before committing
