---
description: Complete bug fix workflow - investigate, fix, review, test
argument-hint: <issue-url-or-description>
allowed-tools: Glob, Grep, Read, Edit, Write, Bash, WebFetch, WebSearch, Task, AskUserQuestion
---

# GLPI Bug Fix Workflow

You are orchestrating a complete bug fix workflow for GLPI. Follow these phases sequentially.

## Input
Bug to fix: $ARGUMENTS

## Phase 1: Investigation

Use the Task tool to delegate investigation to the bug-investigator agent:
- **subagent_type**: `glpi-bug-investigator`
- **description**: Investigate GLPI bug
- **prompt**: Include the bug description/URL from $ARGUMENTS. Ask the agent to produce a bug scenario with root cause, execution path, and proposed fix approach.

DO NOT use Bash commands to invoke agents. Use the Task tool exclusively.

**Ask user to confirm the analysis before proceeding to Phase 2.**

## Phase 2: Implementation

After user confirms the analysis:

1. Implement the fix following GLPI patterns (architecture, conventions from preloaded skills)
2. Keep changes minimal and focused

**Show the diff and ask user to confirm before Phase 3.**

## Phase 3: Code Review

Use the Task tool to delegate review to the code-reviewer agent:
- **subagent_type**: `glpi-code-reviewer`
- **description**: Review GLPI bug fix
- **prompt**: Include the list of modified files. Ask the agent to verify GLPI-native patterns, naming conventions, anti-patterns, and edge cases. Expect a review summary with APPROVED/NEEDS CHANGES assessment.

DO NOT use Bash commands to invoke agents. Use the Task tool exclusively.

**If NEEDS CHANGES, go back to Phase 2.**

## Phase 4: Test Writing

Use the Task tool to delegate test creation to the test-writer agent:
- **subagent_type**: `glpi-test-writer`
- **description**: Write regression test for bug fix
- **prompt**: Include the root cause, fix details, and modified files. Ask the agent to write a regression test that recreates the bug conditions using **raw user inputs** (not pre-normalized state), asserts correct behavior, and would fail if the bug is reintroduced.

DO NOT use Bash commands to invoke agents. Use the Task tool exclusively.

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
