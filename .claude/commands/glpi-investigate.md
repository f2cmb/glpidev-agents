---
description: Investigate a GLPI bug without fixing it
argument-hint: <issue-url-or-description>
allowed-tools: Agent, AskUserQuestion
---

# GLPI Bug Investigation

Delegate to the `glpi-bug-investigator` agent.

## Input

Bug to investigate: $ARGUMENTS

## Execution

Use the Agent tool with:

- **subagent_type**: `glpi-bug-investigator`
- **description**: Investigate a GLPI bug
- **prompt**: Pass `$ARGUMENTS` verbatim as the bug target (issue URL or description). If empty, first use `AskUserQuestion` to ask the user for the issue URL or bug description, then forward the answer to the agent.

The agent gathers context (GitHub issue if URL given, otherwise the description), maps affected components, traces the execution path through entry points, controllers, hooks, DB and templates, compares with working code, and produces a structured bug-scenario report with root-cause analysis, proposed fix strategy, and verification needs. No source files are modified.

Source of truth: `.claude/agents/bug-investigator.md` — do not duplicate logic here.
