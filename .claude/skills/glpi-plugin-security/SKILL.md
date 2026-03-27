---
name: glpi-plugin-security
description: GLPI plugin security audit - entry point auth, CSRF, SQLi, XSS, mass assignment, object instantiation, file upload, path traversal, SSRF, vendor exposure — with GLPI 10 vs 11 migration notes
user-invocable: false
disable-model-invocation: true
---

# GLPI Plugin Security Patterns

Security audit reference based on verified CVEs and security research for `glpi-project/glpi` and its plugin ecosystem.

**Sources**: GitHub Security Advisories (glpi-project), Quarkslab, SensePost, Synacktiv, Lexfo, Almond Offensive.

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

## 1. Entry Point Authentication (S1, S2)

### GLPI 10 (plugin must call explicitly)

Every `front/*.php` and `ajax/*.php` must open with a guard **before any logic**:

```php
Session::checkLoginUser();                              // minimum
Session::checkRight('plugin_pluginname_object', READ); // preferred
```

### GLPI 11 (firewall strategy — opt-out model)

Default behaviour is authenticated. A plugin that needs a **public** endpoint must declare it:

```php
// setup.php
Glpi\Http\Firewall::addPluginStrategyForLegacyScripts(
    'pluginname',
    ['front/public_endpoint.php'],
    \Glpi\Http\Firewall::STRATEGY_NO_CHECK
);
```

**If no strategy is declared in GLPI 11, authentication is enforced automatically.**

**Audit check**: In GLPI 10 plugins, every `front/` and `ajax/` file must have `checkLoginUser`/`checkRight`. In GLPI 11 plugins, flag files that declare `STRATEGY_NO_CHECK` — verify they truly need public access.

**Real examples**:
- Cartography plugin: `front/upload.php` accepted unauthenticated file uploads → RCE (CVE-2022-34128)
- FusionInventory: `front/send_inventory.php` exposed unauthenticated `call_user_func_array()` → arbitrary PHP call (CVE-2019-10477)
- Barcode plugin: `front/send.php`, no auth, no path restriction → arbitrary file read (CVE-2021-43778)

---

## 2. CSRF Protection (S3, S4)

### Declaration in setup.php (mandatory)

```php
$PLUGIN_HOOKS[Hooks::CSRF_COMPLIANT]['pluginname'] = true;
```

**In GLPI < 0.84, a single plugin missing this hook disables CSRF protection system-wide** — for all forms in all plugins and core.

### Validation in POST handlers

```php
Session::checkCSRF($_REQUEST);   // before any $_POST processing
Html::checkAllDatas();           // alternative
```

Forms must use `Html::closeForm()` (not raw `</form>`) to embed the token.

**Vulnerable**:
```php
if (isset($_POST['update'])) {
    $item->update($_POST); // no CSRF validation
}
```

---

## 3. SQL Injection (S5)

### Safe — `$DB->request()` array syntax

```php
$result = $DB->request([
    'FROM'  => MyObject::getTable(),
    'WHERE' => [
        'id'          => (int) $id,
        'entities_id' => $_SESSION['glpiactive_entity'],
    ],
]);
```

### Safe — `$DB->insert()` / `$DB->update()` / `$DB->delete()`

```php
$DB->insert(MyObject::getTable(), ['name' => $name]);
$DB->update(MyObject::getTable(), ['name' => $name], ['id' => $id]);
$DB->delete(MyObject::getTable(), ['id' => $id]);
```

### Vulnerable — `doQuery()` with concatenation

```php
// CRITIQUE — direct injection
$DB->doQuery("SELECT * FROM glpi_plugin_foo WHERE id=" . $_GET['id']);
$DB->doQuery("SELECT * FROM glpi_plugin_foo WHERE name='" . $_POST['name'] . "'");
```

`doQuery()` has zero parameterization — any variable is injectable.

### Vulnerable — `filter_input()` bypass

`filter_input()` reads directly from PHP superglobals **before GLPI's sanitisation pipeline** has processed them:

```php
// VULNERABLE — bypasses GLPI's sanitisation, raw user input
$name = filter_input(INPUT_POST, 'name');
$DB->doQuery("SELECT * FROM table WHERE name='$name'");
```

**This is a GLPI-specific vector** confirmed in FusionInventory (Synacktiv, 2021).

### Vulnerable — Second-order SQLi via session

