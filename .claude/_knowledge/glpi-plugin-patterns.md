# GLPI 11 Plugin Patterns

Modern plugin development patterns for GLPI 11.

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

## setup.php - Plugin Initialization

```php
<?php

use Glpi\Plugin\Hooks;

define('PLUGIN_PLUGINNAME_VERSION', '1.0.0');

function plugin_init_pluginname(): void
{
    global $PLUGIN_HOOKS;

    $PLUGIN_HOOKS[Hooks::CSRF_COMPLIANT]['pluginname'] = true;

    // Config page
    $PLUGIN_HOOKS[Hooks::CONFIG_PAGE]['pluginname'] = 'front/config.form.php';

    // Item hooks
    $PLUGIN_HOOKS[Hooks::ITEM_PURGE]['pluginname'] = [
        'AuthLDAP' => 'plugin_pluginname_item_purge',
    ];

    // Register classes with tabs
    Plugin::registerClass(
        \GlpiPlugin\PluginName\MyClass::class,
        ['addtabon' => ['Computer', 'Ticket']]
    );
}

function plugin_version_pluginname(): array
{
    return [
        'name'           => 'Plugin Name',
        'version'        => PLUGIN_PLUGINNAME_VERSION,
        'author'         => 'Author',
        'license'        => 'GPLv3',
        'homepage'       => 'https://example.com',
        'requirements'   => [
            'glpi' => ['min' => '11.0', 'max' => '11.99'],
            'php'  => ['min' => '8.1'],
        ],
    ];
}
```

### Hook Constants (Glpi\Plugin\Hooks)

| Constant | Purpose |
|----------|---------|
| `Hooks::CSRF_COMPLIANT` | Declare CSRF compliance |
| `Hooks::CONFIG_PAGE` | Plugin configuration page |
| `Hooks::ITEM_ADD` | After item creation |
| `Hooks::ITEM_UPDATE` | After item update |
| `Hooks::ITEM_PURGE` | After item deletion |
| `Hooks::PRE_ITEM_ADD` | Before item creation |
| `Hooks::PRE_ITEM_UPDATE` | Before item update |
| `Hooks::PRE_ITEM_PURGE` | Before item deletion |

## hook.php - Installation

```php
<?php

use GlpiPlugin\PluginName\MyClass;
use GlpiPlugin\PluginName\AnotherClass;

function plugin_pluginname_install(): bool
{
    $migration = new Migration(PLUGIN_PLUGINNAME_VERSION);

    MyClass::install($migration);
    AnotherClass::install($migration);

    $migration->executeMigration();
    return true;
}

function plugin_pluginname_uninstall(): bool
{
    $migration = new Migration(PLUGIN_PLUGINNAME_VERSION);

    MyClass::uninstall($migration);
    AnotherClass::uninstall($migration);

    $migration->executeMigration();
    return true;
}
```

## Class install() Pattern

```php
<?php

namespace GlpiPlugin\PluginName;

use CommonDBTM;
use DBConnection;
use Migration;

class MyClass extends CommonDBTM
{
    public static $rightname = 'plugin_pluginname_myclass';

    public static function install(Migration $migration): void
    {
        global $DB;

        $default_charset   = DBConnection::getDefaultCharset();
        $default_collation = DBConnection::getDefaultCollation();
        $default_key_sign  = DBConnection::getDefaultPrimaryKeySignOption();

        $table = self::getTable();

        if (!$DB->tableExists($table)) {
            $DB->doQuery("
                CREATE TABLE `$table` (
                    `id` int {$default_key_sign} NOT NULL AUTO_INCREMENT,
                    `name` varchar(255) DEFAULT NULL,
                    `entities_id` int {$default_key_sign} NOT NULL DEFAULT '0',
                    `is_recursive` tinyint NOT NULL DEFAULT '0',
                    `date_creation` timestamp NULL DEFAULT NULL,
                    `date_mod` timestamp NULL DEFAULT NULL,
                    PRIMARY KEY (`id`),
                    KEY `entities_id` (`entities_id`),
                    KEY `date_creation` (`date_creation`),
                    KEY `date_mod` (`date_mod`)
                ) ENGINE=InnoDB DEFAULT CHARSET={$default_charset}
                  COLLATE={$default_collation} ROW_FORMAT=DYNAMIC
            ");
        }

        // Add new fields to existing table
        $migration->addField($table, 'new_field', 'string');
        $migration->addKey($table, 'new_field');
    }

    public static function uninstall(Migration $migration): void
    {
        global $DB;
        $DB->doQuery("DROP TABLE IF EXISTS `" . self::getTable() . "`");
    }
}
```

## Table Naming

```php
// Plugin table naming: glpi_plugin_{pluginname}_{tablename}
public static function getTable($classname = null): string
{
    return 'glpi_plugin_pluginname_myclasses';
}
```

## Tab Integration

```php
public function getTabNameForItem(CommonGLPI $item, $withtemplate = 0): string
{
    if ($item instanceof AuthLDAP) {
        return self::createTabEntry(self::getTypeName(Session::getPluralNumber()));
    }
    return '';
}

public static function displayTabContentForItem(
    CommonGLPI $item,
    $tabnum = 1,
    $withtemplate = 0
): bool {
    if ($item instanceof AuthLDAP) {
        self::showForAuthLDAP($item);
        return true;
    }
    return false;
}
```

## PHP 8 Patterns

### Safe Functions

```php
use function Safe\json_encode;
use function Safe\json_decode;
use function Safe\file_get_contents;
use function Safe\preg_match;

// These throw exceptions instead of returning false
$data = json_decode($json, true);
```

### Match Expressions

```php
$result = match ($status) {
    'new'      => __('New'),
    'assigned' => __('Assigned'),
    'solved'   => __('Solved'),
    default    => __('Unknown'),
};
```

### Typed Properties

```php
public static string $rightname = 'plugin_pluginname_myclass';
public static bool $is_entity_assign = true;
```

## Reference Implementation

The `advancedldap` plugin serves as a validated reference:
- `src/SyncFilter.php` - CommonDBTM with install()
- `src/AuthLdapSyncFilter.php` - CommonDBRelation
- `setup.php` - Modern hook registration

## Plugin Anti-Patterns

| Anti-Pattern | Correct Approach |
|--------------|------------------|
| Code in `inc/` | Use `src/` only |
| String hooks (`'item_add'`) | Use `Hooks::ITEM_ADD` |
| Install logic in hook.php | Use `Class::install(Migration)` |
| Missing Safe\ functions | Use Safe\ for json, file, preg |
| Untyped PHPDoc | Add proper type hints |
