---
name: glpi-js
description: GLPI JavaScript / TypeScript conventions to apply when reading, writing or editing any *.js or *.ts file in a GLPI core or plugin codebase. Enforces TypeScript for type safety, ES modules for scope isolation and deferred loading (never IIFE), Vue 3 composition API for components, and prohibits introducing new jQuery code (legacy jQuery still exists in core but no new usage). Activates whenever the model is about to read or modify GLPI front-end JS/TS.
---

# JavaScript/TypeScript Rules for GLPI

## General
- TypeScript for type safety
- Never introduce jQuery code (legacy jQuery exists in GLPI core, but no new usage)
- Use ES modules for scope isolation and deferment — never IIFE
- Vue 3 composition API for new components
