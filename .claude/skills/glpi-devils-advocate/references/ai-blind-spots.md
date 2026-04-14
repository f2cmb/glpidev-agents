# AI Blind Spots in GLPI Development

Where AI-generated GLPI code consistently falls short. Use this checklist when reviewing AI output.

---

## 1. Happy Path Bias

**What happens:** AI generates code that handles the success case but misses failure paths, edge cases, and error states.

**GLPI manifestations:**
- `prepareInputForAdd()` returns `$input` but never returns `false` for invalid states
- No `Session::addMessageAfterRedirect()` when an operation fails silently
- Tests only verify the success scenario, not the "should fail" scenario
- `return false` without any user-facing message or log entry — impossible to debug

**Question:** "What happens when this input is invalid, missing, or malformed?"

---

## 2. Scope Acceptance

**What happens:** AI implements exactly what was asked without questioning whether the request is correct or complete.

**GLPI manifestations:**
- Fix applied only to `Ticket` when the issue exists in `Problem` and `Change` too
- Migration adds a field to one table but the same field is needed in related tables
- Fix addresses one code path (web form) but the same bug exists on the API path
- Feature works for `central` interface but breaks in `helpdesk`

**Question:** "Is this the complete scope, or are there parallel code paths with the same issue?"

---

## 3. Pattern Attraction (Over-Engineering)

**What happens:** AI reaches for familiar software engineering patterns that are foreign to GLPI's architecture.

**GLPI manifestations:**
- Creating service classes, DTOs, or repository layers
- Introducing dependency injection where GLPI uses `global $DB`
- Building event dispatchers instead of using CommonDBTM hooks
- Adding abstraction layers where a direct `$DB->request()` is correct
- Creating helper classes for one-time operations

**Question:** "Does GLPI core solve similar problems this way? Show me a `file:line` reference."

---

## 4. Confidence Without Correctness

**What happens:** AI presents solutions with high confidence even when the underlying analysis is wrong or incomplete.

**GLPI manifestations:**
- Claiming a fix is complete after modifying one file, when the bug spans multiple classes
- Asserting that `canUpdateItem()` checks permissions (it doesn't check global rights)
- Stating that a migration is safe without considering table size or locking
- Describing GLPI's behavior based on general PHP knowledge rather than actual codebase verification

**Question:** "Can you show me in the actual GLPI codebase where this behavior is confirmed?"

---

## 5. Reactive Patching

**What happens:** AI fixes the immediate symptom rather than the root cause, often in the wrong layer.

**GLPI manifestations:**
- Adding input normalization in `front/` controller instead of `prepareInputForAdd()`
- Adding a `WHERE` clause to one query instead of fixing the data model
- Catching an exception in the caller instead of preventing it in the source
- Adding a CSS fix for a rendering issue caused by wrong data in the template context

**Question:** "Is this fixing the symptom or the cause? Would this fix survive if the entry point changes?"

---

## Quick Reference

| Blind Spot | Key Question | Red Flag in GLPI Code |
|-----------|-------------|----------------------|
| Happy path bias | "What happens when this fails?" | No `return false` handling, no error messages |
| Scope acceptance | "Are there parallel code paths?" | Fix on Ticket only, or web form only |
| Pattern attraction | "Does GLPI core do it this way?" | Service classes, DI, repositories |
| Confidence without correctness | "Show me the codebase reference." | Claims about behavior without `file:line` |
| Reactive patching | "Symptom or root cause?" | Fix in `front/` instead of `prepareInput*` |
