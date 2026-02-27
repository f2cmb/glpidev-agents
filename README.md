# GLPI Development Agents

A suite of AI agents to contribute more efficiently to GLPI (core and plugins).

**Compatible with:** Claude Code, GitHub Copilot, Cursor, Google Antigravity, and other AI tools.

## Quick Start

| Tool | Agents | Commands | Skills/Rules | Setup |
|------|--------|----------|-------------|-------|
| **Claude Code** | `.claude/agents/*.md` | `.claude/commands/*.md` | `.claude/skills/`, `.claude/rules/` | `claude --agent .claude/agents/agent.md` or `/glpi-*` |
| **GitHub Copilot** | `.claude/copilot/agents/*.md` | — | `.claude/copilot/instructions/` | Copy to `.github/agents/` |
| **Cursor** | `.claude/cursor/agents/*.chatmode.md` | — | `.claude/cursor/rules/` | Copy to `.cursor/` |
| **Antigravity** | `.claude/antigravity/workflows/*.md` | — | `.claude/antigravity/rules/` | Copy to `.agent/` |

## Structure

```
glpidev-agents/
└── .claude/                            # Everything lives here
    ├── agents/                         # Claude Code agents
    │   ├── bug-investigator.md
    │   ├── code-reviewer.md
    │   ├── glpi-feature-builder.md
    │   ├── php-mentor.md
    │   └── test-writer.md
    │
    ├── commands/                       # Claude Code slash commands
    │   ├── glpi-feature.md
    │   ├── glpi-fix-bug.md
    │   ├── glpi-investigate.md
    │   ├── glpi-review.md
    │   ├── glpi-test.md
    │   └── glpi-learn.md
    │
    ├── skills/                         # Preloaded knowledge (injected into agents)
    │   ├── glpi-architecture/SKILL.md
    │   ├── glpi-conventions/SKILL.md
    │   ├── glpi-plugin-patterns/SKILL.md
    │   └── glpi-testing/SKILL.md
    │
    ├── rules/                          # Auto-applied rules by file type
    │   ├── php.md                      # Applied to **/*.php
    │   └── twig.md                     # Applied to **/*.twig
    │
    ├── _contexts/                      # Universal overlays
    │   ├── core-10.md
    │   ├── core-11.md
    │   └── plugin.md
    │
    ├── _knowledge/                     # Universal knowledge base (source of truth)
    │   ├── glpi-architecture.md
    │   ├── glpi-conventions.md
    │   ├── glpi-plugin-patterns.md
    │   └── glpi-testing.md
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
    └── antigravity/                    # Google Antigravity
        ├── workflows/
        └── rules/
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
claude --agent .claude/agents/bug-investigator.md

# Specify context in prompt
"Investigate issue #12345. Context: GLPI 11 core"
```

Agents use **skills** to preload GLPI knowledge automatically (no manual file reads needed) and **persistent memory** (`memory: project`) to retain learnings across sessions.

**Using slash commands:**

Copy `.claude/commands/` folder to your project's `.claude/` and use them directly:
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
| `/glpi-fix-bug` | Complete workflow: investigate → fix → review → test (uses Task tool for agent orchestration) |
| `/glpi-investigate` | Investigate a bug without making changes |
| `/glpi-review` | Review code changes for GLPI compliance |
| `/glpi-test` | Write PHPUnit tests for a class/method |
| `/glpi-learn` | Explain PHP/GLPI patterns for learning |

**Rules** in `.claude/rules/` are applied automatically when editing PHP or Twig files (naming conventions, architecture patterns, code quality checks).

### GitHub Copilot

1. **Copy to your project:**
```bash
# Agents (specialized assistants)
cp -r .claude/copilot/agents/ /your/project/.github/agents/

# Instructions (auto-applied by file path)
cp .claude/copilot/copilot-instructions.md /your/project/.github/
mkdir -p /your/project/.github/instructions/
cp .claude/copilot/instructions/glpi-core.instructions.md /your/project/.github/instructions/
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
cp -r .claude/cursor/agents/ /your/project/.cursor/agents/

# Rules (auto-applied by glob patterns)
mkdir -p /your/project/.cursor/rules/
cp .claude/cursor/rules/glpi-core.mdc /your/project/.cursor/rules/
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
cp -r .claude/antigravity/workflows/ /your/project/.agent/workflows/

# Rules (project-level rules)
mkdir -p /your/project/.agent/rules/
cp .claude/antigravity/rules/glpi-core.md /your/project/.agent/rules/
```

2. **Use workflows in chat:**
```
/glpi-bug-investigator
/glpi-code-reviewer
```

3. Rules apply automatically when files are opened.

### Other AI Tools

Use universal files as context:
- `.claude/_knowledge/*.md` - GLPI documentation
- `.claude/_contexts/*.md` - Environment specifics

---

## Contexts

| Context | When to use |
|---------|-------------|
| `core-10` | GLPI 10.0.x development |
| `core-11` | GLPI 11.0.x / main branch |
| `plugin` | GLPI 11 plugin development |

## Knowledge Base

| File | Content | Claude Code Skill |
|------|---------|-------------------|
| `glpi-architecture.md` | CommonDBTM hooks, DB layer, Session, rights | `glpi-architecture` |
| `glpi-conventions.md` | Naming, anti-patterns, bug patterns | `glpi-conventions` |
| `glpi-plugin-patterns.md` | Plugin structure, Hooks::*, install() | `glpi-plugin-patterns` |
| `glpi-testing.md` | DbTestCase, PHPUnit, Playwright | `glpi-testing` |

Files in `.claude/_knowledge/` are the **source of truth**, used by all tools. For Claude Code, they are also available as **skills** (`.claude/skills/`) which are preloaded automatically into agents via `skills:` frontmatter — no file reads needed.

---

## Customization

### Adding agents

| Tool | Location | Format |
|------|----------|--------|
| Claude Code | `.claude/agents/` | `.md` with YAML frontmatter (`name`, `description`, `tools`, `model`, `skills`, `memory`) |
| Copilot | `.claude/copilot/agents/` | `.md` with YAML frontmatter |
| Cursor | `.claude/cursor/agents/` | `.chatmode.md` with YAML frontmatter |
| Antigravity | `.claude/antigravity/workflows/` | `.md` with `description:` frontmatter |

### Adding commands (Claude Code)

| Location | Format |
|----------|--------|
| `.claude/commands/` | `.md` with YAML frontmatter (`description`, `argument-hint`, `allowed-tools`) |

Use `$ARGUMENTS` placeholder in the command body to receive user input.

### Adding skills (Claude Code)

| Location | Format |
|----------|--------|
| `.claude/skills/{name}/` | `SKILL.md` with YAML frontmatter (`name`, `description`, `user-invocable`, `disable-model-invocation`) |

Skills are injectable knowledge. Reference them in agents via `skills:` frontmatter to preload content automatically.

### Adding rules

| Tool | Location | Format |
|------|----------|--------|
| Claude Code | `.claude/rules/` | `.md` with `globs:` frontmatter |
| Copilot | `.claude/copilot/instructions/` | `.instructions.md` with `applyTo:` |
| Cursor | `.claude/cursor/rules/` | `.mdc` with `globs:` |
| Antigravity | `.claude/antigravity/rules/` | `.md` (plain markdown) |

---

## License

MIT License - See [LICENSE](LICENSE)
