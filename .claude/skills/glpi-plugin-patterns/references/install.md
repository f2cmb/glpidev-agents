# Install / Uninstall

## hook.php

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
