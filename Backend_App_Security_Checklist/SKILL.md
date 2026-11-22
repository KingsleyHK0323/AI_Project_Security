---
name: backend-app-security-checklist
description: A structured, production-grade security checklist for backend services that power mobile and web apps. Use this skill whenever the user designs, implements, reviews, audits, or hardens a backend API, authentication flow, authorization logic, file upload, payment or points system, webhook receiver, push notification pipeline, or deployment setup, even if they do not explicitly say "security". Trigger on requests such as "review my API", "is this endpoint safe", "audit this backend", "threat model this feature", "pre-release security check", "pull request security review", or any mention of JWT, refresh tokens, OAuth, IDOR, BOLA, rate limiting, idempotency, race conditions, SSRF, secrets management, or audit logging. Covers 10 domains and 28 verifiable controls with severity ratings and a standard report format.
license: MIT
metadata:
  author: Kingsley (@KingsleyHK0323)
  repository: https://github.com/KingsleyHK0323/AI_Project_Security
  version: "1.0.1"
  domain: application-security
---

# Backend App Security Checklist

A practical, defense-in-depth security checklist for backend services that serve mobile and web clients. It consolidates the controls that most often separate a secure production backend from a vulnerable one: authentication, authorization, input handling, abuse resistance, business-logic integrity, secrets, observability, infrastructure, integrations, and release governance.

The skill supports three jobs: **designing** new features securely, **reviewing** existing code or architecture, and **producing a release-gate checklist** for a team.

## Core principles

Apply these principles throughout. They explain *why* the controls exist and help you make sound judgments when a situation does not match the checklist exactly.

1. **Never trust the client.** Mobile apps and browsers are attacker-controlled environments. Any value, flag, price, ID, or timestamp sent by a client is an untrusted claim until the server verifies it.
2. **Authorize on every request, at the object level.** Authentication proves who the caller is; authorization decides what they may touch. Most serious API breaches are authorization failures, not authentication failures.
3. **Default deny.** Access, network exposure, and permissions should be closed unless explicitly opened.
4. **Defense in depth.** No single control is sufficient. Request signing, app attestation, and client-side checks are *signals* that raise attacker cost. They never replace server-side authorization and validation.
5. **Make state changes safe to retry and safe to race.** Networks time out and users double-tap. Writes involving money, points, inventory, or notifications must be idempotent and concurrency-safe.
6. **Fail safely and quietly.** Errors reveal nothing useful to attackers; logs capture what responders need without capturing secrets.
7. **Assume breach.** Plan for credential theft, key leakage, and device loss with revocation, rotation, monitoring, and incident response.

## When to use this skill

Use it when the user is:

- Designing or implementing API endpoints, auth flows, payments, rewards, orders, messaging, or file handling.
- Asking for a code review, architecture review, threat model, or security audit of a backend.
- Preparing a release, pull request, or compliance review and wants a security gate.
- Integrating third-party services (OAuth providers, payment gateways, push providers, webhooks).
- Configuring deployment, containers, databases, caches, or CI/CD pipelines.

