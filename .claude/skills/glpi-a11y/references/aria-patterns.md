# ARIA Authoring Practices — Component Patterns

Authoritative keyboard interactions and ARIA requirements for interactive components found in GLPI.
Source: W3C ARIA Authoring Practices Guide (https://www.w3.org/WAI/ARIA/apg/patterns/)

Load this file when the code contains: combobox, autocomplete, grid, tabs, disclosure/accordion, listbox, or breadcrumb components.

---

## Combobox (search field with autocomplete)

Typical GLPI usage: assignee search, asset type selector, category picker.

**ARIA structure**
```html
<input role="combobox"
       aria-controls="listbox-id"
       aria-expanded="false"
       aria-haspopup="listbox"
       aria-autocomplete="list"
       aria-activedescendant="">

<ul role="listbox" id="listbox-id">
  <li role="option" aria-selected="false" id="opt-1">Option 1</li>
</ul>
```

**Keyboard interactions — Input**
| Key | Action |
|---|---|
| `↓` | Opens popup; focuses first option |
| `↑` | Opens popup; focuses last option |
| `Alt+↓` | Opens popup without moving focus |
| `Alt+↑` | Closes popup |
| `Enter` | Accepts focused option |
| `Escape` | Closes popup; optionally clears field |

**Keyboard interactions — Listbox popup**
| Key | Action |
|---|---|
| `↓` / `↑` | Moves between options |
| `Home` / `End` | First / last option |
| `Enter` | Selects option; closes popup |
| `Escape` | Closes popup; returns focus to input |
| Printable char | Jumps to matching option |

**Critical rule:** DOM focus stays on the input at all times. Move AT focus via `aria-activedescendant` — never move DOM focus to list items.

**Anti-pattern:**
```javascript
// ❌ moving DOM focus to list items
listItem.focus();
```

**Correct pattern:**
```javascript
// ✅ update aria-activedescendant on input, keep DOM focus there
input.setAttribute('aria-activedescendant', listItem.id);
input.setAttribute('aria-expanded', 'true');
```

---

## Grid (interactive data table)

Typical GLPI usage: ticket list, asset inventory, search results with sortable columns.

**Difference from `<table>`:** a grid is interactive — cells can contain focusable widgets (checkboxes, buttons, links). Use `role="grid"` only when cells are interactive, not for display-only tables (use `<table>` with `<th scope>` instead).

**ARIA structure**
```html
<table role="grid" aria-label="Open tickets" aria-rowcount="150">
  <thead>
    <tr role="row">
      <th role="columnheader" aria-sort="ascending" scope="col">Title</th>
      <th role="columnheader" scope="col">Status</th>
    </tr>
  </thead>
  <tbody>
    <tr role="row" aria-rowindex="1">
      <td role="gridcell"><a href="#">Ticket #1234</a></td>
      <td role="gridcell">Open</td>
    </tr>
  </tbody>
</table>
```

**Keyboard interactions**
| Key | Action |
|---|---|
| `→` / `←` | Move one cell horizontally |
| `↓` / `↑` | Move one cell vertically |
| `Home` | First cell in current row |
| `End` | Last cell in current row |
| `Ctrl+Home` | First cell in first row |
| `Ctrl+End` | Last cell in last row |
| `Page Down/Up` | Move several rows |
| `Shift+Space` | Select entire row |
| `Ctrl+Space` | Select entire column |

**Critical rule:** only one grid cell in the tab sequence at a time. All navigation uses arrow keys. The grid itself gets `tabindex="0"`; individual cells get `tabindex="-1"` and receive focus programmatically.

---

## Tabs

Typical GLPI usage: ticket detail tabs (Description / Solution / Follow-ups / Assets).

**ARIA structure**
```html
<div role="tablist" aria-label="Ticket details">
  <button role="tab"
          aria-selected="true"
          aria-controls="panel-desc"
          id="tab-desc">Description</button>
  <button role="tab"
          aria-selected="false"
          aria-controls="panel-sol"
          id="tab-sol"
          tabindex="-1">Solution</button>
</div>

<div role="tabpanel" id="panel-desc" aria-labelledby="tab-desc">...</div>
<div role="tabpanel" id="panel-sol"  aria-labelledby="tab-sol" hidden>...</div>
```

**Keyboard interactions**
| Key | Action |
|---|---|
| `→` | Next tab (wraps to first) |
| `←` | Previous tab (wraps to last) |
| `Home` | First tab |
| `End` | Last tab |
| `Tab` | Moves focus out of tablist into active panel |
| `Space` / `Enter` | Activates focused tab (if not auto-activated) |

**Critical rules:**
- Only the active tab has `tabindex="0"`. All others: `tabindex="-1"`.
- Inactive panels must have `hidden` attribute — not just `display:none` via CSS.
- Prefer automatic activation (activating on arrow key focus) when panel content loads without delay.

**Anti-pattern:**
```twig
{# ❌ tabs without role or keyboard management #}
<ul class="tabs">
  <li class="active"><a href="#desc">Description</a></li>
  <li><a href="#sol">Solution</a></li>
</ul>
```

---

## Disclosure (expandable section / accordion)

Typical GLPI usage: collapsible form sections, filter panels, advanced options.

**ARIA structure**
```html
<!-- ✅ button controls visibility -->
<button type="button"
        aria-expanded="false"
        aria-controls="section-content">
  Advanced options
</button>

<div id="section-content" hidden>
  <!-- content -->
</div>
```

**Keyboard interactions**
| Key | Action |
|---|---|
| `Enter` / `Space` | Toggle visibility |

**Critical rules:**
- The trigger must be a `<button>` (or element with `role="button"`). Never use `<div>` or `<a>`.
- Toggle `aria-expanded` and `hidden` together — not just CSS classes.
- `aria-controls` is optional but recommended for screen reader navigation.

**Anti-pattern:**
```twig
{# ❌ div trigger, CSS-only visibility #}
<div class="accordion-trigger" onclick="toggle()">Advanced options</div>
<div class="accordion-content" style="display:none">...</div>
```

**Correct pattern:**
```javascript
// ✅ toggle both aria-expanded and hidden
trigger.addEventListener('click', function() {
    const expanded = this.getAttribute('aria-expanded') === 'true';
    this.setAttribute('aria-expanded', !expanded);
    document.getElementById(this.getAttribute('aria-controls'))
            .toggleAttribute('hidden');
});
```

---

## Listbox (selectable list)

Typical GLPI usage: multi-select filter lists, observer/watcher selectors, group membership.

**ARIA structure — single select**
```html
<ul role="listbox" aria-label="Priority" aria-activedescendant="opt-2">
  <li role="option" id="opt-1" aria-selected="false">Low</li>
  <li role="option" id="opt-2" aria-selected="true">Medium</li>
  <li role="option" id="opt-3" aria-selected="false">High</li>
</ul>
```

**ARIA structure — multi-select**
```html
<ul role="listbox" aria-label="Observers" aria-multiselectable="true">
  <li role="option" aria-selected="true">Alice</li>
  <li role="option" aria-selected="false">Bob</li>
</ul>
```

**Keyboard interactions**
| Key | Action |
|---|---|
| `↓` / `↑` | Move focus between options |
| `Home` / `End` | First / last option |
| `Space` | Toggle selection (multi-select) |
| `Shift+↓/↑` | Extend selection (multi-select) |
| `Ctrl+A` | Select all (multi-select) |
| Printable char | Jump to matching option |

**Critical rule:** keep DOM focus on the listbox container; move the visual/AT focus indicator via `aria-activedescendant`. Do not use both `aria-selected` and `aria-checked` — pick one.

---

## Breadcrumb

Typical GLPI usage: entity hierarchy navigation, asset location path.

**ARIA structure**
```twig
{# ✅ nav landmark with label + aria-current on last item #}
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="{{ path('root') }}">Home</a></li>
    <li><a href="{{ path('computers') }}">Computers</a></li>
    <li><a href="{{ path('computer', {id: item.id}) }}" aria-current="page">
      {{ item.name }}
    </a></li>
  </ol>
</nav>
```

**Keyboard interactions:** standard link navigation only — no special keyboard pattern.

**Critical rules:**
- Must use `<nav>` landmark with a distinct `aria-label` (to distinguish from main navigation).
- Last item (current page) must have `aria-current="page"`.
- Use `<ol>` (ordered list) — breadcrumbs are ordered by hierarchy.
