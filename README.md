# GLPI Development Agents

A suite of AI agents to contribute more efficiently to GLPI (core and plugins).

**Compatible with:** Claude Code, GitHub Copilot, Cursor, and other AI tools.

## Quick Start

Choose your AI tool:

| Tool | Setup |
|------|-------|
| **Claude Code** | Use agents from `agents/` |
| **GitHub Copilot** | Copy `copilot/` to your project's `.github/` |
| **Cursor** | Copy `cursor/rules/` to your project's `.cursor/rules/` |
| **Other tools** | Use `_knowledge/` files as context |

## Structure

```
glpidev-agents/
├── agents/                 # Claude Code agents
│   ├── bug-investigator.md
│   ├── code-reviewer.md
│   ├── php-mentor.md
│   └── test-writer.md
│
├── copilot/                # GitHub Copilot instructions
│   ├── copilot-instructions.md
│   └── instructions/
│       ├── glpi-core.instructions.md
│       └── glpi-plugin.instructions.md
│
├── cursor/                 # Cursor rules
│   └── rules/
│       ├── glpi-core.mdc
│       └── glpi-plugin.mdc
│
├── _contexts/              # Environment overlays (universal)
│   ├── core-10.md
│   ├── core-11.md
│   └── plugin.md
│
└── _knowledge/             # GLPI knowledge base (universal)
    ├── glpi-architecture.md
    ├── glpi-conventions.md
    ├── glpi-plugin-patterns.md
    └── glpi-testing.md
```

---

## Usage by Tool

### Claude Code

```bash
# Use an agent directly
claude --agent /path/to/glpidev-agents/agents/bug-investigator.md

# Or copy to your project
cp -r agents/ /your/glpi/project/.claude/agents/
```

Specify context in your prompt:
```
"Investigate issue #12345. Context: GLPI 11 core"
```

### GitHub Copilot

1. Copy files to your GLPI project:
```bash
# For core development
cp copilot/copilot-instructions.md /your/glpi/project/.github/
cp copilot/instructions/glpi-core.instructions.md /your/glpi/project/.github/instructions/

# For plugin development
cp copilot/copilot-instructions.md /your/plugin/project/.github/
cp copilot/instructions/glpi-plugin.instructions.md /your/plugin/project/.github/instructions/
```

2. Copilot will automatically apply rules based on file paths (`applyTo` patterns).

### Cursor

1. Copy rules to your project:
```bash
# For core development
cp cursor/rules/glpi-core.mdc /your/glpi/project/.cursor/rules/

# For plugin development
cp cursor/rules/glpi-plugin.mdc /your/plugin/project/.cursor/rules/
```

2. Cursor will automatically apply rules based on glob patterns.

### Other AI Tools

Use the universal `_knowledge/` files as context:

1. Copy relevant files to your project or paste content into your AI tool
2. Reference them in your prompts

```
"Review this code following the conventions in glpi-conventions.md"
```

---

## Agents (Claude Code)

| Agent | Purpose | Model |
|-------|---------|-------|
| `bug-investigator` | Analyze bugs, identify root causes, build resolution plans | opus |
| `code-reviewer` | Review code changes before commit | opus |
| `php-mentor` | Explain PHP concepts and GLPI patterns | sonnet |
| `test-writer` | Write PHPUnit/Cypress tests | sonnet |

## Contexts

| Context | When to use |
|---------|-------------|
| `core-10` | Contributing to GLPI 10.0.x |
| `core-11` | Contributing to GLPI 11.0.x / main |
| `plugin` | Developing a GLPI 11 plugin |

## Knowledge Base

| File | Content |
|------|---------|
| `glpi-architecture.md` | CommonDBTM hooks, DB layer, Session, helpers |
| `glpi-conventions.md` | Naming standards, anti-patterns, bug patterns |
| `glpi-plugin-patterns.md` | GLPI 11 plugin structure, Hooks::*, install() |
| `glpi-testing.md` | DbTestCase helpers, PHPUnit/Cypress patterns |

---

## Customization

### Adding tool-specific rules

- **Copilot**: Add `.instructions.md` files in `copilot/instructions/`
- **Cursor**: Add `.mdc` files in `cursor/rules/`
- **Claude**: Add `.md` files in `agents/`

### Extending knowledge

Add or modify files in `_knowledge/` - these are universal and can be referenced by any tool.

---

## License

MIT License - See [LICENSE](LICENSE)
