# Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Tables | `glpi_` prefix, plural, snake_case | `glpi_computers`, `glpi_tickets` |
| Fields | snake_case | `is_deleted`, `date_creation` |
| Foreign keys | `{itemtype}s_id` | `computers_id`, `users_id` |
| Classes | PascalCase | `Computer`, `TicketValidation` |
| Methods | camelCase | `getFromDB()`, `prepareInputForAdd()` |
| Variables | snake_case | `$old_name`, `$is_deleted`, `$entity_id` |
| Array keys | snake_case | `'title_diff'`, `'content_diff'`, `'date_creation'` |
| Constants | UPPER_SNAKE | `READ`, `CREATE`, `PURGE` |
| Itemtype references | `ClassName::class` | `Computer::class`, `Ticket::class` |

## Common Field Names

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

## File Organization (Core)

```
src/                    # PHP classes (PSR-4 autoload)
front/                  # Entry points (*.php, *.form.php)
ajax/                   # AJAX handlers
templates/              # Twig templates
install/migrations/     # Database migrations
tests/functional/       # PHPUnit tests
tests/cypress/e2e/      # Cypress tests
```

## Method Organization in Classes

1. Properties and constants
2. `getTypeName()`, `getIcon()`
3. `rawSearchOptions()`
4. `prepareInputFor*` hooks
5. `post_*Item` hooks
6. Display methods (`showForm()`, `showTab*()`)
7. Utility methods
