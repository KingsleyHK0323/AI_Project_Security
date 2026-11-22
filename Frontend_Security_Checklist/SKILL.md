---
name: frontend-security-checklist
description: A structured, production-grade security checklist for browser-based web frontends, including single-page apps, server-rendered and static sites, and progressive web apps (React, Vue, Angular, Next.js, Nuxt, Svelte, and vanilla JavaScript). Use this skill whenever the user builds, reviews, audits, or hardens web UI code, browser-side logic, or front-end delivery configuration, even if they do not explicitly say "security". Trigger on requests such as "is this XSS-safe", "review my React component", "where should I store the JWT", "set up a CSP", "which security headers do I need", "audit my web app", "pre-release frontend security check", or any mention of innerHTML, dangerouslySetInnerHTML, v-html, DOMPurify, open redirect, prototype pollution, localStorage tokens, CSRF, SameSite, CSP, HSTS, clickjacking, SRI, npm supply chain, source maps, service workers, postMessage, WebSocket, Permissions-Policy, cookie consent, or Error Boundary. Covers 10 domains and 37 verifiable controls with severity ratings and a standard report format.
license: MIT
metadata:
  author: Kingsley (@KingsleyHK0323)
  repository: https://github.com/KingsleyHK0323
  version: "1.0.0"
  domain: application-security
  companion-skills: frontend-app-security-checklist, backend-security-checklist, backend-app-security-checklist
---

# Frontend Security Checklist

A practical, defense-in-depth security checklist for web applications that run in the browser. It consolidates the controls that most often decide whether a web frontend resists attack: injection and output encoding, authentication and session handling, HTTP security headers, supply chain and build hygiene, client-side storage and offline behavior, cross-window and real-time messaging, browser API permissions, abuse resistance, privacy compliance, and operations.

The skill supports three jobs: **designing** web features securely, **reviewing** existing code, delivery configuration, and build output, and **producing a release-gate checklist** for a team.

## Relationship to companion skills

This skill covers **browser-based web frontends**.

- Use `frontend-app-security-checklist` for **native mobile, hybrid, and desktop clients** (Keychain/Keystore storage, certificate pinning, deep links, Electron and Tauri hardening, WebView bridges). When a native shell hosts web content, apply that skill to the shell and this skill to the web content.
- Use `backend-security-checklist` for **server-side** controls. Many frontend controls (CSRF, CORS, file validation, rate limiting, authorization) are only effective when the server enforces them, so cross-reference it for those findings.
- Use `backend-app-security-checklist` when the backend serves mobile or consumer apps and app-specific concerns apply.

Do not duplicate findings across skills; cite the control ID that best fits.

## Core principles

Apply these principles throughout. They explain *why* the controls exist and help you make sound judgments when a situation does not match the checklist exactly.

1. **The browser is an untrusted runtime.** Users, extensions, DevTools, and any injected script can read and modify everything the page holds. Client-side checks improve usability; they are never an enforcement point.
2. **Prevent script injection first.** Cross-site scripting (XSS) nullifies most other frontend defenses, including token storage choices and CSRF protections. Treat output encoding, sanitization, and Content Security Policy as the foundation.
3. **Keep data out of the client when possible.** What is never sent to or stored in the browser cannot be stolen from it. Minimize, expire, and clear sensitive data.
4. **Use the platform's defenses.** Prefer built-in browser mechanisms (HttpOnly cookies, SameSite, CSP, Trusted Types, SRI, Permissions-Policy) to hand-rolled equivalents.
5. **Every dependency is code you ship.** Third-party packages, CDN scripts, and tag managers run with the full privileges of your origin. Inventory, pin, and constrain them.
6. **Defense in depth.** Layer controls so that one failure (a missed sanitization, a compromised package) does not become a full compromise.
7. **Collect only what is needed, and say why.** Telemetry, cookies, and permissions carry privacy and regulatory obligations.

## When to use this skill

Use it when the user is:

