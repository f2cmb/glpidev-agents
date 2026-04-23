# RGAA Topic 7 — Scripts and Interactive Components

Priority criteria for JavaScript (jQuery/vanilla) and GLPI interactive components.

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
