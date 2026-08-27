---
name: glpi-investigate
description: Investigate a GLPI bug without fixing it — root cause analysis and a resolution plan, read-only.
argument-hint: <issue-url-or-description>
disable-model-invocation: true
context: fork
background: false
agent: glpi-bug-investigator
---

# GLPI Bug Investigation

Investigate this GLPI bug and report. **Modify no source file** — this is a read-only investigation.

## Target

$ARGUMENTS

If the target above is empty, say so immediately and stop: you cannot investigate without an issue URL or a bug description, and you cannot ask for one from here.

If it is a GitHub issue URL, fetch the issue first and work from its content — error messages, reproduction steps, reported version, affected itemtype. If it is a prose description, work from that.

## What to produce

Follow your own methodology end to end: gather context, map the affected components, trace the execution path from entry point through controllers, hooks, DB and templates, compare against structurally equivalent working code, then construct the bug scenario and the resolution plan.

Report in your standard output format. Anchor every claim to a `file:line`. Where a conclusion depends on information the issue does not give, say which information and what it would change — do not fill the gap with a guess.