User preferences stored in the DB are loaded back into `$_SESSION` by `Session::init()` on login **without additional sanitisation**. A first write that injects SQL into a stored preference triggers on next login:

```php
// Attacker writes malicious JSON into a user preference field (e.g. savedsearches_pinned)
// → stored in DB via a mass assignment / first-order injection
// → on next login, Session::init() loads it back → second-order SQLi
```

**Real CVE**: CVE-2023-43813 — SQLi in saved searches via session loading.

### Vulnerable — `SimpleXMLElement` escaping bypass (GLPI 10)

`dbEscapeRecursive()` checks `is_string()` and `is_array()` but not XML objects. Inventory endpoints that parse XML into `SimpleXMLElement` create objects that pass through unsanitised:

```php
// GLPI inventory XML parsing path → SimpleXMLElement bypasses dbEscapeRecursive()
// CVE-2025-24799: pre-auth SQLi via inventory endpoint
```

---

## 4. XSS Prevention (S6, S7)

### Output in Twig

```twig
{{ item.name }}       {# auto-escaped — safe #}
{{ item.content|raw }} {# raw — only for trusted/sanitised HTML — flag in audit #}
```

### Output in PHP

```php
echo htmlspecialchars($value, ENT_QUOTES, 'UTF-8'); // safe
echo Html::entities_to_text($value);                 // safe

echo $_GET['name'];              // VULNERABLE
echo $item->fields['content'];   // VULNERABLE if user-supplied
```

### GLPI-specific: `unclean_cross_side_scripting_deep()`

This function **decodes** HTML entities (converts `&lt;` back to `<`). It is used in some legacy contexts and is the opposite of XSS protection. Any plugin calling it on user data before output is vulnerable (CVE-2020-11036).

### Stored XSS via `prepareInput*`

```php
// Sanitise before storage
public function prepareInputForAdd($input)
{
    return Sanitize::sanitize($input); // GLPI 10
}
```

In GLPI 11, there is no global sanitisation — plugins must sanitise explicitly.

### GLPI-specific: JSON AJAX responses with `Content-Type: text/html`

AJAX endpoints that return JSON but declare `Content-Type: text/html` make stored XSS in item names executable in browser context (CVE-2020-11062). Check AJAX handlers that output JSON.

---

## 5. Mass Assignment (S8)

Passing `$_POST` directly to `CommonDBTM::add()` or `update()` without a field whitelist allows attackers to write arbitrary DB columns:

```php
// VULNERABLE — mass assignment
$item->add($_POST);
$item->update($_POST);

// Safe — whitelist in prepareInputForAdd / prepareInputForUpdate
public function prepareInputForAdd($input)
{
    return array_intersect_key($input, array_flip(['name', 'content', 'entities_id']));
}
```

**Attack impact**: Write to `entities_id`, `profiles_id`, `is_active`, `api_token`, `password_forget_token`, or any security-sensitive column. This enables privilege escalation and second-order SQLi (see above).

**Real CVEs**: CVE-2023-41322 (technician overwrites admin `api_token` via API PUT), SensePost 2024 (mass assignment → profile escalation → RCE chain).

---

## 6. Arbitrary Object Instantiation (S9)

`getItemForItemtype()` instantiates any class passed by name with no whitelist:

```php
// Core pattern — vulnerable when $itemtype comes from user input
$item = getItemForItemtype($_POST['itemtype']);
```

Attacker supplies a class name like `GuzzleHttp\Client` (bundled in GLPI vendor):
- `GuzzleHttp\Client::__call()` proxies to `request()` → **SSRF** from server
- Other vendor classes may have more dangerous gadget chains

**Check**: Any plugin endpoint that passes `$_GET`/`$_POST` to `getItemForItemtype()`, `new $classname()`, or `class_exists($classname)` then instantiates it.

**Real CVEs**: CVE-2021-21327 (unauthenticated), CVE-2024-27098 (authenticated → SSRF via GuzzleHttp).

---

## 7. File Upload Security (S10)

```php
// Correct — use GLPI Document class
$doc = new Document();
$doc->add([
    'name'       => $_FILES['file']['name'],
    'upload_dir' => GLPI_DOC_DIR,
    '_filename'  => $_FILES['file']['tmp_name'],
]);
```

**Vulnerable patterns**:

