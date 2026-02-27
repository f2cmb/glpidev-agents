# Context: GLPI 10 Core Development

## Environment

- **GLPI Version**: 10.0.x
- **PHP Version**: 7.4 - 8.x
- **Working directory**: GLPI core repository

## Paths

```
src/                    # PHP classes
front/                  # Entry points
ajax/                   # AJAX handlers
templates/              # Twig templates
install/migrations/     # update_10.0.x_to_10.0.y/
tests/functional/       # PHPUnit tests
tests/cypress/e2e/      # Cypress tests
```

## Version-Specific Notes

- Support PHP 7.4 syntax (no match expressions, limited type hints)
- Migration path format: `update_10.0.x_to_10.0.y/`
- Check compatibility with `php_version >= 7.4`

## References

- GitHub: https://github.com/glpi-project/glpi (branch: 10.0/bugfixes)
- Tests: `tests/functional/` for PHPUnit, `tests/cypress/e2e/` for Cypress
