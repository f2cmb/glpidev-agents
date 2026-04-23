---
description: RGAA 4.1 accessibility audit on existing GLPI code
argument-hint: [path-or-empty-for-current-branch]
allowed-tools: Glob, Grep, Read, Bash
---

# GLPI A11y Audit

Read-only accessibility audit.

## Input
Scope to audit: $ARGUMENTS
(If empty: files changed on the current branch via `git diff main --name-only`)

## Step 1: Determine the scope

If `$ARGUMENTS` is provided (non-empty): use that path as the scope.
If `$ARGUMENTS` is absent or empty: retrieve the frontend files changed on the current branch:

```bash
git diff main --name-only | grep -E '\.(twig|js|ts|scss|css|php)$'
```

## Step 2: Scan the files

For each file in scope:
- Read the content
- Identify: forms, tables, images, JS components, CSS colors, landmarks
- Apply the relevant RGAA criteria (glpi-a11y skill)

## Step 3: Report

```markdown
## A11y Audit — [scope]

### Summary
[N] violations — [X] critical, [Y] major, [Z] minor

### Violations

#### [RGAA X.X] Title
Severity: Critical | Major | Minor
File: path:line
Issue: …
Fix:
[corrected code]
```

> Static audit only — test with NVDA/VoiceOver and the Focus Order / Structure Revealer bookmarklets (a11y-tools.com/bookmarklets/).
