---
description: Challenge GLPI code, plans, and decisions before they ship
argument-hint: "[code|plan|files] or empty (= asked)"
allowed-tools: Agent, Glob, Grep, Read, Bash, AskUserQuestion
---

# GLPI Devil's Advocate

Delegate to the `glpi-devils-advocate` agent.

## Input

Target to challenge: $ARGUMENTS

## Execution

Use the Agent tool with:

- **subagent_type**: `glpi-devils-advocate`
- **description**: Challenge GLPI output
- **prompt**: Pass `$ARGUMENTS` verbatim as the target. If empty, the agent will prompt the user to pick between code / migration / plan / output of another `/glpi-*` command.

The agent applies the methodology defined in the `glpi-devils-advocate` skill — pre-mortem, inversion, Socratic questioning, GLPI-specific blind spots (entity scoping, rights, migrations, hooks, ITIL divergence, search options, inventory conflicts) — and produces a verdict (Ship it / Ship with changes / Rethink) with up to 7 ranked, actionable concerns.

The skill is the single source of truth for the methodology. Do not duplicate it here.
