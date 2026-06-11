# Code Quality

## Comments

Keep code comments minimal and to the point:

- **Concise, clear, direct.** Comment only what the code can't say itself (a non-obvious *why*). Don't narrate what the code already shows.
- **No process metadata in code.** Never reference PR numbers, commit hashes, or issue IDs in a comment — that belongs in the commit message / PR description, not the source.
- **Compress, don't pad.** If a justification needs several lines, state the core reason in one or two; drop the worked-through reasoning and the edge cases the reader can infer.

A wall of explanatory comment adds cognitive load for the reviewer and reads as AI-generated padding. Favour the shortest comment that still answers "why".

```php
// ❌ Verbose — narrates the reasoning, references an issue
// `allow-same-origin` is mandatory: without it the frame gets an opaque origin and the
// players can't read their own cookies/localStorage, so they refuse to start. It is safe
// here because `$src` is always cross-origin, the Same-Origin Policy still blocks any access
// to the parent GLPI DOM. See PR #24268 / issue #328 for the full discussion.

// ✅ Concise — the core why, no process metadata
// `allow-same-origin` lets the cross-origin player read its own storage; safe because `$src`
// is always a provider host, so the Same-Origin Policy still blocks parent-DOM access.
```

## Before Committing

```bash
make lint          # All quality checks (phpstan, phpcs, phpcsfixer...)
```

## PHPDoc Standards

```php
/**
 * Short description.
 *
 * @param int    $id      Item ID
 * @param array  $options Display options
 * @param bool   $full    Full display mode
 *
 * @return string|false HTML content or false on error
 */
public function showForm(int $id, array $options = [], bool $full = true): string|false
```
