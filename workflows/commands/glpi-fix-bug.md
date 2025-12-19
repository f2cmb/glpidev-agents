---
description: Complete bug fix workflow - investigate, fix, review, test
argument-hint: <issue-url-or-description>
allowed-tools: Glob, Grep, Read, Edit, Write, Bash, WebFetch, WebSearch, Task, AskUserQuestion
---

# GLPI Bug Fix Workflow

You are orchestrating a complete bug fix workflow for GLPI. Follow these phases sequentially.

## Input
Bug to fix: $ARGUMENTS

## Phase 1: Investigation (bug-investigator)

First, thoroughly investigate the bug:

1. If a GitHub issue URL is provided, fetch and analyze it
2. Search the codebase for affected components
3. Trace the execution path from user action to failure
4. Identify the root cause with file:line references

**Output a bug scenario:**
```
## Bug Analysis
### Summary: [description]
### Root Cause: [file:line] - [explanation]
### Proposed Fix: [approach]
```

**Ask user to confirm before proceeding to Phase 2.**

## Phase 2: Implementation

After user confirms the analysis:

1. Implement the fix following GLPI patterns
2. Reference `_knowledge/glpi-architecture.md` for patterns
3. Reference `_knowledge/glpi-conventions.md` for standards
4. Keep changes minimal and focused

**Show the diff and ask user to confirm before Phase 3.**

## Phase 3: Code Review (code-reviewer)

Review the implementation:

1. Verify GLPI-native patterns are used
2. Check naming conventions
3. Verify no anti-patterns introduced
4. Check for edge cases

**Output review summary:**
```
## Code Review
### Assessment: [APPROVED/NEEDS CHANGES]
### Issues: [list if any]
### Recommendations: [list]
```

**If NEEDS CHANGES, go back to Phase 2.**

## Phase 4: Test Writing (test-writer)

Write tests for the fix:

1. Search existing tests for patterns
2. Write a regression test that:
   - Recreates the bug conditions
   - Asserts correct behavior
   - Would fail if bug is reintroduced

**Output the test file location and summary.**

## Final Summary

Provide a summary:
```
## Bug Fix Complete
- Issue: [description]
- Root cause: [file:line]
- Fix: [files modified]
- Tests: [test file]
- Ready for: `make lint` then commit
```
