# Questioning Frameworks for GLPI Code Review

Structured approaches for challenging GLPI code, plans, and decisions.

---

## 1. Pre-Mortem

**Premise:** "This shipped to production. 3 months later it caused an incident. What went wrong?"

Force yourself to imagine failure, then trace backwards.

### GLPI-Specific Pre-Mortem Questions

- "This migration ran on a 500k-row production table. What locked? For how long?"
- "A helpdesk user with minimal rights accessed this endpoint. What data leaked?"
- "Entity A and Entity B both have a 'Server-01' computer. This query returned both. Who saw the other's data?"
- "The inventory agent ran overnight and overwrote the field this feature manages. What's the user's experience the next morning?"
- "A plugin was listening to `post_updateItem()` for this class. This change bypassed the hook. What broke downstream?"

### When to Use

- Reviewing migrations and schema changes
- Evaluating any feature that touches entity-scoped data
- Reviewing changes to CommonDBTM hooks or front controllers

---

## 2. Inversion

**Premise:** "What would guarantee this breaks?"

Instead of asking "does this work?", ask "what would make this fail?" — then check if those conditions exist.

### GLPI-Specific Inversion Questions

- "What would guarantee this breaks in a multi-entity environment?" -> Check entity filtering on every query
- "What would guarantee this leaks data across profiles?" -> Check `$item->can($id, RIGHT)` vs `canUpdateItem()`
- "What would guarantee this migration is irreversible?" -> Check for `dropField()` without transition period
- "What would guarantee the API path doesn't trigger hooks?" -> Check if logic is in `front/` instead of `prepareInput*`/`post_*`
- "What would guarantee this only works for Ticket but breaks for Problem/Change?" -> Check if code uses `Ticket::INCOMING` instead of the parent constant

### When to Use

- Reviewing bug fixes that claim to be "complete"
- Evaluating access control implementations
- Checking ITIL-related changes across Ticket/Problem/Change

---

## 3. Socratic Questioning

**Premise:** "This assumes [X]. What if that assumption is wrong?"

Surface hidden assumptions by naming them explicitly.

### GLPI-Specific Socratic Questions

- "This assumes the user has the UPDATE right in their profile. What if they only have READ?"
- "This assumes `entities_id` is always set. What if the item is in the root entity?"
- "This assumes `post_addItem()` fires for all creation paths. What about API creation?"
- "This assumes `is_recursive` is false. What if the parent entity set it to true?"
- "This assumes all CommonITILObject children share this method. What if Problem overrides it?"
- "This assumes the session entity is set. What if this runs in a CLI/cron context?"

### When to Use

- Reviewing any code that reads from `$_SESSION`
- Evaluating permission logic
- Checking code that assumes CommonITILObject uniformity

---

## 4. Steel-Manning

**Premise:** Before challenging, articulate why the approach is reasonable.

If you can't explain why someone would choose this approach in GLPI's context, your challenge is probably off-base.

### Structure

1. Identify the problem the code solves
2. Explain which GLPI constraints it respects
3. Note what it gets right
4. *Then* challenge what it misses

### Why Steel-Man First

- Prevents drive-by criticism that ignores GLPI's specific constraints
- Forces you to understand the code before criticizing it
- Makes your challenges more credible — you've demonstrated understanding
- Catches cases where the approach is actually correct and your initial instinct was wrong

### GLPI Example

> "Here's what this gets right: it uses `prepareInputForAdd()` for validation, which ensures
> the check runs on both web form and API paths. It correctly uses `$DB->request()` array
> syntax instead of raw SQL. The fix is minimal in scope."
>
> "Now, the concern: it only validates for Ticket, but Problem and Change have the same
> issue with a different code path in `post_addItem()`."
