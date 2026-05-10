# Tab Integration & PHP 8 Patterns

## Tab Integration

```php
public function getTabNameForItem(CommonGLPI $item, $withtemplate = 0): string
{
    if ($item instanceof AuthLDAP) {
        return self::createTabEntry(self::getTypeName(Session::getPluralNumber()));
    }
    return '';
}

public static function displayTabContentForItem(
    CommonGLPI $item,
    $tabnum = 1,
    $withtemplate = 0
): bool {
    if ($item instanceof AuthLDAP) {
        self::showForAuthLDAP($item);
        return true;
    }
    return false;
}
```

## PHP 8 Patterns

### Safe Functions

```php
use function Safe\json_encode;
use function Safe\json_decode;
use function Safe\file_get_contents;
use function Safe\preg_match;

// These throw exceptions instead of returning false
$data = json_decode($json, true);
```

### Match Expressions

```php
$result = match ($status) {
    'new'      => __('New'),
    'assigned' => __('Assigned'),
    'solved'   => __('Solved'),
    default    => __('Unknown'),
};
```

### Typed Properties

```php
public static string $rightname = 'plugin_pluginname_myclass';
public static bool $is_entity_assign = true;
```

## Plugin Anti-Patterns

| Anti-Pattern | Correct Approach |
|--------------|------------------|
| Code in `inc/` | Use `src/` only |
| String hooks (`'item_add'`) | Use `Hooks::ITEM_ADD` |
| Install logic in hook.php | Use `Class::install(Migration)` |
| Missing Safe\ functions | Use Safe\ for json, file, preg |
| Untyped PHPDoc | Add proper type hints |
