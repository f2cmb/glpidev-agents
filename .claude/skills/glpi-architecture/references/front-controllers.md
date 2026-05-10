# Front Controllers — Thin Routing Layer

Front controllers (`front/*.form.php`) handle routing and display only:

```
front/problem.form.php
├── POST path  → $item->add/update/delete($_POST)
└── else path  → Html::header() → $item->display() → Html::footer()
```

**What belongs in front controllers:**
- HTTP method routing (POST vs display)
- Authentication / `Session::checkRight()`
- `Html::header()` / `Html::footer()`
- Calling `$item->add()`, `$item->update()`, `$item->display()`

**What does NOT belong in front controllers:**
- Input normalization (scalar → array, format transformations)
- Business logic, validation, side effects
- Data transformation for linked items

Input normalization belongs in `prepareInputForAdd()` / `prepareInputForUpdate()`. These hooks are called by `$item->add()` / `$item->update()` regardless of entry point (front controller, API, tests), making the logic testable and consistent.

**Rule of thumb**: if a fix in a front controller can't be tested at the class level, the logic is probably in the wrong layer.
