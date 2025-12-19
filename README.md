# GLPI Development Agents

A suite of AI agents to contribute more efficiently to GLPI (core and plugins).

**Compatible with:** Claude Code, GitHub Copilot, Cursor, Google Antigravity, and other AI tools.

## Quick Start


| Tool | Agents | Instructions | Setup |
|------|--------|--------------|-------|
| **Claude Code** | `agents/*.md` | `_contexts/` | `claude --agent path/to/agent.md` |
| **GitHub Copilot** | `copilot/agents/*.md` | `copilot/instructions/` | Copy to `.github/` |
| **Cursor** | `cursor/agents/*.chatmode.md` | `cursor/rules/` | Copy to `.cursor/` |
| **Antigravity** | `antigravity/workflows/*.md` | `antigravity/rules/` | Copy to `.agent/` |

## Structure

```
glpidev-agents/
├── agents/                         # Claude Code agents
│   ├── bug-investigator.md
│   ├── code-reviewer.md
│   ├── php-mentor.md
│   └── test-writer.md
│
├── commands/                       # Claude Code slash commands
│   ├── glpi-fix-bug.md             # Full bug fix cycle
│   ├── glpi-investigate.md         # Bug investigation only
│   ├── glpi-review.md              # Code review
│   ├── glpi-test.md                # Write tests
│   └── glpi-learn.md               # Learn concepts
│
├── copilot/                        # GitHub Copilot
│   ├── agents/
│   ├── instructions/
│   └── copilot-instructions.md
│
├── cursor/                         # Cursor
│   ├── agents/
│   └── rules/
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

## Workflows (Claude Code)

Workflows orchestrate multiple agents in sequence. Install them as slash commands.

### Installation

```bash
# Copy to your GLPI project
cp -r commands/ /your/glpi/project/.claude/commands/
```

### Available Workflows

| Command | Description | Agents Used |
|---------|-------------|-------------|
| `/glpi-fix-bug <issue>` | Complete bug fix cycle | investigate → fix → review → test |
| `/glpi-investigate <issue>` | Investigate without fixing | bug-investigator |
| `/glpi-review [files]` | Review code changes | code-reviewer |
| `/glpi-test <class>` | Write tests | test-writer |
| `/glpi-learn <concept>` | Explain concepts | php-mentor |

### Example Usage

```bash
# Full bug fix workflow
/glpi-fix-bug https://github.com/glpi-project/glpi/issues/12345

# Investigate only (no changes)
/glpi-investigate "Serial validation fails on template creation"

# Review staged changes
/glpi-review

# Review specific files
/glpi-review src/Computer.php src/Item.php

# Write tests for a class
/glpi-test Computer::prepareInputForAdd

# Learn about a concept
/glpi-learn "CommonDBTM hooks"
```

### Workflow: `/glpi-fix-bug`

```
┌─────────────────┐
│ Phase 1:        │
│ Investigation   │──→ Bug scenario + root cause
└────────┬────────┘
         │ (user confirms)
         ▼
┌─────────────────┐
│ Phase 2:        │
│ Implementation  │──→ Code changes
└────────┬────────┘
         │ (user confirms)
         ▼
┌─────────────────┐
│ Phase 3:        │
│ Code Review     │──→ APPROVED / NEEDS CHANGES
└────────┬────────┘
         │ (if approved)
         ▼
┌─────────────────┐
│ Phase 4:        │
│ Test Writing    │──→ Regression test
└────────┬────────┘
         │
         ▼
    Ready for commit
```

---

## Agents

All tools have the same 4 specialized agents:

| Agent | Purpose | Use when... |
|-------|---------|-------------|
| **bug-investigator** | Analyze bugs, trace code, identify root causes | Investigating a GitHub issue |
| **code-reviewer** | Review changes, check conventions | Before committing |
| **php-mentor** | Explain PHP/GLPI patterns | Learning concepts |
| **test-writer** | Write PHPUnit/Cypress tests | Adding test coverage |

---

## Usage by Tool

### Claude Code

```bash
# Use workflows (recommended)
/glpi-fix-bug https://github.com/glpi-project/glpi/issues/12345

# Use agents directly
claude --agent /path/to/agents/bug-investigator.md

# Specify context in prompt
"Investigate issue #12345. Context: GLPI 11 core"
```

### GitHub Copilot

```bash
# Copy to project
cp -r copilot/agents/ /your/project/.github/agents/
cp copilot/copilot-instructions.md /your/project/.github/
cp -r copilot/instructions/ /your/project/.github/instructions/
```

Use in chat:
```
@glpi-bug-investigator investigate issue #12345
@glpi-code-reviewer review my changes
```

### Cursor

```bash
# Copy to project
cp -r cursor/agents/ /your/project/.cursor/agents/
cp -r cursor/rules/ /your/project/.cursor/rules/
```

Switch mode: `/mode glpi-bug-investigator`

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

### Adding workflows (Claude Code)

Create `.md` files in `commands/`:

```markdown
---
description: My custom workflow
argument-hint: <arg>
allowed-tools: Read, Grep, Edit
---

Instructions for the workflow...
Use $ARGUMENTS for user input.
```

### Adding agents

| Tool | Location | Format |
|------|----------|--------|
| Claude | `agents/` | `.md` with YAML frontmatter |
| Copilot | `copilot/agents/` | `.md` with YAML frontmatter |
| Cursor | `cursor/agents/` | `.chatmode.md` with YAML frontmatter |
| Antigravity | `antigravity/workflows/` | `.md` with `description:` frontmatter |

### Adding rules

| Tool | Location | Format |
|------|----------|--------|
| Copilot | `copilot/instructions/` | `.instructions.md` with `applyTo:` |
| Cursor | `cursor/rules/` | `.mdc` with `globs:` |
| Antigravity | `antigravity/rules/` | `.md` (plain markdown) |

---

## License

MIT License - See [LICENSE](LICENSE)
