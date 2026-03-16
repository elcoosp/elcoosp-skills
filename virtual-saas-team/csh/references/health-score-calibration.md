# Health Score Calibration — Benchmarking and Weight Adjustment

CSH maintains customer health scores that are only useful if they predict
real outcomes (retention, expansion, churn). This file guides calibration:
how to set initial weights, how to validate them against actual outcomes,
and how to adjust when they're wrong.

---

## Default Health Score Model

The default model from CSH-001. Weights are starting points — calibrate
against your actual churn data within 90 days of activation.

```
Overall score = weighted average of five dimensions × 100

Dimension weights (default):
  Product usage        35%
  Feature adoption     25%
  Support ticket rate  15%
  NPS / CSAT           15%
  Billing health       10%
```

### Dimension Scoring

**Product usage (35%)**
Measure: DAU or WAU / licensed seats

| Usage rate | Score |
|-----------|-------|
| > 80% | 100 |
| 60–80% | 75 |
| 40–60% | 50 |
| 20–40% | 25 |
| < 20% | 0 |

**Feature adoption (25%)**
Measure: number of core features used at least once in last 30 days

First define "core features" — typically the 3–5 features that are most
correlated with value delivery. This is a product decision; get it from PM-001.

| Core features used | Score |
|-------------------|-------|
| All core features | 100 |
| 3 of 4 | 75 |
| 2 of 4 | 50 |
| 1 of 4 | 25 |
| None | 0 |

**Support ticket rate (15%)**
Measure: tickets submitted in last 30 days

| Tickets/month | Score |
|--------------|-------|
| 0 | 100 |
| 1–2 | 75 |
| 3–5 | 40 |
| > 5 | 0 |

Note: Context matters. High tickets may indicate engagement in early onboarding,
not dissatisfaction. Segment by account age when interpreting.

**NPS / CSAT (15%)**
Measure: most recent survey score

| Score | Health score |
|-------|------------|
| NPS 9–10 / CSAT 5/5 | 100 |
| NPS 7–8 / CSAT 4/5 | 60 |
| NPS 0–6 / CSAT ≤ 3/5 | 0 |
| No response in 90 days | 40 (neutral) |

**Billing health (10%)**
Measure: payment status

| Status | Score |
|--------|-------|
| Current | 100 |
| 1–14 days overdue | 50 |
| 15–29 days overdue | 10 |
| 30+ days or dispute | 0 |

---

## Health Score Bands

| Score | Band | Action |
|-------|------|--------|
| 75–100 | Green — Healthy | Monitor monthly; identify expansion signals |
| 60–74 | Yellow — At risk | Proactive outreach within 5 business days; CSH-004 plan |
| 0–59 | Red — Critical | Escalate to human CSM lead within 48 hours |

---

## Calibration: Validating Weights Against Outcomes

After 90 days of operation, validate whether the health score actually
predicts churn. If it doesn't, the weights are wrong.

**Step 1: Build a calibration dataset**

For every account that churned in the last 90 days:
- What was their health score 30 days before they churned?
- What was their health score 60 days before they churned?
- What was their health score 90 days before they churned?

For every account that renewed in the last 90 days:
- Same 30/60/90 day lookback scores.

**Step 2: Check discrimination — do scores separate churns from renewals?**

A well-calibrated health score should show:
- Churned accounts averaging below 60 at the 30-day mark
- Renewed accounts averaging above 70 at the 30-day mark
- A clear separation between the two groups

If churned and renewed accounts have similar scores, the model is not
discriminating. The weights need adjustment.

**Step 3: Identify which dimensions predict churn**

For each dimension, compare average score for churned vs retained accounts.
The dimensions with the largest gap are the most predictive — increase their weight.
The dimensions with a small gap are less predictive — decrease their weight.

Example finding:
```
Feature adoption — churned avg: 22, retained avg: 74 → gap: 52 → increase weight
Support tickets — churned avg: 45, retained avg: 55 → gap: 10 → decrease weight
```

**Step 4: Adjust weights and re-score**

Adjust weights proportionally. After adjustment, re-score all current accounts
and verify the distribution makes intuitive sense.

If you can't explain why a specific account scored the way it did to a
colleague in 30 seconds, the model is too complex or the weights are wrong.

---

## Override Signals (Automatic Score Adjustments)

Some events should immediately change a health score regardless of the
underlying dimension scores:

| Event | Score adjustment |
|-------|----------------|
| Executive sponsor left the company | -25 points immediately |
| Champion left the company | -20 points immediately |
| Negative press about the customer (layoffs, acquisition) | Flag for human review |
| Customer publicly complains about product | -15 points |
| Customer wins an award or major new contract | +10 points |
| Customer successfully completes onboarding | +15 points |
| Customer expands usage to new team | +20 points |

Override adjustments are additive to the dimension score, not multiplicative.
Document every override in CSH-001.

---

## Segments That Need Different Models

The default model assumes a mid-market B2B account with regular product usage.
Accounts that differ significantly may need adjusted models:

**Seasonal businesses:** Usage drops during off-season even for healthy accounts.
Adjust the usage dimension to compare to the same period last year, not the
trailing 30 days.

**Enterprise accounts with long implementation phases:** Health score during
implementation (before go-live) should be based on implementation milestones,
not product usage (they're not using it yet because they're still setting it up).

**Very small accounts (1–2 users):** Support ticket rate of "3 tickets" means
something very different for a 1-user account vs a 50-user account. Normalise
by licensed seat count.

**Newly onboarded accounts (< 90 days):** High support ticket rate during
onboarding is expected and healthy. Segment by account age when applying
the support ticket dimension.
