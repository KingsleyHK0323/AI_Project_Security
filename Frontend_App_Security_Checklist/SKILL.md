---
name: frontend-app-security-checklist
description: A structured, production-grade security checklist for client-side applications, covering native mobile (iOS, Android), hybrid and cross-platform (React Native, Flutter), WebView-based, and desktop (Electron, Tauri) apps. Use this skill whenever the user builds, reviews, audits, or hardens a client app, even if they do not explicitly say "security". Trigger on requests such as "where should I store this token", "review my app's security", "is this deep link safe", "should I use certificate pinning", "secure my WebView", "harden my Electron app", "pre-release mobile security check", or any mention of Keychain, Keystore, SharedPreferences, localStorage, ATS, cleartext traffic, PKCE, biometrics, JS bridge, Universal Links, App Links, exported components, obfuscation, R8, root/jailbreak detection, source maps, or SDK/privacy compliance. Covers 10 domains and 26 verifiable controls with severity ratings and a standard report format.
license: MIT
metadata:
  author: Kingsley (@KingsleyHK0323)
  repository: https://github.com/KingsleyHK0323/AI_Project_Security
  version: "1.0.1"
  domain: application-security
  companion-skill: backend-app-security-checklist
---

# Frontend App Security Checklist

A practical, defense-in-depth security checklist for client applications that run on devices the user does not control: mobile, hybrid, WebView-based, and desktop apps. It consolidates the controls that most often decide whether a shipped app protects its users' data and its backend: secure storage, transport security, authentication, WebView and hybrid hardening, deep links and IPC, tamper resistance, secrets hygiene, permissions and SDKs, privacy-safe logging, and release governance.

The skill supports three jobs: **designing** features securely, **reviewing** existing code, configuration, and build artifacts, and **producing a release-gate checklist** for a team.

## Core principles

Apply these principles throughout. They explain *why* the controls exist and help you make sound judgments when a situation does not match the checklist exactly.

1. **The client is hostile territory.** Anything shipped inside an app (code, assets, keys, logic) can be extracted, inspected, and modified by an attacker with a rooted device, a debugger, or a proxy. Design on the assumption that the client will be reverse engineered.
2. **The client is a convenience layer, not a security boundary.** Client-side route guards, feature flags, and validation improve user experience. Real enforcement of access control, pricing, and business rules belongs on the server.
3. **Use the platform's secure primitives.** Keychain, Keystore, system browsers, and OS-level protections are maintained by platform vendors. Prefer them over custom cryptography or ad hoc storage.
4. **Minimize what the device holds.** Data that is never stored or collected cannot leak. Keep sensitive data short-lived, scoped, and cleared on logout.
5. **Detection is a signal, not a guarantee.** Root, jailbreak, hook, and tamper detection can be bypassed. Use them to raise attacker cost and feed server-side risk scoring, never as the only defense.
6. **Collect only what is needed, and say why.** Permissions, SDKs, and telemetry carry user-privacy and regulatory consequences.
7. **Plan for the old version in the wild.** Shipped binaries cannot be patched remotely. Build update, deprecation, and forced-upgrade paths from the start.

## When to use this skill

Use it when the user is:

- Building or changing storage, login, session, biometric, payment, or deep-link features in a client app.
- Asking for a code review, configuration review, threat model, or security audit of a mobile, hybrid, or desktop app.
- Preparing a release, pull request, or store submission and wants a security gate.
- Embedding WebViews, integrating third-party SDKs, or handling OAuth sign-in.
- Deciding on hardening measures such as certificate pinning, obfuscation, or jailbreak detection and wants a balanced recommendation.

