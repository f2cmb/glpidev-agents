---
name: glpi-testing
description: >-
  GLPI testing patterns — PHPUnit DbTestCase with auto transaction rollback (createItem, updateItem, deleteItem, checkInput, login/realLogin, setEntity, log_handler), data providers, regression test pattern, plugin test bootstrap, and Playwright E2E for GLPI 11 (glpi_fixture, ProfileSwitcher, EntitySwitcher, api.createItem, page objects FormPage/TicketPage/EntityPage/etc., semantic-first locator policy: getByRole/getByLabel/getByTitle/getByPlaceholder — NEVER raw `.locator()` even with eslint-disable, NEVER data-testid in app code, missing-semantic = enrich markup or stop and ask). Cypress notes for GLPI 10 legacy. Use when writing or reviewing tests for GLPI core or plugins.
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
| PHPUnit `DbTestCase` helpers, structure, data providers, regression pattern, plugin bootstrap, review discipline | [`references/phpunit.md`](references/phpunit.md) |
| Playwright fixtures, page objects, locators, rules (GLPI 11) | [`references/playwright.md`](references/playwright.md) |
| Cypress legacy (GLPI 10) | [`references/cypress.md`](references/cypress.md) |

## Key Rules

1. **No comments in test code** — test names should be self-documenting
2. **No cleanup needed** — `DbTestCase` auto-rollbacks transactions
3. **One concept per test** — multiple assertions OK if testing one behavior
4. **No private method testing** — test public API only
5. **No mocks** — unless existing GLPI tests use them for similar cases
6. **Replicate patterns** — look at existing tests before writing new ones
7. **Playwright: semantic locators only** — `getByRole`/`getByLabel`/`getByTitle`/`getByPlaceholder`. Raw `.locator()` is rejected on review **even with `eslint-disable-next-line playwright/no-raw-locators`**. If no semantic anchor exists, enrich the app markup (`aria-label`, `role`, `<label for>`, `title`) — a11y win + test fix. If you cannot enrich the markup, **stop and surface the blocker** — ask the user when you are in the main conversation, and report it as your result when you are a subagent (which cannot ask). Either way, write no test rather than a raw locator. Never add `data-testid` in app code. See [`references/playwright.md`](references/playwright.md#locator-strategy--semantic-first).

## PHPUnit Data Providers & Assertions

Recurring core-review feedback — apply on every PHPUnit test:

1. **Data providers MUST use named keys**, never positional. `yield 'case name' => ['input' => …, 'expected' => …];`, never `yield [$input, $expected];`. The keys document each column in the failure message; "simplifying" to positional forces the reader to count indices and is rejected on review.
2. **Assert exact output, not substrings.** For deterministic output (generated HTML, strings), one `assertSame($expected_exact, $actual)` beats a parade of `assertStringContainsString()`/`assertStringNotContainsString()` — partial `contains` lets regressions through on everything not asserted. Idiom: a template constant + `sprintf()` in the provider builds each `expected`.
3. **Reuse the existing provider.** Before adding a new `@dataProvider` + test method, check whether one already covers the same method-under-test (`input → expected`); add your cases there. A malicious/edge case is just an `input` with its neutralised `expected`. One entry point per method when the assertion is homogeneous.

See [`references/phpunit.md`](references/phpunit.md#data-providers).