- Writing or reviewing components, templates, routing, forms, or state management that handle untrusted data.
- Deciding how to store tokens, configure cookies, or protect against CSRF and XSS.
- Configuring HTTP security headers, CSP, CORS interaction, or CDN and hosting settings.
- Adding dependencies, third-party scripts, analytics, tag managers, or build tooling.
- Building PWAs, service workers, real-time features, cross-window messaging, or embedded widgets.
- Preparing a release, pull request, or compliance review and wants a security gate.

Do not use it for native mobile client questions (use `frontend-app-security-checklist`), for penetration testing or exploit development, or for formal compliance certification. See [Limitations and responsible use](#limitations-and-responsible-use).

## Workflow

### Step 1: Establish scope and context

Identify, from the conversation and any provided code, what is in scope. Gather only what you cannot infer:

- **Stack:** framework and version, rendering model (SPA, SSR, SSG, hybrid), bundler, package manager.
- **Authentication model:** cookie sessions, JWT, OAuth/OIDC, a backend-for-frontend (BFF), or third-party identity.
- **Sensitive assets:** tokens, PII, payment data, admin capabilities, user-generated HTML.
- **Delivery:** hosting, CDN, reverse proxy, where headers are set, and who controls them.
- **Mode of work:** **Design guidance**, **Code review**, or **Release checklist**.

If the user provided code, build configuration, header configuration, or `package.json`, read them before answering. Base findings on evidence, not assumptions.

### Step 2: Select the relevant domains

Not every domain applies to every task. Map the request to domains, and state which you are covering and why.

| Task | Prioritize domains |
|---|---|
| Rendering user content, rich text, markdown, URLs | 1, 3 |
| Login, tokens, cookies, sessions, logout | 2, 5, 3 |
| Setting up headers, CSP, hosting, or CDN | 3, 10 |
| Adding packages, scripts, analytics, or changing the build | 4, 9 |
| PWA, offline mode, caching, local persistence | 5, 2 |
| iframes, widgets, postMessage, WebSocket | 6, 3 |
| Camera, microphone, location, clipboard, notifications | 7, 3 |
| Payments, forms, voting, rate-sensitive actions | 8, 2, 1 |
| Cookie banners, tracking, telemetry | 9, 4 |
| Pre-release gate or full audit | All |

### Step 3: Evaluate each applicable control

For every applicable control in the [Control catalog](#control-catalog), assign one status:

- **Pass**: Evidence shows the control is implemented correctly.
- **Partial**: Implemented, but with gaps (state the gap).
- **Fail**: Missing or incorrectly implemented.
- **N/A**: Not applicable (state why, for example "no service worker").
- **Not verified**: Evidence is not available (for example, response headers or CDN configuration were not provided). Never mark a control as Pass without evidence.

### Step 4: Rate severity

Rate each Fail or Partial finding using exploitability and impact:

| Severity | Meaning | Examples |
|---|---|---|
| **Critical** | Directly exploitable; leads to account takeover, session theft, or arbitrary script execution for other users | Stored XSS through unsanitized HTML; `eval` on attacker-influenced data; compromised or unpinned third-party script on an authenticated page; secret key in the public bundle |
| **High** | Exploitable with modest effort or a precondition; serious impact | Open redirect on a login flow; missing CSRF protection on cookie-authenticated state changes; wildcard `postMessage` origin handling; prototype pollution reachable from untrusted JSON |
| **Medium** | Weakens defenses or leaks limited information | No CSP or a CSP with `unsafe-inline`; sensitive responses cacheable; telemetry capturing PII; source maps exposed in production |
| **Low** | Hardening gap with limited direct impact | Missing `Referrer-Policy`; missing optional isolation headers |
| **Info** | Best-practice recommendation | Documentation and process improvements |

### Step 5: Report and remediate

Deliver the result using the [Report format](#report-format). For every Fail or Partial finding, give a concrete, minimal remediation that fits the user's framework and hosting. Provide short code or configuration examples when they clarify the fix. Order findings by severity, then by effort to fix.

## Control catalog

Each control has an ID for traceability, the requirement, and what to verify. Use the IDs in reports and pull request comments.

---

### Domain 1: Input handling, output encoding, and injection defense

**XSS-01: DOM XSS sink avoidance**
- *Requirement:* Never pass untrusted data to dangerous sinks: `innerHTML`, `outerHTML`, `document.write()`, `insertAdjacentHTML()`, `eval()`, `new Function()`, and `setTimeout()` / `setInterval()` with a string argument. Prefer `textContent` and framework data binding, which encode output by default.
- *Verify:* Search the code, including third-party UI snippets and jQuery calls such as `.html()`, for these sinks and trace each argument back to its source (URL parameters, `location.hash`, `postMessage`, API data, storage). Where supported, consider enforcing **Trusted Types** through CSP to make unsafe sinks fail closed.

**XSS-02: Framework escape hatches and rich-text sanitization**
- *Requirement:* Avoid `dangerouslySetInnerHTML` (React), `v-html` (Vue), and Angular's `bypassSecurityTrust*` APIs on untrusted content. When rich text must be rendered, sanitize it immediately before insertion with **DOMPurify** configured with a strict allowlist of tags and attributes.
- *Verify:* Every escape-hatch usage has a documented, sanitized source; the sanitizer is kept up to date (sanitizer bypasses are found periodically); sanitized output is not modified afterward; markdown renderers are configured to disallow raw HTML or are sanitized after rendering; the server also sanitizes or encodes stored HTML (see `backend-security-checklist`).

**URL-01: URL, redirect, and scheme safety**
- *Requirement:* Validate dynamic `href`, `src`, `action`, and `window.location` targets. Block `javascript:`, `data:`, and `vbscript:` schemes. For in-site redirects, allow only relative paths or domains matched against an allowlist.
- *Verify:* Parse URLs with the `URL` API and compare scheme and origin exactly rather than using string prefix or `includes` checks; reject protocol-relative (`//evil.com`) and backslash variants (`/\evil.com`); check `next`, `returnTo`, and `redirect` query parameters on login and logout flows.

**PROTO-01: Prototype pollution defense**
- *Requirement:* When merging objects or assigning with dynamic keys (for example `obj[key] = value`), reject `__proto__`, `constructor`, and `prototype`. Prefer `Object.create(null)` or `Map` for attacker-keyed data, and validate input shape with a schema library (for example Zod or Yup).
- *Verify:* Review recursive merge, clone, `set`-by-path, and query-string parsing code, especially when fed by untrusted JSON; confirm that utility libraries (for example deep-merge helpers) are on patched versions; check gadget impact in code that reads from plain objects (configuration, feature flags, template options).

**UPLD-01: Client-side form and file pre-checks**
- *Requirement:* The frontend may check file extension, MIME type, and size to improve user experience, but must **never** be the only line of defense. The backend must re-validate content and magic bytes.
- *Verify:* Confirm that the backend performs equivalent checks (see `backend-security-checklist`, FILE-01); confirm previews of user-selected files use object URLs or safe rendering rather than injecting content as HTML.

---

### Domain 2: Authentication, sessions, and access control

**AUTH-01: Token storage strategy**
- *Requirement:* Avoid keeping high-privilege access or refresh tokens in `localStorage` or `sessionStorage`, where any injected script can read them. Prefer session cookies with `HttpOnly; Secure; SameSite`, or short-lived access tokens held only in memory with a secure refresh mechanism. For SPAs, consider a **backend-for-frontend (BFF)** that keeps tokens server-side.
- *Verify:* Be candid about limits: no browser storage option protects against XSS, because injected script can still use the session or call the API. HttpOnly cookies prevent token theft but not session riding, so XSS prevention (Domain 1 and CSP) remains the primary defense. For cookies, consider the `__Host-` prefix and the narrowest scope and lifetime.

**AUTH-02: CSRF defense**
- *Requirement:* When authenticating with cookies, set `SameSite=Lax` or `Strict` (evaluate `SameSite=None; Secure` explicitly for cross-site embedding), and protect critical state-changing operations with an anti-CSRF token or a custom request header check.
- *Verify:* Do not rely on `SameSite` alone, because it does not cover same-site attacks (for example from a vulnerable sibling subdomain) and defaults vary across browsers; state-changing requests never use `GET`; the frontend sends the CSRF token or header on every mutating request; the server also validates `Origin` or `Sec-Fetch-Site` where practical. Enforcement is server-side, so cross-reference the backend skill.

**AUTH-03: Route guards and UI-level permissions**
- *Requirement:* Block unauthenticated or unauthorized users from restricted routes and redirect to sign-in. Hiding buttons and menu items is a UX measure only; the backend API must enforce authorization on every data access and operation.
- *Verify:* Confirm that calling the API directly with a low-privilege session does not grant access; confirm that restricted bundles or data are not shipped to unauthorized users when that matters (client-side code splitting is not a security boundary).

**AUTH-04: Logout and session cleanup**
- *Requirement:* On logout, clear in-memory state, front-end caches, and related Storage and IndexedDB data, and call the backend to revoke the token or session.
- *Verify:* Check state stores (Redux, Pinia, Zustand), data-fetching caches (for example React Query, SWR), service worker caches, Cache Storage, and IndexedDB; consider the `Clear-Site-Data` response header; confirm that back/forward cache restoration does not redisplay authenticated pages after logout.

**AUTH-05: Step-up confirmation for sensitive actions**
- *Requirement:* High-risk operations such as changing a password, binding a device, large transactions, or deleting an account require re-authentication (password or OTP) or explicit confirmation.
- *Verify:* The server enforces a "recently authenticated" condition rather than trusting the UI flow; confirmation prompts show the specific action and target; the flow is resistant to being triggered silently by a link or script.

---

### Domain 3: HTTP security headers and transport security

**HDR-01: Content Security Policy (CSP)**
- *Requirement:* Define a CSP with explicit `default-src 'self'`, `script-src`, `style-src`, `img-src`, `connect-src`, `font-src`, and `frame-ancestors`. Avoid `unsafe-inline` and `unsafe-eval`; use nonces or hashes where inline code is unavoidable.
- *Verify:* Prefer a strict, nonce-based policy (for example with `strict-dynamic`) over long host allowlists, which are frequently bypassable; set `object-src 'none'` and a restrictive `base-uri`; consider `form-action`; roll out first with `Content-Security-Policy-Report-Only` and fix violations; remember `frame-ancestors` and reporting directives are not honored in a `<meta>` tag, so set CSP as a response header.

**HDR-02: Clickjacking protection**
- *Requirement:* Set `X-Frame-Options: DENY` or `SAMEORIGIN`, and use the CSP `frame-ancestors` directive to precisely control which parent origins may embed the page.
- *Verify:* `frame-ancestors` takes precedence in modern browsers, and `X-Frame-Options` serves as a legacy fallback; pages that must be embeddable list only the specific partner origins; sensitive actions are not exposed in embeddable pages.

**HDR-03: MIME sniffing and HTTPS enforcement**
- *Requirement:* Set `X-Content-Type-Options: nosniff`. Serve the whole site over HTTPS and set `Strict-Transport-Security` (HSTS), ideally with `includeSubDomains` and a long `max-age`.
- *Verify:* Plain HTTP redirects to HTTPS; HSTS is introduced gradually (short `max-age` first) and `preload` is added only after confirming every subdomain supports HTTPS, since preload is difficult to reverse; there is no mixed content.

**HDR-04: Cross-origin isolation and referrer controls**
- *Requirement:* As needed, set `Cross-Origin-Opener-Policy` (COOP), `Cross-Origin-Embedder-Policy` (COEP), and `Cross-Origin-Resource-Policy` (CORP). Set `Referrer-Policy` (for example `strict-origin-when-cross-origin`) so sensitive URLs and query parameters do not leak via the `Referer` header.
- *Verify:* COOP `same-origin` is considered for sensitive apps to reduce cross-window attacks; COEP and CORP are used when cross-origin isolation features are required; sensitive tokens and identifiers are never placed in URLs, which also protects history, logs, and referrers.

**HDR-05: Precise CORS configuration**
- *Requirement:* The backend must not combine a wildcard or reflected `Origin` with `Access-Control-Allow-Credentials: true`. Verify allowed origins and methods individually.
- *Verify:* Origin matching is exact; the `null` origin is not allowed; CORS is understood as a browser read-permission mechanism and not an access-control mechanism. Fixes live on the server; cross-reference `backend-security-checklist`, API-03.

---

### Domain 4: Supply chain, third-party packages, and build security

**SUP-01: Dependency vulnerability management**
- *Requirement:* Commit lockfiles (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`). Run `npm audit` / `pnpm audit` in CI or integrate Dependabot, Renovate, or Snyk, and define a remediation SLA for Critical and High vulnerabilities.
- *Verify:* CI installs from the lockfile (for example `npm ci`); audit results are triaged by reachability and not ignored wholesale; transitive dependencies are covered; abandoned packages are replaced.

**SUP-02: Package provenance and malicious install scripts**
- *Requirement:* Review new dependencies for maintenance status, download volume, ownership, and license. Watch for risky `postinstall` scripts and consider `--ignore-scripts` with explicit review.
- *Verify:* Check for typosquatting and recent maintainer changes; check how the package manager in use treats dependency lifecycle scripts, since defaults differ and have changed; consider a minimum release-age policy so brand-new versions are not installed immediately; prefer packages that publish verifiable provenance.

**SUP-03: Subresource Integrity (SRI)**
- *Requirement:* When loading third-party scripts or styles from a CDN, add `integrity="sha384-..."` and `crossorigin="anonymous"`, and keep a fallback or self-hosted copy.
- *Verify:* SRI is applied only to static, versioned files (a URL whose content changes will break); resources that must change dynamically (many tag managers and widgets) are self-hosted, constrained by CSP, or reassessed; pipelines regenerate hashes on upgrade.

**SUP-04: Third-party scripts (analytics, tag managers, chat widgets)**
- *Requirement:* Assess each third-party SDK's permissions and data flows. Do not load an uncontrolled tag manager directly into high-privilege pages. Monitor third-party scripts for performance and behavioral anomalies.
- *Verify:* Maintain an inventory of every script with an owner and justification; authenticated, admin, and payment pages load the minimum set (payment pages carry specific script-inventory and integrity requirements under PCI DSS v4.x where it applies); consider sandboxed iframes for widgets; gate non-essential scripts on consent (see PRIV-01).

**SUP-05: Build artifacts and secret leakage**
- *Requirement:* Confirm that source, environment variables (especially public-prefixed ones such as `VITE_*`, `NEXT_PUBLIC_*`, `REACT_APP_*`), and Git history contain no API secret keys, private keys, internal hostnames, or test credentials. Disable or protect production source maps (`.map`).
- *Verify:* Treat every value with a public prefix as published to all visitors; scan the built bundle, not just the repository; upload source maps to the error-tracking service and do not serve them publicly; confirm leaked secrets are rotated, not just removed.

---

### Domain 5: Client storage, caching, and offline / PWA security

**STOR-01: No sensitive data at rest in the browser**
- *Requirement:* Do not store plaintext passwords, national ID numbers, card numbers, complete tokens, or confidential business data in `localStorage`, `sessionStorage`, IndexedDB, or Cache Storage.
- *Verify:* Inspect Application storage in DevTools after exercising the app; confirm persisted state libraries (for example Redux Persist) do not persist sensitive slices; confirm that anything persisted has an expiry and is cleared on logout (AUTH-04).

**STOR-02: Cache-Control for sensitive responses**
- *Requirement:* For personalized data, billing, and account settings, send `Cache-Control: no-store` (or at minimum `private, no-cache`) so shared computers, proxies, and browser caches retain nothing.
- *Verify:* Check headers on authenticated HTML and API responses; confirm CDNs do not cache personalized responses (check `Vary` and cache keys); confirm static, public assets still get long-lived caching with content hashes.

**STOR-03: Service worker and PWA cache safety**
- *Requirement:* Review the scope and caching strategies of service workers. Do not store sensitive API responses in offline caches. Serve the service worker script itself over HTTPS with a strict update strategy.
- *Verify:* Authenticated responses are excluded from caches or keyed per user and purged on logout; the service worker file is served with `Cache-Control: no-cache` so updates and fixes propagate; `importScripts` does not load third-party code; there is a kill-switch path to unregister a faulty or compromised worker.

---

### Domain 6: Cross-window messaging, WebSocket, and real-time data

**MSG-01: `window.postMessage` validation**
- *Requirement:* Message listeners must strictly verify `event.origin` using exact allowlist comparison (not substring matching), and also verify `event.source` and the message data structure. When sending, specify an explicit `targetOrigin` rather than `"*"` for anything sensitive.
- *Verify:* Search for `addEventListener("message", ...)` handlers lacking origin checks; confirm handlers validate message shape with a schema and never pass message data to dangerous sinks (XSS-01).

**MSG-02: Reverse tabnabbing protection**
- *Requirement:* Add `rel="noopener noreferrer"` to every `target="_blank"` link. Modern browsers imply `noopener` by default, but explicit declaration remains best practice and `noreferrer` additionally suppresses the referrer.
- *Verify:* Check user-generated links and rendered markdown, where the attribute is most often missing; confirm `window.open` calls with untrusted URLs pass `noopener`.

**MSG-03: WebSocket, SSE, and WebRTC security**
- *Requirement:* Use encrypted channels (`wss://`, `https://`). Authenticate connections and validate the `Origin` on the server. Schema-validate pushed messages, and handle replay and race conditions in state updates.
- *Verify:* The server rejects WebSocket handshakes from unexpected origins (cross-site WebSocket hijacking); authentication does not rely on long-lived tokens in query strings, which leak into logs (prefer short-lived tickets or cookie-based sessions with origin checks); realtime message handlers use the same validation as REST responses; WebRTC signaling is authenticated and TURN credentials are short-lived.

---

### Domain 7: Browser APIs, permissions, and device access

**API-01: Least privilege and user intent**
- *Requirement:* Access to camera, microphone, geolocation, clipboard, notifications, or the file system must be triggered by an explicit user interaction (such as clicking a button), with the purpose clearly explained.
- *Verify:* Permission prompts never fire on page load; the app degrades gracefully when permission is denied; permissions are requested at the moment of need, with an explanation first.

**API-02: Permissions-Policy header**
- *Requirement:* Use the HTTP `Permissions-Policy` header to disable browser features the site does not need (for example `camera=(), microphone=(), geolocation=()`), reducing the damage from embedding and XSS.
- *Verify:* The policy lists features explicitly disabled by default; embedded third-party frames are granted only the specific features they need via the `allow` attribute; the deprecated `Feature-Policy` header is not the only mechanism.

**API-03: Clipboard and canvas / WebGL safety**
- *Requirement:* Do not write hidden or malicious commands to the clipboard (clipboard hijacking). When processing cross-origin images with Canvas or WebGL, account for tainted-canvas restrictions and release memory.
- *Verify:* Copy buttons copy exactly what is displayed; cross-origin image requests use CORS (`crossorigin`) correctly rather than disabling protections; large canvas and texture resources are released to avoid resource exhaustion.

---

### Domain 8: Frontend logic, abuse resistance, and resilience

**LOGIC-01: Frontend validation is UX; backend validation is mandatory**
- *Requirement:* Required fields, format, length, numeric ranges, prices, and state transitions must all be re-validated on the backend, to prevent tampering through DevTools or direct API requests.
- *Verify:* For each client-side rule, locate the server-side equivalent; confirm prices, totals, and permissions are never accepted from the client as authoritative.

**LOGIC-02: Duplicate-submission prevention and rate-limit cooperation**
- *Requirement:* For form submission, payment, verification-code sending, lotteries, and "like" actions, the frontend should lock the control while loading, debounce, and send an idempotency key, while the backend enforces rate limits.
- *Verify:* Buttons are disabled during in-flight requests; idempotency keys are generated per user intent and honored by the server; retry logic does not blindly repeat non-idempotent requests; the UI handles `429` responses gracefully.

**LOGIC-03: Regular-expression and recursion safety (ReDoS / DoS)**
- *Requirement:* Avoid complex regular expressions prone to catastrophic backtracking, and unbounded recursive rendering, when processing very large strings.
- *Verify:* Look for nested quantifiers and overlapping alternations in validation patterns; bound input length before matching; treat server-side rendering and Node-based build or edge code as higher risk, since a ReDoS there affects all users rather than one browser; bound recursion depth in tree and comment renderers.

---

### Domain 9: Privacy, cookie consent, and compliance

**PRIV-01: Consent management**
- *Requirement:* According to the regulations of target markets (for example GDPR, CCPA/CPRA, or local privacy law), do not load non-essential tracking cookies or analytics and advertising scripts before user consent is obtained.
- *Verify:* Observe network traffic and cookies on first load before interacting with the banner; scripts are gated on consent state and respect later withdrawal; opt-out signals such as Global Privacy Control are honored where required; consent choices are recorded.

**PRIV-02: Telemetry and log sanitization**
- *Requirement:* Front-end analytics events and error reports must filter out email addresses, phone numbers, tokens, `Authorization` headers, passwords, and sensitive query-string parameters.
- *Verify:* Configure scrubbing hooks in the error-tracking and analytics SDKs; mask form inputs by default in session-replay tools; strip sensitive URL parameters before reporting page URLs; review breadcrumbs and network-capture options.

---

### Domain 10: Error handling, observability, and operations

**OPS-01: Global error boundaries and exception capture**
- *Requirement:* Configure error boundaries (for example React Error Boundary) plus `window.onerror` and `unhandledrejection` handlers to avoid blank pages from uncaught exceptions, without exposing internal paths or stack traces to users.
- *Verify:* Error boundaries do not catch errors in event handlers or asynchronous code, so those paths are covered separately; user-facing messages are generic and carry a correlation ID; production stack traces go to the error tracker, not the page.

**OPS-02: CSP violation reporting**
- *Requirement:* Configure `report-uri` and/or `report-to` to collect real-user CSP violations, so XSS attempts, hijacked third-party scripts, and wrongly blocked legitimate resources are discovered early.
- *Verify:* A reporting endpoint exists, is rate-limited, and is monitored; reports are triaged with awareness that browser extensions generate noise; new third-party sources go through CSP review rather than being added reflexively.

**OPS-03: Release and rollback**
- *Requirement:* Set up automated security scanning (SAST and dependency checks) and a fast rollback process, so a serious frontend vulnerability can be withdrawn or patched within minutes.
- *Verify:* A previous build can be redeployed quickly and CDN caches can be purged; clients holding an old service worker or cached bundle are accounted for in rollback and forced-update plans; feature flags or kill switches exist for risky features; scan results gate or alert the pipeline.

## Report format

Use this structure for reviews and audits. Adapt length to scope: a single-component review needs only the findings table and remediation.

```markdown
# Security Review: <application or feature name>

## Summary
<2-4 sentences: scope, overall posture, number of findings by severity, top risks>

## Scope and assumptions
- Stack: <framework, rendering model, hosting>
- Reviewed: <files, headers, build output, configurations>
- Not reviewed / not verified: <items>
- Assumptions: <threat model, sensitivity of data>

## Findings

| ID | Control | Severity | Status | Evidence | Recommendation |
|----|---------|----------|--------|----------|----------------|
| XSS-02 | Framework escape hatches | Critical | Fail | `Bio.jsx:9` renders `user.bio` with `dangerouslySetInnerHTML` | Sanitize with DOMPurify allowlist |

## Detailed remediation
### <Finding ID>: <Title>
- **Risk:** <what an attacker can do>
- **Evidence:** <file/line or header observed>
- **Fix:** <minimal change, with a short example>
- **Verification:** <how to confirm the fix, for example a test or a header check>

## Passed controls
<Brief list of controls verified as Pass>

## Recommended next steps
1. <Highest-priority action>
2. ...
```

For **design guidance**, replace the findings table with a short list of required controls per feature, each referencing a control ID, followed by an implementation outline.

For a **release checklist**, output a Markdown task list grouped by domain (`- [ ] XSS-01 ...`) limited to the controls relevant to the release.

## Example

**Input:** "Users can write a bio. Is this component okay?"

```jsx
function Bio({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

**Expected output (abbreviated):**

> **Finding XSS-02 (Critical, Fail):** `dangerouslySetInnerHTML` inserts the string as raw HTML. If `html` contains attacker-controlled content, a payload such as an event-handler attribute executes in every visitor's session (stored XSS), which can then act as the victim or read anything the page can read.
>
> **Fix:** If the bio is plain text, render it as text (`<p>{bio}</p>`), which React encodes automatically. If limited formatting is required, sanitize with a strict allowlist immediately before insertion:
>
> ```jsx
> import { useMemo } from "react";
> import DOMPurify from "dompurify";
>
> const SANITIZE_OPTIONS = {
>   ALLOWED_TAGS: ["b", "i", "em", "strong", "p", "ul", "ol", "li", "br", "a"],
>   ALLOWED_ATTR: ["href"],
> };
>
> function Bio({ html }) {
>   const clean = useMemo(() => DOMPurify.sanitize(html, SANITIZE_OPTIONS), [html]);
>   return <div dangerouslySetInnerHTML={{ __html: clean }} />;
> }
> ```
>
> **Also check:** the server sanitizes or encodes the stored value (defense in depth; see `backend-security-checklist`), HDR-01 (a strict CSP limits the impact of any missed payload), URL-01 (allowed `href` values are restricted to safe schemes), and MSG-02 (links opened with `target="_blank"` carry `rel="noopener noreferrer"`).

## Reference mapping

These controls align with widely used industry references. Use them to give users standard terminology and links for further reading. Check the latest edition of each reference, since category numbering changes between releases.

| Domain | Related references |
|---|---|
| 1. Input and injection | OWASP Top 10 Injection; OWASP XSS Prevention and DOM-based XSS Prevention Cheat Sheets; OWASP Prototype Pollution Prevention Cheat Sheet; Trusted Types |
| 2. Authentication and sessions | OWASP Top 10 Identification and Authentication Failures; OWASP Session Management and CSRF Prevention Cheat Sheets; IETF guidance for OAuth 2.0 for Browser-Based Applications |
| 3. Headers | OWASP HTTP Security Response Headers, Content Security Policy, and Clickjacking Defense Cheat Sheets; W3C CSP specification; MDN Web Security documentation |
| 4. Supply chain | OWASP Top 10 Vulnerable and Outdated Components and Software and Data Integrity Failures; OWASP Third Party JavaScript Management Cheat Sheet; SLSA; NIST SSDF |
| 5. Storage and PWA | OWASP HTML5 Security Cheat Sheet; OWASP Top 10 Security Misconfiguration |
| 6. Messaging | OWASP HTML5 Security Cheat Sheet (postMessage, WebSockets) |
| 7. Browser APIs | W3C Permissions Policy; MDN documentation on the Permissions API |
| 8. Logic and abuse | OWASP Top 10 Insecure Design; OWASP ReDoS guidance |
| 9. Privacy | GDPR, ePrivacy, CCPA/CPRA; Global Privacy Control |
| 10. Operations | OWASP Top 10 Security Logging and Monitoring Failures; W3C Reporting API |

For a comprehensive verification standard, refer users to the OWASP Application Security Verification Standard (ASVS).

## Limitations and responsible use

- **Defensive scope only.** Use this skill to find and fix weaknesses in applications the user owns or is authorized to assess. Do not produce working exploit payload collections, XSS filter-evasion lists, phishing pages, or tooling for attacking other people's sites. Describing a vulnerability class and its remediation is appropriate; weaponizing it is not.
- **Not a certification.** A completed checklist is not a penetration test, an accessibility or privacy audit, or a compliance attestation (for example PCI DSS, SOC 2, GDPR). State this when delivering a report, and recommend independent testing for high-risk applications.
- **Evidence over assurance.** Report only what you have verified. Headers and build output are often set outside the repository, so mark unverified controls as **Not verified** rather than assuming.
- **Context matters.** Strict CSP, Trusted Types, cross-origin isolation, and SRI carry integration cost and can break third-party functionality. Explain trade-offs, suggest incremental rollout (such as report-only mode), and let the user make an informed decision.
- **Browser behavior evolves.** Defaults (SameSite, `noopener`, script handling in package managers, Trusted Types support) change over time. When a recommendation depends on specific browser or tool behavior, say so and encourage the user to confirm against current documentation.
- **Handle secrets carefully.** If the user pastes credentials, keys, or tokens, point it out, advise rotating them immediately, and do not repeat them in your output.

## License

Released under the MIT License. See the `LICENSE` file for the full text.
