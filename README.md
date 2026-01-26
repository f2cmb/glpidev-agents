# GLPI Development Agents

A suite of AI agents to contribute more efficiently to GLPI (core and plugins).

**Compatible with:** Claude Code, GitHub Copilot, Cursor, Google Antigravity, and other AI tools.

## Quick Start

| Tool | Agents | Commands | Setup |
|------|--------|----------|-------|
| **Claude Code** | `agents/*.md` | `commands/*.md` | `claude --agent path/to/agent.md` or `/glpi-*` |
| **GitHub Copilot** | `copilot/agents/*.md` | — | Copy to `.github/` |
| **Cursor** | `cursor/agents/*.chatmode.md` | — | Copy to `.cursor/` |
| **Antigravity** | `antigravity/workflows/*.md` | — | Copy to `.agent/` |

## Structure

```
glpidev-agents/
├── agents/                         # Claude Code agents
│   ├── bug-investigator.md
│   ├── code-reviewer.md
│   ├── glpi-feature-builder.md    # Feature development session manager
│   ├── php-mentor.md
│   └── test-writer.md
│
├── commands/                       # Claude Code slash commands
│   ├── glpi-feature.md            # Start/finalize feature session from issue/PR
│   ├── glpi-fix-bug.md            # Full workflow: investigate → fix → review → test
│   ├── glpi-investigate.md        # Investigate bug without fixing
│   ├── glpi-review.md             # Review code before commit
│   ├── glpi-test.md               # Write tests for code
│   └── glpi-learn.md              # Explain PHP/GLPI concepts
│
├── copilot/                        # GitHub Copilot
│   ├── agents/                     # Specialized agents
│   │   ├── bug-investigator.md
│   │   ├── code-reviewer.md
│   │   ├── php-mentor.md
│   │   └── test-writer.md
│   ├── instructions/               # Path-based rules
│   │   ├── glpi-core.instructions.md
│   │   └── glpi-plugin.instructions.md
│   └── copilot-instructions.md     # Global instructions
│
├── cursor/                         # Cursor
│   ├── agents/                     # Specialized agents
│   │   ├── bug-investigator.chatmode.md
│   │   ├── code-reviewer.chatmode.md
│   │   ├── php-mentor.chatmode.md
│   │   └── test-writer.chatmode.md
│   └── rules/                      # Path-based rules
│       ├── glpi-core.mdc
│       └── glpi-plugin.mdc
│
├── antigravity/                    # Google Antigravity
│   ├── workflows/                  # Specialized workflows
│   │   ├── glpi-bug-investigator.md
│   │   ├── glpi-code-reviewer.md
│   │   ├── glpi-php-mentor.md
│   │   └── glpi-test-writer.md
│   └── rules/                      # Project rules
│       ├── glpi-core.md
│       └── glpi-plugin.md
│
├── _contexts/                      # Universal overlays
│   ├── core-10.md
│   ├── core-11.md
│   └── plugin.md
│
└── _knowledge/                     # Universal knowledge base
    ├── glpi-architecture.md
    ├── glpi-conventions.md
    ├── glpi-plugin-patterns.md
    └── glpi-testing.md
```

---

## Agents

Common agents available across all tools:

| Agent | Purpose | Use when... |
|-------|---------|-------------|
| **bug-investigator** | Analyze bugs, trace code, identify root causes | Investigating a GitHub issue or unexpected behavior |
| **code-reviewer** | Review changes, check conventions | Before committing code |
| **php-mentor** | Explain PHP/GLPI patterns | Learning why code works a certain way |
| **test-writer** | Write PHPUnit/Playwright tests | Adding test coverage |

Claude Code exclusive:

| Agent | Purpose | Use when... |
|-------|---------|-------------|
| **feature-builder** | Manage feature development sessions | Starting work on a GitHub issue/PR, or finalizing a session |

---

## Usage by Tool

### Claude Code

**Using agents:**
```bash
# Start a session with an agent
claude --agent /path/to/glpidev-agents/agents/bug-investigator.md

# Specify context in prompt
"Investigate issue #12345. Context: GLPI 11 core"
```

**Using slash commands:**

