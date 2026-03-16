# Regulatory Matrix — GDPR, CCPA, SOC 2, HIPAA

CPO uses this matrix when conducting architecture security reviews (CPO-003),
writing DPIAs (CPO-001), and assessing compliance gaps in CPO-002. Each
regulation is mapped to the specific controls it requires and the evidence
needed to demonstrate compliance.

---

## GDPR (EU — General Data Protection Regulation)

**Applies to:** Any product that processes personal data of EU residents,
regardless of where the company is based.

**Personal data includes:** Name, email, IP address, device ID, location data,
behavioral data tied to an individual, any data that can identify a person
directly or indirectly.

### Core Requirements Mapped to Controls

| Requirement | Article | Required Control | Evidence |
|-------------|---------|-----------------|---------|
| Lawful basis for processing | Art. 6 | Legal basis documented per data type | DPIA with legal_basis field completed |
| Data minimisation | Art. 5(1)(c) | Only collect data necessary for stated purpose | Data inventory in DPIA shows no excess collection |
| Purpose limitation | Art. 5(1)(b) | Data used only for purpose it was collected for | No re-use of data without new legal basis |
| Storage limitation | Art. 5(1)(e) | Retention periods defined and enforced | Retention policies in SA-001 data architecture |
| Right of access | Art. 15 | User can export their data within 30 days | Data export endpoint implemented and tested |
| Right to erasure | Art. 17 | User can delete their account and all data within 30 days | Deletion endpoint cascades to all services |
| Right to portability | Art. 20 | User can download data in machine-readable format | Export returns JSON or CSV |
| Data breach notification | Art. 33/34 | DPA notified within 72h; users if high risk | Incident response playbook with 72h SLA |
| Privacy by design | Art. 25 | Data minimisation and security built in from design | CPO reviews SA-001 before EL implements |
| DPA agreements | Art. 28 | Signed DPA with every sub-processor | DPA tracker with all third-party vendors |
| International transfers | Art. 44-49 | Adequacy decision or SCCs in place | Transfer impact assessment per destination |

**Sub-processors requiring DPAs (common list):**
AWS, GCP, Azure, Salesforce, HubSpot, Stripe, SendGrid, PostHog, Datadog,
Sentry, PagerDuty, Slack, Notion, GitHub, Vercel.

**Key CPO actions for GDPR:**
- Maintain a Record of Processing Activities (ROPA) updated quarterly
- Review every new third-party integration for sub-processor status
- Run DPIA for any new data collection, new processing purpose, or new third-party share

---

## CCPA / CPRA (California — Consumer Privacy Act)

**Applies to:** For-profit businesses that: (a) have ≥$25M annual gross revenue,
OR (b) buy/sell/receive/share personal data of ≥100,000 consumers/households,
OR (c) derive ≥50% of revenue from selling personal data.

