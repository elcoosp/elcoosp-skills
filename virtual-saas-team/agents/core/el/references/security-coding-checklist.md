# Security Coding Checklist

EL runs this checklist before opening any PR. It maps to the OWASP Top 10
and the guardrails AUD enforces. A failed item blocks merge.

---

## How to Use This Checklist

Run through every section that applies to your change. Mark items N/A if
genuinely not applicable to this PR — but be honest. "N/A" on input validation
for a form handler is a red flag, not a time-saver.

Items marked with 🔴 are P0 security violations. AUD will quarantine any
EL-001 that contains them. Treat them as blockers, not warnings.

---

## 1. Authentication and Authorisation

- [ ] Every protected endpoint checks authentication before executing any logic
- [ ] Authorisation is enforced at the **API layer**, not only in the UI
- [ ] The authenticated user's permissions are checked against the requested resource
- [ ] Privilege escalation is impossible: a user cannot grant themselves permissions they don't have
- [ ] Session tokens are invalidated on logout and password change
- [ ] Password reset tokens are single-use and expire within 15 minutes
- [ ] Admin endpoints require elevated permissions, not just authentication
- [ ] Multi-tenant endpoints enforce tenant isolation: user A cannot access user B's data

---

## 2. Input Validation

- [ ] All inputs are validated **server-side** — client-side validation is UX only
- [ ] File uploads: type checked by MIME type inspection (not extension), size limited, scanned
- [ ] Integer inputs have minimum and maximum bounds enforced
- [ ] String inputs have maximum length enforced
- [ ] Enum inputs reject values not in the allowed set
- [ ] JSON inputs are schema-validated before processing
- [ ] No `eval()`, `exec()`, or dynamic code execution on user-supplied input
- [ ] Regular expressions do not have ReDoS vulnerabilities (avoid catastrophic backtracking)

---

## 3. SQL / NoSQL Injection

🔴 These are automatic P0 violations — AUD quarantines immediately.

- [ ] 🔴 All database queries use parameterised queries or ORMs — zero string concatenation in SQL
- [ ] 🔴 No raw SQL built from user input anywhere in the codebase
- [ ] 🔴 NoSQL queries use typed query builders — no `$where` clauses with user input
- [ ] Stored procedures do not concatenate user input into dynamic SQL

**Wrong:**
```python
query = f"SELECT * FROM users WHERE email = '{email}'"  # NEVER
```

**Right:**
```python
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (email,))
```

---

## 4. Cross-Site Scripting (XSS)

- [ ] All user-supplied data rendered in HTML is escaped/encoded
- [ ] React/Vue: no `dangerouslySetInnerHTML` or `v-html` with user data
- [ ] Content-Security-Policy header is set (no `unsafe-inline` for scripts)
- [ ] Cookies set with `HttpOnly` and `Secure` flags
- [ ] `SameSite=Strict` or `SameSite=Lax` on session cookies

---

## 5. Secrets Management

🔴 These are automatic P0 violations — AUD quarantines immediately.

- [ ] 🔴 No secrets, API keys, tokens, or passwords in source code
- [ ] 🔴 No secrets in environment variables committed to the repo (`.env` files in `.gitignore`)
- [ ] 🔴 No secrets in test fixtures, mock data, or comments
- [ ] 🔴 All secrets accessed via the approved secrets manager at runtime
- [ ] Secret rotation does not require a code deployment
- [ ] Log statements do not emit secret values (check format string arguments)

**Patterns AUD scans for (automatic quarantine):**
```
AKIA[0-9A-Z]{16}          # AWS access key
sk_live_[0-9a-zA-Z]{24}   # Stripe live key
ghp_[0-9a-zA-Z]{36}       # GitHub personal access token
[Pp]assword\s*=\s*["\'][^"\']+["\']  # Hardcoded password
[Aa][Pp][Ii]_?[Kk][Ee][Yy]\s*=\s*["\'][^"\']+["\']  # API key assignment
```

---

## 6. Cryptography

- [ ] Passwords are hashed with bcrypt, Argon2, or scrypt — never MD5, SHA-1, or unsalted SHA-256
- [ ] Encryption uses AES-256-GCM (authenticated encryption) — never AES-ECB
- [ ] TLS 1.2 minimum; TLS 1.3 preferred for all external connections
- [ ] Certificates are validated — no `verify=False` or `InsecureRequestWarning` suppression
- [ ] Random tokens use a cryptographically secure RNG (`secrets` module in Python, `crypto.randomBytes` in Node.js)
- [ ] JWT signing uses RS256 or ES256 — never HS256 with a weak or shared secret

---

## 7. Dependency Security

- [ ] All new dependencies are from maintained, reputable sources
- [ ] No dependencies with known critical CVEs (dependency scan in CI will catch this)
- [ ] Transitive dependencies reviewed for license compatibility
- [ ] `package-lock.json` / `requirements.txt` / `go.sum` committed and up to date
- [ ] No `npm install packagename@latest` in production code — pin versions

---

## 8. Error Handling and Logging

- [ ] Error messages returned to users contain no internal details (stack traces, DB errors, file paths)
- [ ] All errors are logged server-side with enough context to debug (request ID, user ID, timestamp)
- [ ] 🔴 PII is not logged in plaintext: no email, name, phone, SSN, payment card in log lines
- [ ] Log levels are appropriate: debug/info for normal flow, error for actionable failures
- [ ] Unhandled exceptions do not crash the process silently — caught and logged
- [ ] HTTP 500 responses are generic to the client; specific to the log

---

## 9. Infrastructure and Deployment

- [ ] IAM roles follow least privilege: the service role has only the permissions it needs
- [ ] New S3 buckets / storage are private by default — no public-read without explicit intent
- [ ] Security groups / firewall rules allow only required ports and sources
- [ ] All infrastructure changes are in Terraform / IaC — no manual console changes
- [ ] Secrets are in the vault — not in environment variables in the deployment config
- [ ] New services have health checks and structured logging configured

---

## 10. Data Handling

- [ ] PII fields are identified in the code (comments or type annotations)
- [ ] PII is not returned in API responses when not required by the consumer
- [ ] Database migrations that add columns storing PII are noted in EL-001 database_changes
- [ ] Test data does not contain real customer data
- [ ] Data exports / reports are scoped to the authenticated user's accessible data

---

## Pre-PR Final Check

Before opening the PR:

```
git diff --stat HEAD  # Review what's actually changed
git log --oneline -10 # Confirm the artifact ID is in the commit message
```

Commit message format:
```
feat(PM-003-feature-name): brief description [EL-001-feature-name-v1]
```

The `[EL-001-...]` reference is what links the PR to the artifact.
AUD checks for this reference — a PR without it generates a guardrail warning.
