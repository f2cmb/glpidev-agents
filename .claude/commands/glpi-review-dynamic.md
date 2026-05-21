---
description: Walkthrough interactif d'une branche/PR GLPI fichier par fichier, bloc par bloc, avec Q/R entre les blocs.
argument-hint: "[branche|PR#|files...] (défaut: branche courante vs main)"
allowed-tools: Bash, Read, Edit, Grep, Glob, TaskCreate, TaskUpdate, AskUserQuestion, Skill
---

# GLPI Review Dynamic

Revue **interactive** d'une PR GLPI. Pas de délégation à un subagent — l'interaction Q/R exige le main agent.

## Input

Cible : $ARGUMENTS

- vide → branche courante vs `main` (`git diff --stat main...HEAD`)
- `PR#<n>` → `gh pr view <n> --json files`
- autre → liste de fichiers / globs

## Execution

1. Résoudre le scope selon les règles ci-dessus.
2. Charger la skill `glpi-review-dynamic` et appliquer sa méthodologie de bout en bout.
3. Edit/Write **uniquement sur consentement explicite** de l'utilisateur à un bloc donné. Aucun fix non sollicité.
4. Aucune commande `git`/`gh` modifiante (cf. CLAUDE.md utilisateur).

Source of truth: `.claude/skills/glpi-review-dynamic/SKILL.md` — do not duplicate logic here.
