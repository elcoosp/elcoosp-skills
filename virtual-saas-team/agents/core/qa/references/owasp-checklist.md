# OWASP Top 10 — Security Test Cases for QA

QA runs security-focused test cases as part of every QA-001 test plan for
features that handle user input, authentication, data storage, or API calls.
This checklist maps the OWASP Top 10 (2021) to concrete, executable test cases.

---

## A01 — Broken Access Control

The most common critical vulnerability. Test that users cannot access or modify
resources they don't own.

**Test cases:**
- Access a resource (by ID) belonging to another user while authenticated
- Access an admin endpoint with a non-admin user token
- Escalate privileges by modifying a role claim in a JWT or cookie
- Access a resource after permissions have been revoked (cache invalidation)
- IDOR (Insecure Direct Object Reference): iterate sequential IDs to find accessible records
  - e.g., `/api/invoices/1001`, `/api/invoices/1002`, `/api/invoices/1003`
  - Expected: only the authenticated user's invoices are accessible

**Pass criteria:** Every access attempt to another user's resource returns 403 or 404, never 200.

---

## A02 — Cryptographic Failures

Test that data is encrypted in transit and at rest, and that weak algorithms aren't used.

**Test cases:**
- Confirm all API endpoints use HTTPS (no HTTP)
- Confirm TLS version ≥ 1.2 (use `testssl.sh` or SSL Labs)
- Confirm HTTP Strict Transport Security (HSTS) header is present
- Inspect database exports: password fields should be hashed (bcrypt/Argon2), not plaintext or MD5
- Confirm password reset tokens are not predictable (not sequential integers, not timestamp-based)
- Confirm session tokens have sufficient entropy (≥ 128 bits)

**Pass criteria:** No plaintext passwords, no HTTP, TLS ≥ 1.2, HSTS present.

---

## A03 — Injection

SQL injection and its cousins (command injection, LDAP injection, template injection).

**Test cases for SQL injection:**
```
Input field: ' OR '1'='1
Input field: ' OR '1'='1'; DROP TABLE users; --
Input field: 1; SELECT sleep(5); --  (time-based blind)
```

**Test cases for command injection (if any shell commands involved):**
```
Input: ; ls -la
Input: && cat /etc/passwd
Input: | whoami
```

**Test cases for template injection:**
```
Input: {{7*7}}
Input: ${7*7}
Input: <%= 7*7 %>
```

**Pass criteria:** None of these inputs produce unexpected behaviour, errors
exposing internal state, or delayed responses indicating time-based injection.
All return normal validation errors.

---

## A04 — Insecure Design

Test that the application's design doesn't have inherent security weaknesses.

**Test cases:**
- Password reset: verify the reset link is single-use and expires
- Account lockout: verify brute force protection after N failed attempts
- Sensitive operations (delete account, change email): require re-authentication or MFA
- Sensitive data in URL parameters: API keys, tokens, or PII should not appear in query strings
  (URLs are logged by web servers and cached by browsers)

**Pass criteria:** Single-use tokens, lockout after threshold, re-auth for destructive operations, no secrets in URLs.

---

## A05 — Security Misconfiguration

Test that the deployment isn't exposing default credentials, verbose errors,
or unnecessary services.

**Test cases:**
- Access `/.env`, `/config.yml`, `/web.config`, `/.git/config` — should 404, not return content
- Access `/admin`, `/phpmyadmin`, `/actuator`, `/swagger-ui` — only accessible if intentional
- Confirm error responses don't include stack traces or internal file paths
- Check that unused HTTP methods (TRACE, OPTIONS) don't expose internal info
- Confirm `X-Powered-By`, `Server` headers are suppressed (don't reveal tech stack)

**Pass criteria:** No config file exposure, no verbose errors in production, no unintentional admin surfaces.

---

## A06 — Vulnerable and Outdated Components

This is largely caught by automated dependency scanning in CI (EL responsibility).
QA verifies the scan is passing.

**Test cases:**
- Confirm CI dependency scan is green (no critical CVEs) before sign-off
- Check the `npm audit` / `pip-audit` / `trivy` output in the CI run for this PR

**Pass criteria:** No critical or high severity CVEs in direct or transitive dependencies.

---

## A07 — Identification and Authentication Failures

**Test cases:**
- Try to use an expired JWT: should return 401
- Try to use a JWT with a modified `exp` claim: should return 401 (signature invalid)
- Try to use a JWT with `"alg": "none"`: should return 401 (algorithm must be specified and enforced)
- After password change: old session tokens should be invalidated
- After logout: token should not be accepted on next request

**Pass criteria:** All invalid or expired tokens return 401. `"alg": "none"` is rejected.

---

## A08 — Software and Data Integrity Failures

**Test cases:**
- If the app accepts serialised objects (JSON, XML): attempt to inject unexpected types
- If the app uses auto-update mechanisms: verify update packages are signature-verified
- CI/CD: verify that deployments require PRs (no direct commits to main)
- Verify that infrastructure changes require Terraform plan review (no manual console changes)

**Pass criteria:** Serialisation edge cases handled gracefully; deployment controls in place.

---

## A09 — Security Logging and Monitoring Failures

**Test cases:**
- Trigger a known security event (failed login × 5, accessing another user's resource)
- Verify that the event appears in logs with: timestamp, user ID, IP, action, outcome
- Verify that sensitive data (passwords, tokens, PII) is NOT in the log output
- Verify that AUD receives an event for any guardrail violation

**Pass criteria:** Security events are logged with full context; PII is absent from logs.

---

## A10 — Server-Side Request Forgery (SSRF)

Relevant for features that fetch external URLs or make server-side HTTP requests
based on user input (e.g., URL preview, webhook configuration, import from URL).

**Test cases:**
```
Input URL: http://169.254.169.254/latest/meta-data/  (AWS instance metadata)
Input URL: http://localhost/admin
Input URL: http://internal-service.company.internal/
Input URL: file:///etc/passwd
```

**Pass criteria:** All of the above are rejected with a validation error before
any outbound request is made. The application must validate URLs against an
allowlist of permitted schemes and destinations, not a blocklist.

---

## Running the OWASP Tests

For manual execution, use Burp Suite Community Edition or OWASP ZAP (both free).

For automated scanning, integrate OWASP ZAP into the CI pipeline for scheduled
scans against the staging environment. ZAP can be run headlessly:

```bash
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://staging.product.com \
  -r zap-report.html
```

The ZAP baseline scan runs passive checks only (no active injection).
For active injection testing, use the full scan:

```bash
docker run -t owasp/zap2docker-stable zap-full-scan.py \
  -t https://staging.product.com \
  -r zap-full-report.html
```

Always run full scans against staging, never production.
