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

## After the agent returns — MANDATORY

1. Present the agent's report verbatim. Do NOT summarize, paraphrase, or skip findings.
2. STOP. Do NOT call Edit, Write, NotebookEdit, or any other modification tool. Do NOT run Bash commands that mutate files (`sed -i`, `awk -i inplace`, `>`, `>>`, etc.). The agent's `Fix:` entries are recommendations, not instructions to execute.
3. Echo this exact prompt to the user (renumber from the report's `Issues Found`):
   > Quels findings veux-tu corriger ? Réponds par numéro(s), ou « aucun » pour clore.
   > 1. [titre + sévérité du finding 1]
   > 2. [titre + sévérité du finding 2]
   > ...
4. Wait for the user's reply. Apply ONLY the specific finding(s) the user confirms — never bundle, never act on un-confirmed items, never enchaîner sur d'autres findings sans nouvelle confirmation.

Source of truth: `.claude/agents/code-reviewer.md` — do not duplicate logic here.
