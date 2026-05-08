# JavaScript — Skeleton & GLPI gotchas

This reference covers the three JS realities in a GLPI codebase: modern TypeScript / ES modules, jQuery legacy, and Vue components.

## Section skeleton (used in the body of the produced doc)

```
## Le contexte GLPI
   (Identifie le style en jeu : ES modules / TS, jQuery legacy, ou Vue)

## La mécanique JS en jeu
   ├── Si TypeScript / ES modules : imports, scope isolation, pas d'IIFE
   ├── Si jQuery legacy           : sélecteurs, événements, on/off, $.ajax patterns
   └── Si Vue                     : composant, props, lifecycle, store

## L'interaction avec GLPI
   - API REST GLPI (apirest.php)
   - Globals exposés (CFG_GLPI, etc.)
   - Intégration depuis Twig (data-attributes, scripts en pied de template)

## Pièges courants
## Pour aller plus loin
```

## What goes in each section

**Le contexte GLPI** — first identify the style in play by looking at the file: ES module syntax (`import`/`export`), jQuery (`$()`, `.on(`), or Vue (`<template>`, `defineComponent`). Don't lecture on a style absent from the file.

**La mécanique JS en jeu** — explain only the style(s) actually present. If the file mixes jQuery and ES modules, explain the bridge. Don't introduce TS concepts in a pure jQuery file.

**L'interaction avec GLPI** — REST API entry point (`apirest.php`), global config (`CFG_GLPI`), data passed from Twig via `data-*` attributes, CSRF token handling.

**Pièges courants** — pick the relevant ones below.

**Pour aller plus loin** — related modules, the asset manifest, equivalent patterns elsewhere in the codebase.

## GLPI gotchas to surface (when relevant)

- **No new jQuery code** — convention `js.md` says: never introduce jQuery. If you read jQuery, treat it as legacy to maintain, not a model.
- **No IIFE** — ES modules give scope; IIFE is obsolete here.
- **TypeScript preferred for new code** — for type safety.
- **Vue mounting in non-SPA pages** — GLPI is mostly server-rendered. A Vue component lives in a specific mount point in a Twig template; lifecycle and hydration must be reasoned about explicitly.
- **REST auth** — calls to `apirest.php` need proper session/token handling. Don't reinvent.
- **CSRF** — POST endpoints require the CSRF token. Don't forge requests without it.

## Anti-patterns to flag in debrief mode

- **New jQuery in a diff** → violates `js.md`. Refactor to ES module / TS.
- **IIFE introduced** (`(function(){...})()`) → use ES module scope.
- **Inline `<script>` in a new Twig file** → must be a separate JS/TS module.
- **`window.foo = ...`** (global pollution) → expose via module exports or a registered namespace.
- **Hand-rolled fetch to GLPI without CSRF** → use the established helper.
- **Vue introduced where a small TS module would do** → YAGNI.
- **TS rewrite of legacy jQuery without functional reason** → leave it; YAGNI applies here too.
