# PHPUnit — DbTestCase

All GLPI tests extend `DbTestCase` which provides automatic transaction rollback.

## Available Helpers

**Note:** `createItem()` returns the loaded `CommonDBTM` object, not the ID. Use `$item->getID()` when you need the ID.

```php
// Create items (returns CommonDBTM object)
$computer = $this->createItem(Computer::class, [
    'name'        => 'Test PC',
    'entities_id' => 0,
]);

$ticket = $this->createItem(Ticket::class, [
    'name'        => 'Test ticket',
    'content'     => 'Description',
    'entities_id' => 0,
]);

// Update items
$this->updateItem(Computer::class, $computer->getID(), [
    'name' => 'Updated name',
]);

// Delete items (GLPI 11 only)
$this->deleteItem(Computer::class, $computer->getID());

// Verify field values
$this->checkInput(Computer::class, $computer->getID(), [
    'name' => 'Expected name',
    'serial' => 'ABC123',
]);

// Authentication
$this->login('glpi', 'glpi');           // Fast fake login
$this->realLogin('glpi', 'glpi');       // Full authentication

// Entity context
$this->setEntity('_test_root_entity', true);  // name, recursive

// Access log handler
$this->log_handler->hasRecordThatContains('message', 'warning');
```

## Test Structure

```php
<?php

namespace Glpi\Tests;

use DbTestCase;
use Computer;

class ComputerTest extends DbTestCase
{
    public function testSerialValidationRejectsEmpty(): void
    {
        $computer = new Computer();

        $result = $computer->add([
            'name'        => 'Test',
            'serial'      => '',
            'entities_id' => 0,
        ]);

        $this->assertFalse($result);
    }

    public function testSerialValidationAcceptsValid(): void
    {
        $computer = $this->createItem(Computer::class, [
            'name'        => 'Test',
            'serial'      => 'ABC123',
            'entities_id' => 0,
        ]);

        $this->assertGreaterThan(0, $computer->getID());
    }
}
```

## Data Providers

```php
public static function serialProvider(): iterable
{
    yield 'empty serial' => [
        'serial'   => '',
        'expected' => false,
    ];
    yield 'valid serial' => [
        'serial'   => 'ABC123',
        'expected' => true,
    ];
    yield 'serial with spaces' => [
        'serial'   => 'ABC 123',
        'expected' => true,
    ];
}

/**
 * @dataProvider serialProvider
 */
public function testSerialValidation(string $serial, bool $expected): void
{
    $computer = new Computer();
    $result = $computer->add([
        'name'        => 'Test',
        'serial'      => $serial,
        'entities_id' => 0,
    ]);

    if ($expected) {
        $this->assertGreaterThan(0, $result);
    } else {
        $this->assertFalse($result);
    }
}
```

## Regression Test Pattern

```php
/**
 * Regression test for issue #12345
 * Serial validation was not triggered on template creation
 */
public function testSerialValidationOnTemplateCreation(): void
{
    // 1. Recreate exact bug conditions
    $template = $this->createItem(Computer::class, [
        'name'        => 'Template',
        'is_template' => 1,
        'entities_id' => 0,
    ]);

    $computer = new Computer();

    // 2. Action that triggered the bug
    $result = $computer->add([
        'name'             => 'From template',
        'serial'           => '',
        'id'               => $template->getID(),
        '_create_from_tpl' => true,
        'entities_id'      => 0,
    ]);

    // 3. Assert correct behavior (not bug behavior)
    $this->assertFalse($result, 'Empty serial should be rejected');
}
```

## Plugin Test Bootstrap

```php
<?php
// tests/bootstrap.php

define('GLPI_ROOT', dirname(__DIR__, 3));
require_once GLPI_ROOT . '/tests/bootstrap.php';

// Load and activate plugin
$plugin = new Plugin();
$plugin->load('pluginname');
```

## Running

```bash
# All tests
vendor/bin/phpunit

# Specific file
vendor/bin/phpunit tests/functional/ComputerTest.php

# Specific method
vendor/bin/phpunit --filter testSerialValidation
```
