---
name: glpi-test
description: Write tests for GLPI code — PHPUnit (DbTestCase) or Playwright E2E.
argument-hint: "[e2e|unit] <class-or-method-to-test>"
disable-model-invocation: true
context: fork
agent: glpi-test-writer
background: false
---

# GLPI Test Writing

Write the test for the target below, into the working checkout.

## Target

$ARGUMENTS

Parse the prefix yourself:

- `e2e <target>` → Playwright E2E spec
- `unit <target>` → PHPUnit test
- `<target>` alone → auto-detect from the project structure (`tests/e2e/` present → Playwright, otherwise PHPUnit)

If the target above is empty, or if you cannot tell which class or method it names, stop and report what you need. Do not write a speculative test.

## What to produce

Locate the target, find the existing tests closest to it, and replicate their patterns rather than inventing any. Check for an existing data provider covering the same method before adding a parallel one.

Write the file at the location the environment overlay specifies, then report in your standard output format — including the `make` command to run it. Do not run it yourself.
