# Test Case Templates by Feature Type

QA uses these templates when writing QA-001 test plans. Each template covers
the test categories that matter most for that feature type. Adapt — don't copy
blindly. The failure modes in your feature may differ from the template.

---

## Template 1: User Authentication

Core risk: unauthorised access, credential exposure, session management bugs.

```
TC-AUTH-001: Successful login with valid credentials
  Preconditions: User account exists and is active
  Steps: Submit valid email + password
  Expected: JWT token returned; user session created; redirect to dashboard
  Priority: P0

TC-AUTH-002: Login with invalid password
  Preconditions: User account exists
  Steps: Submit valid email + wrong password
  Expected: HTTP 401; error message does not confirm whether email exists;
            no token returned
  Priority: P0

TC-AUTH-003: Login with non-existent email
  Preconditions: Email not in system
  Steps: Submit unknown email + any password
  Expected: HTTP 401; identical error message to TC-AUTH-002 (no user enumeration)
  Priority: P0

TC-AUTH-004: Brute force protection
  Preconditions: Fresh account
  Steps: Submit wrong password 10 times consecutively
  Expected: Account locked or CAPTCHA triggered after configured threshold
  Priority: P1

TC-AUTH-005: Session expiry
  Preconditions: Valid session token
  Steps: Wait for token expiry (or manipulate exp claim); attempt authenticated request
  Expected: HTTP 401; token rejected; user redirected to login
  Priority: P0

TC-AUTH-006: Concurrent session behaviour
  Preconditions: User logged in on Device A
  Steps: Login on Device B
  Expected: Behaviour matches configured policy (new session valid; old session
            valid or invalidated, per design)
  Priority: P1

TC-AUTH-007: Password reset token single-use
  Preconditions: Password reset requested; reset token issued
  Steps: Use token to reset password; attempt to use same token again
  Expected: Second use returns error (token expired or already used)
  Priority: P0

TC-AUTH-008: JWT signature verification
  Preconditions: Valid JWT from login
  Steps: Modify JWT payload (e.g., change role to admin); submit modified token
  Expected: HTTP 401; modified token rejected
  Priority: P0
```

---

## Template 2: Multi-Tenant Data Isolation

Core risk: tenant A accessing tenant B's data — either directly or through
a missing WHERE clause.

```
TC-TENANT-001: User can only read own tenant's resources
  Preconditions: Two tenants (Tenant A, Tenant B) with separate users and data
  Steps: Authenticate as Tenant A user; request Tenant B's resource ID directly
  Expected: HTTP 403 or HTTP 404 (not HTTP 200 with Tenant B's data)
  Priority: P0

TC-TENANT-002: User cannot modify another tenant's resources
  Preconditions: Same as TC-TENANT-001
  Steps: PUT/PATCH/DELETE Tenant B's resource as Tenant A
  Expected: HTTP 403 or HTTP 404
  Priority: P0

TC-TENANT-003: List endpoints return only own tenant's data
  Preconditions: Both tenants have records
  Steps: GET /resources as Tenant A
  Expected: Response contains only Tenant A's records; Tenant B's records absent
  Priority: P0

TC-TENANT-004: Search/filter cannot expose cross-tenant records
  Preconditions: Both tenants have records matching same search term
  Steps: Search for term that exists in both tenants as Tenant A user
  Expected: Only Tenant A's matching records returned
  Priority: P0

TC-TENANT-005: Webhook/event callbacks scoped to correct tenant
  Preconditions: Both tenants subscribed to same event type
  Steps: Trigger event in Tenant A
  Expected: Tenant A's webhook fires; Tenant B's webhook does NOT fire
  Priority: P0
```

---

## Template 3: Payment and Billing

Core risk: incorrect charges, failed payments without notification, revenue leakage.

