# RGAA Topic 5 — Tables

Priority criteria for GLPI data tables.

## Criterion 5.4 — Headers declared correctly

Each data column or row must have a `<th>` element with a `scope` attribute.

| scope value | Usage |
|---|---|
| `scope="col"` | Column header |
| `scope="row"` | Row header |
| `scope="colgroup"` | Column group header |
| `scope="rowgroup"` | Row group header |

**Anti-pattern:**
```twig
{# ❌ th without scope #}
<table>
  <tr>
    <th>Name</th>
    <th>Status</th>
    <th>Date</th>
  </tr>
```

**Fix:**
```twig
{# ✅ th with scope #}
<table>
  <thead>
    <tr>
      <th scope="col">Name</th>
      <th scope="col">Status</th>
      <th scope="col">Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Ticket #1234</th>
      <td>Open</td>
      <td>2026-04-23</td>
    </tr>
  </tbody>
</table>
```

## Criterion 5.7 — Table caption

Complex data tables must have a `<caption>` describing their content.

**Anti-pattern:**
```twig
{# ❌ table without caption #}
<table>
  <thead>...</thead>
```

**Fix:**
```twig
{# ✅ caption present #}
<table>
  <caption>List of open tickets — Main entity</caption>
  <thead>...</thead>
</table>
```

## Criterion 5.3 — Layout tables

Tables used for layout purposes (layout tables) must not contain semantic data table elements.

If a table is used solely for layout (legacy case):
```twig
{# ✅ role="presentation" on a legacy layout table #}
<table role="presentation">
  <tr>
    <td>...</td>
    <td>...</td>
  </tr>
</table>
```

## Complex tables — multiple headers

For tables with headers spanning multiple levels, use `id` + `headers`:

```twig
{# ✅ explicit cell ↔ header binding #}
<table>
  <thead>
    <tr>
      <th id="col-name" scope="col">Name</th>
      <th id="col-status" scope="col">Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td headers="col-name">Ticket #1234</td>
      <td headers="col-status">Open</td>
    </tr>
  </tbody>
</table>
```
