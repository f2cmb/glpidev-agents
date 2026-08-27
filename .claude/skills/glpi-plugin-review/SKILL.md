---
name: glpi-plugin-review
description: Audit a GLPI plugin for security vulnerabilities (23 checks) and GLPI 11 structural conformance.
argument-hint: <path/to/plugin/>
disable-model-invocation: true
context: fork
background: false
agent: glpi-plugin-reviewer
---

# GLPI Plugin Review

Audit the plugin below for security and GLPI 11 conformance, and produce your complete normalized report.

## Target

$ARGUMENTS

If the target above is empty, say so immediately and stop — you cannot guess which plugin to audit, and you cannot ask from here.

## Scope

Run your four phases in order: plugin mapping, then the S1–S23 security audit, then structural conformance, then contradiction detection.

Two things make a report incomplete, and both are on you:

- **The companion files.** Load `checks.md` and `audit-commands.md` from the `glpi-plugin-security` skill before grading any check. An audit run from the summary table alone is not reproducible.
- **The "Checks Without Findings" table.** All 23 checks appear, each with an explicit `Non applicable`, `Aucun`, or a `→ S-00x` reference. A check you skipped silently is indistinguishable from a check that passed.

Establish the GLPI target version in Phase 1 — it changes the severity of several checks on its own.

Report language: French for severity labels, descriptions and risks; technical patterns and file paths stay as they are.
