# GLPI Plugin Security — Audit Commands

Grep / find commands to run for each check S1–S22 against a plugin directory `{PLUGIN_DIR}`.
For each finding: read the flagged file in full, then cross-reference against the vulnerable patterns documented in [`checks.md`](checks.md).

---

## S1, S2 — Entry Point Authentication

```bash
# Find entry points
find {PLUGIN_DIR}/front {PLUGIN_DIR}/ajax -name "*.php" 2>/dev/null | sort

# Which ones have an auth check
grep -rln "Session::check\|checkLoginUser\|checkRight" {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null

# Which ones opt out in GLPI 11
grep -rn "STRATEGY_NO_CHECK\|STRATEGY_FAQ_ACCESS" {PLUGIN_DIR}/ 2>/dev/null
```

Read each `front/` and `ajax/` file individually. The check must appear **before any logic** — not inside a conditional branch.

---

## S3, S4 — CSRF

```bash
grep -n "CSRF_COMPLIANT" {PLUGIN_DIR}/setup.php
grep -rn "checkCSRF\|checkAllDatas\|Html::closeForm" {PLUGIN_DIR}/front/ 2>/dev/null

# Flag forms that use raw </form> instead of Html::closeForm()
grep -rn "</form>" {PLUGIN_DIR}/templates/ {PLUGIN_DIR}/front/ 2>/dev/null
```

---

## S5 — SQL Injection

```bash
# Raw doQuery with variables
grep -rn "doQuery" {PLUGIN_DIR}/ --include="*.php"

# filter_input usage (bypasses GLPI sanitisation pipeline)
grep -rn "filter_input" {PLUGIN_DIR}/ --include="*.php"
```

Read surrounding code for every `doQuery()` hit containing a variable. For `filter_input()` hits, trace whether the result flows into a DB operation.

---

## S6 — filter_input bypass (input sanitization)

Covered by the `filter_input` grep above (S5). Cross-reference: `checks.md` § 10.

---

## S7 — Second-order SQLi via preferences

```bash
# Direct $_GET/$_POST usage feeding DB operations
grep -rn '\$_GET\|\$_POST\|\$_REQUEST' {PLUGIN_DIR}/src/ {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null

# User preferences stored as structured data (second-order SQLi risk)
grep -rn "savedsearches\|preferences_save\|config_context\|user_pref" {PLUGIN_DIR}/src/ 2>/dev/null
```

---

## S8, S9 — XSS

```bash
# Direct echo of variables
grep -rn "echo \$\|echo \$_\|print \$" {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ {PLUGIN_DIR}/src/ 2>/dev/null

# Twig raw filter
grep -rn "|raw" {PLUGIN_DIR}/templates/ 2>/dev/null

# Dangerous GLPI function that DECODES entities (opposite of protection)
grep -rn "unclean_cross_side_scripting_deep" {PLUGIN_DIR}/ --include="*.php"

# prepareInputForAdd / prepareInputForUpdate — check for Sanitize calls
grep -rn "prepareInputFor" {PLUGIN_DIR}/src/ 2>/dev/null

# AJAX endpoints with Content-Type text/html returning JSON
grep -rn "Content-Type.*text/html\|json_encode" {PLUGIN_DIR}/ajax/ 2>/dev/null
```

---

## S10 — Mass Assignment

```bash
# Dangerous: direct $_POST to CommonDBTM methods
grep -rn "->add(\$_POST\|->update(\$_POST\|->add(\$_REQUEST\|->update(\$_REQUEST" {PLUGIN_DIR}/ --include="*.php"

# Safer but still check: all calls to add() / update() in front/ and ajax/
grep -rn "->add(\|->update(" {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null
```

For each `add($_POST)` / `update($_POST)` hit: check whether `prepareInputForAdd`/`prepareInputForUpdate` exists in the class and whitelists fields.

---

## S11 — Arbitrary Object Instantiation

