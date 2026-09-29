# RGAA Topic 7 — Scripts and Interactive Components

Priority criteria for JavaScript (jQuery/vanilla) and GLPI interactive components.

## ARIA Foundational Rules

Five rules that govern every use of ARIA. Apply them before writing any role, attribute, or property.

### Rule 1 — Prefer native HTML
Use a native element (`<button>`, `<input>`, `<details>`) instead of a custom ARIA component wherever possible. ARIA equivalents are always slightly less reliable: keyboard shortcuts may break, screen reader shortcuts may not apply.

**Justified exceptions:** styling constraints, inconsistent browser support, immutable HTML that can only be patched with JS.

### Rule 2 — Never override native semantics
Changing an element's role removes it from AT navigation shortcuts (heading list, list navigation, etc.).

```twig
{# ❌ breaks heading navigation — screen reader users lose the FAQ in their heading list #}
<h3 role="button" tabindex="0" aria-expanded="false">What if I lose my item?</h3>

{# ✅ nest — h3 keeps its heading role, button gets its button role #}
<h3><button type="button" aria-expanded="false">What if I lose my item?</button></h3>
```

### Rule 3 — Keyboard operability is non-negotiable
Any element given an interactive ARIA role must receive `tabindex="0"` and respond to keyboard events. See [Criterion 7.3](#criterion-73--keyboard-operability) below.

### Rule 4 — Do not suppress semantics of visible interactive elements
`role="presentation"` cancels the semantic of an element **and all its children**. `aria-hidden="true"` hides an element from assistive technologies regardless of visual display state.

```twig
{# ❌ role="presentation" on the li strips semantics from the button inside #}
<ul role="tablist">
  <li role="presentation">
    <button role="tab" aria-controls="panel-1" aria-selected="false">Tab 1</button>
  </li>
</ul>

{# ✅ either let li be neutral, or apply the tab role directly on li #}
<ul role="tablist">
  <li>
    <button role="tab" aria-controls="panel-1" aria-selected="false">Tab 1</button>
  </li>
</ul>
```

Safe use of `aria-hidden`: hide purely decorative elements (icon fonts, duplicate SVG) from AT.

```twig
{# ✅ icon hidden, visible text carries the label #}
<button type="button">
  <span class="fa fa-trash" aria-hidden="true"></span>
  Delete
</button>
```

### Rule 5 — Every interactive element must have an accessible name
See [RGAA Forms — Accessible Name Computation](./rgaa-forms.md).

---

## Criterion 7.1 — Compatibility with Assistive Technologies

Every interactive JS component must expose its role, state, and value via ARIA.

**General rule:** if a non-native element (`<div>`, `<span>`) is made interactive, assign it the appropriate ARIA `role`.

| Component | ARIA Role |
|---|---|
| Custom button | `role="button"` + `tabindex="0"` |
| Tabs | `role="tablist"` > `role="tab"` > `role="tabpanel"` |
| Modal | `role="dialog"` + `aria-modal="true"` |
| Dropdown menu | `role="menu"` > `role="menuitem"` |
| Dynamic alert | `role="alert"` or `aria-live="polite"` |

**Basic rule:** prefer native HTML elements (`<button>`, `<dialog>`) over custom ARIA components.

## Criterion 7.3 — Keyboard Operability

Every feature triggered by a mouse event must have a keyboard equivalent.

**Anti-pattern:**
```javascript
// ❌ mouse click only
$('.dropdown-toggle').on('click', function() { openDropdown(this); });
```

**Fix:**
```javascript
// ✅ click + enter/space
$('.dropdown-toggle').on('click keydown', function(e) {
    if (e.type === 'click' || e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        openDropdown(this);
    }
});
```

## Focus Management — Modals

On opening a modal: move focus inside the modal.
On closing: return focus to the triggering element.
While open: trap focus inside (Tab/Shift+Tab cycle within the modal).
Escape key: closes the modal.

**Pattern:**
```javascript
// ✅ focus on open
function openModal(modal, trigger) {
    modal.setAttribute('aria-hidden', 'false');
    modal.removeAttribute('hidden');
    const firstFocusable = modal.querySelector('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
    if (firstFocusable) firstFocusable.focus();
    modal._trigger = trigger;
}

// ✅ focus on return
function closeModal(modal) {
    modal.setAttribute('aria-hidden', 'true');
    modal.setAttribute('hidden', '');
    if (modal._trigger) modal._trigger.focus();
}

// ✅ Escape to close
document.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') {
        const openModal = document.querySelector('[role="dialog"]:not([hidden])');
        if (openModal) closeModal(openModal);
    }
});
```

**Anti-pattern in Twig:**
```twig
{# ❌ modal without aria-modal or aria-labelledby #}
<div class="modal" id="confirm-modal">
  <div class="modal-content">...</div>
</div>
```

**Fix:**
```twig
{# ✅ complete ARIA attributes #}
<div class="modal"
     id="confirm-modal"
     role="dialog"
     aria-modal="true"
     aria-labelledby="confirm-modal-title"
     hidden>
  <h2 id="confirm-modal-title">Confirm deletion</h2>
  <div class="modal-content">...</div>
  <button type="button" class="modal-close" aria-label="Close">×</button>
</div>
```

### Modal with long content — `role="document"` pattern

`role="dialog"` automatically switches screen readers into application mode (form mode): arrow keys are captured by the browser, standard reading shortcuts are lost. For modals containing navigable content (long text, headings, lists), restore reading mode inside the modal with `role="document"`.

```twig
{# ✅ role="document" restores arrow-key reading navigation inside the dialog #}
<div role="dialog"
     tabindex="-1"
     aria-modal="true"
     aria-labelledby="modal-title"
     hidden>
  <button type="button" class="modal-close" aria-label="Close">×</button>
  <div role="document" tabindex="0">
    <h2 id="modal-title">Ticket details</h2>
    <p>Long description content that users need to read with arrow keys...</p>
  </div>
</div>
```

**Why `tabindex="0"` on the document div:** it makes the wrapper focusable so screen readers recognize the role switch and restore navigation mode automatically. Without it, NVDA and JAWS may not apply the `document` role.

---

## ARIA States — Components

Interactive component states must be updated dynamically.

| State | ARIA Attribute | Values |
|---|---|---|
| Open/closed (dropdown, accordion) | `aria-expanded` | `"true"` / `"false"` |
| Selected (tab, option) | `aria-selected` | `"true"` / `"false"` |
| Checked (custom checkbox) | `aria-checked` | `"true"` / `"false"` |
| Disabled | `aria-disabled` | `"true"` |
| Required | `aria-required` | `"true"` |
| Invalid | `aria-invalid` | `"true"` |

**jQuery pattern:**
```javascript
// ✅ update state on interaction
$('.accordion-trigger').on('click', function() {
    const expanded = $(this).attr('aria-expanded') === 'true';
    $(this).attr('aria-expanded', !expanded);
    $(this).next('.accordion-panel').toggle();
});
```

## Criterion 7.4 — Predictable Context Changes

Any automatically triggered context change (e.g. partial reload, redirect) must be announced or under user control.

Use `aria-live` to announce dynamic updates without a page reload:

```twig
{# ✅ live notification region #}
<div aria-live="polite" aria-atomic="true" class="sr-only" id="live-region"></div>
```

```javascript
// ✅ inject the message into the live region
document.getElementById('live-region').textContent = 'Ticket #1234 updated successfully.';
```
