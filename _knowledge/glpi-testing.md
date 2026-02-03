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

## Playwright E2E Tests (GLPI)

### Test Location

Tests are in `tests/e2e/specs/` organized by feature domain.

### Available Fixtures

```typescript
import { test, expect } from '../../utils/fixtures';

test('...', async ({
  page,           // Authenticated Playwright page
  profile,        // ProfileSwitcher - switch user profiles
  entity,         // EntitySwitcher - switch entities
  api,            // REST API client for fast data setup
  csrf,           // CSRF token fetcher
  formImporter    // Import form JSON fixtures
}) => { ... });
```

### Profile Management

```typescript
import { Profiles } from '../../utils/Profiles';

// Available: SuperAdmin, Admin, Technician, Supervisor,
//            Hotliner, Observer, SelfService, ReadOnly
await profile.set(Profiles.SuperAdmin);
await profile.set(Profiles.Technician);
```

### API Data Creation (Preferred)

```typescript
import { getWorkerEntityId } from '../../utils/WorkerEntities';

// Create via API (fast, no UI interaction)
const form_id = await api.createItem('Glpi\\Form\\Form', {
    name: `Test Form - ${crypto.randomUUID()}`,
    entities_id: getWorkerEntityId(),
    is_active: true,
});

const ticket_id = await api.createItem('Ticket', {
    name: `Ticket - ${crypto.randomUUID()}`,
    content: 'Test content',
    entities_id: getWorkerEntityId(),
});
```

### Page Objects

All extend `GlpiPage` base class with helpers:

```typescript
import { FormPage } from '../../pages/FormPage';

const form = new FormPage(page);

// Semantic locators
form.getButton('Save');
form.getTextbox('Name');
form.getCheckbox('Active');
form.getTab('Settings');

// Navigation
await form.doGoToTab('Settings');
await form.goto(form_id);
```

Available page objects: `FormPage`, `FormRenderPage`, `EntityPage`, `KnowbaseItemPage`, `TicketPage`, `DocumentPage`, `ServiceCatalogPage`.

### Select2 Dropdowns

```typescript
// Set dropdown value (handles grouped options)
await page_obj.doSetDropdownValue('Entity', 'Root entity');

// Get available options
const options = await page_obj.getDropdownOptions('Category');
```

### TinyMCE Rich Text

```typescript
// Initialize editor (required for first interaction)
await page_obj.initRichTextByLabel('Content');

// Get editor and type
const editor = page_obj.getRichTextByLabel('Content');
await editor.fill('My content');
```

### Test Structure

```typescript
import { test, expect } from '../../utils/fixtures';
import { Profiles } from '../../utils/Profiles';
import { getWorkerEntityId } from '../../utils/WorkerEntities';
import { FormPage } from '../../pages/FormPage';

test('user can create form', async ({ page, profile, api }) => {
    // 1. Set profile
    await profile.set(Profiles.SuperAdmin);

    // 2. Create test data via API
    const form_id = await api.createItem('Glpi\\Form\\Form', {
        name: `Test - ${crypto.randomUUID()}`,
        entities_id: getWorkerEntityId(),
    });

    // 3. Navigate
    const form = new FormPage(page);
    await form.goto(form_id);

    // 4. Action
    await form.doSaveFormEditor();

    // 5. Assert
    await expect(form.editor_save_success_alert).toBeVisible();
});
```

### Serial Tests

```typescript
test.describe('Feature with shared state', () => {
    test.describe.configure({ mode: 'serial' });

    test('step 1', async ({ page }) => { ... });
    test('step 2', async ({ page }) => { ... });
});
```

### Query Priority

```typescript
// 1. Role queries (best)
await page.getByRole('button', { name: /save/i });
await page.getByRole('checkbox', { name: /active/i });

// 2. Label queries
await page.getByLabel(/name/i);

// 3. Test ID (last resort)
await page.getByTestId('complex-widget');
```

### Playwright Rules

- **DON'T** create data via UI when API available
- **DON'T** use raw CSS selectors - use page object helpers
- **DON'T** hardcode entity IDs - use `getWorkerEntityId()`
- **DON'T** use `waitForTimeout()` - use web-first assertions
- **DON'T** login manually - use authenticated page fixture

## Running Tests

```bash
# PHPUnit - All tests
vendor/bin/phpunit

# PHPUnit - Specific file
vendor/bin/phpunit tests/functional/ComputerTest.php

# PHPUnit - Specific method
vendor/bin/phpunit --filter testSerialValidation

# Cypress (core only)
npx cypress run
npx cypress open  # Interactive

# Playwright E2E
npm run test:e2e
npx playwright test tests/e2e/specs/Feature/test.spec.ts
npx playwright test --ui  # Interactive
```
