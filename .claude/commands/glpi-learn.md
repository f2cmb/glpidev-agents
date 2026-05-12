---
description: Produce a structured French learning document about a GLPI subject or recent change
argument-hint: <subject-or-debrief> [#issue PR <url>] dans <output-path>/
allowed-tools: Agent, AskUserQuestion
---

# GLPI Learn

Delegate to the `glpi-mentor` agent (which itself follows the `glpi-learn` skill).

## Input

User invocation: $ARGUMENTS

## Execution

Use the Agent tool with:

- **subagent_type**: `glpi-mentor`
- **description**: Produce a French GLPI learning document
- **prompt**: Pass `$ARGUMENTS` verbatim. Two supported modes:
  - Standalone — `<subject> dans <path>/`
  - Debrief — `debrief #<issue> PR <url> dans <path>/`
  If `$ARGUMENTS` is empty, use `AskUserQuestion` to collect (a) the subject or debrief reference and (b) the output path, then forward to the agent.

The agent detects mode and domains, reads matching `.claude/skills/glpi-learn/references/*.md`, anchors every claim to a real GLPI `file:line`, and writes a single dated French Markdown document via `Write`.

Source of truth: `.claude/agents/glpi-mentor.md` and `.claude/skills/glpi-learn/` — do not duplicate logic here.
