---
name: glpi-plugin-security
description: GLPI plugin security audit - entry point auth, CSRF, SQLi, XSS, mass assignment, object instantiation, file upload, path traversal, SSRF, vendor exposure — with GLPI 10 vs 11 migration notes
user-invocable: false
---

# GLPI Plugin Security Patterns

Security audit reference based on verified CVEs and security research for `glpi-project/glpi` and its plugin ecosystem.

**Sources**: [GitHub Security Advisories — glpi-project/glpi](https://github.com/glpi-project/glpi/security/advisories), Quarkslab, SensePost, Synacktiv, Lexfo, Almond Offensive.

**Detailed patterns and code examples**: see [`checks.md`](checks.md) — 18 sections with vulnerable/safe examples and real CVEs for each check.

---

## GLPI 10 vs GLPI 11 — Security Model Differences

Critical context before auditing: the security model changed significantly between versions.

| Area | GLPI 10 | GLPI 11 |
|------|---------|---------|
| **Auto-sanitisation** | `$_GET`/`$_POST` sanitised globally by `includes.php` | **Removed.** No global sanitisation — each plugin handles its own input. |
| **Entry point auth** | No default auth — plugins must call `Session::checkLoginUser()` manually | **Default: authenticated.** Scripts require a session unless the plugin opts out with `Glpi\Http\Firewall::addPluginStrategyForLegacyScripts()`. |
| **Webroot** | Optional | Required: webroot = `public/` directory; direct access to non-public PHP blocked. |
| **SQL escaping** | Applied via `dbEscapeRecursive()` in global pipeline | Applied automatically by `$DB->request()` query builder — `doQuery()` has no protection. |
| **`Sanitize` class** | `Sanitize::sanitize()` used globally | Class restructured; global pipeline removed. |

**GLPI 10 → 11 migration risk**: Plugins that relied on `includes.php` sanitisation silently lose that protection. Plugins that called `addslashes()` manually now double-escape data. Both are bugs.

---

## Security Checklist

| # | Check | Pattern | Severity if missing |
|---|-------|---------|---------------------|
| S1 | Auth on `front/` entry points | `Session::checkLoginUser()` / `checkRight()` at top (GLPI 10), or `STRATEGY_NO_CHECK` justified (GLPI 11) | CRITIQUE |
| S2 | Auth on `ajax/` entry points | Same as above | CRITIQUE |
| S3 | CSRF hook declared | `PLUGIN_HOOKS[Hooks::CSRF_COMPLIANT]` in setup.php | MAJEUR |
| S4 | CSRF validated in POST handlers | `Session::checkCSRF()` before any `$_POST` use | MAJEUR |
| S5 | No raw SQL concatenation | `$DB->request()` array syntax — no string concat in `doQuery()` | CRITIQUE |
| S6 | No `filter_input()` feeding SQL | `filter_input()` bypasses GLPI pipeline | MAJEUR |
| S7 | No second-order SQLi via preferences | User preference fields sanitised before storage | MAJEUR |
| S8 | XSS — output escaped | `htmlspecialchars()` / Twig auto-escape / no `unclean_cross_side_scripting_deep()` on output | MAJEUR |
| S9 | XSS — stored (prepareInput*) | Explicit sanitisation before DB write (GLPI 11 has no global pipeline) | MAJEUR |
| S10 | No mass assignment | `prepareInputForAdd`/`prepareInputForUpdate` whitelist fields — no raw `add($_POST)` | MAJEUR |
| S11 | No arbitrary object instantiation | `getItemForItemtype()` / `new $class()` not fed from user input | MAJEUR |
| S12 | File upload via Document class | No raw `move_uploaded_file` — extension checks are bypassable | CRITIQUE |
| S13 | Path traversal in file-serving | `realpath()` + prefix check before any `readfile()`/`include()` | CRITIQUE |
| S14 | No SSRF via user-supplied URLs | Scheme validation + optional host allowlist | MAJEUR |
| S15 | Rights via `can()` | Never use `canViewItem()`/`canUpdateItem()` as access gates | MAJEUR |
| S16 | Vendor library exposure | No web-accessible test/debug scripts in `vendor/` | CRITIQUE |
| S17 | SRI on external scripts/styles | `integrity` + `crossorigin` on all CDN assets, or embed locally | MAJEUR |
| S18 | Timing-safe secret comparison | `hash_equals()` for all secret/token comparisons — never `===`/`!==` | MAJEUR |
| S19 | No secrets in logs | Mask API keys, passwords, HMAC secrets before `Toolbox::logDebug` | MINEUR |
| S20 | No open redirect | `Html::back()` instead of raw `HTTP_REFERER` — validate user-supplied redirect URLs | MINEUR |
| S21 | Rate limiting on public endpoints | No unauthenticated endpoint writes to DB/logs on every request without limit | MAJEUR |
| S22 | No PII in logs | Phone numbers, emails, names masked or excluded from log messages | MINEUR |
