# Code Quality

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
