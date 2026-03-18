# Website Security Audit
Generated: 2026-03-18

---

## CRITICAL

### 1. Open Redirect via startsWith() Whitelist Bypass
**File:** subscribe.html:135, manage.html:108
**Root cause:** Stripe URL validation uses `startsWith()` which only checks the beginning of the URL. An attacker who controls the Edge Function response could return `https://checkout.stripe.com.attacker.com/malicious` — this passes the check but redirects to an attacker-controlled domain.
**Risk:** Users redirected to phishing pages styled to look like Stripe. Credentials or payment details captured by attacker.

### 2. Supabase Project ID Hardcoded in Public Source
**File:** subscribe.html:66, manage.html:48, admin.html:376
**Root cause:** The Supabase project ID `ycoesxmrbwtcsoyixqjp` is hardcoded in plain JavaScript, publicly readable in the GitHub Pages source.
**Risk:** Attacker can enumerate Edge Function endpoints, target known Supabase CVEs, perform reconnaissance on storage buckets and DB schema.

---

## HIGH

### 1. Admin Panel Exposed Publicly with Password-Only Auth
**File:** admin.html (entire file)
**Root cause:** admin.html is on a public GitHub Pages site with only a client-side password check (`secret-input`). robots.txt disallows it but robots.txt is not a security control — any attacker ignores it.
**Risk:** Brute-force the admin secret → full access to farmer moderation queue, ban/unban farmers, approve/reject photos.

### 2. Admin Secret Has No Rate Limiting or Lockout
**File:** admin.html:431-436
**Root cause:** The admin login has no rate limiting, no account lockout, no CAPTCHA. Validation is client-side string comparison. Unlimited brute-force attempts possible.
**Risk:** Weak or medium-strength secret can be brute-forced with no throttle.

### 3. Session Token Passed in URL Hash
**File:** subscribe.html:81-90, manage.html:57-65
**Root cause:** Auth token passed via URL hash. `history.replaceState()` clears it from the address bar but if the page crashes, reloads, or is bookmarked before replaceState executes, the token is in browser history or system logs.
**Risk:** Token exposure in browser history, crash dumps, or debugging tools. Session tokens in URLs violate security best practices.

### 4. No CSRF Protection on Edge Function Calls
**File:** subscribe.html:120, manage.html:94, admin.html:415
**Root cause:** Bearer token sent to Edge Functions with no state parameter or same-origin verification. No check that the token belongs to the current session.
**Risk:** If an attacker can craft a URL with a valid token in the hash, they can trigger authenticated actions on behalf of the victim.

---

## MEDIUM

### 1. No Content Security Policy (CSP)
**File:** All HTML files
**Root cause:** No CSP meta tags or headers. Inline scripts run without restriction. No external script source allowlist.
**Risk:** If GitHub Pages CDN or DNS is compromised, or any injected content reaches the page, scripts execute with full page privileges.

### 2. Photo URL Path Traversal in Admin Panel
**File:** admin.html:394-406
**Root cause:** Photo URL validation checks hostname and protocol via `new URL()` but only uses `startsWith()` for pathname. A URL like `https://ycoesxmrbwtcsoyixqjp.supabase.co/storage/v1/object/public/../../secret-file` could pass validation.
**Risk:** An attacker who can control the `photo_url` field in the moderation queue could access unintended storage paths or trigger SSRF.

### 3. No HSTS Header
**File:** All HTML files
**Root cause:** No Strict-Transport-Security header. GitHub Pages serves HTTPS but does not enforce it on first visit.
**Risk:** HTTP downgrade attack on first visit if user types `http://` — tokens transmitted over HTTP can be intercepted.

### 4. Admin Secret Vulnerable to Memory Inspection
**File:** admin.html:385, 435-436
**Root cause:** Admin secret stored as a plain string in a JavaScript closure. Accessible via browser debugger breakpoints or memory dumps.
**Risk:** Local attacker or malware with browser access can extract the secret from the closure scope.

---

## LOW / INFO

### 1. robots.txt Advertises Admin Panel Existence
**File:** robots.txt:5
**Root cause:** `Disallow: /admin.html` explicitly tells attackers something sensitive is at that path.
**Risk:** Low — but advertising the path is counterproductive. Not listing it at all is better.

### 2. Supabase Edge Function URL Structure Is Predictable
**File:** subscribe.html:116, manage.html:90
**Root cause:** Endpoints follow standard Supabase pattern `/functions/v1/create-checkout-session` etc. Enumerable.
**Risk:** Low — endpoints require valid auth tokens, but pattern is guessable.

### 3. No SRI Hashes on External Resources
**File:** All HTML files
**Root cause:** No Subresource Integrity hashes. No external scripts currently loaded, so low risk now.
**Risk:** If external libraries (Stripe.js, analytics) are added in future without SRI, a CDN compromise could inject malicious code.

### 4. JWT Format Validated Client-Side Only (Structure, Not Signature)
**File:** subscribe.html:76-77, manage.html:52-53
**Root cause:** Frontend checks JWT has three base64url segments but does not verify signature or expiry. Comment in code acknowledges this — Supabase validates server-side.
**Risk:** Acceptable IF backend validates correctly. A security assumption worth noting.

---

## CLEAN (checked, no issue found)

- XSS in admin.html: all user-controlled data rendered via `textContent`, not `innerHTML` ✅
- All external links use `https://` ✅
- External links in new tabs use `rel="noopener"` ✅
- No API keys, secrets, or credentials in HTML comments ✅
- No localStorage used — only sessionStorage (cleared on tab close) ✅
- No hidden .env or config files with secrets in the repo ✅
- No iframes or frames (no clickjacking surface) ✅
- CNAME correctly configured ✅
