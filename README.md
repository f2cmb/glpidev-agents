# GLPI Development Agents

A suite of AI agents and slash commands to contribute more efficiently to GLPI (core and plugins).

Built for **Claude Code**, this kit provides specialized agents, slash commands, and a GLPI knowledge base — including file-type skills that activate automatically when editing PHP, JS/TS, or Twig files.

---

## Installation

### Recommended — Claude Code plugin

```bash
claude plugin install github.com/f2cmb/glpidev-agents
```

Agents, commands, and skills become available immediately in any session.

### Legacy — manual copy

If you prefer not to use the plugin system:

```bash
cp -r .claude/ /path/to/your/glpi-project/.claude/
```

---

## Slash Commands

Use directly in Claude Code with `/command-name`.

| Command | Argument | Purpose |
|---------|----------|---------|
| `/glpi-fix-bug` | `<issue-url-or-description>` | Full workflow: investigate → fix → review → test |
| `/glpi-investigate` | `<issue-url-or-description>` | Analyze a bug without making any changes (read-only) |
| `/glpi-review` | `[files]` or empty (= staged) | Review code for GLPI compliance before committing |
| `/glpi-test` | `[e2e\|unit] <Class::method>` | Generate PHPUnit or Playwright tests for a class or method |
| `/glpi-learn` | `<subject> dans <path>/` or `debrief #<issue> PR <url> dans <path>/` | Produce a structured French learning document about a GLPI subject or a recent change (post-PR debrief) |
| `/glpi-plugin-review` | `<path/to/plugin/>` | Full plugin audit: security (22 checks) + GLPI 11 structural conformance |
| `/glpi-devils-advocate` | `[code\|plan\|files]` or empty (= asked) | Challenge AI-generated code, plans, or decisions before they ship |

---

## Agents

Specialized personas loaded automatically with the relevant GLPI knowledge.

| Agent | Purpose | When to use |
|-------|---------|-------------|
| **glpi-bug-investigator** | Systematic bug analysis, root cause identification, resolution plan | Investigating a GitHub issue or unexpected behavior |
| **glpi-code-reviewer** | Code review: GLPI-native patterns, naming conventions, anti-patterns, edge cases | Before any commit or PR |
| **glpi-mentor** | Structured French learning documents anchored to real GLPI files — covers PHP, Twig, JS/TS (jQuery legacy + Vue), SCSS, and the build chain | Post-fix/PR debrief or focused deep-dive on a GLPI subject |
| **glpi-plugin-reviewer** | Security audit (22 checks) and GLPI 11 structural conformance | Verifying a plugin before release or integration |
| **glpi-test-writer** | Write PHPUnit (DbTestCase) and Playwright E2E tests | Adding test coverage |
| **glpi-devils-advocate** | Challenges AI-generated code, plans, and decisions — entity scoping, rights, migrations, hooks, ITIL divergence | Before merging any AI-produced fix or feature |

---

## Knowledge Base (Skills)

Skills are injected automatically into agents — no manual file reads needed.

| Skill | Content |
|-------|---------|
| `glpi-architecture` | CommonDBTM hooks, DB layer (`$DB->request()`), Session, rights (`can()`) |
| `glpi-conventions` | Naming (tables, fields, classes), anti-patterns, common pitfalls |
| `glpi-learn` | Learning-doc methodology: domain detection (PHP/Twig/JS/SCSS/build), citation discipline (file:line), French output, per-domain skeletons in `references/` |
| `glpi-plugin-patterns` | GLPI 11 plugin structure, namespaces, `setup.php`, `hook.php`, PHP 8 patterns |
| `glpi-plugin-security` | 22 security checks: entry point auth, CSRF, SQLi, XSS, mass assignment, file upload, path traversal, SSRF… |
| `glpi-testing` | DbTestCase, PHPUnit fixtures, Playwright page objects, test patterns for core and plugins |
| `glpi-devils-advocate` | Pre-mortem methodology, GLPI-specific blind spots, AI-specific blind spots, questioning frameworks (Socratic, inversion, pre-mortem) |

---

## File-type Skills

Auto-invocable skills that Claude pulls in when about to edit a file of the matching type. They are not preloaded into agents — they activate on demand from the file context.

| Skill | Triggers on | Conventions enforced |
|-------|-------------|----------------------|
| `glpi-php` | `**/*.php` | snake_case variables/keys, `ClassName::class`, CommonDBTM hooks, `$DB->request()`, `$item->can()`, `Toolbox::logDebug()`, `_s()` |
| `glpi-js` | `**/*.{js,ts}` | TypeScript for type safety, ES modules, no jQuery, Vue 3 composition API |
| `glpi-twig` | `**/*.twig` | `TemplateRenderer`, auto-escaping, no raw HTML output from PHP, no inline `<script>` |

---

## Contexts

Specify in your prompts to adapt behavior to your environment.

| Context | When to use |
|---------|-------------|
| `core-10` | GLPI 10.0.x development |
| `core-11` | GLPI 11 / `main` branch development |
| `plugin` | GLPI 11 plugin development |

Example: *"Investigate issue #12345. Context: GLPI 11 core"*

---

## Repository Structure

```
glpidev-agents/
├── .gitignore
├── .claude-plugin/
│   └── plugin.json                 # Claude Code plugin manifest
└── .claude/
    ├── agents/                     # 6 specialized agents
    │   ├── bug-investigator.md
    │   ├── code-reviewer.md
    │   ├── devils-advocate.md
    │   ├── glpi-mentor.md
    │   ├── plugin-reviewer.md
    │   └── test-writer.md
    │
    ├── commands/                   # 7 slash commands
    │   ├── glpi-devils-advocate.md
    │   ├── glpi-fix-bug.md
    │   ├── glpi-investigate.md
    │   ├── glpi-learn.md
    │   ├── glpi-plugin-review.md
    │   ├── glpi-review.md
    │   └── glpi-test.md
    │
    ├── skills/                     # GLPI knowledge base
    │   ├── glpi-architecture/      # + references/
    │   ├── glpi-conventions/       # + references/
    │   ├── glpi-devils-advocate/   # + references/
    │   ├── glpi-js/                # auto-invocable on *.js / *.ts
    │   ├── glpi-learn/             # + references/ (php, twig, javascript, scss, build)
    │   ├── glpi-php/               # auto-invocable on *.php
    │   ├── glpi-plugin-patterns/   # + references/
    │   ├── glpi-plugin-security/   # + checks.md (22 detailed security checks)
    │   ├── glpi-testing/           # + references/
    │   └── glpi-twig/              # auto-invocable on *.twig
    │
    └── _contexts/                  # Environment overlays (manual mention)
        ├── core-10.md
        ├── core-11.md
        └── plugin.md
```

---

## License

MIT — see [LICENSE](LICENSE)