```bash
# getItemForItemtype with user-controlled input
grep -rn "getItemForItemtype\|new \$" {PLUGIN_DIR}/ --include="*.php"

# class_exists then instantiate
grep -rn "class_exists" {PLUGIN_DIR}/ --include="*.php"
```

Read surrounding code to check whether `$itemtype` / `$classname` originates from user input.

---

## S12 — File Upload

```bash
grep -rn "move_uploaded_file\|tmp_name\|_filename" {PLUGIN_DIR}/ --include="*.php"
grep -rn "PATHINFO_EXTENSION\|\.php\b" {PLUGIN_DIR}/ --include="*.php" | grep -i "ext\|upload\|file"
```

---

## S13 — Path Traversal in File-Serving

```bash
# Find file-serving scripts
find {PLUGIN_DIR} -name "send.php" -o -name "*.send.php" -o -name "download.php" -o -name "icon*.php" 2>/dev/null

# readfile / file_get_contents / include with user input
grep -rn "readfile\|file_get_contents\|include\b\|require\b" {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null | grep -v "//\|inc/includes\|autoload"

# realpath validation (presence is good, absence is bad)
grep -rn "realpath" {PLUGIN_DIR}/ --include="*.php"
```

Read every file-serving script in full.

---

## S14 — SSRF

```bash
grep -rn "file_get_contents\|curl_setopt\|curl_exec\|Requests::\|GuzzleHttp\|Http::request" {PLUGIN_DIR}/src/ {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null
grep -rn "parse_url\|filter_var.*URL\|FILTER_VALIDATE_URL" {PLUGIN_DIR}/ --include="*.php"
```

Flag any HTTP request where the URL is derived from user input without host/scheme validation.

---

## S15 — Rights Verification

```bash
grep -rn "canViewItem\|canUpdateItem\|canDeleteItem" {PLUGIN_DIR}/src/ {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null
grep -rn "Session::haveRight\|->can(\b\|checkRight\b" {PLUGIN_DIR}/ --include="*.php" | head -30
```

---

## S16 — Vendor Library Exposure

```bash
# vendor directory presence and web-accessible test scripts
find {PLUGIN_DIR}/vendor -name "*Test*" -o -name "*Demo*" -o -name "*Example*" 2>/dev/null | grep "\.php$"
find {PLUGIN_DIR}/vendor -name "htmLawed*" 2>/dev/null

# Check if .htaccess or equivalent protects vendor/
ls {PLUGIN_DIR}/vendor/.htaccess 2>/dev/null || echo "No .htaccess in vendor/"
cat {PLUGIN_DIR}/public/.htaccess 2>/dev/null | head -10
```

---

## S17 — External Script Integrity (SRI)

```bash
# External CDN scripts/styles without integrity attribute
grep -rn "src=['\"]https://\|href=['\"]https://" {PLUGIN_DIR}/templates/ {PLUGIN_DIR}/front/ {PLUGIN_DIR}/src/ 2>/dev/null | grep -v "integrity"
```

Flag any `<script src="https://...">` or `<link href="https://...">` without an `integrity` attribute. Note whether the library could be embedded locally instead.

---

## S18 — Timing-Safe Secret Comparisons

```bash
# Comparisons on secret values using non-constant-time operators
grep -rn "!== \$.*key\|!== \$.*token\|!== \$.*secret\|!== \$.*hmac\|!== \$.*signature\|== \$.*key\|== \$.*token\|== \$.*secret" {PLUGIN_DIR}/ --include="*.php"

# hash_equals usage (presence is good)
grep -rn "hash_equals" {PLUGIN_DIR}/ --include="*.php"
```

Read the surrounding code for any comparison involving `key`, `token`, `secret`, `hmac`, or `signature` to determine if it should use `hash_equals()`.

---

## S19 — Secrets in Debug Logs

