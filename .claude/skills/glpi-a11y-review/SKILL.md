---
name: glpi-a11y-review
description: Read-only RGAA 4.1 / WCAG AA accessibility audit on existing GLPI code.
argument-hint: "[path-or-empty-for-current-branch]"
disable-model-invocation: true
context: fork
background: false
agent: glpi-a11y-reviewer
---

# GLPI A11y Review

Audit the scope below for accessibility and produce your complete report.

## Scope

$ARGUMENTS

If the scope above is empty, audit the frontend files changed on the current branch:

!`git diff main --name-only | grep -E '\.(twig|js|ts|scss|css|php)$' || echo "(no frontend files changed)"`

If both are empty, say so immediately and stop — you cannot ask from here.

## Report

For each file: identify forms, tables, images, JS components, CSS colors and landmarks, then apply the relevant RGAA criteria from the `glpi-a11y` skill.

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
