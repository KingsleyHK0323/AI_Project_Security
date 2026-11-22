---
name: backend-security-checklist
description: A structured, production-grade security baseline for general-purpose backend services, web APIs, and server-side applications. Use this skill whenever the user writes, reviews, audits, or hardens server-side code or infrastructure, even if they do not explicitly say "security". Trigger on requests such as "review this API", "is this query safe", "audit my backend", "pre-release security review", "pull request security check", "add rate limiting", "secure my file upload", "harden my CI/CD pipeline", or any mention of SQL injection, command injection, path traversal, SSRF, XXE, insecure deserialization, JWT, session management, password hashing, IDOR, BOLA, CORS, GraphQL limits, webhooks, secrets, TLS, audit logging, least-privilege database accounts, container hardening, idempotency, race conditions, or dependency scanning. Covers 10 domains and 35 verifiable controls with severity ratings and a standard report format.
license: MIT
metadata:
  author: Kingsley (@KingsleyHK0323)
  repository: https://github.com/KingsleyHK0323/AI_Project_Security
  version: "1.0.1"
  domain: application-security
  companion-skills: backend-app-security-checklist, frontend-app-security-checklist
---

# Backend Security Checklist

A practical, defense-in-depth security baseline for server-side software. It consolidates the controls that most often separate a hardened production backend from a vulnerable one: injection defense, authentication and session management, access control, API abuse resistance, secrets and cryptography, file and integration handling, observability, infrastructure, business-logic integrity, and software supply chain and operations.

The skill supports three jobs: **designing** services securely, **reviewing** existing code, configuration, and pipelines, and **producing a release-gate checklist** for a team.

## Relationship to companion skills

This is the general-purpose backend baseline, applicable to any server-side system (web apps, public APIs, internal services, SaaS back offices).

- Use `backend-app-security-checklist` in addition when the backend serves **mobile or consumer apps** and app-specific concerns apply: device binding, request signing and anti-replay, push notifications, deep links, and forced-upgrade compatibility.
- Use `frontend-app-security-checklist` for the **client side** of mobile, hybrid, and desktop apps.

When both this skill and the app-focused backend skill apply, treat this skill as the baseline and use the other for the app-specific layer. Do not duplicate findings; cite the control ID that best fits.

## Core principles

Apply these principles throughout. They explain *why* the controls exist and help you make sound judgments when a situation does not match the checklist exactly.

1. **Treat all input as hostile.** Data from clients, queues, webhooks, files, headers, and even internal services is untrusted until validated at the boundary where it enters your code.
2. **Separate code from data.** Injection flaws (SQL, shell, XML, templates, deserialization) share one root cause: attacker-controlled data interpreted as instructions. Parameterize, allowlist, and avoid interpreters wherever possible.
3. **Authorize every request at the object level.** Authentication answers "who are you"; authorization answers "may you do this to this object". Most serious API breaches are authorization failures.
4. **Default deny, least privilege.** Permissions, network paths, database grants, and CORS origins start closed and are opened explicitly and narrowly.
5. **Make state changes safe to retry and safe to race.** Networks time out and users double-click. Money, points, inventory, and notifications need idempotency and concurrency control.
6. **Fail safely and quietly.** Errors reveal nothing useful to attackers; logs record what responders need without recording secrets.
7. **Defense in depth, with an assume-breach mindset.** No single control is sufficient. Plan for key leakage, credential theft, and compromised dependencies with rotation, revocation, monitoring, and rehearsed response.

## When to use this skill

Use it when the user is:

- Designing or implementing endpoints, authentication, authorization, uploads, payments, integrations, or background jobs.
- Asking for a code review, architecture review, threat model, or security audit of a backend.
- Preparing a release, pull request, or compliance review and wants a security gate.
- Configuring databases, caches, queues, containers, CI/CD pipelines, or cloud networking.
- Receiving webhooks, calling third-party URLs, or parsing untrusted files or serialized data.

