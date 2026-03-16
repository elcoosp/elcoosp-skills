# DPIA Examples — Annotated Good and Bad

CPO uses these examples when writing CPO-001 (Data Protection Impact Assessment).
The good examples show what completeness looks like. The bad examples show the
failure modes that produce a DPIA that looks done but provides no protection.

---

## When a DPIA Is Required (Checklist)

A DPIA is required before a feature can be scheduled if any of the following:
- [ ] The feature collects a new category of personal data
- [ ] The feature shares personal data with a new third party
- [ ] The feature uses personal data in a new way (re-purposing)
- [ ] The feature processes data at significantly greater scale
- [ ] The feature involves special category data (health, biometric, financial, race/ethnicity, etc.)
- [ ] The feature uses automated decision-making that affects users (scoring, profiling, filtering)
- [ ] The feature processes location data

If none apply, document the reasoning briefly and proceed without a full DPIA.

---

## Good DPIA: Email-Based Onboarding Campaign

This feature sends a sequence of onboarding emails to new users.

```yaml
artifact_id: CPO-001-onboarding-email-campaign-v1
feature_reference: PM-003-onboarding-campaign-v1

data_inventory:
  - category: identity
    data_type: email_address
    source: user_registration
    storage_location: SendGrid contact list + our users table
    retention_period: duration of account + 90 days post-deletion
    third_party_access:
      - vendor: SendGrid
        purpose: email delivery only
        dpa_signed: true
        data_transferred: email_address, first_name, onboarding_step
    encryption_at_rest: AES-256 (our DB); SendGrid handles their storage
    encryption_in_transit: TLS 1.3

  - category: behavioral
    data_type: email_open_and_click_events
    source: SendGrid webhook
    storage_location: our analytics table
    retention_period: 24 months
    third_party_access: none (collected back from SendGrid into our systems)
    encryption_at_rest: AES-256
    encryption_in_transit: TLS 1.3

risk_assessment:
  - description: User receives unwanted emails after unsubscribing
    likelihood: 2
    impact: 3
    risk_score: 6
    mitigation: Unsubscribe mechanism in every email; 48h processing SLA;
                 suppression list checked before every send
    residual_risk: 2

  - description: SendGrid breach exposes user email addresses
    likelihood: 2
    impact: 4
    risk_score: 8
    mitigation: Signed DPA with SendGrid; data minimisation (only email + first name sent);
                 breach notification clause in DPA
    residual_risk: 4

  - description: Email tracking pixels used to build behavioural profile beyond stated purpose
    likelihood: 1
    impact: 3
    risk_score: 3
    mitigation: Open/click tracking limited to onboarding sequence measurement;
                 not fed into any ad targeting or sold to third parties;
                 documented in privacy policy
    residual_risk: 1

legal_basis: contract
  # Users signed up for the product; onboarding emails are necessary to deliver the service

data_subject_rights:
  access_mechanism: User can request their data via support@product.com; 30-day SLA
  deletion_mechanism: Account deletion cascades to SendGrid suppression list;
                       all behavioral data deleted from analytics table within 30 days
  portability_mechanism: Data export endpoint returns all user data as JSON
  objection_mechanism: Unsubscribe link in every email; preference centre at /settings/notifications

approval:
  cpo_approval: true
  human_legal_review_required: false
    # Risk scores all below 15; no special category data; established vendor with DPA
  notes: Standard email marketing. Low residual risk. Proceed.
```

**Why this is a good DPIA:**
- Every data field is inventoried with its source, storage, retention, and third-party access
- Risks are specific (not "data breach" but "SendGrid breach exposes emails")
- Mitigations are concrete and verifiable (not "we take security seriously")
- Legal basis is stated and justified
- Residual risks are re-scored after mitigation — showing the actual remaining risk
- CPO's reasoning for not requiring human legal review is documented

---

## Bad DPIA: Email-Based Onboarding Campaign

The same feature, done badly.

```yaml
artifact_id: CPO-001-onboarding-email-campaign-v1
feature_reference: PM-003-onboarding-campaign-v1

data_inventory:
  - category: identity
    data_type: user data
    source: our database
    storage_location: cloud
    retention_period: as needed
    third_party_access: SendGrid
    encryption_at_rest: yes
    encryption_in_transit: yes

risk_assessment:
  - description: Data breach
    likelihood: 1
    impact: 1
    risk_score: 1
    mitigation: We use encryption
    residual_risk: 1

legal_basis: consent

data_subject_rights:
  access_mechanism: contact support
  deletion_mechanism: account deletion
  portability_mechanism: export
  objection_mechanism: unsubscribe

approval:
  cpo_approval: true
  human_legal_review_required: false
```

