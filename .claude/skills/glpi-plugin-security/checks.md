# GLPI Plugin Security — Detailed Check Patterns

Full descriptions, code examples, and real CVEs for each check in the security checklist.
Reference: [GitHub Security Advisories — glpi-project/glpi](https://github.com/glpi-project/glpi/security/advisories)

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

## 4. XSS Prevention (S8, S9)

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

### Inert `data-*` attributes — do not build a per-context sanitizer

`data-*` attributes are **inert**: the browser triggers no behaviour from them (unlike `on*`, `src`,
`href`, `style`). Allowing specific `data-*` attributes through the HTML sanitizer **globally** does
not widen the XSS surface, so a context-specific sanitizer just to permit them is complexity with no
security gain.

```php
// ❌ Two HtmlSanitizerConfig variants just to allow data-* in one context
private static function getHtmlSanitizer(bool $allow_video_embeds): HtmlSanitizer {
    if ($allow_video_embeds) {
        return self::$kb_sanitizer ??= self::buildConfig(allow_data: true);  // duplicated state
    }
    return self::$sanitizer ??= self::buildConfig(allow_data: false);
}

// ✅ Allow the inert data-* attributes once, in the single shared config
private static function getHtmlSanitizer(): HtmlSanitizer {
    return self::$sanitizer ??= self::buildConfig();   // allows data-video-* everywhere — still safe
}
```

**Where the real defense lives:** dynamic content is validated **at the point of render**, not by
multiplying sanitizer configs. The renderer rebuilds the trusted markup (e.g. an `<iframe>`) from a
**provider allowlist** + a **strict ID regex** — the sanitizer never has to police `data-*` values.

**Audit rule:** flag any new `HtmlSanitizerConfig`/sanitizer variant whose only difference is the set
of allowed `data-*` attributes. Ask: "is this branch protecting anything, or duplicating state for an
inert attribute?" Push the validation to the render point (allowlist + regex) instead.

---

## 5. Mass Assignment (S10)

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

## 6. Arbitrary Object Instantiation (S11)

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

## 7. File Upload Security (S12)

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

## 8. Path Traversal in File-Serving Endpoints (S13)

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

## 9. SSRF Prevention (S14)

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
- `getItemForItemtype()` + `GuzzleHttp\Client` (see S11)

---

## 10. Input Sanitization (S6, S7)

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

## 11. Access Rights Verification (S15)

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

## 12. Vendor Library Exposure (S16)

Plugins that bundle third-party libraries must ensure test/debug scripts are not web-accessible.

**Pattern to flag**:
- `vendor/` directory under plugin root that is web-accessible
- Test scripts: `*Test.php`, `*Demo.php`, `*Example.php` in vendor packages
- Libraries with known CVEs: check `composer.json` for unmaintained dependencies

**Real CVE**: CVE-2022-35914 — `vendor/htmlawed/htmlawed/htmLawedTest.php` accessible from GLPI root, unauthenticated RCE (CVSS 9.8, exploited in the wild).

---

## 13. External Script Integrity (S17)

Plugins that load JavaScript or CSS from external CDNs must include Subresource Integrity (SRI) attributes.

**Vulnerable**:
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"></script>
<script src="https://unpkg.com/vis-network@9.1.9/standalone/umd/vis-network.min.js"></script>
```

**Correct**:
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"
        integrity="sha384-..."
        crossorigin="anonymous"></script>
```

Without SRI, a CDN compromise or DNS hijack executes attacker JavaScript in the session of any authenticated GLPI admin viewing the page — full account takeover.

**Preferred alternative**: embed the library locally in the plugin (`public/lib/`), removing the CDN dependency entirely.

**Check**: any `<script src="https://...">` or `<link href="https://...">` without `integrity` attribute.

---

## 14. Timing-Safe Secret Comparisons (S18)

PHP's `===` and `!==` operators are vulnerable to timing side-channel attacks when comparing secret values (tokens, HMAC signatures, API keys). An attacker with network access can measure response time differences to recover secrets character by character.

```php
// Vulnerable — timing attack possible
if ($request_key !== $expected_key) { ... }
if ($token !== $expected) { ... }
if ($provided_hmac == $computed_hmac) { ... }

// Correct — constant-time comparison
if (!hash_equals($expected_key, $request_key)) { ... }
if (!hash_equals($expected, $token)) { ... }
if (!hash_equals($computed_hmac, $provided_hmac)) { ... }
```

**Applies to**: webhook secret validation, HMAC signature verification, API token comparison, CSRF token fallback validation, any user-supplied secret compared to a stored value.

---

## 15. Secrets in Debug Logs (S19)

Logging sensitive values (API keys, passwords, HMAC secrets, tokens) to disk creates a persistent exposure risk.

**Vulnerable**:
```php
// Logs full URL including client_secret as query param
Toolbox::logDebug("API request: {$url}?client_secret={$secret}");

// Logs HMAC signing input containing secret
Toolbox::logDebug("Sign input: {$toSign}");

// Logs full credentials
error_log("LDAP bind: user={$user} pass={$password}");
```

**Correct**:
```php
// Mask or truncate sensitive values
Toolbox::logDebug("API request to: " . preg_replace('/client_secret=[^&]+/', 'client_secret=***', $url));
Toolbox::logDebug("Sign input length: " . strlen($toSign));
```

**Check**: any `Toolbox::logDebug`, `error_log`, `Toolbox::logWarning` call that includes variable values that may contain secrets, passwords, tokens, or private keys — especially in API client classes and authentication handlers.

---

## 16. Open Redirect (S20)

Redirecting to a user-controlled URL without validation allows phishing attacks against authenticated users.

**Vulnerable**:
```php
// HTTP_REFERER is client-controlled
Html::redirect($_SERVER['HTTP_REFERER'] ?? $CFG_GLPI['root_doc']);

// User-supplied return URL
Html::redirect($_GET['redirect']);
```

**Correct**:
```php
// Use GLPI's built-in back() which validates the referer against the base URL
Html::back();

// Or validate explicitly
$url = $_GET['redirect'] ?? '';
if (strpos($url, $CFG_GLPI['url_base']) === 0) {
    Html::redirect($url);
} else {
    Html::redirect($CFG_GLPI['root_doc']);
}
```

**Note**: `Html::back()` is the standard GLPI pattern for redirecting to the previous page — prefer it over manual `HTTP_REFERER` handling.

---

## 17. DoS via Unauthenticated or Rate-Unlimited Endpoints (S21)

Webhooks and other public endpoints that perform writes (DB, filesystem, logs) on every request without rate limiting are vulnerable to denial-of-service by flooding.

**Vulnerable pattern**:
```php
// front/webhook.php — no rate limiting, writes on every request
$raw = file_get_contents('php://input');
Toolbox::logInFile('plugin_webhook', $raw);       // disk saturation
$history->add(['event' => $data['event'], ...]);  // DB saturation
```

**Risk factors** (cumulative):
- No authentication or optional-only secret → anyone can flood
- DB write on every request → table growth, lock contention
- Log write on every request → disk saturation
- No payload size limit → memory exhaustion

**Mitigations to check for**:
```php
// Payload size limit
if ((int)($_SERVER['CONTENT_LENGTH'] ?? 0) > 1_048_576) {
    http_response_code(413); exit;
}

// Log only in debug mode
if ($CFG_GLPI['debug_sql'] ?? false) {
    Toolbox::logInFile('plugin_webhook', $raw);
}
```

**Check**: any endpoint registered as stateless (`STRATEGY_NO_CHECK`) or unauthenticated (GLPI 10) that writes to DB, logs, or filesystem on every call without rate limiting or payload size validation.

---

## 18. PII in Logs (S22)

Personal data (phone numbers, email addresses, names) written to log files is a GDPR exposure risk, distinct from credential logging (S19).

**Vulnerable**:
```php
Toolbox::logError('Failed to link ticket ' . $id . ' to phone ' . $phone);
Toolbox::logDebug('Sending notification to ' . $email);
Toolbox::logInFile('webhook', json_encode($payload)); // payload contains contact data
```

**Correct**:
```php
// Truncate or anonymise PII
Toolbox::logError('Failed to link ticket ' . $id . ' to phone ***' . substr($phone, -4));
Toolbox::logDebug('Notification queued for user ID ' . $users_id);
```

**Applies to**: plugins processing WhatsApp/SMS numbers, emails, contact data. Check webhook handlers, API clients, sync classes, cron jobs.

---

## 19. iframe Sandbox Least-Privilege (S23)

When embedding third-party content in an `<iframe sandbox="...">`, every token re-enables a capability
the empty sandbox removed. The burden of proof is on the author: **test without the token first, add
it only if the embedded content actually breaks.** A narrative comment ("needed for cross-origin
playback") is not proof — only "it fails without it" is.

```html
<!-- ❌ over-permissive: tokens added "to be safe", justified by a comment -->
<iframe src="https://www.youtube-nocookie.com/embed/abc"
        sandbox="allow-scripts allow-same-origin allow-presentation allow-popups"></iframe>

<!-- ✅ least-privilege: only what the player provably needs -->
<iframe src="https://www.youtube-nocookie.com/embed/abc"
        sandbox="allow-scripts allow-presentation"></iframe>
```

### Token risk table

| Token | What it grants | Audit note |
|-------|----------------|------------|
| `allow-scripts` | JS execution inside the frame | Usually required for video players |
| `allow-same-origin` | frame keeps its origin → can reach parent DOM/storage if same-origin | **Sensitive.** Only if the embed is genuinely same-origin and needs it |
| `allow-popups` | frame can open new windows/tabs | Rarely needed — drop unless the player opens external links |
| `allow-presentation` | Presentation API (fullscreen casting) | Low risk; keep only if used |

### Red flag — `allow-same-origin` + `allow-scripts` together

With **both** tokens, a same-origin framed document can run scripts *and* reach the embedding page —
effectively escaping the sandbox (it can remove its own `sandbox` attribute and reload). For a
cross-origin embed (`youtube.com`, `vimeo.com`), `allow-same-origin` is almost never required: the
`src` is already a different origin, so the player does not gain same-origin access to *your* page —
it only keeps access to *its own*. Treat the pair as a finding unless proven necessary.

### Also: don't re-emit browser defaults

`referrerpolicy="strict-origin-when-cross-origin"` is the modern-browser default — specifying it adds
noise without effect. Same for other default-valued attributes. Drop them.

**Audit rule:** for each `<iframe sandbox>`, list every token and require a proof-of-necessity. Flag
`allow-same-origin` + `allow-scripts` on a cross-origin embed, and any `allow-popups` without a
demonstrated need. Default = the most restrictive sandbox that still renders the content.
