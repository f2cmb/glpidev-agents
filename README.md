# GLPI Development Agents

A suite of AI agents and skills to contribute more efficiently to GLPI (core and plugins).

Built for **Claude Code**, this kit provides specialized agents and a GLPI knowledge base, both reachable as slash commands — plus file-type skills gated on a `paths:` glob, so the PHP, JS/TS and Twig conventions load only when those files are in play.

---

## Installation

### Recommended — Claude Code plugin

```bash
claude plugin install github.com/f2cmb/glpidev-agents
```

Agents and skills become available immediately in any session.

### Legacy — manual copy

If you prefer not to use the plugin system:

```bash
cp -r .claude/ /path/to/your/glpi-project/.claude/
```

---

## Slash Commands

Every command is a skill under `.claude/skills/`. Use it with `/command-name`.

| Command | Argument | Purpose | Runs in |
|---------|----------|---------|---------|
| `/glpi-investigate` | `<issue-url-or-description>` | Analyze a bug without making any changes (read-only) | forked subagent |
| `/glpi-review` | `[files]` or empty (= staged) | Review code for GLPI compliance, then ask which findings to fix | main session → subagent |
| `/glpi-review-dynamic` | `[branch\|PR#\|files]` or empty (= current branch vs `main`) | Interactive walkthrough of a branch / PR — file by file, block by block, with Q&A between blocks | main session |
| `/glpi-test` | `[e2e\|unit] <Class::method>` | Generate PHPUnit or Playwright tests for a class or method | forked subagent |
| `/glpi-learn` | `<subject> dans <path>/` or `debrief #<issue> PR <url> dans <path>/` | Produce a structured French learning document about a GLPI subject or a recent change (post-PR debrief) | main session |
| `/glpi-plugin-review` | `<path/to/plugin/>` | Full plugin audit: security (23 checks) + GLPI 11 structural conformance | forked subagent |
| `/glpi-devils-advocate` | `[code\|plan\|files]` or empty (= asked) | Challenge AI-generated code, plans, or decisions before they ship | main session |

**Why the split.** A command that only delegates and returns a report carries `context: fork` — the work happens in an isolated context and never fills your conversation. A command that has to talk to you stays in the main session, because `AskUserQuestion` is withheld from every subagent. Anything conversational — the devil's advocate menu, the block-by-block walkthrough, the "which findings do you want fixed?" prompt — must therefore run in the main session.

---

## Agents

Specialized personas loaded with the relevant GLPI knowledge preloaded as skills.

| Agent | Purpose | When to use |
|-------|---------|-------------|
| **glpi-bug-investigator** | Systematic bug analysis, root cause identification, resolution plan | Investigating a GitHub issue or unexpected behavior |
| **glpi-code-reviewer** | Code review: GLPI-native patterns, naming conventions, anti-patterns, edge cases | Before any commit or PR |
| **glpi-mentor** | Structured French learning documents anchored to real GLPI files — covers PHP, Twig, JS/TS (jQuery legacy + Vue), SCSS, and the build chain | Post-fix/PR debrief or focused deep-dive on a GLPI subject |
| **glpi-plugin-reviewer** | Security audit (23 checks) and GLPI 11 structural conformance | Verifying a plugin before release or integration |
| **glpi-test-writer** | Write PHPUnit (DbTestCase) and Playwright E2E tests | Adding test coverage |
| **glpi-devils-advocate** | Challenges AI-generated code, plans, and decisions — entity scoping, rights, migrations, hooks, ITIL divergence | Before merging any AI-produced fix or feature |

No agent declares `AskUserQuestion` — Claude Code withholds it from every subagent, foreground or background. An agent that lacks information reports what it needs instead of guessing.

---

## Knowledge Base (Skills)

Preloaded into the agents that need them — no manual file reads.

| Skill | Content |
|-------|---------|
| `glpi-architecture` | CommonDBTM hooks, DB layer (`$DB->request()`), Session, rights (`can()`) |
| `glpi-conventions` | Naming (tables, fields, classes), anti-patterns, code quality, PHPDoc, error handling, review discipline |
| `glpi-learn` | Learning-doc methodology: domain detection (PHP/Twig/JS/SCSS/build), citation discipline (file:line), French output, per-domain skeletons in `references/` |
| `glpi-plugin-patterns` | GLPI 11 plugin structure, namespaces, `setup.php`, `hook.php`, PHP 8 patterns |
| `glpi-plugin-security` | 23 security checks (S1–S23) + the vulnerable/safe patterns and CVEs in `checks.md`, and one grep command per check in `audit-commands.md` |
| `glpi-review-dynamic` | Interactive PR walkthrough — one block at a time, pedagogy first, user controls the pace |
| `glpi-testing` | DbTestCase, PHPUnit fixtures, data providers, Playwright page objects and locator policy, review discipline |
| `glpi-devils-advocate` | Pre-mortem methodology, GLPI-specific blind spots, AI-specific blind spots, questioning frameworks (Socratic, inversion, pre-mortem) |

---

## File-type Skills

Gated by a `paths:` glob, so Claude loads them only when the files in play match. They are not preloaded into agents; an agent pulls one in with the `Skill` tool when it needs the per-language rules.