Do not use it for purely frontend or UI security questions, for penetration-testing or exploit development, or for formal compliance certification. See [Limitations and responsible use](#limitations-and-responsible-use).

## Workflow

### Step 1: Establish scope and context

Identify, from the conversation and any provided code, what is in scope. Gather only what you cannot infer:

- Stack and framework (language, web framework, ORM, database, cache, queue).
- Client types (iOS, Android, web, third-party) and authentication method (sessions, JWT, OAuth/OIDC).
- Sensitive assets (PII, payment data, health data, virtual currency, inventory).
- Deployment model (cloud provider, containers, serverless, VPC layout).
- Mode of work: **Design guidance**, **Code review**, or **Release checklist**.

If the user provided code or configuration, read it before answering. Base findings on evidence, not assumptions.

### Step 2: Select the relevant domains

Not every domain applies to every task. Map the request to domains using this guide, and state which domains you are covering and why.

| Task | Prioritize domains |
|---|---|
| New or changed login, signup, password reset, OTP | 1, 4, 7 |
| New CRUD endpoint or resource | 2, 3, 7 |
| File upload or download | 3, 2, 8 |
| Payment, rewards, points, check-in, limited-quantity sales | 5, 4, 2, 7 |
| Webhook receiver or third-party API call | 4, 9, 6 |
| Push notifications or deep links | 9, 2 |
| Deployment, containers, database, CI/CD | 8, 6, 10 |
| Full audit or pre-release gate | All |

### Step 3: Evaluate each applicable control

For every applicable control in the [Control catalog](#control-catalog), assign one status:

- **Pass**: Evidence shows the control is implemented correctly.
- **Partial**: Implemented, but with gaps (state the gap).
- **Fail**: Missing or incorrectly implemented.
- **N/A**: Not applicable (state why).
- **Not verified**: Evidence is not available (for example, infrastructure config not provided). Never mark a control as Pass without evidence.

### Step 4: Rate severity

Rate each Fail or Partial finding using exploitability and impact:

| Severity | Meaning | Examples |
|---|---|---|
| **Critical** | Directly exploitable; leads to account takeover, large-scale data exposure, financial loss, or remote code execution | Missing object-level authorization on a user-data endpoint; SQL injection; hardcoded production secret in the repository |
| **High** | Exploitable with modest effort or a precondition; serious impact | No rate limiting on OTP verification; refresh tokens that cannot be revoked; race condition allowing negative balances |
| **Medium** | Weakens defenses or leaks limited information | Verbose error messages; missing pagination caps; weak log redaction |
| **Low** | Hardening gap with limited direct impact | Missing security headers on non-sensitive responses |
| **Info** | Best-practice recommendation | Documentation and process improvements |

### Step 5: Report and remediate

Deliver the result using the [Report format](#report-format). For every Fail or Partial finding, give a concrete, minimal remediation that fits the user's stack. Provide short code or configuration examples when they clarify the fix. Order findings by severity, then by effort to fix.

## Control catalog

Each control has an ID for traceability, the requirement, and what to verify. Use the IDs in reports and pull request comments.

---

### Domain 1: Authentication and token lifecycle

**AUTH-01: Brute-force resistance and login abuse controls**
- *Requirement:* Login, OTP/SMS verification, password reset, and device-binding endpoints enforce rate limiting, account lockout, or progressive delay (exponential backoff). Responses must not reveal whether an account exists.
- *Verify:* Limits exist per account, per IP, and per device; identical response body, status code, and timing for "unknown user" and "wrong password"; OTP attempts are capped and codes expire quickly; password reset tokens are single-use and short-lived.

**AUTH-02: Secure credential storage**
- *Requirement:* Passwords are hashed with **Argon2id** (preferred), **bcrypt**, or **PBKDF2**, using a unique salt and an appropriate cost factor. Plaintext, MD5, SHA-1, and any fast general-purpose hash (including salted SHA-256) are prohibited for passwords.
- *Verify:* Inspect the hashing call and parameters; confirm cost factors are configurable and periodically reviewed; confirm legacy hashes are upgraded on next login; for bcrypt, account for its 72-byte input limit.

**AUTH-03: Access token and refresh token lifecycle**
- *Requirement:* Access tokens are short-lived. Refresh tokens rotate on every use with **reuse detection** (a replayed old token revokes the whole token family). Logout, password change, and device loss revoke tokens and sessions server-side immediately.
- *Verify:* Access token lifetime (minutes, not days); refresh tokens are stored hashed or are opaque and revocable; a server-side revocation or denylist mechanism exists; password change invalidates existing sessions.

**AUTH-04: OAuth 2.0, OIDC, and PKCE validation**
- *Requirement:* Validate `state`, `nonce`, and the PKCE `code_verifier`. The backend verifies ID tokens and access tokens against the authoritative issuer's keys (JWKS), checking signature, `iss`, `aud`, and `exp`.
- *Verify:* Algorithm is pinned (reject `none` and unexpected algorithms); `aud` matches this application; JWKS is fetched from the issuer's published endpoint and cached with sensible refresh; redirect URIs are exact-match allowlisted.

---

### Domain 2: Authorization and object-level security (BOLA / IDOR)

**AUTHZ-01: Object-level authorization**
- *Requirement:* Every read, update, or delete of a resource (orders, messages, files, account data) verifies that the current user or device owns the resource or has a legitimate grant. Never rely on an ID supplied by the client.
- *Verify:* Each handler that takes a resource ID also checks ownership or permission, preferably through a shared policy layer or query scoping (for example, `WHERE id = ? AND owner_id = ?`); test with two different users' accounts; check list, export, and bulk endpoints as well as single-object ones.

**AUTHZ-02: Function-level and role-based access control (RBAC / ABAC)**
- *Requirement:* Administrative endpoints, internal operations, and sensitive APIs enforce role and permission checks in backend middleware with a **default-deny** posture, preventing both vertical (privilege escalation) and horizontal (peer access) abuse.
- *Verify:* Authorization is enforced centrally, not per-developer discretion; new routes are denied until explicitly granted; admin routes are not merely hidden in the client UI.

**AUTHZ-03: Device and account binding checks**
- *Requirement:* Where the product supports multi-device login, trusted-device lists, or single-device restrictions, the backend verifies device identity and credential state to prevent unauthorized device impersonation.
- *Verify:* Device identifiers are bound to tokens server-side; trust revocation takes effect immediately; client-reported device IDs alone are never treated as proof.

---

### Domain 3: Input validation, injection defense, and file handling

**INPUT-01: Strict schema validation and mass-assignment protection**
- *Requirement:* Validate type, length, format, and range for path, query, header, and body inputs using a schema library (for example Zod, Joi, Pydantic). Reject unknown fields. Prevent mass assignment of sensitive attributes such as `role`, `balance`, or `is_verified`.
- *Verify:* Handlers bind requests to explicit allowlisted DTOs rather than passing raw request bodies to the ORM; sensitive fields can only be set by server-side logic.

**INPUT-02: Injection prevention**
- *Requirement:*
  - Use parameterized queries or ORM bindings for all database access (SQL and NoSQL).
  - Avoid invoking a system shell. If an external command is unavoidable, pass an argument array and enforce a strict allowlist on inputs.
  - Resolve file paths to a canonical form and verify they remain within an allowed root directory (path traversal defense).
- *Verify:* Search for string-concatenated queries, `exec`/`system`/`shell=True` patterns, and user-controlled path segments; confirm NoSQL operators from user input are rejected.

**INPUT-03: Secure file upload and download**
- *Requirement:* Validate file size, extension allowlist, and actual content (magic bytes, or re-encode images). Store under random filenames in isolated storage (for example object storage). Serve with correct `Content-Type`, `Content-Disposition`, and `X-Content-Type-Options: nosniff`.
- *Verify:* Original filenames are never used as storage paths; uploads are not served from the application origin in an executable context; downloads enforce authorization (see AUTHZ-01); size limits are enforced before full buffering.

---

### Domain 4: Transport security, anti-replay, and abuse resistance

**NET-01: Enforced TLS**
- *Requirement:* Accept only HTTPS and WSS. Use TLS 1.2 or later (prefer TLS 1.3). Disable obsolete protocols and weak cipher suites.
- *Verify:* HTTP redirects or is refused; HSTS is set for browser-facing surfaces; certificate validation is not disabled in outbound clients.

**NET-02: Request signing and anti-replay**
- *Requirement:* For critical writes, reward claims, check-ins, transactions, and other high-cost APIs, consider combining a timestamp (with a tight validity window), a nonce, and a signature to reduce replay and forgery. Pair with app attestation (for example Apple App Attest, Google Play Integrity) and risk signals as supporting evidence.
- *Verify:* Nonces are stored and rejected on reuse within the window; signing keys are not extractable from the client in a way that defeats the purpose; the design never depends *solely* on client-reported flags. Treat this control as defense in depth, not a substitute for AUTHZ and BIZ controls.

**NET-03: Rate limiting, quotas, and payload limits**
- *Requirement:* Apply multi-dimensional rate limits (IP, user ID, device/token) to authentication, SMS/email sending, search, and public APIs. Enforce a maximum page size on list queries. Limit request body size and JSON nesting depth.
- *Verify:* Limits exist at the gateway or middleware layer; costly operations (SMS, email, exports) have per-user quotas; `limit` parameters are clamped server-side.

**NET-04: SSRF defense and webhook verification**
- *Requirement:* When the backend fetches user-supplied URLs, block internal and link-local IP ranges, cloud metadata endpoints, and unexpected protocols. When receiving third-party webhooks (payments, push, partners), verify the HMAC signature, enforce a timestamp window, and process idempotently.
- *Verify:* URL fetching resolves and validates the destination IP *after* DNS resolution and on each redirect; webhook signatures use constant-time comparison; duplicate webhook deliveries do not produce duplicate effects.

---

### Domain 5: Business logic, transactional integrity, and concurrency

**BIZ-01: Server-side authoritative computation**
- *Requirement:* Prices, discounts, taxes, point or virtual-item grants, tiers, and state transitions are computed and validated on the backend from authoritative data. Never trust client-calculated results or state flags.
- *Verify:* Order totals are recomputed server-side from catalog data; state machines reject illegal transitions; quantities and amounts reject negative, zero, or overflow values where inappropriate.

**BIZ-02: Idempotency**
- *Requirement:* Payments, orders, point deductions, transfers, and notification triggers support an idempotency key or a unique-constraint guard so that timeouts, retries, and double-taps cannot cause duplicate charges or grants.
- *Verify:* The idempotency key is scoped to the user and operation; the result of the first request is returned on retry; a database unique constraint backs the logic.

**BIZ-03: Concurrency control and transaction isolation**
- *Requirement:* For balance deduction, stock reservation, limited-quantity sales, and quota redemption, use database transactions with pessimistic locks (`SELECT ... FOR UPDATE`) or optimistic concurrency (version column / compare-and-swap) to prevent overselling and negative balances.
- *Verify:* Read-check-write sequences are atomic; check constraints (for example `balance >= 0`) exist at the database level; test with concurrent requests rather than reasoning alone.

---

### Domain 6: Secrets, cryptography, and key lifecycle

**SEC-01: No hardcoded secrets**
- *Requirement:* API secrets, database credentials, JWT signing keys, push credentials (APNs/FCM), and third-party private keys never appear in source code or Git history. Inject them through a secret manager or controlled environment variables, and enable secret scanning.
- *Verify:* Run secret scanning over the repository and history; confirm `.env` files are ignored; confirm leaked secrets are rotated, not merely deleted from the latest commit.

**SEC-02: Encryption at rest and key management**
- *Requirement:* Encrypt sensitive PII, medical, financial, and confidential configuration data at the database and storage layers as required by compliance needs. Manage keys in a KMS with rotation and emergency revocation procedures.
- *Verify:* Keys are separated from the data they protect; rotation is documented and tested; backups and snapshots are encrypted.

---

### Domain 7: Error handling, audit logging, and observability

**OBS-01: Safe error responses**
- *Requirement:* Return only standard error codes, a user-friendly message, and a correlation ID (request or trace ID). Never expose stack traces, SQL errors, internal paths, or system versions.
- *Verify:* A global exception handler normalizes errors; debug mode is off in production; error messages for authentication failures are uniform.

**OBS-02: Log redaction and audit trail**
- *Requirement:*
  - Record audit events for authentication, permission changes, sensitive data access, and key transactions, and defend against log injection.
  - Strictly mask or exclude passwords, tokens, session IDs, card numbers, national ID numbers, and full secret-bearing headers from application logs, crash reporters, and analytics systems.
- *Verify:* Sanitize newline and control characters in logged user input; review logging of request bodies and `Authorization` headers; audit logs are tamper-resistant and access-controlled.

**OBS-03: Monitoring and security alerting**
- *Requirement:* Monitor spikes in 401/403/429 responses, rising 5xx rates, slow queries, and anomalous payment or request-frequency behavior. Maintain real-time alerting and a response runbook.
- *Verify:* Alerts have owners and thresholds; there is a tested escalation path.

---

### Domain 8: Database, cache, and infrastructure security

**INFRA-01: Least privilege and network isolation**
- *Requirement:* Databases, Redis, message queues, and internal services are not exposed to the public internet. Allow access only from authorized VPC/private networks on required ports. The application's database account holds only the table and operation permissions it needs.
- *Verify:* Review security groups, firewall rules, and bind addresses; confirm the application does not use a database superuser; confirm cache and queue services require authentication.

**INFRA-02: Container, runtime, and dependency security**
- *Requirement:* Use trusted, minimal base images and run services as a non-root user. Integrate dependency vulnerability scanning in CI/CD (for example `npm audit`, `pip-audit`, `cargo audit`, Trivy, Dependabot) and container image scanning.
- *Verify:* Dockerfile uses a pinned, minimal base and a `USER` directive; CI fails or alerts on high-severity vulnerabilities; lockfiles are committed.

---

### Domain 9: Push notifications, deep links, and service-to-service security

**INTEG-01: Push notification and deep link safety**
- *Requirement:* Do not place sensitive personal data or verification codes in plaintext push titles or bodies. Treat deep-link parameters as untrusted: re-verify permissions when the app opens and again at the backend API.
- *Verify:* Notification payloads contain identifiers or generic text rather than sensitive content; deep-link targets go through the same authorization checks as any other request.

**INTEG-02: Internal service authentication**
- *Requirement:* Service-to-service calls authenticate the caller using mTLS, internal tokens, or strict network/IAM policy, so that internal admin and batch endpoints cannot be reached directly from outside.
- *Verify:* Internal endpoints are not routable from the public ingress; service identities are individually scoped; shared static internal tokens are avoided or rotated.

---

### Domain 10: Release governance, backward compatibility, and incident response

**REL-01: API versioning and graceful upgrades**
- *Requirement:* Older app versions remain in the field for some time. When changing fields, validation rules, or security mechanisms, plan version compatibility, deprecation notices, and a forced-upgrade strategy.
- *Verify:* Versioned routes or headers exist; a minimum-supported-version gate can be enforced server-side; security fixes have a path to reach old clients (or retire them).

**REL-02: Safe release, rollback, and incident response**
- *Requirement:* Maintain automated tests, database migration backups, and fast rollback procedures. Keep standard operating procedures for key rotation, token revocation, and security incident response.
- *Verify:* Migrations are reversible or backed up; a rollback has been rehearsed; an incident runbook names roles and contact paths.

## Report format

Use this structure for reviews and audits. Adapt length to scope: a single-endpoint review needs only the findings table and remediation.

```markdown
# Security Review: <system or feature name>

## Summary
<2-4 sentences: scope, overall posture, number of findings by severity, top risks>

## Scope and assumptions
- Reviewed: <files, services, configurations>
- Not reviewed / not verified: <items>
- Assumptions: <stack, deployment model>

## Findings

| ID | Control | Severity | Status | Evidence | Recommendation |
|----|---------|----------|--------|----------|----------------|
| AUTHZ-01 | Object-level authorization | Critical | Fail | `orders.py:42` fetches order by ID without owner check | Scope query by `owner_id` |

## Detailed remediation
### <Finding ID>: <Title>
- **Risk:** <what an attacker can do>
- **Evidence:** <file/line or configuration>
- **Fix:** <minimal change, with a short example>
- **Verification:** <how to confirm the fix, for example a test>

## Passed controls
<Brief list of controls verified as Pass>

## Recommended next steps
1. <Highest-priority action>
2. ...
```

For **design guidance**, replace the findings table with a short list of required controls per feature, each referencing a control ID, followed by an implementation outline.

For a **release checklist**, output a Markdown task list grouped by domain (`- [ ] AUTH-01 ...`) limited to the controls relevant to the release.

## Example

**Input:** "Here is my `GET /api/orders/{id}` handler. Is it secure?"

```python
@app.get("/api/orders/{order_id}")
def get_order(order_id: int, user=Depends(current_user)):
    return db.query(Order).filter(Order.id == order_id).first()
```

**Expected output (abbreviated):**

> **Finding AUTHZ-01 (Critical, Fail):** The handler authenticates the caller but never checks that the order belongs to them. Any authenticated user can read any order by iterating IDs (BOLA/IDOR).
>
> **Fix:**
> ```python
> order = (db.query(Order)
>            .filter(Order.id == order_id, Order.user_id == user.id)
>            .first())
> if order is None:
>     raise HTTPException(status_code=404)  # same response for "missing" and "not yours"
> return OrderResponse.model_validate(order)  # explicit response schema, no raw ORM leak
> ```
>
> **Also check:** INPUT-01 (response schema excludes internal fields), OBS-02 (the access is logged if orders are sensitive), and the sibling list/update/delete handlers for the same flaw.

## Reference mapping

These controls align with widely used industry references. Use them to give users standard terminology and links for further reading.

| Domain | Related references |
|---|---|
| 1. Authentication | OWASP API Security Top 10 (2023) API2 Broken Authentication; OWASP ASVS authentication and session management chapters |
| 2. Authorization | API1 Broken Object Level Authorization; API3 Broken Object Property Level Authorization; API5 Broken Function Level Authorization |
| 3. Input and files | OWASP Top 10 Injection; ASVS validation and file-handling chapters |
| 4. Transport and abuse | API4 Unrestricted Resource Consumption; API6 Unrestricted Access to Sensitive Business Flows; API7 Server Side Request Forgery |
| 5. Business logic | API6; ASVS business-logic verification |
| 6. Secrets and crypto | OWASP Top 10 Cryptographic Failures; ASVS cryptography chapter |
| 7. Logging and errors | OWASP Top 10 Security Logging and Monitoring Failures |
| 8. Infrastructure | API8 Security Misconfiguration; OWASP Top 10 Vulnerable and Outdated Components |
| 10. Release and versioning | API9 Improper Inventory Management |

For mobile-client-specific concerns, refer users to OWASP MASVS. This skill focuses on the backend.

## Limitations and responsible use

- **Defensive scope only.** Use this skill to find and fix weaknesses in systems the user owns or is authorized to assess. Do not produce working exploits, attack tooling, or step-by-step instructions for attacking third-party systems. Describing a vulnerability class and its remediation is appropriate; weaponizing it is not.
- **Not a certification.** A completed checklist is not a penetration test, a compliance attestation (for example SOC 2, PCI DSS, ISO 27001, HIPAA, GDPR), or a guarantee of security. State this when delivering a report, and recommend independent testing for high-risk systems.
- **Evidence over assurance.** Report only what you have verified. Mark unverified controls as **Not verified**. Do not present assumptions as findings.
- **Context matters.** Some controls (for example request signing, KMS-managed encryption) carry cost and complexity and may be disproportionate for low-risk systems. Explain trade-offs and let the user make an informed decision rather than applying every control mechanically.
- **Handle secrets carefully.** If the user pastes credentials, keys, or tokens, point it out, advise rotating them immediately, and do not repeat them in your output.

## License

Released under the MIT License. See the `LICENSE` file for the full text.