**Key differences from GDPR:**
- Opt-out model (vs GDPR's opt-in for most processing)
- "Do Not Sell or Share My Personal Information" link required
- No DPA requirement for service providers (but written contract required)
- 45-day response window for consumer requests (vs GDPR's 30 days)

### Core Requirements Mapped to Controls

| Requirement | Required Control | Evidence |
|-------------|-----------------|---------|
| Right to know | User can request what data is collected and why | Privacy policy + data inventory |
| Right to delete | User deletion within 45 days, including service providers | Deletion cascade verified in QA |
| Right to opt out of sale/share | "Do Not Sell" mechanism; no selling without opt-in | UI mechanism + upstream data partner controls |
| Right to correct | User can correct inaccurate personal data | Edit profile endpoint |
| Right to limit use of sensitive PI | Opt-out of use of sensitive PI beyond core service | Preference centre |
| Non-discrimination | Cannot deny service for exercising privacy rights | No feature gating based on consent |
| Data security | Reasonable security measures | SOC 2 or equivalent controls |

**Sensitive personal information under CPRA:**
SSN, financial account numbers, health data, genetic data, biometric data,
precise geolocation, race/ethnicity/religion/union membership, private communications.

---

## SOC 2 Type II

**Applies to:** Any B2B SaaS company selling to enterprises. Required by most
enterprise procurement processes. Type I = point-in-time; Type II = 6–12 month
audit period of operational effectiveness.

**Trust Service Criteria (TSC):**
SOC 2 is audited against five criteria. Security (CC) is mandatory. Others
are optional but often included for SaaS.

### Security Criteria — Required Controls (CC series)

| Control Area | Key Requirements | CPO Verification |
|-------------|-----------------|-----------------|
| CC1 — Control environment | Defined security policies; background checks; security training | Policy docs exist; training records |
| CC2 — Communication | Security policies communicated to employees | Security awareness training completion |
| CC3 — Risk assessment | Annual risk assessment; risk register maintained | CPO-002 updated annually |
| CC4 — Monitoring | Continuous monitoring of controls | AUD audit log; security dashboards |
| CC5 — Control activities | Change management; access reviews; vendor management | RM-001 for releases; quarterly access review |
| CC6 — Logical access | MFA required; least privilege enforced; access reviewed quarterly | MFA enabled in permissions.yaml; access log |
| CC7 — System operations | Incident detection and response; vulnerability management | Incident playbook; pen test schedule |
| CC8 — Change management | All changes via version control; testing before production | CI/CD pipeline; QA-001 sign-off |
| CC9 — Risk mitigation | Vendor risk management; business continuity | DPA tracker; DR plan |

### Availability Criteria (A series — commonly included)

| Requirement | Target | Measurement |
|-------------|--------|------------|
| Uptime SLA | ≥99.9% (common enterprise requirement) | Monitoring dashboard |
| RTO (Recovery Time Objective) | ≤4 hours | DR test result |
| RPO (Recovery Point Objective) | ≤1 hour | Backup configuration + test |
| Performance monitoring | P95 latency tracked and alerted | Observability stack |

**SOC 2 audit readiness checklist (CPO owns):**
- [ ] All security policies documented and dated
- [ ] Employee training records current
- [ ] Pen test completed within last 12 months
- [ ] Vulnerability scan results current
- [ ] Access review completed in last 90 days
- [ ] Incident log maintained (even if no incidents)
- [ ] Vendor/sub-processor list current with contracts
- [ ] Change management evidence (PR history, QA sign-offs)
- [ ] Monitoring dashboards and alert configs documented

---

## HIPAA (US — Health Insurance Portability and Accountability Act)

**Applies to:** Covered entities (healthcare providers, health plans, clearinghouses)
AND their business associates (any vendor handling PHI on their behalf).

**Triggers for a SaaS product:** If any customer is a covered entity AND the
product processes, stores, or transmits Protected Health Information (PHI),
a Business Associate Agreement (BAA) is required.

**PHI includes:** Any individually identifiable health information — diagnosis,
treatment, lab results, prescription, health history — when combined with
a person identifier (name, DOB, address, SSN, device ID, etc.).

### Safeguard Requirements

**Administrative safeguards:**
- Designated Security Officer
- Risk analysis and risk management program (annual minimum)
- Workforce training on PHI handling
- Access management procedures (need-to-know)
- Incident response procedures
- Business associate contracts (BAA) with all sub-processors handling PHI

**Physical safeguards:**
- Facility access controls (relevant for on-prem; cloud providers handle for IaaS)
- Workstation use and security policies
- Device controls (encryption at rest on all devices with PHI access)

**Technical safeguards:**
- Access controls: unique user identification; automatic logoff; encryption
- Audit controls: hardware, software, and procedural mechanisms to record PHI access
- Integrity controls: mechanism to authenticate PHI has not been altered
- Transmission security: encrypt PHI in transit (TLS 1.2+ minimum; 1.3 preferred)

**Minimum necessary rule:** Only access, use, or disclose the minimum PHI
necessary to accomplish the purpose. This must be enforced at the data model
level — not just policy.

### HIPAA CPO Actions

- Get a signed BAA from every sub-processor before any PHI flows to them
- Encrypt PHI at rest with AES-256; in transit with TLS 1.3
- Log all PHI access with user identity and timestamp
- Conduct annual risk assessment
- Train all team members who handle PHI
- Establish and test breach notification procedure (60-day notification window)

---

## Regulation Applicability Decision Tree

```
Does the product collect or process data about people?
  NO → No data protection regulation applies
  YES ↓

Are any of those people EU residents?
  YES → GDPR applies

Are any of those people California residents?
  YES → Check CCPA revenue/volume thresholds

Is any customer a covered healthcare entity?
  YES → Check if product touches PHI → If yes, HIPAA applies + BAA required

Does the product sell to enterprise B2B customers?
  YES → SOC 2 audit strongly recommended (often contractually required)
```

Multiple regulations can apply simultaneously. GDPR + SOC 2 is the most
common combination for B2B SaaS with EU customers.

When multiple regulations apply, implement the most stringent requirement
for each control. GDPR's 30-day data subject response vs CCPA's 45-day:
implement 30 days to satisfy both.
