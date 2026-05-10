# Database Layer

## DBmysql Abstraction

```php
global $DB;

// Query with iterator (preferred)
$iterator = $DB->request([
    'SELECT' => ['id', 'name'],
    'FROM'   => 'glpi_computers',
    'WHERE'  => ['is_deleted' => 0],
    'ORDER'  => 'name ASC',
    'LIMIT'  => 10
]);
foreach ($iterator as $row) {
    // process $row
}

// Insert
$DB->insert('glpi_tablename', ['field' => 'value']);

// Update
$DB->update('glpi_tablename', ['field' => 'newvalue'], ['id' => $id]);

// Delete
$DB->delete('glpi_tablename', ['id' => $id]);
```

## Migration Class

For schema changes in `install/migrations/`:

```php
$migration->addField('glpi_tablename', 'new_field', 'string');
$migration->addKey('glpi_tablename', 'new_field');
$migration->dropField('glpi_tablename', 'old_field');
$migration->changeField('glpi_tablename', 'field', 'field', 'integer');
```
