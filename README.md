# GLPI Development Agents

A suite of AI agents and slash commands to contribute more efficiently to GLPI (core and plugins).

Built for **Claude Code**, this kit provides specialized agents, slash commands, a GLPI knowledge base, and rules applied automatically based on file type.

---

## Installation

Copy the `.claude/` directory to the root of your GLPI project:

```bash
cp -r .claude/ /path/to/your/glpi-project/.claude/
```

Agents, commands, and rules are immediately available in Claude Code.

---

## Slash Commands

Use directly in Claude Code with `/command-name`.

| Command | Argument | Purpose |
|---------|----------|---------|
| `/glpi-feature` | `<issue-or-pr-url>` or `finalize` | Start or wrap up a feature development session from a GitHub issue or PR |
| `/glpi-fix-bug` | `<issue-url-or-description>` | Full workflow: investigate → fix → review → test |
| `/glpi-investigate` | `<issue-url-or-description>` | Analyze a bug without making any changes (read-only) |
| `/glpi-review` | `[files]` or empty (= staged) | Review code for GLPI compliance before committing |
| `/glpi-test` | `[e2e\|unit] <Class::method>` | Generate PHPUnit or Playwright tests for a class or method |
| `/glpi-learn` | `<concept-or-code-snippet>` | Explain a PHP/GLPI pattern to build understanding |
| `/glpi-plugin-review` | `<path/to/plugin/>` | Full plugin audit: security (22 checks) + GLPI 11 structural conformance |
| `/glpi-devils-advocate` | `[code\|plan\|files]` or empty (= asked) | Challenge AI-generated code, plans, or decisions before they ship |
| `/glpi-a11y` | `[path]` or empty (= changed files on current branch) | Read-only RGAA 4.1 / WCAG AA accessibility audit on existing GLPI code |

---

## Agents

Specialized personas loaded automatically with the relevant GLPI knowledge.

| Agent | Purpose | When to use |
|-------|---------|-------------|
| **glpi-bug-investigator** | Systematic bug analysis, root cause identification, resolution plan | Investigating a GitHub issue or unexpected behavior |
| **glpi-code-reviewer** | Code review: GLPI-native patterns, naming conventions, anti-patterns, edge cases | Before any commit or PR |
| **glpi-feature-builder** | Full feature development cycle: issue analysis → plan → implementation → tests → review | Working on a new feature |
| **glpi-php-mentor** | PHP explanations, GLPI patterns, OOP principles | Understanding why code works a certain way |
| **glpi-plugin-reviewer** | Security audit (22 checks) and GLPI 11 structural conformance | Verifying a plugin before release or integration |
| **glpi-test-writer** | Write PHPUnit (DbTestCase) and Playwright E2E tests | Adding test coverage |
| **glpi-devils-advocate** | Challenges AI-generated code, plans, and decisions — entity scoping, rights, migrations, hooks, ITIL divergence | Before merging any AI-produced fix or feature |
| **glpi-a11y-reviewer** | Read-only RGAA 4.1 / WCAG AA audit — scans Twig, JS, CSS, PHP and produces a prioritized report with concrete fixes | Auditing existing code for accessibility issues |

---

## Knowledge Base (Skills)

Skills are injected automatically into agents — no manual file reads needed.

| Skill | Content |
|-------|---------|
| `glpi-architecture` | CommonDBTM hooks, DB layer (`$DB->request()`), Session, rights (`can()`) |
| `glpi-conventions` | Naming (tables, fields, classes), anti-patterns, common pitfalls |
| `glpi-plugin-patterns` | GLPI 11 plugin structure, namespaces, `setup.php`, `hook.php`, PHP 8 patterns |
| `glpi-plugin-security` | 22 security checks: entry point auth, CSRF, SQLi, XSS, mass assignment, file upload, path traversal, SSRF… |
| `glpi-testing` | DbTestCase, PHPUnit fixtures, Playwright page objects, test patterns for core and plugins |
| `glpi-devils-advocate` | Pre-mortem methodology, GLPI-specific blind spots, AI-specific blind spots, questioning frameworks (Socratic, inversion, pre-mortem) |
| `glpi-a11y` | RGAA 4.1 / WCAG AA criteria for Twig, JS, CSS, PHP legacy — injected into `glpi-code-reviewer` and `glpi-feature-builder`. Covers images, colors, tables, forms, scripts, navigation, and W3C APG keyboard patterns (combobox, tabs, disclosure, grid, listbox, breadcrumb) |

