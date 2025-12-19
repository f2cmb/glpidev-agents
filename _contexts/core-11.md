# Context: GLPI 11 Core Development

## Environment

- **GLPI Version**: 11.0.x
- **PHP Version**: 8.1+
- **Working directory**: GLPI core repository

## Paths

```
src/                    # PHP classes
front/                  # Entry points
ajax/                   # AJAX handlers
templates/              # Twig templates
install/migrations/     # update_11.0.x_to_11.0.y/
tests/functional/       # PHPUnit tests
tests/cypress/e2e/      # Cypress tests
```

## Version-Specific Notes

- Use PHP 8.x features: match expressions, constructor promotion, union types
- Migration path format: `update_11.0.x_to_11.0.y/`
- Null safety with `?->` operator
- Typed properties encouraged

## PHP 8 Patterns Available

```php
// Match expression
$label = match ($status) {
    'new'    => __('New'),
    'closed' => __('Closed'),
    default  => __('Unknown'),
};

// Constructor promotion
public function __construct(
    private readonly int $id,
    private string $name,
) {}

// Null-safe operator
$name = $item?->getField('name');
```

## References

- GitHub: https://github.com/glpi-project/glpi (branch: main)
- Tests: `tests/functional/` for PHPUnit, `tests/cypress/e2e/` for Cypress