```
TC-BILLING-001: Successful subscription creation
  Preconditions: Test card available (use Stripe test cards)
  Steps: Complete checkout with valid test card
  Expected: Stripe subscription created; local subscription record created;
            confirmation email sent; user gains access to paid features
  Priority: P0

TC-BILLING-002: Failed payment — declined card
  Preconditions: Stripe test card for decline (4000000000000002)
  Steps: Attempt checkout with declining card
  Expected: Clear error message; no subscription created; no charge;
            user remains on free tier
  Priority: P0

TC-BILLING-003: Subscription renewal
  Preconditions: Active subscription
  Steps: Advance test clock to renewal date
  Expected: Renewal charge processed; subscription period extended;
            no service interruption
  Priority: P0

TC-BILLING-004: Payment failure at renewal — grace period
  Preconditions: Active subscription; card set to fail at renewal
  Steps: Advance test clock; payment fails
  Expected: Grace period begins; user notified by email; access maintained;
            Stripe retries per retry schedule
  Priority: P0

TC-BILLING-005: Subscription cancellation
  Preconditions: Active subscription
  Steps: Cancel subscription
  Expected: Cancellation recorded in Stripe; access maintained until period end;
            no further charges; confirmation email sent
  Priority: P0

TC-BILLING-006: Webhook idempotency
  Preconditions: Stripe webhook configured
  Steps: Send same Stripe webhook event twice (simulate duplicate delivery)
  Expected: Event processed once; no duplicate records created
  Priority: P0

TC-BILLING-007: Upgrade mid-cycle — proration
  Preconditions: Active monthly subscription on Tier 1
  Steps: Upgrade to Tier 2 mid-cycle
  Expected: Prorated charge for remaining days on old tier + new tier;
            immediate access to new tier features
  Priority: P1
```

---

## Template 4: File Upload

Core risk: malicious file execution, storage abuse, data exposure.

```
TC-UPLOAD-001: Valid file upload — happy path
  Preconditions: Authenticated user; valid file type and size
  Steps: Upload file within permitted type and size limits
  Expected: File stored; reference returned; file accessible via returned URL
  Priority: P1

TC-UPLOAD-002: File type restriction enforcement
  Preconditions: Endpoint accepts only images (jpeg, png, gif)
  Steps: Upload a PHP file renamed as image.jpg; upload an actual .exe
  Expected: Both rejected with HTTP 400; MIME type checked by content inspection,
            not by file extension
  Priority: P0

TC-UPLOAD-003: File size limit enforcement
  Preconditions: 10MB limit configured
  Steps: Upload 11MB file
  Expected: HTTP 413 or rejected before upload completes; no partial file stored
  Priority: P1

TC-UPLOAD-004: Malicious file content (XSS in SVG)
  Preconditions: SVG uploads permitted
  Steps: Upload SVG containing embedded JavaScript
  Expected: SVG sanitised before storage; script tag removed or file rejected
  Priority: P0

TC-UPLOAD-005: Path traversal attempt
  Preconditions: File upload endpoint
  Steps: Submit filename as "../../etc/passwd" or "../../../secret.env"
  Expected: Filename sanitised; file stored with generated safe name
  Priority: P0

TC-UPLOAD-006: Unauthenticated upload attempt
  Preconditions: No session token
  Steps: POST to upload endpoint without auth
  Expected: HTTP 401; no file stored
  Priority: P0
```

---

## Template 5: API Rate Limiting

Core risk: abuse, denial of service, cost runaway from API calls.

```
TC-RATE-001: Rate limit enforced per user
  Preconditions: Rate limit of N requests per minute configured
  Steps: Send N+1 requests within 1 minute from same authenticated user
  Expected: First N requests succeed; N+1th returns HTTP 429 with
            Retry-After header
  Priority: P1

TC-RATE-002: Rate limit resets after window
  Preconditions: Rate limited user from TC-RATE-001
  Steps: Wait for window to expire; send one request
  Expected: Request succeeds (limit reset)
  Priority: P1

TC-RATE-003: Rate limit is per-user, not global
  Preconditions: User A at rate limit
  Steps: Send request from User B
  Expected: User B's request succeeds; User A's rate limit does not affect User B
  Priority: P1

TC-RATE-004: Rate limit headers present on all responses
  Steps: Any authenticated API request
  Expected: Response includes X-RateLimit-Limit, X-RateLimit-Remaining,
            X-RateLimit-Reset headers
  Priority: P2
```

---

## Priority Definitions (Reminder)

| Priority | Meaning | If found before release |
|----------|---------|------------------------|
| P0 | System unusable, data loss, security vulnerability | Block release; immediate human notification |
| P1 | Core feature broken for most users | Block release |
| P2 | Feature degraded; workaround exists | Ship with documented limitation + fix-in-sprint |
| P3 | Minor cosmetic or edge case | Ship with ticket created |