```php
// CRITIQUE — arbitrary upload
move_uploaded_file($_FILES['file']['tmp_name'], GLPI_DOC_DIR . '/' . $_FILES['file']['name']);

// CRITIQUE — bypassable extension check (.php5, .phtml, .phar)
if (pathinfo($name, PATHINFO_EXTENSION) !== 'php') { move_uploaded_file(...); }

// CRITIQUE — uploads stored in web-accessible directory
move_uploaded_file(..., GLPI_ROOT . '/plugins/foo/uploads/' . $filename);
```

**GLPI-specific escalation path**: SQLi into `glpi_documenttypes` table to add `.php` as allowed extension → converts SQLi into RCE-capable file upload.

**Real CVEs**: CVE-2022-34128 (Cartography, unauthenticated upload), CVE-2023-42802 (unauthenticated PHP script execution), CVE-2024-37149 (technician → plugin loader hijack via upload).

---

## 8. Path Traversal in File-Serving Endpoints (S11)

Plugins that serve files via `front/send.php` or similar commonly concatenate user input with a base path:

```php
// VULNERABLE
$file = $_GET['file'];
readfile(GLPI_DOC_DIR . '/' . $file); // traversal: ../../etc/passwd

// VULNERABLE — regex bypass with backslash on Windows or URL encoding
if (preg_match('/\.\.\//', $file)) { die(); }
// Bypass: ..%2f or ..\ or ../../\\..\\..
```

**Correct pattern**:
```php
$file = $_GET['file'];
$real  = realpath(GLPI_DOC_DIR . '/' . $file);
$base  = realpath(GLPI_DOC_DIR);
if ($real === false || strpos($real, $base . DIRECTORY_SEPARATOR) !== 0) {
    Http::sendErrorResponse(403);
}
readfile($real);
```

**Real CVEs**: CVE-2021-43778 (Barcode — no restriction), CVE-2021-43779 (Addressing), Glpiinventory LFI via `b/deploy/index.php` (backslash bypass).

---

## 9. SSRF Prevention (S12)

```php
// Vulnerable — unvalidated URL
$response = file_get_contents($_POST['url']);
curl_setopt($ch, CURLOPT_URL, $_POST['webhook_url']);

// Minimum: scheme validation
$parsed = parse_url($url);
if (!in_array($parsed['scheme'] ?? '', ['http', 'https'], true)) {
    throw new \InvalidArgumentException('Invalid URL scheme');
}
// Stronger: allowlist of authorized hosts
```

**GLPI-specific vectors**:
- RSS/external calendar features: fetch user-supplied URLs server-side → blind SSRF
- Webhook configuration: admin-controlled URLs to internal services (CVE-2026-22247)
- `getItemForItemtype()` + `GuzzleHttp\Client` (see S9)

---

## 10. Input Sanitization (S13)

```php
// GLPI 10 — Sanitize all POST/GET arrays
$input = Sanitize::sanitize($_POST);

// Integer IDs — always cast and validate
$id = (int) ($_GET['id'] ?? 0);
if ($id <= 0) {
    Html::displayErrorAndDie('Invalid ID');
}

// Avoid filter_input() — bypasses GLPI's pipeline
// $name = filter_input(INPUT_POST, 'name'); // DO NOT USE
```

---

## 11. Access Rights Verification (S14)

```php
// Correct — checks global profile rights
$item->can($id, READ);
$item->can($id, UPDATE);
$item->can($id, PURGE);

if (!Session::haveRight('plugin_pluginname_object', READ)) {
    Html::displayRightError();
}

// Incorrect — bypasses global profile check
$item->canViewItem();    // do NOT use as access gate
$item->canUpdateItem();  // do NOT use as access gate
$item->canDeleteItem();  // do NOT use as access gate
```

**Real CVEs**: CVE-2023-28855 (Fields plugin — any authenticated user writes to any container), CVE-2023-34107 (missing permission check → KnowbaseItems exposure).

---

## 12. Vendor Library Exposure (S15)

Plugins that bundle third-party libraries must ensure test/debug scripts are not web-accessible.

**Pattern to flag**:
- `vendor/` directory under plugin root that is web-accessible
- Test scripts: `*Test.php`, `*Demo.php`, `*Example.php` in vendor packages
- Libraries with known CVEs: check `composer.json` for unmaintained dependencies

**Real CVE**: CVE-2022-35914 — `vendor/htmlawed/htmlawed/htmLawedTest.php` accessible from GLPI root, unauthenticated RCE (CVSS 9.8, exploited in the wild).

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
