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

**Always name the keys in each `yield` — never positional arguments.** The keys become named
arguments and document every column directly in the failure message. Passing positional rows
(`yield [$serial, $expected];`) forces the reader to count indices against the test signature, and
core reviewers reject it on sight. Do not "simplify" a named provider to positional.

**Reuse an existing provider before creating a new one.** If a provider already feeds the same
method-under-test as `input → expected`, add your cases to it rather than spawning a parallel
`@dataProvider` + test method. A malicious or edge case is just an `input` whose `expected` is the
neutralised output — it belongs in the same provider, as long as the assertion stays homogeneous
(see *Assertion Style* below).

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

## Assertion Style — exact output, not substrings

For deterministic output (generated HTML, rendered strings, serialised data), assert the **whole
expected value** with `assertSame()`. A stack of `assertStringContainsString()` /
`assertStringNotContainsString()` is harder to read and lets regressions through on everything not
explicitly asserted (stray markup, attribute order, leftover fragments). One exact `expected` per
case states the intent and gives full coverage.

```php
// ❌ Partial assertions — hard to read, silent gaps
$out = (new VideoEmbedRenderer())->renderAllAsLink($html);
$this->assertStringContainsString('<a href="https://youtu.be/abc"', $out);
$this->assertStringNotContainsString('<iframe', $out);
$this->assertStringNotContainsString('data-video-provider', $out);

// ✅ Exact output, data-provider driven
public static function renderAsLinkProvider(): iterable
{
    yield 'youtube' => [
        'html'     => '<div data-video-provider="youtube" data-video-id="abc"></div>',
        'expected' => sprintf(self::LINK_TEMPLATE, 'https://youtu.be/abc', 'https://youtu.be/abc'),
    ];
}

private const LINK_TEMPLATE = '<a href="%s" target="_blank" rel="noopener">%s</a>';

/** @dataProvider renderAsLinkProvider */
public function testRenderAllAsLink(string $html, string $expected): void
{
    $this->assertSame($expected, (new VideoEmbedRenderer())->renderAllAsLink($html));
}
```

The template-constant + `sprintf()` idiom keeps the provider readable while each `expected` stays
an exact string. Switching XSS/edge cases to an exact `expected` is also what lets them fold into
the main provider (see *Data Providers* above) instead of a separate test method.

## Regression Test Pattern

```php
/**
 * Serial validation was not triggered on template creation.
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
