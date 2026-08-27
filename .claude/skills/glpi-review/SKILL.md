---
name: glpi-review
description: Review GLPI code changes for convention and pattern compliance before committing.
argument-hint: "[files-or-empty-for-staged]"
disable-model-invocation: true
allowed-tools: Agent, Bash(git diff:*), Bash(git status:*)
---

# GLPI Code Review

## Target

$ARGUMENTS

If empty, the review target is the staged changes.

## Step 1 — Resolve the scope yourself

`glpi-code-reviewer` has no shell: it reads and greps, it cannot run `git`. So the diff is yours to produce before delegating.

- Target given → that file list is the scope.
- Target empty → `git diff --cached --name-only` for the list, then `git diff --cached` for the content. If nothing is staged, say so and stop; do not silently fall back to the working tree or to `HEAD`, which reviews something the user did not ask about.

## Step 2 — Delegate

Use the `Agent` tool with `subagent_type: glpi-code-reviewer`, and put **both** in the prompt:

- the list of changed files, so the agent knows the scope;
- the diff itself, so the agent reviews *the change* rather than the current state of each file. Several of its checks — no scope creep, no PHPDoc on untouched code, does the fix cover every equivalent path — are only decidable from a diff.

This step runs in a subagent on purpose: a review that fills the main conversation with diff reading leaves no room for the discussion that follows.

## Step 3 — Report, then stop

When the agent returns:

1. **Present its report verbatim.** Do not summarize, paraphrase, reorder or drop findings. The severities are the agent's to assign, not yours to soften.
2. **Stop.** Call no `Edit`, `Write` or `NotebookEdit`, and run no Bash command that mutates a file (`sed -i`, `awk -i inplace`, `>`, `>>`). The agent's `Fix:` entries are recommendations, not a work queue.
3. **Ask which findings to act on**, renumbering from the report's `Issues Found`:

   > Quels findings veux-tu corriger ? Réponds par numéro(s), ou « aucun » pour clore.
   > 1. [titre + sévérité du finding 1]
   > 2. [titre + sévérité du finding 2]
   > …

4. **Wait.** Apply only the findings the user names. Never bundle neighbouring findings into a confirmed one, and never continue to the next finding without a fresh confirmation.

The point of this whole step is that a review and a refactor are two decisions. Merging them takes the second one away from the user.
