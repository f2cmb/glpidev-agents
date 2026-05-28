# Playwright E2E Tests (GLPI 11)

## Test Location

Tests are in `tests/e2e/specs/` organized by feature domain.

## Available Fixtures

```typescript
import { test, expect } from '../../fixtures/glpi_fixture';

test('...', async ({
  page,           // Authenticated Playwright page
  profile,        // ProfileSwitcher - switch user profiles
  entity,         // EntitySwitcher - switch entities
  api,            // REST API client for fast data setup
  csrf,           // CSRF token fetcher
  formImporter    // Import form JSON fixtures
}) => { ... });
```

## Profile Management

```typescript
import { Profiles } from '../../utils/Profiles';

// Available: SuperAdmin, Admin, Technician, Supervisor,
//            Hotliner, Observer, SelfService, ReadOnly
await profile.set(Profiles.SuperAdmin);
await profile.set(Profiles.Technician);
```

## API Data Creation (Preferred)

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

## Page Objects

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

## Select2 Dropdowns

```typescript
// Set dropdown value (handles grouped options)
await page_obj.doSetDropdownValue('Entity', 'Root entity');

// Get available options
const options = await page_obj.getDropdownOptions('Category');
```

## TinyMCE Rich Text

```typescript
// Initialize editor (required for first interaction)
await page_obj.initRichTextByLabel('Content');

// Get editor and type
const editor = page_obj.getRichTextByLabel('Content');
await editor.fill('My content');
```

## Test Structure

```typescript
import { test, expect } from '../../fixtures/glpi_fixture';
import { Profiles } from '../../utils/Profiles';
import { getWorkerEntityId } from '../../utils/WorkerEntities';
import { FormPage } from '../../pages/FormPage';

test('user can create form', async ({ page, profile, api }) => {
    await profile.set(Profiles.SuperAdmin);

    const form_id = await api.createItem('Glpi\\Form\\Form', {
        name: `Test - ${crypto.randomUUID()}`,
        entities_id: getWorkerEntityId(),
    });

    const form = new FormPage(page);
    await form.goto(form_id);

    await form.doSaveFormEditor();

    await expect(form.editor_save_success_alert).toBeVisible();
});
```

## Serial Tests

```typescript
test.describe('Feature with shared state', () => {
    test.describe.configure({ mode: 'serial' });

    test('step 1', async ({ page }) => { ... });
    test('step 2', async ({ page }) => { ... });
});
```

## Locator Strategy — Semantic First

GLPI reviewers reject **every** raw locator on sight, even with `eslint-disable-next-line playwright/no-raw-locators` and a justification. Locators MUST target attributes that are coherent with the user-facing semantics of the element. CSS classes, internal `data-*` flags and DOM structure are NOT semantic.

### Mandatory search order

Before writing any locator, walk this list and stop at the first match:

1. **`getByRole(role, { name })`** — unique interactive elements. `button`, `link`, `textbox`, `checkbox`, `radio`, `combobox`, `dialog`, `tab`, `tabpanel`, `alert`, `heading`, `listitem`, `row`, `cell`. The implicit role of native HTML usually works (a `<button>`, an `<input type="checkbox">`, an `<a href>`).
2. **`getByLabel(name)`** — form controls associated to a `<label>`. Works for `input`, `select`, `textarea`.
3. **`getByPlaceholder(text)`** — `<input placeholder>` when there is no label.
4. **`getByTitle(text)`** — elements with a `title` attribute (icon buttons, badges).
5. **`getByAltText(text)`** — images.
6. **`getByText(text)`** — only outside modals/forms (see pitfall below). Use `{ exact: true }` whenever possible.

### When no semantic locator exists

Do not reach for `.locator('.css-class')` or `.locator('[data-something]')`. Treat the absence of a semantic anchor as a **bug in the application markup**, not a tooling problem.

Take one of these actions, in order:

