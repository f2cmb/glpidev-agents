# GLPI Development Agents

A suite of AI agents to contribute more efficiently to GLPI (core and plugins).

## Quick Start

1. Copy the agent you need from `agents/` to your AI tool
2. Include the appropriate context from `_contexts/`
3. Reference `_knowledge/` files as needed

## Agents

| Agent | Purpose | Model |
|-------|---------|-------|
| `bug-investigator` | Analyze bugs, identify root causes, build resolution plans | opus |
| `code-reviewer` | Review code changes before commit | opus |
| `php-mentor` | Explain PHP concepts and GLPI patterns | sonnet |
| `test-writer` | Write PHPUnit/Cypress tests | sonnet |

## Structure

```
glpidev-agents/
├── agents/                 # AI agent definitions
│   ├── bug-investigator.md
│   ├── code-reviewer.md
│   ├── php-mentor.md
│   └── test-writer.md
│
├── _contexts/              # Environment-specific overlays
│   ├── core-10.md          # GLPI 10 core development
│   ├── core-11.md          # GLPI 11 core development
│   └── plugin.md           # GLPI 11 plugin development
│
└── _knowledge/             # Shared GLPI knowledge base
    ├── glpi-architecture.md
    ├── glpi-conventions.md
    ├── glpi-plugin-patterns.md
    └── glpi-testing.md
```

## Usage

### With Claude Code

```bash
# In your GLPI project directory
claude --agent /path/to/glpidev-agents/agents/bug-investigator.md

# Or copy agent content to your project's .claude/ directory
```

### With Other AI Tools (Copilot, Cursor, etc.)

The `_knowledge/` and `_contexts/` files are pure Markdown and can be used as context with any AI tool:

1. Copy relevant files to your project
2. Include them in your AI tool's context/instructions
3. Reference them in your prompts

### Example Workflow

**Investigating a bug in GLPI 11 core:**

```
Prompt: "Investigate issue #12345. Context: GLPI 11 core"

The agent will:
1. Read _contexts/core-11.md for environment specifics
2. Consult _knowledge/glpi-architecture.md for patterns
3. Follow investigation methodology
4. Produce a bug scenario with file:line references
```

**Reviewing plugin code:**

```
Prompt: "Review my changes to SyncFilter.php. Context: plugin"

The agent will:
1. Read _contexts/plugin.md for plugin patterns
2. Check against _knowledge/glpi-conventions.md
3. Compare with GLPI core patterns
4. Produce a structured review
```

## Contexts

Choose the context matching your working environment:

| Context | When to use |
|---------|-------------|
| `core-10` | Contributing to GLPI 10.0.x branch |
| `core-11` | Contributing to GLPI 11.0.x / main branch |
| `plugin` | Developing a GLPI 11 plugin |

## Knowledge Base

The `_knowledge/` folder contains reusable GLPI documentation:

| File | Content |
|------|---------|
| `glpi-architecture.md` | CommonDBTM hooks, DB layer, Session, helpers |
| `glpi-conventions.md` | Naming standards, anti-patterns, bug patterns |
| `glpi-plugin-patterns.md` | GLPI 11 plugin structure, Hooks::*, install() |
| `glpi-testing.md` | DbTestCase helpers, PHPUnit/Cypress patterns |

These files can be used standalone or combined with agents.

## Customization

### Adding a new context

Create a file in `_contexts/` with:
- Environment details (GLPI version, PHP version)
- Relevant paths
- Version-specific notes

### Extending knowledge

Add or modify files in `_knowledge/` for:
- New GLPI patterns
- Project-specific conventions
- Additional references

## License

MIT License - See [LICENSE](LICENSE)
