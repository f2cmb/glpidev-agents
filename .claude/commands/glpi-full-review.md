---
description: Revue complète — conventions GLPI + qualité générale (PR ou changements locaux)
argument-hint: [pr-number]
allowed-tools: Glob, Grep, Read, Bash, WebSearch, Skill
---

# GLPI Full Review

## Étape 1 : Détecter le contexte

**Si `$ARGUMENTS` contient un numéro de PR :**
Utiliser ce numéro pour les deux revues.

**Si pas d'argument :**
Tenter de détecter une PR associée à la branche courante :
```bash
gh pr list --head $(git branch --show-current) --json number,title
```
Si une PR est trouvée, l'utiliser pour la revue générale.
Sinon, la revue générale s'appuie sur les changements locaux (non stagés, stagés, commits non pushés).

---

## Étape 2 : Revue conventions GLPI

Invoquer le skill `glpi-review` avec les arguments appropriés :
- Si PR : passer le diff de la PR comme contexte des fichiers à analyser
- Si local : pas d'argument (le skill gère les changements locaux)

---

## Étape 3 : Revue générale

Invoquer le skill `review` :
- Si PR disponible (fournie ou détectée) : passer le numéro de PR
- Si pas de PR : analyser les changements locaux sur les mêmes critères (exactitude, qualité, performance, tests, sécurité)

---

## Étape 4 : Synthèse unifiée

Produire un **rapport unique** qui consolide les résultats des deux revues :

```markdown
## Revue complète — [titre PR ou description des changements]

### Conventions GLPI
[Résultat de glpi-review]

### Qualité générale
[Résultat de review]

### Synthèse
**Verdict global : [APPROUVÉ / À CORRIGER / REJETÉ]**
Points bloquants (le cas échéant) :
- ...
```
