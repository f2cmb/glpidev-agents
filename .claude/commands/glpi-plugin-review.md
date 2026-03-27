---
description: Audit a GLPI plugin for security vulnerabilities and GLPI 11 conformance
argument-hint: <path/to/plugin/>
allowed-tools: Glob, Grep, Read, Bash, AskUserQuestion
---

# GLPI Plugin Review

Audit the plugin at the given path for security issues and GLPI 11 conformance.

## Input

Plugin path: $ARGUMENTS

## Execution

Use the `glpi-plugin-reviewer` agent to perform the audit.

The agent will:
1. Map the plugin structure
2. Audit all `front/` and `ajax/` entry points for authentication and CSRF
3. Scan for SQL injection, XSS, file upload, and SSRF vulnerabilities
4. Check GLPI 11 structural conformance
5. Flag any contradictions between security and structural findings
6. Produce a normalized report with CRITIQUE / MAJEUR / MINEUR findings

## Output

The agent produces a complete audit report including:
- Per-finding severity, file, line, risk description, and remediation
- Findings numbered S-001… (security) and P-001… (patterns)
- Explicit contradiction section
- Token budget summary at the end