```bash
# Log calls that may include sensitive values
grep -rn "logDebug\|logWarning\|error_log\|Toolbox::log" {PLUGIN_DIR}/src/ {PLUGIN_DIR}/inc/ 2>/dev/null | grep -i "secret\|password\|key\|token\|credential\|sign\|auth"

# Also check API client classes specifically
find {PLUGIN_DIR} -name "*api*" -o -name "*client*" -o -name "*auth*" 2>/dev/null | grep "\.php$" | xargs grep -ln "logDebug\|error_log" 2>/dev/null
```

---

## S20 — Open Redirect

```bash
# HTTP_REFERER used for redirect
grep -rn "HTTP_REFERER" {PLUGIN_DIR}/ --include="*.php"

# User-supplied redirect parameter
grep -rn "Html::redirect\|header.*Location" {PLUGIN_DIR}/ --include="*.php" | grep -i "GET\|POST\|redirect\|back\|return"
```

Flag any `Html::redirect()` or `header('Location: ...')` where the URL is derived from `$_GET`, `$_POST`, or `$_SERVER['HTTP_REFERER']` without validation against `$CFG_GLPI['url_base']`. Preferred pattern: `Html::back()`.

---

## S21 — DoS via Rate-Unlimited Public Endpoints

```bash
# Stateless / unauthenticated endpoints
grep -rn "STRATEGY_NO_CHECK\|GLPI_USE_CSRF_CHECK\|registerPluginStatelessPath" {PLUGIN_DIR}/ --include="*.php"

# DB writes in those endpoints
grep -rn "->add(\|->update(\|->delete(\|doQuery\|logInFile" {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null

# Rate limiting or payload size guards (presence is good)
grep -rn "CONTENT_LENGTH\|ratelimit\|rate_limit\|429\|413" {PLUGIN_DIR}/ --include="*.php"
```

For every stateless/public endpoint found: check whether it writes to DB or logs on every call, and whether any payload size or rate limit guard exists.

---

## S22 — PII in Logs

```bash
# Log calls that may include personal data
grep -rn "logDebug\|logError\|logWarning\|logInFile\|error_log" {PLUGIN_DIR}/src/ {PLUGIN_DIR}/inc/ {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ 2>/dev/null | grep -i "phone\|email\|mail\|name\|contact\|numero\|tel\b\|sms\|whatsapp"

# Webhook handlers and sync classes logging full payloads
find {PLUGIN_DIR} -name "*webhook*" -o -name "*sync*" -o -name "*handler*" 2>/dev/null | grep "\.php$" | xargs grep -ln "logInFile\|logDebug\|error_log" 2>/dev/null
```

---

## Structural Conformance (Phase 3)

```bash
# Deprecated inc/ directory
ls {PLUGIN_DIR}/inc/ 2>/dev/null && echo "DEPRECATED: inc/ found"

# Namespace check
grep -rn "^namespace" {PLUGIN_DIR}/src/ 2>/dev/null | head -10

# Hook constants vs string literals
grep -rn "PLUGIN_HOOKS\['" {PLUGIN_DIR}/setup.php
grep -n "Hooks::CSRF_COMPLIANT\|Hooks::CONFIG_PAGE" {PLUGIN_DIR}/setup.php

# Install pattern
grep -n "Class::install\|->install\|new Migration" {PLUGIN_DIR}/hook.php 2>/dev/null

# PHP 8 patterns
grep -rn "use function Safe\\\\" {PLUGIN_DIR}/src/ 2>/dev/null | head -5

# GPL license headers
grep -rL "GNU General Public License\|GPL" {PLUGIN_DIR}/src/ {PLUGIN_DIR}/inc/ {PLUGIN_DIR}/front/ {PLUGIN_DIR}/ajax/ {PLUGIN_DIR}/setup.php {PLUGIN_DIR}/hook.php 2>/dev/null | grep "\.php$"

# CSS/JS injected after Html::footer() (renders after </body>)
grep -n "Html::footer\|footer()" {PLUGIN_DIR}/front/*.php {PLUGIN_DIR}/inc/*.php 2>/dev/null
```