---

## Accessibility

The `glpi-a11y` skill and `glpi-a11y-reviewer` agent provide **static analysis assistance** against RGAA 4.1 (primary) and WCAG 2.2 AA (baseline). They detect common violations in Twig templates, JS, CSS, and PHP with inline HTML, and suggest concrete fixes with criterion references.

**What these tools do:**
- Flag missing labels, insufficient contrast, broken keyboard patterns, missing ARIA attributes, inaccessible tables, and more
- Reference the exact RGAA criterion for each issue
- Propose corrected code snippets inline

**What these tools do not replace:**
- Testing with a real screen reader (NVDA on Windows, VoiceOver on macOS/iOS, TalkBack on Android)
- Keyboard-only navigation testing by a human
- A formal RGAA audit conducted by an accessibility specialist
- User testing with people who rely on assistive technologies

Static analysis catches structural issues early in development. It does not validate actual AT behavior, focus order perception, reading order, or cognitive load. **Always complement with manual testing.**

> Recommended bookmarklets to use alongside: **Focus Order**, **Structure Revealer** — [a11y-tools.com/bookmarklets](https://a11y-tools.com/bookmarklets/)

---

## Automatic Rules

Applied automatically when editing files, based on file type — no configuration needed.

| Files | Rules |
|-------|-------|
| `**/*.php` | snake_case, PascalCase classes, CommonDBTM hooks, `$DB->request()`, `can()`, `_s()` |
| `**/*.{js,ts}` | TypeScript for type safety, ES modules, no jQuery |
| `**/*.twig` | TemplateRenderer, auto-escaping, no raw HTML output from PHP |

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
└── .claude/
    ├── agents/                     # 8 specialized agents
    │   ├── a11y-reviewer.md
    │   ├── bug-investigator.md
    │   ├── code-reviewer.md
    │   ├── devils-advocate.md
    │   ├── glpi-feature-builder.md
    │   ├── php-mentor.md
    │   ├── plugin-reviewer.md
    │   └── test-writer.md
    │
    ├── commands/                   # 9 slash commands
    │   ├── glpi-a11y.md
    │   ├── glpi-devils-advocate.md
    │   ├── glpi-feature.md
    │   ├── glpi-fix-bug.md
    │   ├── glpi-investigate.md
    │   ├── glpi-learn.md
    │   ├── glpi-plugin-review.md
    │   ├── glpi-review.md
    │   └── glpi-test.md
    │
    ├── skills/                     # GLPI knowledge base
    │   ├── glpi-a11y/              # RGAA 4.1 / WCAG AA — injected in code-reviewer + feature-builder
    │   │   ├── SKILL.md
    │   │   └── references/
    │   │       ├── aria-patterns.md      # W3C APG: combobox, grid, tabs, disclosure, listbox, breadcrumb
    │   │       ├── rgaa-colors.md        # Thématique 3: contrast, info not by color alone
    │   │       ├── rgaa-forms.md         # Thématique 11: labels, errors, field grouping
    │   │       ├── rgaa-images.md        # Thématique 1: alt text, decorative vs informative, SVG
    │   │       ├── rgaa-navigation.md    # Thématique 12: skip links, landmarks, keyboard traps
    │   │       ├── rgaa-scripts.md       # Thématique 7: ARIA states, keyboard operability, live regions
    │   │       └── rgaa-tables.md        # Thématique 5: th+scope, caption, complex headers
    │   ├── glpi-architecture/
    │   ├── glpi-conventions/
    │   ├── glpi-devils-advocate/   # + references/
    │   │   └── references/
    │   │       ├── ai-blind-spots.md
    │   │       ├── glpi-blind-spots.md
    │   │       └── questioning-frameworks.md
    │   ├── glpi-plugin-patterns/
    │   ├── glpi-plugin-security/   # + checks.md (22 detailed security checks)
    │   └── glpi-testing/
    │
    ├── rules/                      # Rules by file type
    │   ├── php.md
    │   ├── js.md
    │   └── twig.md
    │
    └── _contexts/                  # Environment overlays
        ├── core-10.md
        ├── core-11.md
        └── plugin.md
```

---

## License

MIT — see [LICENSE](LICENSE)
