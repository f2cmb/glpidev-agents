# GLPI Conventions & Standards

Coding standards and conventions for GLPI development.

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Tables | `glpi_` prefix, plural, snake_case | `glpi_computers`, `glpi_tickets` |
| Fields | snake_case | `is_deleted`, `date_creation` |
| Foreign keys | `{itemtype}s_id` | `computers_id`, `users_id` |
| Classes | PascalCase | `Computer`, `TicketValidation` |
| Methods | camelCase | `getFromDB()`, `prepareInputForAdd()` |
| Constants | UPPER_SNAKE | `READ`, `CREATE`, `PURGE` |

### Common Field Names

```php
'id'            // Primary key (auto)
'name'          // Display name
'comment'       // Description/notes
'entities_id'   // Entity ownership
'is_recursive'  // Recursive visibility
'is_deleted'    // Soft delete flag
'date_creation' // Creation timestamp
'date_mod'      // Last modification timestamp
'users_id'      // Owner/creator
```

## Code Structure

### File Organization (Core)

```
src/                    # PHP classes (PSR-4 autoload)
front/                  # Entry points (*.php, *.form.php)
ajax/                   # AJAX handlers
templates/              # Twig templates
install/migrations/     # Database migrations
tests/functional/       # PHPUnit tests
tests/cypress/e2e/      # Cypress tests
```

### Method Organization in Classes

1. Properties and constants
2. `getTypeName()`, `getIcon()`
3. `rawSearchOptions()`
4. `prepareInputFor*` hooks
5. `post_*Item` hooks
6. Display methods (`showForm()`, `showTab*()`)
7. Utility methods

## Anti-Patterns to Avoid

### Architecture Anti-Patterns

| Anti-Pattern | Why It's Wrong | GLPI Way |
|--------------|----------------|----------|
| Service classes | Not in GLPI architecture | Use static methods or CommonDBTM hooks |
| Dependency injection | Foreign to GLPI | Use `global $DB`, `Session::*` |
| Repository pattern | Over-abstraction | Use `$item->getFromDB()`, `$DB->request()` |
| DTOs | Unnecessary complexity | Use arrays |
| Event dispatchers | Bypasses GLPI hooks | Use `post_addItem()`, `post_updateItem()` |

### Code Anti-Patterns

| Anti-Pattern | Correct Approach |
|--------------|------------------|
| Raw SQL queries | Use `$DB->request()`, `$DB->insert()`, etc. |
| `var_dump()`, `print_r()` | Use `Toolbox::logDebug()` |
| Hardcoded IDs | Use constants or config |
| Magic numbers | Define constants |
| Direct `$_POST`/`$_GET` | Use GLPI's input handling |
| `echo` in classes | Return data, use TemplateRenderer |

## Common Bug Patterns

Quick reference for investigation:

| Symptom | Likely Cause | Where to Look |
|---------|--------------|---------------|
| Data corruption | Missing validation | `prepareInputForAdd/Update()` |
| Unauthorized access | Permission bypass | `Session::haveRight()` checks |
| Twig errors | Undefined variable | Controller data passing |
| DB errors after upgrade | Schema mismatch | Migration files |
| Side effects missing | Hook not triggered | `post_*Item()` registration |
| Blank page | PHP fatal error | `files/_log/php-errors.log` |
| AJAX failure | Wrong response format | `Ajax::returnJson()` |

## Code Quality

### Before Committing

```bash
make lint          # All quality checks
make phpstan       # Static analysis
make phpcs         # Code style
```

### PHPDoc Standards

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

## Critical Rules

1. **GLPI-native only**: Follow existing patterns, never "improve" with external patterns
2. **Simplicity first**: Simplest working solution is best
3. **Evidence-based**: Reference existing GLPI code for patterns
4. **Minimal scope**: Change only what's necessary
5. **No assumptions**: Verify patterns in codebase before applying
