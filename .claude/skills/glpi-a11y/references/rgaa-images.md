# RGAA Topic 1 — Images

Priority criteria for images and SVG in GLPI templates.

## Criterion 1.1 — alt attribute present

Every `<img>` tag must have an `alt` attribute, even if empty.

**Anti-pattern:**
```twig
{# ❌ no alt attribute #}
<img src="{{ asset('img/logo.png') }}">
```

**Fix:**
```twig
{# ✅ alt present (informative) #}
<img src="{{ asset('img/logo.png') }}" alt="GLPI — IT asset management">

{# ✅ empty alt (decorative) #}
<img src="{{ asset('img/separator.png') }}" alt="">
```

## Criteria 1.2 / 1.3 — Decorative vs informative images

| Type | Treatment |
|---|---|
| Image conveying information | descriptive `alt` of the information transmitted |
| Decorative image (background, separator, redundant icon) | `alt=""` |
| Image-link | `alt` = link destination |
| Image-button | `alt` = button action |

**Rule:** if the image is the only thing inside an `<a>` or `<button>`, its `alt` is the label of the link/button.

```twig
{# ✅ image-link: alt = destination #}
<a href="{{ path('glpi_home') }}">
  <img src="{{ asset('img/logo.png') }}" alt="GLPI Home">
</a>
```

## Criterion 1.6 — Image with caption

If an image is accompanied by a visible caption, use `<figure>` + `<figcaption>` and link them with `aria-labelledby`.

```twig
{# ✅ figure + figcaption #}
<figure>
  <img src="{{ asset('img/chart.png') }}"
       alt="Chart: 47 open tickets this month"
       aria-labelledby="chart-caption">
  <figcaption id="chart-caption">Open tickets — April 2026</figcaption>
</figure>
```

## Accessible SVG

Inline SVG elements conveying information must be correctly labelled.

**Anti-pattern:**
```twig
{# ❌ SVG without title or role #}
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <path d="..."/>
</svg>
```

**Fix:**
```twig
{# ✅ informative SVG #}
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"
     role="img"
     aria-labelledby="icon-title-1">
  <title id="icon-title-1">Download file</title>
  <path d="..."/>
</svg>

{# ✅ decorative SVG #}
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"
     aria-hidden="true"
     focusable="false">
  <path d="..."/>
</svg>
```

## FontAwesome icons

FontAwesome icons (`<i class="fa fa-*">`) must have `aria-hidden="true"` when decorative, and be accompanied by visible text or an `aria-label` on the parent element when they carry meaning.

```twig
{# ✅ decorative FA icon (text present) #}
<button type="button">
  <i class="fa fa-save" aria-hidden="true"></i>
  Save
</button>

{# ✅ informative FA icon (no visible text) #}
<button type="button" aria-label="Save">
  <i class="fa fa-save" aria-hidden="true"></i>
</button>
```