Do not use it for purely client-side security questions (use `frontend-app-security-checklist`), for penetration testing or exploit development, or for formal compliance certification. See [Limitations and responsible use](#limitations-and-responsible-use).

## Workflow

### Step 1: Establish scope and context

Identify, from the conversation and any provided code, what is in scope. Gather only what you cannot infer:

- **Stack:** language, framework, ORM, database, cache, queue, API style (REST, GraphQL, RPC).
- **Authentication model:** sessions with cookies, JWT, OAuth/OIDC, API keys, mTLS.
- **Sensitive assets:** PII, payment data, health data, credentials, virtual currency, inventory.
- **Deployment model:** cloud provider, containers, serverless, network layout, CI/CD tooling.
- **Mode of work:** **Design guidance**, **Code review**, or **Release checklist**.

If the user provided code, configuration, Dockerfiles, IaC, or pipeline definitions, read them before answering. Base findings on evidence, not assumptions.

### Step 2: Select the relevant domains

Not every domain applies to every task. Map the request to domains, and state which you are covering and why.

| Task | Prioritize domains |
|---|---|
| New or changed endpoint that reads or writes data | 1, 3, 4, 7 |
| Login, signup, password reset, MFA, sessions, JWT | 2, 4, 7 |
| File upload, download, or document parsing | 6, 1, 3 |
| Outbound URL fetching, webhooks, third-party calls | 1, 6, 5 |
| Payments, points, coupons, inventory, limited quotas | 9, 3, 4 |
| Database, container, network, or cloud configuration | 8, 5 |
| Pipeline, dependency, or deployment changes | 10, 5 |
| Pre-release gate or full audit | All |

### Step 3: Evaluate each applicable control

For every applicable control in the [Control catalog](#control-catalog), assign one status:

- **Pass**: Evidence shows the control is implemented correctly.
- **Partial**: Implemented, but with gaps (state the gap).
- **Fail**: Missing or incorrectly implemented.
- **N/A**: Not applicable (state why, for example "no GraphQL endpoint").
- **Not verified**: Evidence is not available (for example, infrastructure configuration was not provided). Never mark a control as Pass without evidence.

### Step 4: Rate severity

Rate each Fail or Partial finding using exploitability and impact:

| Severity | Meaning | Examples |
|---|---|---|
| **Critical** | Directly exploitable; leads to remote code execution, account takeover, large-scale data exposure, or financial loss | SQL injection on a reachable endpoint; missing object-level authorization on user data; production secret committed to the repository; unauthenticated deserialization of attacker data |
| **High** | Exploitable with modest effort or a precondition; serious impact | SSRF reaching a cloud metadata endpoint; no rate limiting on login or OTP verification; race condition allowing negative balances; JWT algorithm not pinned |
| **Medium** | Weakens defenses or leaks limited information | Verbose error messages; missing pagination caps; permissive CORS with limited impact; weak log redaction |
| **Low** | Hardening gap with limited direct impact | Missing optional security headers; outdated but unexploitable TLS option |
| **Info** | Best-practice recommendation | Documentation and process improvements |

### Step 5: Report and remediate

Deliver the result using the [Report format](#report-format). For every Fail or Partial finding, give a concrete, minimal remediation that fits the user's stack. Provide short code or configuration examples when they clarify the fix. Order findings by severity, then by effort to fix.

## Control catalog

Each control has an ID for traceability, the requirement, and what to verify. Use the IDs in reports and pull request comments.

---

### Domain 1: Input validation, injection defense, and data handling

**INJ-01: Parameterized queries**
- *Requirement:* All database access uses prepared statements or ORM parameter binding. Never build SQL or NoSQL queries by string concatenation or interpolation.
- *Verify:* Search for string-built queries, f-strings, template literals, and `+` concatenation near query calls. Remember that parameters cannot bind identifiers: table names, column names, and `ORDER BY` fields must come from a server-side allowlist. For NoSQL, reject user-supplied operator objects (for example `$ne`, `$where`).

**INJ-02: Command injection prevention**
- *Requirement:* Avoid executing external system commands (`exec`, `spawn`, `system`, `Runtime.exec`). If unavoidable, pass an argument array instead of a shell string, never enable a shell, and strictly allowlist argument values.
- *Verify:* Search for `shell=True`, backtick execution, and shell strings built from user input; prefer a native library over shelling out; guard against option injection (for example arguments beginning with `-`).

**INJ-03: Path traversal prevention**
- *Requirement:* For file read, download, or delete operations, resolve the canonical absolute path (for example `path.resolve`, `realpath`) and confirm it stays inside the permitted root directory, blocking `../` sequences and symbolic-link escapes.
- *Verify:* Confirm the containment check compares against the root *with a trailing path separator* (so `/data/uploads-evil` does not match `/data/uploads`), is performed after resolution, and that archive extraction (zip/tar) validates each entry path before writing.

**INJ-04: Server-side request forgery (SSRF) defense**
- *Requirement:* When the backend fetches external URLs, webhooks, or images, allow only `http`/`https`, apply a domain or IP allowlist where feasible, and block loopback, link-local, and private ranges (`127.0.0.0/8`, `169.254.169.254` and `169.254.0.0/16`, RFC 1918 ranges, carrier-grade NAT, and their IPv6 equivalents), as well as redirects into those ranges.
- *Verify:* Resolve DNS and validate the destination IP *after* resolution, and connect to the validated IP to resist DNS rebinding; disable automatic redirects or re-validate each hop; handle IPv4-mapped IPv6 and alternative IP encodings; enforce timeouts and response-size limits; on cloud platforms, require the hardened metadata service (for example AWS IMDSv2) and restrict instance role permissions.

**INJ-05: Structured schema validation and type checking**
- *Requirement:* Validate request bodies, query strings, path parameters, and headers against a strict schema (for example Zod, Joi, Pydantic). Reject unexpected fields, wrong types, negative numbers, and out-of-range or overflowing values.
- *Verify:* Handlers bind to explicit allowlisted models rather than passing raw bodies to the ORM (mass assignment); numeric fields have bounds; string fields have maximum lengths; validation happens server-side regardless of client validation.

**INJ-06: Insecure deserialization prevention**
- *Requirement:* Do not deserialize untrusted data with native object serializers (Python `pickle`, Java native serialization, PHP `unserialize`, .NET `BinaryFormatter`, unsafe YAML loaders). Use data-only formats such as JSON and validate the structure.
- *Verify:* Search for these APIs on any code path that touches external input, caches shared with other tenants, or message queues; confirm safe YAML loading (for example `safe_load`); if native deserialization is unavoidable, require a strict class allowlist and integrity protection (signed payloads).

**INJ-07: XML and XXE defense**
- *Requirement:* When parsing XML, disable external entity resolution and DTD processing.
- *Verify:* Inspect parser configuration (for example feature flags in Java parsers, `defusedxml` in Python); confirm limits on entity expansion and document size to resist "billion laughs" attacks; also check XML-based formats such as SVG, SAML, and Office documents.

---

### Domain 2: Authentication and session management

**AUTH-01: Password hashing**
- *Requirement:* Never store passwords in plaintext or with fast hashes such as MD5, SHA-1, or SHA-256 (salted or not). Use a slow, salted, memory- or compute-hard algorithm: **Argon2id** (preferred), **bcrypt**, or **PBKDF2**, with an appropriate cost factor.
- *Verify:* Inspect the hashing call and parameters against current OWASP Password Storage guidance (cost factors rise over time, so confirm current recommendations); confirm unique per-password salts; for bcrypt, account for its 72-byte input limit; confirm legacy hashes are upgraded on next successful login; consider screening new passwords against known-breached lists and allowing long passphrases rather than imposing arbitrary composition rules.

**AUTH-02: Brute-force and credential-stuffing protection**
- *Requirement:* Login, password-reset, and OTP verification endpoints enforce rate limiting, account lockout, or progressive delay (exponential backoff). Responses must not reveal whether an account exists (prevent username enumeration).
- *Verify:* Limits apply per account, per IP, and per device where available; "unknown user" and "wrong password" return identical status, body, and similar timing; lockout design cannot be abused to deny service to legitimate users (prefer progressive delay and step-up challenges over permanent lockout); consider MFA and bot-detection signals for credential-stuffing resistance.

**AUTH-03: Token and session security**
- *Requirement:*
  - Session IDs come from a cryptographically secure random generator (CSPRNG), are regenerated after login (preventing session fixation), and are invalidated server-side on logout.
  - If using JWT, strictly validate the algorithm (reject `alg: none` and algorithm confusion), signing-key strength, `exp`, `iss`, and `aud`, and plan a revocation or denylist mechanism.
- *Verify:* For cookie-based sessions, confirm `HttpOnly`, `Secure`, and an appropriate `SameSite` attribute, and confirm CSRF protection on state-changing endpoints; for JWT, confirm the verification call pins accepted algorithms and does not trust the token's own header to choose the key or algorithm; confirm access-token lifetimes are short and that password change and logout invalidate existing sessions.

**AUTH-04: Password reset and MFA design**
- *Requirement:* Reset links and tokens are highly random, short-lived, and single-use. MFA/OTP flows limit attempts and resist replay.
- *Verify:* Reset tokens have at least 128 bits of entropy, are stored hashed, expire quickly, and are invalidated after use or password change; reset responses do not reveal account existence; TOTP codes cannot be reused within their validity window; recovery codes are single-use and stored hashed.

---

### Domain 3: Access control and authorization

**ACL-01: Object-level authorization (IDOR / BOLA prevention)**
- *Requirement:* Every access or modification of a specific resource (for example `/api/orders/{id}`, `/api/files/{id}`) compares the authenticated principal with the resource's owner or owning organization. Do not rely on an ID supplied by the client or on a route merely existing.
- *Verify:* Each handler taking a resource ID also enforces ownership or tenancy, ideally through a shared policy layer or query scoping (for example `WHERE id = ? AND owner_id = ?`); test with two separate user accounts; check list, export, search, bulk, and background-job paths as well as single-object reads; return the same response for "not found" and "not yours" where existence is sensitive.

**ACL-02: Function-level and administrative endpoint control**
- *Requirement:* Admin consoles, internal APIs, and sensitive operations validate role and permission (RBAC / ABAC) in middleware, and **default to deny**.
- *Verify:* Authorization is enforced centrally; new routes are denied until explicitly granted; admin routes are protected on the server, not merely hidden in the UI; internal and debug endpoints are not reachable from the public internet.

**ACL-03: Horizontal and vertical privilege escalation prevention**
- *Requirement:* Ordinary users cannot read other users' data (horizontal), and cannot elevate to administrator by changing a role field, request parameter, or calling an API directly (vertical).
- *Verify:* Role and ownership attributes are never accepted from request bodies; privilege changes require an authorized actor and are audit-logged; test by replaying a low-privilege session against admin routes and a second user's resource IDs.

---

### Domain 4: API security, traffic control, and abuse resistance

**API-01: Rate limiting and quotas**
- *Requirement:* Public APIs, login, SMS and email sending, search, and computationally expensive endpoints are limited by IP, account, or API key, to resist brute force, scraping, and application-layer denial of service.
- *Verify:* Limits are enforced at a gateway or middleware layer; costly side-effect operations (SMS, email, exports) have per-user quotas; limiter state cannot be bypassed by rotating headers such as `X-Forwarded-For` when the service is not behind a trusted proxy.

**API-02: Pagination and payload limits**
- *Requirement:* List endpoints enforce a default and a maximum `limit` (for example a cap of 100). Request body size and JSON nesting depth are limited to prevent resource exhaustion.
- *Verify:* `limit` and `offset` parameters are clamped server-side; body-size limits are enforced before full buffering; file, array, and string field sizes are bounded.

**API-03: CORS and cross-origin policy**
- *Requirement:* Configure `Access-Control-Allow-Origin` as a precise allowlist. Do not combine a wildcard or blindly reflected `Origin` with `Access-Control-Allow-Credentials: true`. Permit only the necessary methods and headers.
- *Verify:* Origin matching is exact (no substring or suffix matching that `trusted.com.evil.com` can defeat); the `null` origin is not allowed; preflight responses are cached sensibly; CORS is not treated as an authorization mechanism, since it only governs browsers.

**API-04: GraphQL and RPC protections (where applicable)**
- *Requirement:* Limit query depth, query complexity, and batch/alias counts. Disable introspection in production where the API is not intentionally public.
- *Verify:* Depth and cost limits exist and are tested; batching and aliasing cannot multiply expensive operations or bypass rate limits (for example repeated login mutations in one request); field-level authorization is enforced in resolvers; error messages do not leak schema hints.

---

### Domain 5: Secrets, cryptography, and key management

**CRYP-01: No hardcoded secrets**
- *Requirement:* API keys, database passwords, JWT secrets, private keys, and cloud credentials must not be written in code or committed to Git. Inject them via a secrets manager or environment variables, and use secret-scanning tools to check.
- *Verify:* Scan the repository, its full history, container images, and CI logs; confirm `.env` files are ignored; confirm leaked secrets are rotated and revoked, not merely deleted from the latest commit; confirm secrets are scoped to least privilege.

**CRYP-02: Encryption in transit**
- *Requirement:* Enforce HTTPS site-wide using TLS 1.2 or later (TLS 1.3 recommended). Disable weak cipher suites and obsolete protocols (SSLv3, TLS 1.0, TLS 1.1).
- *Verify:* Plain HTTP redirects or is refused; HSTS is set for browser-facing surfaces; outbound clients validate certificates and do not disable verification; internal service traffic and database connections are also encrypted where the threat model warrants.

**CRYP-03: Encryption at rest and key management**
- *Requirement:* Encrypt sensitive PII, medical or financial records, and credentials at the database and file-storage layers as compliance requires, and manage keys in a KMS with a defined key-rotation process.
- *Verify:* Keys are stored separately from the data they protect; rotation and emergency revocation are documented and tested; backups and snapshots are encrypted; application-level encryption uses vetted libraries and authenticated modes rather than custom cryptography.

---

### Domain 6: File upload and third-party integration security

**FILE-01: Secure file upload and download**
- *Requirement:*
  - Validate file size, an extension allowlist, and real content (magic bytes, or re-encode images).
  - Regenerate a random storage filename so the original name cannot overwrite system files or place executable content.
  - Store uploads on an isolated domain or object storage (for example S3) with execute permission disabled, and serve with correct `Content-Type`, `Content-Disposition`, and `X-Content-Type-Options: nosniff`.
- *Verify:* Original filenames are never used as storage paths; uploads are not served from the application origin in an executable context; downloads enforce authorization (see ACL-01); size limits are enforced during streaming; archives are checked for zip-slip and decompression bombs; consider malware scanning for user-shared files.

**HOOK-01: Webhook and third-party callback verification**
- *Requirement:* When receiving webhooks (for example Stripe, GitHub, LINE), verify the HMAC signature and timestamp (preventing replay attacks), and process through an asynchronous queue that handles retries and idempotency.
- *Verify:* The signature is computed over the raw request body and compared in constant time; timestamps outside a tight window are rejected; the event ID is stored so duplicate deliveries do not repeat side effects; the endpoint responds quickly and defers heavy work.

---

### Domain 7: Error handling, logging, and observability

**OBS-01: Safe error responses**
- *Requirement:* In production, catch unexpected exceptions and return only a generic message with a trace identifier (Request ID / Trace ID). Never leak SQL statements, stack traces, file paths, or framework versions.
- *Verify:* A global exception handler normalizes errors; debug mode is disabled in production; authentication and authorization failures return uniform messages; 404 and 403 behavior does not leak resource existence where that matters.

**OBS-02: Audit logging and log sanitization**
- *Requirement:*
  - Record audit events: login success and failure, permission changes, critical data changes, and anomalous access.
  - Before writing logs, **strictly mask or exclude** passwords, tokens, session IDs, card numbers, national ID numbers, and sensitive request and response headers, and defend against log injection (CRLF).
- *Verify:* Use structured logging and sanitize newline and control characters in logged user input; review logging of full request bodies and `Authorization` or `Cookie` headers; confirm log access is restricted and audit logs are tamper-resistant and retained per policy.

**OBS-03: Security alerting and monitoring**
- *Requirement:* Monitor for spikes in 401/403/429 responses, slow database queries, and sudden error-rate increases, with real-time alerting.
- *Verify:* Alerts have owners, thresholds, and a tested escalation path; dashboards cover authentication failures, privilege changes, and unusual data-export volume.

---

### Domain 8: Database and infrastructure security

**INFRA-01: Database least privilege**
- *Requirement:* The application's database account holds only the privileges it needs (`SELECT`, `INSERT`, `UPDATE`, `DELETE` on specific tables). Never connect as `root` or a superuser, and avoid unnecessary DDL privileges.
- *Verify:* Review grants; use a separate, more privileged account only for migrations, run from the deployment pipeline; confirm read-only workloads use read-only credentials; confirm default and sample accounts are removed.

**INFRA-02: Network isolation and firewalling**
- *Requirement:* Databases, caches (Redis), message queues (RabbitMQ, Kafka), and internal microservices are not exposed to the public internet. Permit access only from trusted VPC or private networks and specific ports.
- *Verify:* Review security groups, firewall rules, and bind addresses; confirm caches and queues require authentication and TLS where appropriate; confirm management consoles and debug ports are not publicly reachable.

**INFRA-03: Container and runtime hardening**
- *Requirement:* Use minimal base images from trusted sources, run the application as a non-root user, and regularly scan the OS layer and packages for vulnerabilities.
- *Verify:* The Dockerfile pins a base image (ideally by digest) and sets a `USER` directive; consider a read-only root filesystem and dropped Linux capabilities; no secrets are baked into image layers; image scanning runs in CI and on a schedule.

---

### Domain 9: Transaction consistency, concurrency, and business logic

**BIZ-01: Idempotency**
- *Requirement:* Critical writes such as payment, order creation, and issuing points or coupons use an idempotency key or a unique constraint so network retries and double-clicks cannot execute twice.
- *Verify:* The key is scoped to the user and operation; the first result is returned on retry; a database unique constraint backs the application logic; background-job and message-consumer paths are also idempotent (at-least-once delivery is the norm).

**BIZ-02: Race conditions and transaction isolation**
- *Requirement:* For balance deduction, stock reservation, or limited-quantity sales, use database transactions, pessimistic locks (`SELECT ... FOR UPDATE`), or optimistic concurrency (version column / compare-and-swap) to prevent overselling and negative balances.
- *Verify:* Read-check-write sequences are atomic; database-level constraints (for example `balance >= 0`) exist as a last line of defense; validate with concurrent-request tests rather than reasoning alone.

**BIZ-03: Complete business-rule validation**
- *Requirement:* Product prices, discount calculations, state-machine transitions, and quantity limits are computed and verified on the backend from authoritative data. Never trust client-supplied computed results.
- *Verify:* Totals are recomputed server-side from catalog data; illegal state transitions are rejected; coupons and promotions enforce eligibility, usage caps, and expiry on the server; amounts reject negative, zero, or overflow values where inappropriate.

---

### Domain 10: Supply chain, CI/CD, and operations

**OPS-01: Dependency vulnerability scanning**
- *Requirement:* Integrate `npm audit`, `pip-audit`, `cargo audit`, Trivy, or Dependabot into CI, and patch known vulnerabilities promptly.
- *Verify:* Lockfiles are committed and builds are reproducible; scanning gates or alerts on high-severity findings with an owner and an SLA; consider generating an SBOM; review new dependencies for maintenance health and typosquatting before adoption.

**OPS-02: Secure deployment and rollback**
- *Requirement:* Prohibit manual changes and deployments directly on production. Maintain automated tests, database migration backups, and a fast rollback procedure.
- *Verify:* Deployments flow only through the pipeline with reviewed changes (branch protection, required reviews); pipeline credentials are least-privilege and short-lived; third-party CI actions and plugins are pinned; migrations are reversible or backed up; a rollback has been rehearsed.

**OPS-03: Regular security testing and exercises**
- *Requirement:* Write security-focused unit and integration tests for core APIs, run SAST and DAST scans, and schedule penetration tests and incident-response exercises according to risk.
- *Verify:* Tests cover authorization (cross-user access), validation, and abuse cases; scan results are triaged; an incident-response runbook names roles and contact paths and has been exercised; findings feed back into the checklist.

## Report format

Use this structure for reviews and audits. Adapt length to scope: a single-endpoint review needs only the findings table and remediation.

```markdown
# Security Review: <system or feature name>

## Summary
<2-4 sentences: scope, overall posture, number of findings by severity, top risks>

## Scope and assumptions
- Reviewed: <files, services, configurations, pipelines>
- Not reviewed / not verified: <items>
- Assumptions: <stack, deployment model, threat model>

## Findings

| ID | Control | Severity | Status | Evidence | Recommendation |
|----|---------|----------|--------|----------|----------------|
| INJ-01 | Parameterized queries | Critical | Fail | `users.py:27` interpolates `name` into SQL | Bind parameters; allowlist sort field |

## Detailed remediation
### <Finding ID>: <Title>
- **Risk:** <what an attacker can do>
- **Evidence:** <file/line or configuration>
- **Fix:** <minimal change, with a short example>
- **Verification:** <how to confirm the fix, for example a regression test>

## Passed controls
<Brief list of controls verified as Pass>

## Recommended next steps
1. <Highest-priority action>
2. ...
```

For **design guidance**, replace the findings table with a short list of required controls per feature, each referencing a control ID, followed by an implementation outline.

For a **release checklist**, output a Markdown task list grouped by domain (`- [ ] INJ-01 ...`) limited to the controls relevant to the release.

## Example

**Input:** "Is this search endpoint okay?"

```python
@app.get("/api/users")
def search_users(name: str, sort: str = "created_at", limit: int = 50):
    sql = f"SELECT id, name FROM users WHERE name = '{name}' ORDER BY {sort} LIMIT {limit}"
    return db.execute(sql).fetchall()
```

**Expected output (abbreviated):**

> **Finding INJ-01 (Critical, Fail):** `name`, `sort`, and `limit` are interpolated directly into the SQL string, so an attacker can inject arbitrary SQL (for example through `name`, or by sending a malicious `sort` value). Parameter binding fixes `name` and `limit`, but cannot protect the `ORDER BY` identifier, which needs an allowlist.
>
> **Fix:**
> ```python
> ALLOWED_SORT = {"created_at": "created_at", "name": "name"}
>
> @app.get("/api/users")
> def search_users(name: str, sort: str = "created_at", limit: int = 50):
>     order_by = ALLOWED_SORT.get(sort)
>     if order_by is None:
>         raise HTTPException(status_code=400, detail="Invalid sort field")
>     limit = max(1, min(limit, 100))  # enforce a server-side maximum (API-02)
>     stmt = text(
>         f"SELECT id, name FROM users WHERE name = :name ORDER BY {order_by} LIMIT :limit"
>     )
>     return db.execute(stmt, {"name": name, "limit": limit}).fetchall()
> ```
>
> **Also check:** ACL-01 (should this endpoint expose all users, or only the caller's tenant?), INJ-05 (declare a validated request model), API-01 (rate-limit search), and OBS-01 (database errors must not reach the client).

## Reference mapping

These controls align with widely used industry references. Use them to give users standard terminology and links for further reading. Check the latest edition of each reference, since category numbering changes between releases.

| Domain | Related references |
|---|---|
| 1. Input and injection | OWASP Top 10 Injection, Insecure Design, Software and Data Integrity Failures (deserialization), Server-Side Request Forgery; OWASP Cheat Sheet Series (SQL Injection Prevention, Deserialization, XXE Prevention, SSRF Prevention) |
| 2. Authentication | OWASP Top 10 Identification and Authentication Failures; OWASP API Security Top 10 Broken Authentication; NIST SP 800-63B; OWASP Password Storage Cheat Sheet |
| 3. Access control | OWASP Top 10 Broken Access Control; OWASP API Security Top 10 Broken Object Level Authorization, Broken Function Level Authorization |
| 4. API and abuse | OWASP API Security Top 10 Unrestricted Resource Consumption; OWASP GraphQL Cheat Sheet |
| 5. Secrets and crypto | OWASP Top 10 Cryptographic Failures; OWASP Secrets Management Cheat Sheet |
| 6. File and integrations | OWASP File Upload Cheat Sheet; OWASP API Security Top 10 Unsafe Consumption of APIs |
| 7. Logging and errors | OWASP Top 10 Security Logging and Monitoring Failures; OWASP Logging Cheat Sheet |
| 8. Infrastructure | OWASP Top 10 Security Misconfiguration; CIS Benchmarks; OWASP Docker Security Cheat Sheet |
| 9. Business logic | OWASP API Security Top 10 Unrestricted Access to Sensitive Business Flows; OWASP Top 10 Insecure Design |
| 10. Supply chain and operations | OWASP Top 10 Vulnerable and Outdated Components; software supply chain frameworks such as SLSA; NIST SSDF |

For a comprehensive verification standard, refer users to the OWASP Application Security Verification Standard (ASVS).

## Limitations and responsible use

- **Defensive scope only.** Use this skill to find and fix weaknesses in systems the user owns or is authorized to assess. Do not produce working exploits, attack payload collections, or step-by-step instructions for attacking third-party systems. Describing a vulnerability class and its remediation is appropriate; weaponizing it is not.
- **Not a certification.** A completed checklist is not a penetration test, a compliance attestation (for example SOC 2, PCI DSS, ISO 27001, HIPAA, GDPR), or a guarantee of security. State this when delivering a report, and recommend independent testing for high-risk systems.
- **Evidence over assurance.** Report only what you have verified. Mark unverified controls as **Not verified**. Do not present assumptions as findings.
- **Context matters.** Some controls (for example application-level encryption, GraphQL cost analysis, SBOM generation) carry cost and complexity and may be disproportionate for low-risk systems. Explain trade-offs and let the user make an informed decision rather than applying every control mechanically.
- **Recommendations age.** Cost factors, protocol versions, and framework defaults change. When a recommendation depends on a specific parameter or version, say so and encourage the user to confirm against current official guidance.
- **Handle secrets carefully.** If the user pastes credentials, keys, or tokens, point it out, advise rotating them immediately, and do not repeat them in your output.

## License

Released under the MIT License. See the `LICENSE` file for the full text.