Do not use it for backend-only questions (use the companion skill `backend-app-security-checklist`), for reverse engineering or exploit development, or for formal compliance certification. See [Limitations and responsible use](#limitations-and-responsible-use).

## Workflow

### Step 1: Establish scope and context

Identify, from the conversation and any provided code, what is in scope. Gather only what you cannot infer:

- **Platform and stack:** iOS (Swift/Objective-C), Android (Kotlin/Java), React Native, Flutter, Electron, Tauri, or a web-based shell.
- **Authentication model:** OAuth/OIDC, passwords, social login, biometrics, session tokens.
- **Sensitive assets:** tokens, keys, PII, payment data, health data, in-app currency, anti-cheat-sensitive state.
- **Risk profile:** a financial, health, or competitive-game app warrants stronger tamper resistance than a content reader.
- **Mode of work:** **Design guidance**, **Code review**, or **Release checklist**.

If the user provided code, manifests, `Info.plist`, `AndroidManifest.xml`, build configuration, or Electron/Tauri configuration, read them before answering. Base findings on evidence, not assumptions.

### Step 2: Select the relevant domains

Not every domain applies to every task or platform. Map the request to domains, and state which you are covering and why.

| Task | Prioritize domains |
|---|---|
| Token, credential, or personal data storage | 1, 9, 7 |
| Login, OAuth, biometrics, session handling | 3, 1, 5 |
| API client or networking changes | 2, 7, 10 |
| WebView, hybrid, or Electron/Tauri work | 4, 5, 7 |
| Deep links, push-to-open flows, Android components | 5, 3, 4 |
| Anti-cheat, fintech, or high-value app hardening | 6, 2, 10 |
| Adding an SDK, requesting a permission, or analytics | 8, 9 |
| Pre-release gate or full audit | All |

### Step 3: Evaluate each applicable control

For every applicable control in the [Control catalog](#control-catalog), assign one status:

- **Pass**: Evidence shows the control is implemented correctly.
- **Partial**: Implemented, but with gaps (state the gap).
- **Fail**: Missing or incorrectly implemented.
- **N/A**: Not applicable to this platform or app (state why).
- **Not verified**: Evidence is not available (for example, build artifacts or store configuration were not provided). Never mark a control as Pass without evidence.

### Step 4: Rate severity

Rate each Fail or Partial finding using exploitability and impact:

| Severity | Meaning | Examples |
|---|---|---|
| **Critical** | Directly exploitable; leads to account takeover, credential or key theft, or large-scale data exposure | Refresh token in plaintext `SharedPreferences` or `localStorage`; production secret key embedded in the app; Electron renderer with `nodeIntegration` enabled loading remote content |
| **High** | Exploitable with modest effort or a precondition; serious impact | Cleartext HTTP traffic allowed globally; sensitive OAuth callback on a hijackable custom URL scheme; biometric result stored as a client-side boolean |
| **Medium** | Weakens defenses or leaks limited information | Missing background-screen masking; overly permissive exported Android component with limited impact; tokens in debug logs |
| **Low** | Hardening gap with limited direct impact | Missing obfuscation on a low-risk app; verbose permission rationale |
| **Info** | Best-practice recommendation | Documentation and process improvements |

### Step 5: Report and remediate

Deliver the result using the [Report format](#report-format). For every Fail or Partial finding, give a concrete, minimal remediation that fits the user's platform and framework. Provide short code or configuration examples when they clarify the fix. Order findings by severity, then by effort to fix.

## Control catalog

Each control has an ID for traceability, the requirement, and what to verify. Use the IDs in reports and pull request comments.

---

### Domain 1: Secure local storage and sensitive data protection

**STORE-01: Secure storage of credentials and keys**
- *Requirement:* Access tokens, refresh tokens, session credentials, and cryptographic keys must never be stored in plaintext files, `NSUserDefaults` (iOS), `SharedPreferences` (Android), `localStorage`/`AsyncStorage`, or an unencrypted SQLite database or filesystem location.
  - **iOS:** Use **Keychain Services** with an appropriate `kSecAttrAccessible` value, for example `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`, which also keeps the item out of backups and device migration.
  - **Android:** Use the **Android Keystore System** to hold keys, and encrypt stored data with Keystore-backed keys. Note that Google has deprecated the Jetpack Security library that provides `EncryptedSharedPreferences`; verify current platform guidance and prefer a maintained approach, such as Keystore-backed keys used through a maintained cryptography library.
  - **Desktop (Electron / Tauri):** Use the operating system's secure credential store (macOS Keychain, Windows DPAPI / Credential Manager, Linux Secret Service). Never write secrets to plaintext configuration files.
- *Verify:* Search for token and key writes to preferences, files, and web storage; inspect an app backup or sandbox dump for secrets; confirm Android `allowBackup` and backup rules do not export sensitive data; confirm accessibility attributes on Keychain items are the most restrictive that the product allows.

**STORE-02: Memory, cache, and background-snapshot hygiene**
- *Requirement:* On logout or account switch, immediately clear sensitive objects in memory, cached files, and database records. Prevent sensitive screens from appearing in the app switcher snapshot by masking the screen when the app enters the background. Consider screenshot and screen-recording protection where the risk justifies it (for example, Android `FLAG_SECURE`; on iOS, detect capture state and mask content).
- *Verify:* Trace the logout path and confirm tokens, cached responses, image caches, and user databases are all removed; test the app switcher preview on sensitive screens; confirm HTTP caches do not retain authenticated responses unnecessarily.

**STORE-03: Clipboard and keyboard cache controls**
- *Requirement:* For password, OTP, key, and sensitive account fields, disable autocorrect and keyboard suggestion caching and restrict third-party keyboards where appropriate. Avoid writing sensitive content to the system clipboard, and when copy is necessary, use short expiry or sensitive-content flags offered by the platform.
- *Verify:* Check input field attributes (for example secure text entry, `autofill` and `autocorrect` settings, input types); review copy-to-clipboard features for secrets, recovery codes, and addresses.

---

### Domain 2: Network and transport security

**NET-01: Enforced HTTPS/TLS**
- *Requirement:* Use `https://` and `wss://` everywhere. Cleartext `http://` is prohibited. Keep iOS App Transport Security enabled (do not set `NSAllowsArbitraryLoads`) and Android cleartext traffic disabled (`usesCleartextTraffic` and Network Security Config). Any exception must be narrowly scoped (for example, a specific development host) and absent from release builds.
- *Verify:* Inspect `Info.plist` ATS keys, `AndroidManifest.xml`, and `network_security_config.xml`; confirm release builds do not trust user-installed CAs unless intentionally required; confirm no custom `TrustManager` or delegate disables validation.

**NET-02: Certificate validation and certificate/public-key pinning**
- *Requirement:* Properly validate the server certificate chain, hostname, and validity period, and never bypass validation (no "trust all" handlers). For high-security apps, and only if the team can operate it, implement **certificate or public-key (SPKI) pinning** with backup pins and an emergency update path, so that certificate rotation or expiry cannot lock users out of the app.
- *Verify:* Search for permissive `TrustManager`, `HostnameVerifier`, `URLSession` challenge handlers, or `rejectUnauthorized: false`; for pinning, confirm at least one backup pin, pins on public keys rather than short-lived leaf certificates, pin expiry handling, and a tested recovery process. Treat pinning as a trade-off: it adds protection against a compromised or rogue CA at a real operational cost.

**NET-03: Request signing and anti-replay**
- *Requirement:* For high-risk APIs (transactions, check-ins, reward claims, sensitive lookups), consider adding a timestamp, a nonce, and a signature to reduce packet replay and forgery. Pair with platform attestation (Apple App Attest, Google Play Integrity) as supporting evidence.
- *Verify:* Confirm that any signing key embedded in the client is understood to be extractable, so the server must not depend on the signature alone; confirm the backend verifies timestamps, nonces, and attestation tokens. This control is defense in depth and does not replace server-side authorization (see REL-02 below and the companion backend skill).

---

### Domain 3: Authentication, sessions, and access control

**AUTH-01: Secure sign-in and OAuth with PKCE**
- *Requirement:* Perform OAuth/OIDC sign-in in the **system browser** (`ASWebAuthenticationSession` on iOS, Chrome Custom Tabs on Android) rather than an embedded WebView, so the app never sees the user's third-party credentials and the user can rely on the browser's session and anti-phishing protections. Native apps are public clients and must use **PKCE (Proof Key for Code Exchange)** with the `S256` method.
- *Verify:* Search for OAuth flows loaded in WebView; confirm `code_challenge_method=S256`; confirm `state` is generated and verified; confirm no client secret exists in the app; prefer claimed HTTPS redirect URIs over custom schemes.

**AUTH-02: Secure biometric integration**
- *Requirement:* Biometrics must unlock something real: a private key or token stored in Keychain or Keystore and protected by access control that requires user authentication. Never treat "biometric succeeded" as a client-side boolean such as `isAuthenticated = true` that gates logic, because that flag can be patched or hooked.
  - **iOS:** Store the secret with a Keychain access control that requires biometry or device passcode, and let the system enforce it.
  - **Android:** Use a Keystore key that requires user authentication, and use `BiometricPrompt` with a `CryptoObject` so a successful authentication releases the cryptographic operation.
- *Verify:* Trace what happens after a successful biometric callback. If the only effect is flipping a flag or navigating, it is a Fail. Confirm enrollment-change behavior is deliberate (for example invalidating keys when biometrics change).

**AUTH-03: Client-side route guards backed by server-side enforcement**
- *Requirement:* Block unauthenticated or unauthorized users from restricted screens and features for good UX, but treat this as cosmetic. Every data access and operation must be authorized by the backend API.
- *Verify:* Confirm that bypassing the UI (calling the API directly with a valid low-privilege token) does not grant access. This pairs with the backend skill's authorization controls.

---

### Domain 4: WebView and hybrid container security

**WEB-01: Restrict JavaScript bridges and file access**
- *Requirement:* If the app embeds a WebView, do not expose high-privilege native bridges to untrusted or external origins. Configure file access conservatively (for example, Android `allowFileAccess`, `allowFileAccessFromFileURLs`, `allowUniversalAccessFromFileURLs` disabled unless essential) to prevent malicious scripts from reading local files.
- *Verify:* Enumerate every bridge method (`addJavascriptInterface`, `WKScriptMessageHandler`, custom schemes, JS channels) and confirm each is exposed only to trusted first-party origins, validates its inputs, and performs only the minimum required action; prefer serving local content through a safe asset loader rather than `file://`.

**WEB-02: URL allowlisting and navigation control**
- *Requirement:* Before a WebView loads an external link or follows a redirect, strictly validate the domain and scheme against an allowlist. This defends against open redirects, XSS that reaches native functionality through a bridge, and abuse of `file://` and `javascript:` URLs.
- *Verify:* Review `shouldOverrideUrlLoading`, `decidePolicyFor navigationAction`, and equivalents; confirm host comparison is exact (no substring or suffix matching that `evil.com/trusted.com` can defeat); confirm external-origin pages are not given bridge access; open untrusted links in the system browser.

**WEB-03: Hybrid framework hardening (Electron, Tauri, React Native, Flutter)**
- *Requirement:*
  - **Electron:** Keep `contextIsolation: true` and `sandbox: true`; keep `nodeIntegration` disabled unless strictly isolated and necessary; expose a minimal API from `preload` scripts (through `contextBridge`) and validate the sender and arguments of every IPC message; do not load remote content in a privileged window; apply a Content Security Policy; validate URLs before `shell.openExternal`.
  - **Tauri:** Grant only the capabilities and permissions each window needs, scope filesystem and shell access narrowly, and configure a Content Security Policy.
  - **React Native / Flutter:** Do not store high-privilege secrets in the JS bundle or Dart code (bytecode and compiled output can be decompiled); validate all input parameters at native module and platform channel boundaries.
- *Verify:* Review `BrowserWindow` `webPreferences`, preload scripts, IPC handlers, Tauri capability files, and native module or platform channel entry points.

---

### Domain 5: Deep links, universal links, and inter-app communication

**LINK-01: Verified Universal Links and App Links**
- *Requirement:* Prefer domain-verified links: **Universal Links** on iOS (`apple-app-site-association`) and **Android App Links** (`assetlinks.json` with auto-verify). Do not rely solely on custom URL schemes for sensitive callbacks or password-reset flows, because other apps can register the same scheme and intercept the data.
- *Verify:* Confirm the association files are served and correct, and that sensitive flows use verified links; confirm links that trigger sensitive actions (for example password reset) require user confirmation or re-authentication instead of acting silently.

**LINK-02: Deep link parameter sanitization**
- *Requirement:* Treat all parameters from a deep link or Intent as untrusted input. Validate type and format, and guard against open redirects, injection, and unexpected state transitions or navigation to screens the user has not been authorized to see.
- *Verify:* Trace each deep-link handler from parsing to action; confirm unknown routes fail safely, redirect targets are allowlisted, and the target screen performs its own authorization check.

**LINK-03: Android component and IPC permission control**
- *Requirement:* Set `android:exported="false"` for every Activity, Service, BroadcastReceiver, and ContentProvider that does not need to be reachable by other apps. For exported components, validate the caller and require appropriate permissions, to avoid Intent leakage or privilege abuse. Use immutable `PendingIntent` objects unless mutability is required.
- *Verify:* Review `AndroidManifest.xml` for every component and intent filter (Android 12 and later require an explicit `exported` declaration for components with intent filters); inspect exported ContentProviders for SQL injection and path traversal; check that Intents received from other apps are not blindly forwarded (Intent redirection).

---

### Domain 6: Anti-tampering and runtime integrity

**TAMP-01: Obfuscation and build hardening**
- *Requirement:* Enable code shrinking, optimization, and obfuscation in release builds (Android R8 / ProGuard; stripped symbols on iOS; minification and removal of debug logging for JavaScript and TypeScript bundles). Remove test backdoors, mock accounts, debug menus, and debug endpoints from release builds.
- *Verify:* Inspect release build configuration; decompile the release artifact and look for readable secrets, debug flags, test hosts, and feature toggles. Remember that obfuscation slows analysis but does not prevent it.

**TAMP-02: Repackaging resistance and signature verification**
- *Requirement:* Protect against attackers decompiling the app, inserting malicious code, and redistributing a repackaged build. Use platform signing correctly and protect signing keys. For stronger assurance, rely on **server-verified attestation** (App Attest, Play Integrity) rather than only client-side signature checks, which can themselves be patched out.
- *Verify:* Confirm that the backend validates attestation verdicts and that the app's behavior degrades sensibly, or backend access is limited, when attestation fails.

**TAMP-03: Root, jailbreak, hook, and emulator detection**
- *Requirement:* For financial, anti-cheat, or other high-sensitivity apps, consider detecting rooted or jailbroken devices, hooking frameworks (for example Frida, Xposed), attached debuggers, and emulators. Acknowledge that **client-side detection can be bypassed**, and pair it with server-side risk scoring. Use a reasonable compatibility policy so legitimate testers, accessibility tools, and specific device models are not wrongly blocked.
- *Verify:* Confirm detection results are reported to the server and weighed as signals rather than enforced only in client code; confirm there is a support path for false positives. For low-risk apps, recommend against investing heavily here.

---

### Domain 7: Secrets management and build artifact hygiene

**SECR-01: No high-privilege secrets in the client**
- *Requirement:* API secret keys, private keys, database connection strings, and internal keys must not be hardcoded in source, resource files, or environment variables bundled into the client. Anything shipped to a client must be treated as public. Public identifiers (public keys, public client IDs) need compensating restrictions such as bundle ID or package-name and signature restrictions, domain restrictions, and server-side rate limiting.
- *Verify:* Run secret scanning over the repository, the Git history, and the built artifact (decompiled APK/AAB, IPA, JS bundle, Electron `asar`); flag any key that grants privileged access; confirm leaked secrets are rotated, not just removed.

**SECR-02: Configuration files and source map handling**
- *Requirement:* Review `google-services.json`, `GoogleService-Info.plist`, `.env` files, and packaged output to confirm there are no stray test keys or internal environment details. These Firebase configuration files are generally not secrets, but the services behind them still need access rules and abuse protection (for example, security rules and app attestation). Production source maps and symbol files should be uploaded to the crash-reporting platform with restricted access and must not ship inside the public app or web bundle.
- *Verify:* Unpack the release artifact and search for `.map` files, `.env` files, staging URLs, and internal hostnames.

---

### Domain 8: Device permissions, media, and third-party SDKs

**PERM-01: Least-privilege permissions with clear rationale**
- *Requirement:* Request only the permissions the product truly needs (camera, microphone, location, Bluetooth, contacts, photos, notifications). Ask at the moment of need, and explain purpose and timing clearly before the system prompt. Prefer narrower alternatives where they exist (for example, a system photo picker instead of full photo-library access, or approximate instead of precise location).
- *Verify:* Compare the declared permissions in `AndroidManifest.xml` and `Info.plist` usage descriptions with actual features; remove unused permissions inherited from SDKs.

**PERM-02: Third-party SDK review and updates**
- *Requirement:* Regularly inventory analytics, advertising, push, payment, and customer-support SDKs. Review their permissions, data-collection behavior, and known vulnerabilities, and upgrade to patched versions promptly. Pin dependency versions and commit lockfiles.
- *Verify:* Produce a dependency and SDK list (an SBOM if available); run dependency vulnerability scanning; check for SDK privacy declarations required by the app stores; confirm SDKs cannot read data the app does not intend to share.

---

### Domain 9: Error handling, logging, and privacy compliance

**PRIV-01: Log redaction and disabled verbose output**
- *Requirement:* Disable sensitive debug logging in release builds. Ensure crash reports, analytics events, and local logs never contain passwords, tokens, card numbers, national ID numbers, or full personal data (PII).
- *Verify:* Search for logging calls near authentication and payment code; review crash-reporter and analytics payload scrubbing; confirm release builds strip or gate debug logs; check that network-inspection logging interceptors are not active in release.

**PRIV-02: Consent management and data deletion**
- *Requirement:* Provide a clear privacy policy, tracking and advertising consent controls, and an account and data deletion flow, according to the regulations that apply (for example GDPR, CCPA/CPRA, PIPA, and platform store policies, which commonly require an in-app way to initiate account deletion). Do not collect or upload sensitive data before consent is obtained.
- *Verify:* Confirm that analytics and advertising SDKs initialize only after consent where required; confirm the deletion flow actually triggers backend deletion; confirm store privacy disclosures match real behavior.

---

### Domain 10: Release governance, updates, and incident response

**REL-01: Secure updates and forced-upgrade mechanism**
- *Requirement:* Maintain version control, careful custody of signing keys, and an emergency **forced-update** mechanism, so that when a severe vulnerability or protocol incompatibility appears, users can be directed to upgrade quickly.
- *Verify:* Confirm the backend can declare a minimum supported app version and that the app handles it gracefully; confirm signing keys live in a protected store with limited access (for example, managed app signing) and have a recovery plan.

**REL-02: End-to-end coordinated review**
- *Requirement:* Client-side checks must be reviewed together with backend authorization, risk controls, and data validation. The test is simple: if an attacker bypasses the app UI and calls the API directly, they must still be unable to exceed their privileges or tamper with data.
- *Verify:* For each sensitive client-side restriction, find the matching server-side enforcement; where it is missing, raise it as a finding against the backend and cross-reference the companion skill `backend-app-security-checklist`.

## Report format

Use this structure for reviews and audits. Adapt length to scope: a single-feature review needs only the findings table and remediation.

```markdown
# Security Review: <app or feature name>

## Summary
<2-4 sentences: scope, overall posture, number of findings by severity, top risks>

## Scope and assumptions
- Platform(s): <iOS / Android / React Native / Electron / ...>
- Reviewed: <files, manifests, configurations, build artifacts>
- Not reviewed / not verified: <items>
- Assumptions: <risk profile, threat model>

## Findings

| ID | Control | Severity | Status | Evidence | Recommendation |
|----|---------|----------|--------|----------|----------------|
| STORE-01 | Secure credential storage | Critical | Fail | `AuthStore.kt:31` writes refresh token to `SharedPreferences` | Move to Keystore-backed encrypted storage |

## Detailed remediation
### <Finding ID>: <Title>
- **Risk:** <what an attacker can do>
- **Evidence:** <file/line or configuration>
- **Fix:** <minimal change, with a short example>
- **Verification:** <how to confirm the fix, for example a test or a sandbox inspection>

## Passed controls
<Brief list of controls verified as Pass>

## Recommended next steps
1. <Highest-priority action>
2. ...
```

For **design guidance**, replace the findings table with a short list of required controls per feature, each referencing a control ID, followed by an implementation outline.

For a **release checklist**, output a Markdown task list grouped by domain (`- [ ] STORE-01 ...`) limited to the controls relevant to the release and platform.

## Example

**Input:** "My iOS app stores the refresh token like this. Is that okay?"

```swift
UserDefaults.standard.set(refreshToken, forKey: "refresh_token")
```

**Expected output (abbreviated):**

> **Finding STORE-01 (Critical, Fail):** `UserDefaults` is stored as an unencrypted property list in the app sandbox and is included in device backups. A refresh token is a long-lived credential, so anyone who obtains a backup, a jailbroken file system, or malware-level access can take over the account.
>
> **Fix:** Store the token in the Keychain with a restrictive accessibility class, and remove the old value on migration.
>
> ```swift
> import Security
>
> func saveRefreshToken(_ token: String) throws {
>     let base: [String: Any] = [
>         kSecClass as String: kSecClassGenericPassword,
>         kSecAttrService as String: "com.example.app.auth",
>         kSecAttrAccount as String: "refresh_token"
>     ]
>     SecItemDelete(base as CFDictionary) // replace any existing item
>
>     var item = base
>     item[kSecValueData as String] = Data(token.utf8)
>     item[kSecAttrAccessible as String] = kSecAttrAccessibleWhenUnlockedThisDeviceOnly
>
>     let status = SecItemAdd(item as CFDictionary, nil)
>     guard status == errSecSuccess else { throw KeychainError.unhandled(status) }
> }
> ```
>
> **Also check:** STORE-02 (delete the Keychain item and cached data on logout), PRIV-01 (the token is not logged), and the backend's refresh-token rotation and revocation (see `backend-app-security-checklist`, AUTH-03).

## Reference mapping

These controls align with widely used industry references. Use them to give users standard terminology and links for further reading.

| Domain | Related references |
|---|---|
| 1. Secure storage | OWASP MASVS: MASVS-STORAGE; MASTG data storage tests |
| 2. Network | OWASP MASVS: MASVS-NETWORK; platform guidance on ATS and Network Security Config |
| 3. Authentication | OWASP MASVS: MASVS-AUTH; RFC 8252 (OAuth 2.0 for Native Apps); RFC 7636 (PKCE) |
| 4. WebView and hybrid | OWASP MASVS: MASVS-PLATFORM; Electron and Tauri security documentation |
| 5. Deep links and IPC | OWASP MASVS: MASVS-PLATFORM; Apple and Android developer documentation on verified links |
| 6. Anti-tampering | OWASP MASVS: MASVS-RESILIENCE (defense-in-depth profile) |
| 7. Secrets and build hygiene | OWASP MASVS: MASVS-CODE, MASVS-CRYPTO |
| 8. Permissions and SDKs | OWASP MASVS: MASVS-PRIVACY, MASVS-CODE (third-party components) |
| 9. Logging and privacy | OWASP MASVS: MASVS-STORAGE, MASVS-PRIVACY |
| 10. Release and update | OWASP MASVS: MASVS-CODE; OWASP Mobile Top 10 |

OWASP MASVS is the standard verification baseline and the Mobile Application Security Testing Guide (MASTG) provides test procedures for each requirement. This skill focuses on the client; for server-side controls, use the companion skill `backend-app-security-checklist`.

## Limitations and responsible use

- **Defensive scope only.** Use this skill to find and fix weaknesses in apps the user owns or is authorized to assess. Do not produce working exploits, bypass scripts for detection or pinning, instructions for repackaging or cracking third-party apps, or tooling for attacking other people's software. Describing a vulnerability class and its remediation is appropriate; weaponizing it is not.
- **Not a certification.** A completed checklist is not a penetration test, a MASVS certification, a store-review guarantee, or a regulatory compliance attestation. State this when delivering a report, and recommend independent testing for high-risk apps.
- **Evidence over assurance.** Report only what you have verified. Mark unverified controls as **Not verified**. Do not present assumptions as findings.
- **Proportionality.** Controls such as certificate pinning, root detection, and heavy obfuscation add operational cost and can harm legitimate users. Explain the trade-offs and recommend them in proportion to the app's risk profile, rather than applying every control mechanically.
- **Platform guidance changes.** APIs, defaults, and store policies evolve. When a recommendation depends on a specific platform API or policy, say so and encourage the user to confirm against current official documentation.
- **Handle secrets carefully.** If the user pastes credentials, keys, or tokens, point it out, advise rotating them immediately, and do not repeat them in your output.

## License

Released under the MIT License. See the `LICENSE` file for the full text.
