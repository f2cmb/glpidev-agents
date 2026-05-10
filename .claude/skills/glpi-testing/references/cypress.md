# Cypress — E2E Tests (GLPI 10, legacy)

> **For GLPI 11, use Playwright** (see `playwright.md`). Cypress is documented here for maintaining existing tests only.

## Test Structure

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

## Custom Commands

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

## Selector Best Practices

```javascript
// Preferred: data-testid attributes
cy.get('[data-testid="submit-btn"]');

// Acceptable: semantic selectors
cy.get('form#computer-form button[type="submit"]');

// Avoid: fragile class selectors
cy.get('.btn-primary');  // May change with UI updates
```

## Running

```bash
npx cypress run
npx cypress open  # Interactive
```
