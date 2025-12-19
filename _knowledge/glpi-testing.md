# GLPI Testing Patterns

Testing conventions and helpers for GLPI development.

## Test Locations

| Context | PHPUnit Location | Cypress Location |
|---------|------------------|------------------|
| GLPI Core | `tests/functional/` | `tests/cypress/e2e/` |
| Plugins | `tests/` | N/A (PHPUnit only) |

## PHPUnit - DbTestCase

All GLPI tests extend `DbTestCase` which provides automatic transaction rollback.

### Available Helpers

```php
// Create items
$computer_id = $this->createItem('Computer', [
    'name'        => 'Test PC',
    'entities_id' => 0,
]);

$ticket_id = $this->createItem('Ticket', [
    'name'        => 'Test ticket',
    'content'     => 'Description',
    'entities_id' => 0,
]);

// Update items
$this->updateItem('Computer', $computer_id, [
    'name' => 'Updated name',
]);

// Delete items
$this->deleteItem('Computer', $computer_id);

// Verify field values
$this->checkInput('Computer', $computer_id, [
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

### Test Structure

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
        $id = $this->createItem('Computer', [
            'name'        => 'Test',
            'serial'      => 'ABC123',
            'entities_id' => 0,
        ]);

        $this->assertGreaterThan(0, $id);
    }
}
```

### Data Providers

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

## Cypress - E2E Tests (Core Only)

### Test Structure

```javascript
describe('Computer Form', () => {
    beforeEach(() => {
        cy.login();
    });

    it('should create computer with valid serial', () => {
        cy.visit('/front/computer.form.php');

        cy.get('[data-testid="name-input"]').type('Test Computer');
        cy.get('[data-testid="serial-input"]').type('ABC123');
        cy.get('[data-testid="submit-btn"]').click();

        cy.get('.alert-success').should('be.visible');
    });

    it('should reject empty serial', () => {
        cy.visit('/front/computer.form.php');

        cy.get('[data-testid="name-input"]').type('Test Computer');
        cy.get('[data-testid="submit-btn"]').click();

        cy.get('.alert-danger').should('contain', 'Serial is required');
    });
});
```

### Custom Commands

Located in `tests/cypress/support/`:

```javascript
// Login command
cy.login();                    // Default admin
cy.login('tech', 'tech');      // Specific user

// Navigation helpers
cy.visitFront('/computer.php');
cy.visitAjax('/ajax/common.tabs.php');

// Form helpers
cy.selectDropdown('locations_id', 'Room 101');
cy.fillTinyMCE('content', 'Description text');

// Wait for AJAX
cy.waitForAjax();
```

### Selector Best Practices

```javascript
// Preferred: data-testid attributes
cy.get('[data-testid="submit-btn"]');

// Acceptable: semantic selectors
cy.get('form#computer-form button[type="submit"]');

// Avoid: fragile class selectors
cy.get('.btn-primary');  // May change with UI updates
```

## Regression Test Pattern

When testing a bug fix:

```php
/**
 * Regression test for issue #12345
 * Serial validation was not triggered on template creation
 */
public function testSerialValidationOnTemplateCreation(): void
{
    // 1. Recreate exact bug conditions
    $template_id = $this->createItem('Computer', [
        'name'        => 'Template',
        'is_template' => 1,
        'entities_id' => 0,
    ]);

    $computer = new Computer();

    // 2. Action that triggered the bug
    $result = $computer->add([
        'name'             => 'From template',
        'serial'           => '',  // Empty serial that should fail
        'id'               => $template_id,
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

## Key Rules

1. **No comments in test code** - Test names should be self-documenting
2. **No cleanup needed** - DbTestCase auto-rollbacks transactions
3. **One concept per test** - Multiple assertions OK if testing one behavior
4. **No private method testing** - Test public API only
5. **No mocks** - Unless existing GLPI tests use them for similar cases
6. **Replicate patterns** - Look at existing tests before writing new ones

## Running Tests

```bash
# All tests
vendor/bin/phpunit

# Specific test file
vendor/bin/phpunit tests/functional/ComputerTest.php

# Specific test method
vendor/bin/phpunit --filter testSerialValidation

# Cypress (core only)
npx cypress run
npx cypress open  # Interactive
```
