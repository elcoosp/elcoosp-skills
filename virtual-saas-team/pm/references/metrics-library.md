# PM Metrics Library

Standard SaaS metrics the PM agent uses when defining success criteria.
Always include: metric name, formula, data source, target benchmark, and
which tool tracks it.

---

## Revenue Metrics

| Metric | Formula | Benchmark | Source |
|--------|---------|-----------|--------|
| MRR | Sum of all active subscription revenue per month | — | Billing system |
| ARR | MRR × 12 | — | Billing system |
| NRR (Net Revenue Retention) | (MRR start + expansion − contraction − churn) / MRR start × 100 | >110% healthy | CRM + billing |
| GRR (Gross Revenue Retention) | (MRR start − contraction − churn) / MRR start × 100 | >85% healthy | CRM + billing |
| ACV | ARR / number of customers | — | CRM |
| LTV | ARPU × gross margin % / churn rate | >3× CAC | CRM + billing |
| CAC | Total sales + marketing spend / new customers acquired | <1/3 LTV | CRM + finance |
| LTV:CAC ratio | LTV / CAC | >3:1 target, >5:1 excellent | Derived |
| Payback period | CAC / (ARPU × gross margin %) | <12 months | Derived |

## Engagement & Product Metrics

| Metric | Formula | Benchmark | Source |
|--------|---------|-----------|--------|
| DAU/MAU ratio | DAU / MAU | >20% healthy | Product analytics |
| Time-to-first-value | Time from signup to first meaningful action | Set per product | Product analytics |
| Feature adoption rate | Users who used feature / total eligible users | >30% for core features | Product analytics |
| D1/D7/D30 retention | Users active on day N / users who signed up | D30 >25% SaaS baseline | Product analytics |
| Session duration | Avg time in product per session | — | Product analytics |
| NPS | % promoters − % detractors | >30 good, >50 excellent | Survey tool |
| CSAT | Avg satisfaction score | >4.0/5.0 | Survey tool |

## Acquisition Metrics

| Metric | Formula | Benchmark | Source |
|--------|---------|-----------|--------|
| MQL | Marketing-qualified leads (meets ICP criteria) | — | CRM |
| SQL | Sales-qualified leads (SDR accepted) | — | CRM |
| MQL→SQL conversion | SQLs / MQLs | 10–25% B2B | CRM |
| SQL→opportunity | Opportunities / SQLs | 50–70% | CRM |
| Win rate | Closed-won / total closed | 20–30% SMB, 15–25% enterprise | CRM |
| Sales cycle length | Avg days from opportunity to close | — | CRM |
| Pipeline coverage | Open pipeline value / revenue target | 3×+ | CRM |

## Customer Health

| Metric | Formula | Notes |
|--------|---------|-------|
| Health score | Weighted composite (usage + support + NPS + billing) | Define weights per product |
| Churn rate (logo) | Churned customers / total customers | <5% annual healthy |
| Churn rate (revenue) | Churned MRR / total MRR | <1% monthly healthy |
| Time-to-onboard | Days from contract sign to first active use | — |
| Support ticket rate | Tickets per customer per month | Declining = good |

## Counter-Metrics (What Not to Break)

Always define counter-metrics alongside primary metrics:
- Optimizing activation → don't break D30 retention (users who activate fast but leave faster)
- Optimizing for upsell → don't erode NPS (users who feel pushed into upgrades leave)
- Optimizing for new features → don't break core feature adoption