Copy `commands/` folder to your project and use them directly:
```bash
/glpi-feature https://github.com/glpi-project/glpi/issues/12345
/glpi-feature https://github.com/glpi-project/glpi/pull/54321
/glpi-feature finalize          # End-of-session review
/glpi-fix-bug https://github.com/glpi-project/glpi/issues/12345
/glpi-investigate "Search not working on tickets"
/glpi-review                    # Review staged changes
/glpi-test Computer::prepareInputForAdd
/glpi-learn "CommonDBTM hooks"
```

| Command | Purpose |
|---------|---------|
| `/glpi-feature` | Start/finalize feature session from GitHub issue or PR |
| `/glpi-fix-bug` | Complete workflow: investigate → fix → review → test |
| `/glpi-investigate` | Investigate a bug without making changes |
| `/glpi-review` | Review code changes for GLPI compliance |
| `/glpi-test` | Write PHPUnit tests for a class/method |
| `/glpi-learn` | Explain PHP/GLPI patterns for learning |

### GitHub Copilot

1. **Copy to your project:**
```bash
# Agents (specialized assistants)
cp -r copilot/agents/ /your/project/.github/agents/

# Instructions (auto-applied by file path)
cp copilot/copilot-instructions.md /your/project/.github/
mkdir -p /your/project/.github/instructions/
cp copilot/instructions/glpi-core.instructions.md /your/project/.github/instructions/
```

2. **Use agents in chat:**
```
@glpi-bug-investigator investigate issue #12345
@glpi-code-reviewer review my changes
```

3. Instructions apply automatically based on `applyTo` patterns.

### Cursor

1. **Copy to your project:**
```bash
# Agents (chat modes)
cp -r cursor/agents/ /your/project/.cursor/agents/

# Rules (auto-applied by glob patterns)
mkdir -p /your/project/.cursor/rules/
cp cursor/rules/glpi-core.mdc /your/project/.cursor/rules/
```

2. **Switch agent in chat** using the mode selector or:
```
/mode glpi-bug-investigator
```

3. Rules apply automatically based on glob patterns.

### Google Antigravity

1. **Copy to your project:**
```bash
# Workflows (specialized agents)
cp -r antigravity/workflows/ /your/project/.agent/workflows/

# Rules (project-level rules)
mkdir -p /your/project/.agent/rules/
cp antigravity/rules/glpi-core.md /your/project/.agent/rules/
```

2. **Use workflows in chat:**
```
/glpi-bug-investigator
/glpi-code-reviewer
```

3. Rules apply automatically when files are opened.

### Other AI Tools

Use universal files as context:
- `_knowledge/*.md` - GLPI documentation
- `_contexts/*.md` - Environment specifics

---

## Contexts

| Context | When to use |
|---------|-------------|
| `core-10` | GLPI 10.0.x development |
| `core-11` | GLPI 11.0.x / main branch |
| `plugin` | GLPI 11 plugin development |

## Knowledge Base

| File | Content |
|------|---------|
| `glpi-architecture.md` | CommonDBTM hooks, DB layer, Session |
| `glpi-conventions.md` | Naming, anti-patterns, bug patterns |
| `glpi-plugin-patterns.md` | Plugin structure, Hooks::*, install() |
| `glpi-testing.md` | DbTestCase, PHPUnit, Cypress |

---

## Customization

### Adding agents

| Tool | Location | Format |
|------|----------|--------|
| Claude Code | `agents/` | `.md` with YAML frontmatter (`name`, `description`, `tools`, `model`) |
| Copilot | `copilot/agents/` | `.md` with YAML frontmatter |
| Cursor | `cursor/agents/` | `.chatmode.md` with YAML frontmatter |
| Antigravity | `antigravity/workflows/` | `.md` with `description:` frontmatter |

### Adding commands (Claude Code)

| Location | Format |
|----------|--------|
| `commands/` | `.md` with YAML frontmatter (`description`, `argument-hint`, `allowed-tools`) |

Use `$ARGUMENTS` placeholder in the command body to receive user input.

### Adding rules

| Tool | Location | Format |
|------|----------|--------|
| Copilot | `copilot/instructions/` | `.instructions.md` with `applyTo:` |
| Cursor | `cursor/rules/` | `.mdc` with `globs:` |
| Antigravity | `antigravity/rules/` | `.md` (plain markdown) |

---

## License

MIT License - See [LICENSE](LICENSE)
