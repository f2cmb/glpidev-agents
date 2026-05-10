# setup.php — Plugin Initialization

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

## Hook Constants (Glpi\Plugin\Hooks)

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
