# Revenue Recognition — ASC 606 Guidelines for SaaS

CPQ applies these guidelines when structuring contracts and reporting revenue.
ASC 606 (IFRS 15 internationally) is the standard for recognising revenue from
customer contracts. Getting this wrong has accounting and audit consequences.

This is a practical reference, not legal or accounting advice. Any complex
contracts or edge cases require review by a qualified accountant.

---

## The Five-Step Model

ASC 606 requires revenue to be recognised following five steps:

**Step 1 — Identify the contract**
A contract exists when: both parties approve it, each party's rights are
identifiable, payment terms are identifiable, the contract has commercial
substance, and collection is probable.

For SaaS: a signed order form or accepted terms of service + payment method
on file constitutes a contract. A free trial with no commitment does not.

**Step 2 — Identify the performance obligations**
A performance obligation is a promise to deliver a distinct good or service.
For SaaS, common performance obligations include:
- Ongoing software access (SaaS subscription — recognised ratably over time)
- Implementation services (recognised as delivered, or at completion)
- Training (recognised at delivery)
- Professional services / consulting (recognised as delivered)

**Step 3 — Determine the transaction price**
The transaction price is the amount you expect to receive. Includes:
- Fixed fees
- Variable fees (usage, overages) — estimated using expected value
- Discounts already agreed
Excludes: amounts collected for third parties (taxes)

**Step 4 — Allocate the transaction price**
If a contract has multiple performance obligations, allocate the price based
on standalone selling price (SSP) of each component.

Example: $12,000 annual contract = $10,000 for SaaS + $2,000 for implementation
- SaaS SSP: $10,000 → recognised monthly ($833/month over 12 months)
- Implementation SSP: $2,000 → recognised at go-live (point in time)

**Step 5 — Recognise revenue when (or as) performance obligation is satisfied**
- Over time: SaaS subscription access — recognised ratably (monthly)
- Point in time: implementation complete, training delivered

---

## SaaS Revenue Recognition — Practical Rules

### Annual Subscription Paid Upfront

```
Contract: $12,000 annual subscription, paid on January 1
Deferred revenue (January 1): $12,000
Revenue recognised per month: $1,000
Deferred revenue balance (March 31): $9,000

Journal entries:
  Jan 1:  Dr Cash $12,000 / Cr Deferred Revenue $12,000
  Jan 31: Dr Deferred Revenue $1,000 / Cr Revenue $1,000
  (repeat monthly)
```

Cash ≠ Revenue. The $12,000 is not revenue until it is earned (i.e., service delivered).

### Monthly Subscription

```
Contract: $1,000/month, billed monthly
Revenue recognised: $1,000 per month as the month is completed
Deferred revenue: zero (no advance payment)
```

### Mid-Period Upgrade

```
Customer upgrades from $1,000/month to $1,500/month on March 15
March revenue: ($1,000 × 14/31) + ($1,500 × 17/31) = $452 + $823 = $1,275
April onwards: $1,500/month
```

### Discount — Commit to Annual vs Month-to-Month

```
Monthly rate: $1,200/month
Annual rate:  $10,000/year (17% discount)
SSP of annual: $14,400 (what 12 months would cost at monthly rate)
Discount:     $4,400

Under ASC 606, the discount should be allocated proportionally across
performance obligations, not treated as a lump-sum reduction.
For simple SaaS with single performance obligation: recognise $10,000/12 = $833/month.
```

---

## Common SaaS Contracts and Revenue Timing

| Contract type | When revenue is recognised |
|--------------|--------------------------|
| Monthly SaaS | End of each month (as service is delivered) |
| Annual SaaS, paid upfront | Ratably over 12 months |
| Annual SaaS, paid monthly | Monthly as each payment is earned |
| Multi-year SaaS | Ratably over contract term |
| Implementation fee | At go-live (or % completion if using percentage-of-completion method) |
| Training | At time of training delivery |
| Usage-based (metered) | As usage occurs (or when measurable) |
| Professional services | As delivered (time-and-materials) or at completion (fixed-fee) |

---

## Key Metrics CPQ Tracks for Revenue Reporting

### MRR Movements (Monthly)

```
Opening MRR
+ New MRR         (new customers × their MRR)
+ Expansion MRR   (upsells and upgrades to existing customers)
- Contraction MRR (downgrades, partial cancellations)
- Churned MRR     (full cancellations)
= Closing MRR

Net New MRR = New + Expansion - Contraction - Churn
```

### Deferred Revenue (Balance Sheet)

Deferred revenue = cash received for services not yet delivered.
CPQ tracks this as: sum of all upfront payments × (remaining contract months / total contract months).

### ARR Calculation

ARR = MRR × 12. Simple.
Do NOT include one-time fees (implementation, training) in ARR.
Do NOT include MRR from customers past their contract end date.

### Bookings vs Revenue

Bookings = total contract value at signing (the number salespeople celebrate).
Revenue = what is recognised under ASC 606 (the number finance cares about).

A $120,000 three-year contract = $120,000 booking but only $40,000/year in recognised revenue.
CPQ reports both, clearly labelled, to avoid confusion.
