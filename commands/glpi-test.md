---
description: Write tests for GLPI code (PHPUnit or Playwright e2e)
argument-hint: "[e2e|unit] <class-or-method-to-test>"
allowed-tools: Glob, Grep, Read, Edit, Write, Bash
---

# GLPI Test Writing Workflow

Write minimal, effective tests following GLPI patterns.

## Input

Arguments: $ARGUMENTS

Parse arguments:
- If starts with `e2e` → Playwright E2E test (remove `e2e` from target)
- If starts with `unit` → PHPUnit test (remove `unit` from target)
- Otherwise → Auto-detect based on project structure

## Step 1: Detect Test Type

Check project structure:
```
# If tests/e2e/specs/ exists → Playwright available
# If tests/functional/ exists → PHPUnit available
```

For ambiguous cases, prefer:
- `e2e` for UI flows, forms, user interactions
- `unit` for backend logic, models, API

## Step 2: Analyze Target

1. Read the code to understand what needs testing
2. Identify methods/behaviors to test
3. Find existing similar tests

## Step 3: Find Existing Patterns

### For PHPUnit

```bash
# Find related test files
find tests/functional/ -name "*Test.php" | head -20

# Search for patterns
grep -r "createItem" tests/functional/ | head -10
```

### For Playwright E2E

```bash
# Find related spec files
find tests/e2e/specs/ -name "*.spec.ts" | head -20

# Search for patterns
grep -r "api.createItem" tests/e2e/ | head -10
```

## Step 4: Write Test

### PHPUnit Template

```php
<?php

namespace Glpi\Tests;

use DbTestCase;

class {ClassName}Test extends DbTestCase
{
    public function test{DescriptiveName}(): void
    {
        $id = $this->createItem({ClassName}::class, [
            'name' => 'Test item',
            'entities_id' => 0,
        ]);

        $item = new {ClassName}();
        $item->getFromDB($id);
        $result = $item->{methodToTest}();

        $this->assertTrue($result);
    }
}
```

### Playwright E2E Template

```typescript
import { test, expect } from '../../utils/fixtures';
import { Profiles } from '../../utils/Profiles';
import { getWorkerEntityId } from '../../utils/WorkerEntities';

test('user can {action}', async ({ page, profile, api }) => {
    await profile.set(Profiles.SuperAdmin);

    const item_id = await api.createItem('{ItemType}', {
        name: `Test - ${crypto.randomUUID()}`,
        entities_id: getWorkerEntityId(),
    });

    await page.goto(`/front/{item}.form.php?id=${item_id}`);

    // Action
    await page.getByRole('button', { name: /save/i }).click();

    // Assert
    await expect(page.getByRole('alert')).toContainText(/success/i);
});
```

### Regression Test (for bug fixes)

**PHPUnit:**
```php
/**
 * Regression test for issue #{number}
 */
public function test{BugScenarioDescription}(): void
{
    // Recreate exact conditions that triggered bug
    // Assert correct behavior
}
```

**Playwright:**
```typescript
test('regression #{number}: {scenario}', async ({ page, profile, api }) => {
    // Recreate exact conditions that triggered bug
    // Assert correct behavior
});
```

## Step 5: Output

1. Create test file at appropriate location:
   - PHPUnit Core: `tests/functional/{ClassName}Test.php`
   - PHPUnit Plugin: `tests/{ClassName}Test.php`
   - Playwright: `tests/e2e/specs/{Feature}/{feature}.spec.ts`

2. Summary:
```markdown
## Test Created

- **Type**: PHPUnit | Playwright E2E
- **File**: `path/to/test`
- **Tests**: [list of test methods]
- **Covers**: [what's being tested]

### Run with:
\`\`\`bash
# PHPUnit
vendor/bin/phpunit path/to/test

# Playwright
npx playwright test path/to/test
\`\`\`
```

## Rules

- No comments in test code
- One assertion per test concept
- Test public methods/user behaviors only
- Replicate existing patterns exactly
- Always use `ClassName::class` for itemtype references, never string literals
- For Playwright: prefer API data creation over UI
- For Playwright: use page object helpers, not raw selectors
- For Playwright: never `.locator()` with CSS selectors (`playwright/no-raw-locators` ESLint rule)
- For Playwright: never add `data-testid` in application code — use existing semantic locators only
- For Playwright: avoid `getByText()` in modals (ambiguous), prefer `getByRole()`
