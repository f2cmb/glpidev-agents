# Build chain — Skeleton & GLPI gotchas

## Section skeleton (used in the body of the produced doc)

```
## Le contexte GLPI
## Le pipeline (Webpack / scripts npm / composer)
## Le manifest et l'intégration core ↔ plugins
## Pièges courants
## Pour aller plus loin
```

## What goes in each section

**Le contexte GLPI** — what build artefact is in play (compiled JS, compiled CSS, manifest), where it's consumed (Twig templates loading via `data-glpi-asset` or equivalent).

**Le pipeline** — entry config, npm scripts, watch mode, prod vs dev outputs. Cite the actual config files.

**Le manifest et l'intégration core ↔ plugins** — how a plugin registers its assets so they're loaded on the right pages. The core asset pipeline does not magically pick up plugin files.

**Pièges courants** — pick from the list.

**Pour aller plus loin** — related config files, the loader code that consumes the manifest.

## GLPI gotchas to surface (when relevant)

- **Dev vs prod** — different bundles, different cache strategies. A change that works in dev may break in prod minification.
- **Cache busting** — manifests use hashed filenames. Hardcoding a path that bypasses the manifest breaks deploys.
- **Plugin asset registration** — a plugin must declare its assets explicitly; copying files into core directories is wrong.
- **HMR / watch** — if the user is editing live, the watch must include the right paths. Otherwise edits don't apply.
- **Source maps** — should be off in prod, on in dev.

## Anti-patterns to flag in debrief mode

- Hardcoded asset URL bypassing the manifest.
- Plugin writing files directly into a core asset directory.
- New entry added to the bundle without justification (bundle bloat).
- Source maps enabled in a prod build path.
- `process.env.NODE_ENV` checks scattered instead of centralized.
