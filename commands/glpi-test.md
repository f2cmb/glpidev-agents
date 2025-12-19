---
description: Write tests for GLPI code
argument-hint: <class-or-method-to-test>
allowed-tools: Glob, Grep, Read, Edit, Write, Bash
---

# GLPI Test Writing Workflow

Write minimal, effective tests following GLPI patterns.

## Input
Code to test: $ARGUMENTS

## Step 1: Analyze Target

1. Read the code to understand what needs testing
2. Identify public methods to test
3. Determine test type needed:
   - PHPUnit (backend logic)
   - Cypress (frontend/e2e) - core only

## Step 2: Find Existing Patterns

Search for similar tests:

```bash
# Find related test files
find tests/ -name "*Test.php" | head -20

# Search for similar test patterns
grep -r "createItem" tests/functional/ | head -10
```

Examine:
- Test structure and naming
- Helper methods used
- Assertion patterns

## Step 3: Write Test

### PHPUnit Template

```php
<?php

namespace Glpi\Tests;

use DbTestCase;

class {ClassName}Test extends DbTestCase
{
    public function test{DescriptiveName}(): void
    {
        // Setup
        $id = $this->createItem('{ItemType}', [
            'name' => 'Test item',
            'entities_id' => 0,
        ]);

        // Action
        $item = new {ClassName}();
        $item->getFromDB($id);
        $result = $item->{methodToTest}();

        // Assert
        $this->assertTrue($result);
    }
}
```

### Regression Test (for bug fixes)

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

## Step 4: Output

1. Create test file at appropriate location:
   - Core: `tests/functional/{ClassName}Test.php`
   - Plugin: `tests/{ClassName}Test.php`

2. Summary:
```markdown
## Test Created

- File: `tests/functional/{ClassName}Test.php`
- Tests: [list of test methods]
- Covers: [what's being tested]

### Run with:
\`\`\`bash
vendor/bin/phpunit tests/functional/{ClassName}Test.php
\`\`\`
```

## Rules

- No comments in test code
- One assertion per test concept
- Test public methods only
- Replicate existing patterns exactly
