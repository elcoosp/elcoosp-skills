# Channel Benchmarks — CPL, CAC, and Conversion by Channel and Segment

MK uses these benchmarks when setting targets in MK-001 (GTM Plan) and
evaluating campaign performance in MK-004 (Weekly Marketing Report).
Benchmarks are industry medians for B2B SaaS — your numbers will vary.
Use these to sanity-check targets, not as hard commitments.

Last updated: March 2026. Review quarterly.

---

## How to Use Benchmarks

A benchmark is a reference point, not a target. If your CAC is 3× the
benchmark for your channel, investigate why — don't assume the benchmark
is wrong. Common legitimate reasons to beat or miss a benchmark: your ICP
is unusually well or poorly defined, your product has unusually high or low
ACV, your sales cycle is shorter or longer than average.

Always use your own historical data when you have it. Benchmarks are for
when you don't yet have data.

---

## CPL (Cost Per Lead) by Channel — B2B SaaS

| Channel | Low | Median | High | Best for |
|---------|-----|--------|------|---------|
| LinkedIn Ads | $40 | $75 | $200+ | Enterprise, professional roles, niche verticals |
| Google Search (branded) | $5 | $15 | $40 | Bottom-funnel capture of existing demand |
| Google Search (non-branded) | $25 | $60 | $150 | Mid-funnel intent |
| Content/SEO (organic) | $8 | $20 | $50 | Long-term, compounding, lower volume |
| Email (outbound) | $15 | $35 | $80 | Targeted accounts; volume-dependent |
| Webinars/Events | $30 | $80 | $200 | Thought leadership; long sales cycle |
| Partner/Affiliate | $20 | $45 | $100 | Trusted referrals; higher close rate |
| Product-Led (PLG) | $2 | $8 | $25 | Self-serve; high volume; lower ACV often |
| Cold social (Twitter/X) | $30 | $70 | $180 | Developer/technical audiences |

**Notes:**
- LinkedIn CPL is high but lead quality is often higher — evaluate CPL alongside MQL rate
- SEO CPL includes content production cost amortised over traffic lifetime
- PLG CPL is deceptively low — include product cost (infrastructure, support) in true CAC
- Outbound email CPL rises sharply as list quality degrades

---

## MQL → SQL Conversion Rate by Source

| Lead Source | Typical MQL → SQL | Why |
|-------------|------------------|-----|
| Inbound (website form, demo request) | 20–35% | High intent — they came to you |
| Content/SEO | 10–20% | Mid-intent — researching, not buying yet |
| LinkedIn Ads | 8–18% | Intent varies by ad type; demo request ads higher |
| Outbound email | 5–15% | Cold; lower intent by definition |
| Webinar attendees | 15–30% | Self-selected interest; longer nurture needed |
| Partner referrals | 25–40% | Warm intro; higher trust |
| PLG (free → paid signal) | 30–50% | Strong behavioural signal |
| Trade show / event | 10–20% | Mixed intent; depends heavily on qualification |

---

## SQL → Opportunity → Close Rate by Segment

| Segment | SQL → Opportunity | Opportunity → Close | Avg Sales Cycle |
|---------|-----------------|-------------------|-----------------|
| SMB (< 100 employees) | 60–75% | 25–40% | 14–30 days |
| Mid-market (100–1000 employees) | 50–65% | 20–35% | 30–90 days |
| Enterprise (1000+ employees) | 40–55% | 15–25% | 90–180 days |
| PLG-led (product qualified) | 65–80% | 35–55% | 7–21 days |

---

## CAC by Segment and Channel Mix

Blended CAC (all sales + marketing spend / new customers) across B2B SaaS:

| Company ARR | Median blended CAC | CAC:LTV healthy range |
|-------------|-------------------|----------------------|
| < $1M ARR | $200 – $800 | LTV > 3× CAC |
| $1M – $10M ARR | $600 – $3,000 | LTV > 3× CAC |
| $10M – $50M ARR | $2,000 – $10,000 | LTV > 3× CAC |
| Enterprise-focused | $8,000 – $50,000+ | LTV > 3× CAC |

**CAC payback period benchmarks:**
- Efficient growth: < 12 months
- Acceptable: 12–18 months
- Concerning: 18–24 months
- Problem: > 24 months (either fix the CAC or raise ACV)

---

## Email Marketing Benchmarks — B2B SaaS

| Metric | Low | Median | High |
|--------|-----|--------|------|
| Open rate (transactional) | 35% | 50% | 70% |
| Open rate (marketing/nurture) | 18% | 28% | 45% |
| Click-to-open rate (marketing) | 8% | 15% | 25% |
| Unsubscribe rate (per send) | 0.05% | 0.15% | 0.5% |
| Spam complaint rate | 0.01% | 0.03% | 0.08% |

**Alert thresholds:**
- Unsubscribe rate > 0.5% on a campaign: review audience targeting and message relevance
- Spam complaint rate > 0.1%: review list hygiene; SendGrid/AWS SES may throttle your domain
- Open rate < 15% on a nurture sequence: review subject lines and send timing

---

## Paid Channel Budget Allocation — Early-Stage B2B SaaS

Suggested starting allocation when total paid budget is < $20k/month:

| Channel | Allocation | Rationale |
|---------|-----------|-----------|
| Google Search (branded) | 15% | Protect your brand; high intent; low waste |
| Google Search (non-branded, bottom funnel) | 25% | Capture existing demand in category |
| LinkedIn Ads (one campaign) | 35% | ICP targeting; test before scaling |
| Content promotion (amplify SEO) | 15% | Compound returns on organic investment |
| Retargeting | 10% | Low CPL for warm audiences |

**Don't spread thin.** At < $20k/month, running 6 channels simultaneously means
none have enough budget to reach statistical significance. Run 2–3 channels,
prove them, then expand.

---

## Attribution Model Comparison

| Model | What it attributes to | Best used for |
|-------|----------------------|--------------|
| Last touch | Final touchpoint before conversion | Simple; overstates bottom-funnel |
| First touch | First touchpoint | Evaluating awareness/discovery channels |
| Linear | Equal credit to all touchpoints | Understanding the full journey |
| Time decay | More credit to recent touchpoints | Short sales cycles |
| Data-driven | ML-based; proportional to conversion probability | Large data volumes (100k+ conversions) |

**Recommendation for early-stage:** Use linear attribution for strategic decisions;
track first touch to understand what creates awareness; track last touch to
understand what closes. Never make budget decisions on last-touch alone.
