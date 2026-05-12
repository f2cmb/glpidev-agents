---
name: glpi-testing
description: GLPI testing patterns — PHPUnit DbTestCase with auto transaction rollback (createItem, updateItem, deleteItem, checkInput, login/realLogin, setEntity, log_handler), data providers, regression test pattern, plugin test bootstrap, and Playwright E2E for GLPI 11 (glpi_fixture, ProfileSwitcher, EntitySwitcher, api.createItem, page objects FormPage/TicketPage/EntityPage/etc., getByRole-first locator priority, Playwright rules: no waitForTimeout, no .locator(CSS), no data-testid in app code, no getByText in modals). Cypress notes for GLPI 10 legacy. Use when writing or reviewing tests for GLPI core or plugins.
user-invocable: false
---

# GLPI Testing Patterns

Testing conventions and helpers for GLPI development. Sections live in `references/`.

## Test Locations

| Context | PHPUnit | Playwright E2E | Cypress (GLPI 11 legacy) |
|---------|---------|----------------|--------------------------|
| GLPI 11 Core | `tests/functional/` | `tests/e2e/specs/` | `tests/cypress/e2e/` |
| GLPI 10 Core | `tests/functional/` | N/A | N/A |
| Plugins | `tests/` | N/A | N/A |

## Sections

| Topic | Reference |
|---|---|
| PHPUnit `DbTestCase` helpers, structure, data providers, regression pattern, plugin bootstrap | [`references/phpunit.md`](references/phpunit.md) |
| Playwright fixtures, page objects, locators, rules (GLPI 11) | [`references/playwright.md`](references/playwright.md) |
| Cypress legacy (GLPI 10) | [`references/cypress.md`](references/cypress.md) |

## Key Rules

1. **No comments in test code** — test names should be self-documenting
2. **No cleanup needed** — `DbTestCase` auto-rollbacks transactions
3. **One concept per test** — multiple assertions OK if testing one behavior
4. **No private method testing** — test public API only
5. **No mocks** — unless existing GLPI tests use them for similar cases
6. **Replicate patterns** — look at existing tests before writing new ones
