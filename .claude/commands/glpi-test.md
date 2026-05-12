---
description: Write tests for GLPI code (PHPUnit or Playwright e2e)
argument-hint: "[e2e|unit] <class-or-method-to-test>"
allowed-tools: Agent, AskUserQuestion
---

# GLPI Test Writing

Delegate to the `glpi-test-writer` agent.

## Input

Arguments: $ARGUMENTS

## Execution

Use the Agent tool with:

- **subagent_type**: `glpi-test-writer`
- **description**: Write GLPI tests
- **prompt**: Pass `$ARGUMENTS` verbatim. The agent parses the prefix itself:
  - `e2e <target>` → Playwright E2E spec
  - `unit <target>` → PHPUnit test
  - `<target>` alone → auto-detect from project structure
  If `$ARGUMENTS` is empty, use `AskUserQuestion` to ask for the class/method (and optionally the `e2e`/`unit` prefix), then forward to the agent.

The agent locates the target, finds existing similar tests, replicates project patterns (DbTestCase + createItem for PHPUnit; glpi_fixture + api.createItem + page objects + role-based locators for Playwright), writes the test at the correct location, and prints the run command.

Source of truth: `.claude/agents/test-writer.md` and the `glpi-testing` skill — do not duplicate logic here.