1. **Look harder.** Open the rendered HTML in DevTools (or read the Twig/Vue source) and check parents/ancestors — the role often lives one level up (`<button>` wrapping a `<svg>`, `<dialog role="dialog">` wrapping the dialog body, etc.). Scope with `within` (`page.getByRole('dialog').getByRole('button', { name: /close/i })`).
2. **Add the missing accessibility attribute** to the application code (Twig template, Vue component, PHP-rendered HTML). Prefer `aria-label`, `role`, `<label for=…>`, `<button title=…>`. This is an a11y win that also fixes the test — exactly what reviewers want.
3. **STOP and ask the user.** If you cannot add semantics yourself (third-party widget, frozen markup), do not write the test with a raw locator. Surface the blocker so the user decides: enrich the markup, refactor the widget, or grant an exception.

### Forbidden escape hatches

These patterns will be flagged by reviewers regardless of the comment attached:

| Anti-pattern | Why it gets rejected |
|---|---|
| `this.page.locator('.image-dialog')` | CSS class — not semantic |
| `this.page.locator('[data-video-provider]')` | Internal data attribute — not semantic |
| `this.page.locator('.video-embed-iframe')` | CSS class — even a "descriptive" name is not a role |
| `// eslint-disable-next-line playwright/no-raw-locators -- no ARIA role available` | The eslint comment is not a free pass. The reviewer reads code, not comments. |
| `data-testid="…"` added to a `.twig` / `.vue` file | Test attributes must NEVER live in app code |

The justifications "no ARIA role available", "semantic class", "semantic data attribute" are all rationalizations. A class is not a role. A `data-*` is not a label. If you find yourself writing one of those comments, restart at the "Look harder" step.

### Examples

```typescript
// ❌ Raw locator (rejected on review)
this.page.locator('.video-dialog');

// ✅ Role on the dialog itself (add role="dialog" or <dialog> in the Vue/Twig template)
this.page.getByRole('dialog', { name: /video/i });

// ❌ Raw locator on a data attribute
this.page.locator('[data-video-provider]');

// ✅ Either:
//    a) scope by the surrounding heading/region, or
//    b) add aria-label="Video embed (YouTube)" on the placeholder element
this.page.getByRole('img', { name: /youtube video/i });

// ❌ CSS class on an iframe
this.page.locator('.video-embed-iframe');

// ✅ <iframe title="…"> is required by WCAG anyway
this.page.getByTitle(/video player/i);
```

> **Pitfall: `getByText()` ambiguity in modals**
> In GLPI modals that combine a form and a list (e.g. permissions modal), `getByText('Entity')` can match **multiple elements**: a `<option>` in a dropdown, a substring in an entity name `<span>`, and a badge `<span>`. Playwright raises a `strict mode violation`. Prefer `getByRole()` scoped inside `getByRole('dialog')`.

## Playwright Rules

- **DON'T** create data via UI when API available
- **DON'T** hardcode entity IDs — use `getWorkerEntityId()`
- **DON'T** use `waitForTimeout()` — use web-first assertions
- **DON'T** login manually — use authenticated page fixture
- **NEVER** use `.locator()` with CSS selectors, `data-*` selectors, XPath, or any other raw selector. The ESLint rule `playwright/no-raw-locators` is informational — **the project policy is stricter than the rule**. `eslint-disable-next-line playwright/no-raw-locators` does NOT make a raw locator acceptable.
- **NEVER** add `data-testid` (or any test-only attribute) in application code — Twig, Vue, PHP echo, etc.
- **NEVER** use `getByText()` in modals with forms — same text often appears in dropdown options, entity names, and badges causing strict mode violations. Prefer `getByRole()` scoped inside `getByRole('dialog')`.
- **DO** enrich the application markup with `aria-label`, `role`, `<label for>`, `title` when a semantic anchor is missing. This benefits accessibility and is the reviewer-approved path.
- **DO** stop and ask the user when no semantic locator exists and you cannot enrich the markup yourself.

## Running

```bash
npm run test:e2e
npx playwright test tests/e2e/specs/Feature/test.spec.ts
npx playwright test --ui  # Interactive
```
