---
description: Review GLPI code changes before commit
argument-hint: "[files-or-empty-for-staged]"
allowed-tools: Agent, Bash
---

# GLPI Code Review

Delegate to the `glpi-code-reviewer` agent.

## Input

Files to review: $ARGUMENTS
(If empty, the agent reviews staged changes via `git diff --cached`.)

## Execution

Use the Agent tool with:

- **subagent_type**: `glpi-code-reviewer`
- **description**: Review GLPI code changes
- **prompt**: If `$ARGUMENTS` is non-empty, pass the file list verbatim as the review target. If empty, instruct the agent to gather staged changes itself using `git diff --cached --name-only` and `git diff --cached`, then review those.

The agent checks GLPI convention compliance (naming, CommonDBTM hooks, DB abstraction, templates, rights, logging), flags anti-patterns (service classes, DI, raw SQL, hardcoded IDs, var_dump, string-literal itemtypes, etc.), compares against GLPI core patterns, and produces a verdict (APPROVED / NEEDS CHANGES / REJECTED) with per-issue severity, location, GLPI reference, and fix.

Source of truth: `.claude/agents/code-reviewer.md` — do not duplicate logic here.
