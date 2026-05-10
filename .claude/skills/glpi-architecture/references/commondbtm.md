# CommonDBTM — Inheritance & Hooks

All GLPI items inherit from `CommonDBTM`. Key hook methods:

| Hook | Purpose | Return |
|------|---------|--------|
| `prepareInputForAdd($input)` | Validate/transform before INSERT | `$input` or `false` to abort |
| `prepareInputForUpdate($input)` | Validate/transform before UPDATE | `$input` or `false` to abort |
| `post_addItem()` | Side effects after INSERT | void |
| `post_updateItem($history)` | Side effects after UPDATE | void |
| `pre_deleteItem()` | Checks before DELETE | `true` to proceed |
| `post_deleteItem()` | Cleanup after DELETE | void |

## Inheritance Hierarchy

```
CommonDBTM
├── CommonDropdown          # Simple dropdowns (Location, Category...)
│   └── CommonTreeDropdown  # Hierarchical dropdowns
├── CommonDBRelation        # M:N relations
├── CommonDBChild           # 1:N child items
├── CommonITILObject        # Ticket, Problem, Change
└── Asset                   # Computer, Monitor, NetworkEquipment...
```
