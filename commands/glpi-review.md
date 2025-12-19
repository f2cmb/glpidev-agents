---
description: Review GLPI code changes before commit
argument-hint: [files-or-empty-for-staged]
allowed-tools: Glob, Grep, Read, Bash, WebSearch
---

# GLPI Code Review Workflow

Review code changes for GLPI compliance.

## Input
Files to review: $ARGUMENTS
(If empty, review staged changes via `git diff --cached`)

## Step 1: Gather Changes

```bash
# If no arguments, get staged changes
git diff --cached --name-only
git diff --cached
```

## Step 2: Analyze Each File

For each modified file:

### Convention Compliance

| Check | Status |
|-------|--------|
| Naming (tables, fields, classes) | ✅/❌ |
| CommonDBTM hooks usage | ✅/❌ |
| Database abstraction (no raw SQL) | ✅/❌ |
| Template patterns | ✅/❌ |
| Rights handling | ✅/❌ |
| Logging (Toolbox::logDebug) | ✅/❌ |

### Anti-Patterns Check

Flag if found:
- Service classes, DI, repositories
- Raw SQL queries
- Hardcoded IDs
- Bypassing hooks
- var_dump/print_r

### GLPI Pattern Comparison

Search codebase for similar implementations:
- How does GLPI core handle this?
- Reference specific files

## Step 3: Output Review

```markdown
## Code Review Summary

### Overall: [APPROVED / NEEDS CHANGES / REJECTED]

### Files Reviewed
- `file1.php` - [status]
- `file2.php` - [status]

### Issues Found
1. **[Severity]** [Description]
   - Location: `file:line`
   - GLPI Pattern: [reference]
   - Fix: [suggestion]

### Recommendations
1. [recommendation]

### Next Steps
- [ ] Address issues above
- [ ] Run `make lint`
- [ ] Ready to commit
```
