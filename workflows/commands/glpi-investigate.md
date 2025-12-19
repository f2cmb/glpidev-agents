---
description: Investigate a GLPI bug without fixing it
argument-hint: <issue-url-or-description>
allowed-tools: Glob, Grep, Read, WebFetch, WebSearch
---

# GLPI Bug Investigation Workflow

Investigate a bug to understand root cause without making changes.

## Input
Bug to investigate: $ARGUMENTS

## Phase 1: Context Gathering

### From GitHub Issue (if URL provided)
Fetch and extract:
- Error messages / stack traces
- Steps to reproduce
- Expected vs actual behavior
- GLPI version affected
- User comments and hints

### From Description
Identify:
- Affected itemtype/feature area
- Frontend (JS/Twig) or backend (PHP)
- Potential entry points

## Phase 2: Codebase Analysis

### Map Components
```bash
# Find related files
find src/ -name "*.php" | xargs grep -l "{keyword}" | head -10

# Trace inheritance
grep -rn "extends CommonDBTM" src/ | grep "{Class}"
```

### Trace Execution Path
1. Entry point (form submission, AJAX call, etc.)
2. Controller/front file
3. Class method called
4. Hook chain (prepareInput*, post_*, etc.)
5. Database operations
6. Template rendering

### Compare with Working Code
Find similar functionality that works:
- Same pattern in different class
- Recent changes that might have broken it

## Phase 3: Bug Scenario

Output detailed scenario:

```markdown
## Bug Analysis: {Issue #/Title}

### Summary
[2-3 sentence description]

### Trigger Conditions
- User role/permissions: [specific roles]
- Data state: [prerequisites]
- Action sequence: [steps]
- Environment: [config, multi-entity, etc.]

### Execution Path
```
{EntryPoint}
└─ src/{File}.php:{line} → {method}()
   └─ src/{Parent}.php:{line} → {parentMethod}()
      └─ ROOT CAUSE: {description}
```

### Affected Components
**Primary:** `src/{Class}.php:{line}`
- Method: `{method}()`
- Issue: {specific problem}

**Secondary:**
- `templates/{file}.twig:{line}` - {issue}
- `ajax/{file}.php:{line}` - {issue}

### Root Cause Analysis
{Detailed explanation of why the bug occurs}

### Proposed Fix Strategy
1. {approach}
2. {considerations}
3. {side effects to check}

### Verification Needs
- [ ] Test scenario 1
- [ ] Test scenario 2
- [ ] Regression areas to check
```

## Phase 4: Questions

If anything is unclear, list questions:
- Reproduction steps clarification
- Edge cases to consider
- Priority of different approaches
