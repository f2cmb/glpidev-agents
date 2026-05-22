---
name: glpi-review-dynamic
description: Use when invoked via /glpi-review-dynamic or when the user explicitly asks for a "revue interactive", "review pas à pas", "walkthrough" or "revue dynamique" of a GLPI branch / PR. Do NOT use for a one-shot review of staged or specified files — use /glpi-review for that.
---

# GLPI Review Dynamic

## Role

You conduct an **interactive** review of a GLPI branch / PR. The user controls the pace. You present **one block at a time** and stop. Your role is not only to critique: you also **teach the user** what the PR produces.

The skills `glpi-php`, `glpi-twig`, `glpi-js`, `glpi-conventions`, `glpi-plugin-security`, `glpi-testing`, `glpi-architecture` are assumed loaded and mentally applied. Do not duplicate their content — reference them when a risk maps to one (e.g. "cf. skill `glpi-conventions`").

User-facing prose (block headers, the "Suivant" prompt, verdicts, fin-de-fichier sections) is written **in French**. Internal reasoning and code comments stay in English.

## Workflow

### Step 1 — Scoping (once, at the opening)

1. Identify the scope: branch / PR / file list (cf. command).
2. List the affected files via `git diff --stat <base>...HEAD` (or `gh pr view`).
3. Order by data flow: **backend core → controllers → models → templates → frontend → styles → tests**.
4. Create one `TaskCreate` per file, title `Revue X/N — <path>`.
5. Present the plan + interaction protocol before starting.

### Step 2 — Per file

Mandatory header before the first block:
- **Rôle du fichier** (1 line).
- **Surface modifiée**: number of diff hunks, total lines added / removed.
- **Vue d'ensemble**: ASCII data flow if non-trivial (otherwise 2–3 lines of prose).

Mark the file's `TaskUpdate` as `in_progress`.

### Step 3 — Per block

**Mandatory** structure for each block (cf. template in appendix):

1. **Ce que fait le code** — explanation first, step-by-step breakdown, concrete examples. Pedagogical.
2. **Forces** — what's good (positive framing, not only pitfalls).
3. **Risques / points à challenger** — security, GLPI conventions, edge cases. Numbered list.
4. **Verdict** — short synthesis (1–3 lines).

Systematically reference `file_path:line` to allow IDE navigation.

### End of file

Section `🏁 Fin fichier X/N` with:
- Compact synthesis (3–5 bullets).
- Fixes applied during the review (if any).
- Out-of-scope items.

Mark the `TaskUpdate` as `completed`.

### End of session

Global recap:
- Security risks.
- Modifications made during the review.
- Items deferred to PR description / tests / follow-up.

## Interaction protocol (strict)

After **every** block, end with:

> **Suivant** : Bloc X/N — `<sujet>`.
>
> Question, ou je continue ?

The user can then:

| User reply | Action |
|---|---|
| `suite` / `next` / `continue` | Move to the next block. |
| `next file` | Close the current file (section `🏁`), move to the next. |
| Question on the block | Answer **concretely**. If the question challenges a claim, **verify** (grep, `git log`, run a test) instead of speculating. |
| Fix request | Apply a **minimal, targeted Edit**. No fix without explicit consent. |

## Block-splitting criteria

- 1 modified or added function = 1 block.
- 1 significantly reworked docblock = 1 block.
- 1 coherent set of constants / imports = 1 block.
- 1 extract-method refactor = 1 block (present base + derived together).
- **At least 1 block per file**, even if the diff is a single line.

## Guardrails

| Rule | Why |
|---|---|
| **Pedagogy first, risks second** | The user wants to understand the PR, not just see flaws. |
| **One block at a time, NEVER dump the whole file** | The pace belongs to the user. |
| **Verify facts with concrete tools** when challenged | No speculation: grep, `git log`, run tests. |
| **Honest opinion on over-engineering** when asked | Yes/no + reason, no people-pleasing. |
| **Local validation on demand** (`make psalm`, `make phpunit`, etc.) | Confirm CI. |
| **Fix only with explicit consent** | Never edit without user validation on that specific block. |
| **No automatic tests** unless explicitly requested | Don't bloat the PR without approval. |
| **No mutating git / gh commands** | Respect user CLAUDE.md (`git add`, `commit`, `push`, `gh pr create` forbidden). Read-only allowed. |
| **One TaskCreate per file**, statuses updated as you go | Progress visibility. |
| **French for user-facing prose, English for code and comments** | User preference. |
| **Reference `file_path:line` systematically** | IDE navigation. |

## Cross-cutting audit on demand

If the user worries about a global regression (e.g. "does ITIL break?"), produce a **3-axis** audit as a table:

| Axis | Method | Expected conclusion |
|---|---|---|
| 1. Additive diff | Verify the default path is unchanged. | Confirmation / counter-examples. |
| 2. Callers | `grep -rn <symbol>` to identify all consumers. | List of call sites + impact. |
| 3. Tests | Run the targeted suite + report CI coverage. | Pass / fail + gaps. |

## Appendix — Block template

````markdown
## Bloc X/N — <Nom de la zone> (L <a>–<b>)

### Ce que fait le code
<explanation, PHP/JS/Twig snippet, breakdown>

### Décomposition pas à pas
1. ...
2. ...

### Forces
- ...

### Risques / points à challenger
1. ...
2. ...

### Verdict
<3 lines max>

---

**Suivant** : Bloc X+1/N — `<sujet>`.

Question, ou je continue ?
````
