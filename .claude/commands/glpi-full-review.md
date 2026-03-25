---
description: Full review — GLPI conventions + general code quality (PR or local changes)
argument-hint: [pr-number]
allowed-tools: Glob, Grep, Read, Bash, WebSearch, Skill
---

# GLPI Full Review

## Step 1: Detect context

**If `$ARGUMENTS` contains a PR number:**
Use that number for both reviews.

**If no argument:**
Try to detect a PR associated with the current branch:
```bash
gh pr list --head $(git branch --show-current) --json number,title
```
If a PR is found, use it for the general review.
Otherwise, fall back to local changes (unstaged, staged, unpushed commits).

---

## Step 2: GLPI conventions review

Invoke the `glpi-review` skill with the appropriate context:
- If PR: pass the PR diff as the list of files to analyze
- If local: no argument (the skill handles local changes)

---

## Step 3: General code review

Invoke the `review` skill:
- If a PR is available (provided or detected): pass the PR number
- If no PR: analyze local changes against the same criteria (correctness, quality, performance, tests, security)

---

## Step 4: Unified summary

Produce a **single report** consolidating results from both reviews:

```markdown
## Full Review — [PR title or change description]

### GLPI Conventions
[Result from glpi-review]

### General Quality
[Result from review]

### Summary
**Overall verdict: [APPROVED / NEEDS CHANGES / REJECTED]**
Blocking issues (if any):
- ...
```
