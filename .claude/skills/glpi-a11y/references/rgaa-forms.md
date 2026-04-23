# RGAA Topic 11 — Forms

Priority criteria for GLPI forms (PHP/Twig).

## RGAA 11.1 — Does each field have a label?

Every `<input>`, `<select>`, `<textarea>` must have an accessible label.

**Accepted methods (in order of preference):**
1. `<label for="id">` associated by `id`
2. `aria-labelledby="label-id"`
3. `aria-label="label text"`

**Anti-pattern:**
```twig
{# ❌ input without a label #}
<input type="text" name="search" placeholder="Search...">
```

**Fix:**
```twig
{# ✅ explicit label #}
<label for="search">Search</label>
<input type="text" id="search" name="search" placeholder="Search...">

{# ✅ aria-label when a visible label is not possible #}
<input type="text" id="search" name="search" aria-label="Search tickets">
```

## RGAA 11.2 — Is the label correctly linked to its field?

The `for` attribute of the `<label>` must exactly match the `id` of the field.

**Anti-pattern:**
```twig
{# ❌ for does not match the id #}
<label for="name">Last name</label>
<input type="text" id="firstname" name="name">
```

**Fix:**
```twig
{# ✅ for = id #}
<label for="name">Last name</label>
<input type="text" id="name" name="name">
```

## RGAA 11.3 — Is the label relevant?

The label text must describe the nature of the field, not its format.

**Anti-pattern:**
```twig
{# ❌ empty or non-descriptive label #}
<label for="date1">*</label>
<input type="text" id="date1">
```

**Fix:**
```twig
{# ✅ descriptive label #}
<label for="date_creation">Creation date <span aria-hidden="true">*</span></label>
<input type="text" id="date_creation" aria-required="true">
```

## RGAA 11.5 / 11.6 — Grouping fields

Fields of the same nature (e.g. address, date range, title) must be grouped in a `<fieldset>` with a `<legend>`.

**Anti-pattern:**
```twig
{# ❌ related fields without grouping #}
<label for="date_start">From</label>
<input type="date" id="date_start">
<label for="date_end">To</label>
<input type="date" id="date_end">
```

**Fix:**
```twig
{# ✅ fieldset + legend #}
<fieldset>
  <legend>Date range</legend>
  <label for="date_start">From</label>
  <input type="date" id="date_start">
  <label for="date_end">To</label>
  <input type="date" id="date_end">
</fieldset>
```

## RGAA 11.9 — Is the label of each button relevant?

A `<button>` or `<input type="submit">` must have an explicit label.

**Anti-pattern:**
```twig
{# ❌ icon button without an accessible label #}
<button class="btn-icon"><i class="fa fa-trash"></i></button>

{# ❌ empty submit input #}
<input type="submit" value="">
```

**Fix:**
```twig
{# ✅ aria-label on icon button #}
<button class="btn-icon" aria-label="Delete ticket">
  <i class="fa fa-trash" aria-hidden="true"></i>
</button>

{# ✅ explicit value #}
<input type="submit" value="Save">
```

## RGAA 11.10 / 11.11 — Error handling

Validation error messages must be associated with the relevant field.

**Anti-pattern:**
```twig
{# ❌ error not associated with the field #}
<p class="error">This field is required.</p>
<input type="text" id="name" name="name">
```

**Fix:**
```twig
{# ✅ aria-describedby + id on the message #}
<label for="name">Last name <span aria-hidden="true">*</span></label>
<input type="text" id="name" name="name"
       aria-required="true"
       aria-describedby="name-error"
       aria-invalid="true">
<p id="name-error" class="error" role="alert">This field is required.</p>
```

## RGAA 11.13 — Autocomplete for personal identity fields

Fields collecting personal data must carry the `autocomplete` attribute.

| Field | autocomplete value |
|---|---|
| Last name | `family-name` |
| First name | `given-name` |
| Email | `email` |
| Phone | `tel` |
| Organization | `organization` |

**Fix:**
```twig
<input type="email" id="email" name="email" autocomplete="email">
```
