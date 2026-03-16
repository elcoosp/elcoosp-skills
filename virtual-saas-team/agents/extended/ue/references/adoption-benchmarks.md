# Adoption Benchmarks — Onboarding Completion, Feature Adoption, Engagement

UE uses these benchmarks when setting targets in UE-001 and UE-002 and
evaluating educational intervention effectiveness in UE-004. These are
industry medians — your numbers depend heavily on product complexity, ICP,
and onboarding investment.

Use these to calibrate whether your results are normal, strong, or weak —
not as hard targets.

---

## Onboarding Benchmarks

### Time-to-First-Value

| Product complexity | Median time to first value | Notes |
|-------------------|--------------------------|-------|
| Simple (single-purpose tool) | < 5 minutes | e.g., Calendly, Loom |
| Medium (multi-feature SaaS) | 20–60 minutes | e.g., HubSpot, Notion |
| Complex (enterprise platform) | 1–5 days | e.g., Salesforce, Workday |

If your time-to-first-value exceeds the benchmark for your complexity tier,
the onboarding flow is the first place to investigate.

### Onboarding Checklist Completion

| Completion rate | Assessment |
|----------------|-----------|
| > 70% | Excellent — checklist is well-designed and valuable |
| 50–70% | Acceptable — review drop-off points |
| 30–50% | Poor — checklist may be too long or items too complex |
| < 30% | Critical — redesign the checklist or the onboarding flow |

The most common failure: a checklist with too many items. Users who see 8+
items feel overwhelmed before they start. Cap at 5.

### D1, D7, D30 Retention After Onboarding

These are retention rates for users who completed onboarding (reached first
value event) vs those who didn't.

| Cohort | D7 retention (completed onboarding) | D7 retention (did not complete) |
|--------|------------------------------------|---------------------------------|
| Self-serve B2B SaaS | 55–70% | 15–30% |
| Consumer SaaS | 35–55% | 8–20% |

The gap between these two numbers is the value of onboarding improvement.
If users who complete onboarding retain at 65% vs 20% for non-completers,
getting 10% more users to complete onboarding has a measurable retention impact.

---

## Feature Adoption Benchmarks

### Core Feature Adoption (% of users who use a core feature within 30 days)

| Feature type | Healthy adoption rate |
|-------------|----------------------|
| Features required for basic value delivery | > 80% |
| Features that extend or enhance core value | 30–60% |
| Advanced / power features | 15–30% |
| Integration features | 20–40% (depends on integration availability) |

If a feature required for basic value delivery has adoption below 50%, either:
- The feature is hard to find or use (UX/UE problem)
- The feature doesn't actually deliver the value it was designed to (product problem)
- Users are accomplishing the same goal another way (possible — investigate before assuming it's a problem)

### Feature Discovery Rate

| % of users who find the feature within 14 days | Assessment |
|-------------------------------------------------|-----------|
| > 60% | Strong discoverability |
| 30–60% | Acceptable |
| < 30% | Feature is likely buried — consider a feature announcement or onboarding prompt |

---

## In-App Message Performance Benchmarks

### Modal (One-Time, Onboarding)

| Metric | Low | Median | High |
|--------|-----|--------|------|
| CTA click rate | 15% | 30% | 55% |
| Dismiss rate | 20% | 35% | 55% |
| Completion of prompted action (within 24h) | 10% | 25% | 45% |

If your CTA click rate is below 15%: the copy is likely too generic or the
timing is wrong (showing the modal too early or too late in the session).

If your dismiss rate is above 55%: the modal is appearing at the wrong
moment or the content isn't relevant to what the user is trying to do.

### Feature Announcement Banner

| Metric | Low | Median | High |
|--------|-----|--------|------|
| Click-to-try rate | 5% | 12% | 25% |
| Permanent dismiss rate | 40% | 60% | 80% |
| Feature adoption at 7 days (banner cohort) | 8% | 18% | 35% |

Compare the feature adoption rate of the banner cohort to the non-banner
cohort. A banner that doesn't improve adoption is just noise.

### Onboarding Email (for comparison with in-product)

| Metric | Low | Median | High |
|--------|-----|--------|------|
| Open rate | 30% | 50% | 70% |
| Click rate | 5% | 15% | 30% |
| Completion of prompted action (within 48h) | 3% | 8% | 20% |

In-product prompts consistently outperform email for onboarding. If you
have to choose where to invest: in-product first, email second.

---

## Measuring Intervention Lift

When UE runs an adoption campaign (UE-002), always measure lift vs a control group:

```
Lift = (Adoption rate in intervention group) - (Adoption rate in control group)
       -----------------------------------------------------------------------
                         Adoption rate in control group

Example:
  Control group:      18% adopted the feature within 7 days
  Intervention group: 27% adopted the feature within 7 days
  
  Lift = (27 - 18) / 18 = 50% relative lift

What this means: the intervention increased adoption by 50% relative to doing nothing.
```

**Minimum meaningful lift to justify ongoing investment:** > 20% relative lift.

If lift is < 20%: the intervention may not be worth the maintenance cost.
Investigate whether: the feature has a discovery problem (try different placement),
or an adoption problem (try different messaging), or neither (the feature may not
be relevant to this segment).

---

## Benchmarks by Product Stage

| Stage | Expected activation rate | Expected D30 retention |
|-------|--------------------------|----------------------|
| Pre-launch / alpha | N/A | Qualitative only |
| Private beta | 60–80% (invited, motivated users) | 40–60% |
| Public beta | 40–60% | 25–45% |
| General availability (first 6 months) | 30–50% | 20–40% |
| Mature product (1+ years) | 25–45% | 25–50% |

Activation rate = % of signups who reach the first value event.
D30 retention = % of activated users who are still active 30 days later.

Mature products often have lower activation (broader, less targeted audience)
but higher D30 retention (product-market fit more established, ICP better defined).