| Skill | `paths:` | Conventions enforced |
|-------|----------|----------------------|
| `glpi-php` | `**/*.php` | snake_case variables/keys, `ClassName::class`, CommonDBTM hooks, `$DB->request()`, `$item->can()`, `Toolbox::logDebug()`, `_s()` |
| `glpi-js` | `**/*.js`, `**/*.ts`, `**/*.vue` | TypeScript for type safety, ES modules, no jQuery, Vue 3 composition API |
| `glpi-twig` | `**/*.twig` | `TemplateRenderer`, auto-escaping, no raw HTML output from PHP, no inline `<script>` |

---

## Environment Contexts

Overlays fixing the version-specific facts: PHP baseline, directory layout, migration path format, which test frameworks exist, and which security model applies. Invoke one explicitly, or let Claude load it from the checkout.

| Context | When to use |
|---------|-------------|
| `/glpi-context-core-10` | GLPI 10.0.x core — PHP 7.4 constraints, PHPUnit only |
| `/glpi-context-core-11` | GLPI 11 / `main` core — PHP 8.1+, PHPUnit + Playwright |
| `/glpi-context-plugin` | GLPI 11 plugin development — auto-loads under `plugins/**` and `marketplace/**` |

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
    └── skills/
        ├── glpi-investigate/       # entry points — /command
        ├── glpi-review/
        ├── glpi-review-dynamic/
        ├── glpi-test/
        ├── glpi-learn/             # + references/ (php, twig, javascript, scss, build)
        ├── glpi-plugin-review/
        ├── glpi-devils-advocate/   # + references/
        │
        ├── glpi-architecture/      # knowledge — preloaded into agents
        ├── glpi-conventions/       # + references/
        ├── glpi-plugin-patterns/   # + references/
        ├── glpi-plugin-security/   # + checks.md + audit-commands.md
        ├── glpi-testing/           # + references/
        │
        ├── glpi-php/               # file-type — paths: **/*.php
        ├── glpi-js/                # paths: **/*.{js,ts,vue}
        ├── glpi-twig/              # paths: **/*.twig
        │
        ├── glpi-context-core-10/   # environment overlays
        ├── glpi-context-core-11/
        └── glpi-context-plugin/
```

---

## Conventions du repo

1. **Un skill = une seule source de vérité.** Chaque règle GLPI (naming, hook, pattern, anti-pattern, check de sécurité…) vit dans **un seul** skill sous `.claude/skills/<nom-du-skill>/`. Si la même règle apparaît ailleurs — dans le corps d'un agent ou d'un skill d'entrée — c'est une duplication, donc un bug à corriger. Le but est qu'une mise à jour de doctrine se fasse en un seul endroit.

2. **Les agents préchargent leurs skills.** Le frontmatter d'agent supporte `skills: [...]` (cf. [doc Claude Code, § "Preload skills into subagents"](https://code.claude.com/docs/en/sub-agents#preload-skills-into-subagents)). Le contenu du `SKILL.md` de chaque skill listé est injecté automatiquement dans le contexte de l'agent à son démarrage. Les fichiers `references/*.md` ne sont **pas** préchargés : l'agent les charge à la demande via `Read`, ce qui garde le contexte initial léger. Tout agent porte aussi le tool `Skill`, pour tirer un skill non préchargé — typiquement un skill file-type ou un overlay de contexte.

3. **Piège `disable-model-invocation: true`.** La doc précise : *« You cannot preload skills that set `disable-model-invocation: true` »*. Si ce flag est posé sur un skill, toute déclaration `skills:` qui le référence devient un no-op silencieux. Conséquence concrète pour ce repo : `glpi-devils-advocate` et `glpi-learn` sont préchargés dans un agent, donc ils **ne portent pas** le flag ; les skills d'entrée qui ne sont préchargés nulle part (`glpi-investigate`, `glpi-review`, `glpi-review-dynamic`, `glpi-test`, `glpi-plugin-review`) le portent, parce que ce sont des commandes utilisateur à déclenchement explicite. `user-invocable: false` est en revanche neutre vis-à-vis du préchargement.

4. **Les skills d'entrée sont minces, et leur mode d'exécution est un choix motivé.** Un skill d'entrée décrit la cible, le mode et le livrable — jamais la méthodologie GLPI, qui vit dans l'agent ou dans un skill de connaissance. Il porte `context: fork` + `agent:` quand il ne fait que déléguer et rendre un rapport ; il reste en session principale dès qu'il doit dialoguer avec l'utilisateur, puisqu'un sous-agent ne peut pas poser de question. Toute logique métier embarquée dans un skill d'entrée est un bug à corriger : elle doit remonter dans l'agent ou le skill de connaissance.

5. **Pas de duplication de connaissance GLPI dans les agents.** Le corps d'un agent décrit son rôle, sa méthodologie et son format de sortie. Il ne ré-énumère pas les règles GLPI déjà présentes dans les skills préchargés — il les cite. Si une règle apparaît à la fois dans un agent et dans un skill, c'est une dette de duplication à résorber : la règle reste dans le skill, l'agent s'y appuie.

6. **Le frontmatter doit parser en YAML strict.** Un `: ` non échappé dans une `description` passe le parseur tolérant de Claude Code mais casse tout parseur YAML conforme — et un frontmatter malformé charge le skill **avec des métadonnées vides**, donc sans `name`, sans `description` et sans ses flags. Utiliser un scalaire de bloc (`description: >-`) dès que la valeur contient de la ponctuation. Vérification :

```bash
claude plugin validate . --strict
```

Toute évolution du repo (nouvel agent, nouveau skill) doit respecter ces 6 règles. Une violation est une dette à corriger immédiatement, pas un cas particulier acceptable.

---

## License

MIT — see [LICENSE](LICENSE)