**Why this DPIA is useless:**
- "User data" is not a data type. What specifically is stored?
- "Cloud" is not a storage location. Which service, which region?
- "As needed" is not a retention period. Regulators require specifics.
- "SendGrid" lists the vendor but doesn't state the purpose, what data is transferred, or whether a DPA exists.
- "Data breach" is a category, not a risk. What kind of breach? Via what vector?
- Risk score of 1/1 = 1 for a public-facing email system is implausible optimism.
- "We use encryption" is not a mitigation. Encryption of what, at which layer?
- "Consent" as legal basis for a service email is wrong — the legal basis for onboarding emails is contract (necessary to deliver the service the user signed up for). Consent requires a clear affirmative opt-in.
- This DPIA would fail a GDPR audit.

---

## High-Risk DPIA: User Behavioural Scoring

This feature scores users on likelihood to churn and uses the score to trigger
targeted intervention campaigns. This is automated decision-making with
significant effect on users.

```yaml
artifact_id: CPO-001-churn-scoring-v1
feature_reference: PM-003-churn-prediction-v1

data_inventory:
  - category: behavioral
    data_type: product_usage_events
    [... detailed inventory as above ...]

  - category: behavioral
    data_type: churn_probability_score_0_to_1
    source: ML model output (derived from usage events)
    storage_location: our analytics DB
    retention_period: 12 months rolling
    third_party_access: none
    notes: DERIVED DATA — this score is not observed but calculated.
           Under GDPR Art. 22, automated decision-making that significantly
           affects users requires explicit disclosure and the right to
           contest the decision.

risk_assessment:
  - description: Scoring model encodes historical bias (e.g., users from
                 certain regions score higher churn risk due to historical
                 data skew, not actual behaviour)
    likelihood: 3
    impact: 4
    risk_score: 12
    mitigation: Monthly bias audit across demographic segments (CSH + CPO);
                 model retrained if disparate impact detected
    residual_risk: 6

  - description: Aggressive intervention triggered by false-positive high-churn score
                 damages relationship with healthy customer
    likelihood: 3
    impact: 3
    risk_score: 9
    mitigation: Human CSM reviews any account before intervention is triggered
                 (score is input to human decision, not automated action)
    residual_risk: 3

  - description: Churn score used for decisions user was not informed about
                 (e.g., pricing discrimination)
    likelihood: 2
    impact: 5
    risk_score: 10
    mitigation: Churn score used ONLY for proactive support outreach;
                 prohibited from use in pricing, access decisions, or
                 any downstream use not stated in privacy policy
    residual_risk: 3

legal_basis: legitimate_interest
  justification: Customer retention outreach is a legitimate business interest;
                 balanced against user's privacy interest (no third-party sharing;
                 score used only for proactive support; user can opt out of profiling)

automated_decision_making:
  involves_automated_decisions: false
    # Score informs human CSM decision; no automated action taken solely on score
  explanation: The churn score is one input to a human CSM's review process.
               No account action (pricing change, feature restriction, contract terms)
               is taken solely on the basis of the score without human review.

data_subject_rights:
  access_mechanism: User can request their churn score and the factors contributing to it
  deletion_mechanism: Score deleted when account deleted; historical scores purged after 12 months
  portability_mechanism: Score and underlying usage data included in data export
  objection_mechanism: Users can opt out of behavioural profiling via /settings/privacy;
                        opting out disables churn scoring for that account

approval:
  cpo_approval: true
  human_legal_review_required: true
    # Risk score of 12 (bias risk) exceeds threshold; legitimate interest basis
    # requires balancing test documented by legal; automated decision-making
    # disclosure language needs legal review before shipping
```

**Key lessons from this DPIA:**
- Derived data (ML model outputs) is personal data under GDPR and requires its own inventory entry
- Risk score of 12 (≥15 threshold not reached but close) triggers human legal review given the sensitivity of automated scoring
- Mitigation must be specific: "human CSM reviews before action" is verifiable; "we will be careful" is not
- Legal basis for profiling is legitimate interest, which requires a documented balancing test
