---
name: glpi-conventions
description: GLPI naming conventions and code-quality standards — table prefixing (glpi_*, plural, snake_case), foreign keys ({itemtype}s_id), class PascalCase / method camelCase / variable+array-key snake_case / constant UPPER_SNAKE / itemtype ClassName::class, file layout (src/ + front/ + ajax/ + templates/ + install/migrations/ + tests/), method ordering inside classes, and a catalogue of architecture/code anti-patterns (no service classes, no DI, no repositories, no DTOs, no event dispatchers, no raw SQL, no var_dump/echo, no hardcoded IDs, no canUpdateItem for access control, no string-literal itemtypes, no camelCase keys), with common bug-pattern signatures (data corruption, permission bypass, ITIL false analogy). Use when verifying that code follows GLPI conventions or when investigating recurring bug shapes.
user-invocable: false
---

# GLPI Conventions & Standards

Coding standards and conventions for GLPI development. Sections live in `references/`.

## Sections

| Topic | Reference |
|---|---|
| Naming, file layout, method ordering | [`references/naming.md`](references/naming.md) |
| Architecture & code anti-patterns + common bug signatures | [`references/anti-patterns.md`](references/anti-patterns.md) |
| Code quality (`make lint`, PHPDoc) | [`references/quality.md`](references/quality.md) |

## Critical Rules

1. **GLPI-native only**: Follow existing patterns, never "improve" with external patterns
2. **Simplicity first**: Simplest working solution is best
3. **Evidence-based**: Reference existing GLPI code for patterns
4. **Minimal scope**: Change only what's necessary
5. **No assumptions**: Verify patterns in codebase before applying
