# Billing Integration Patterns — Stripe, Paddle, Chargebee

CPQ uses these patterns when configuring billing integrations and specifying
quote-to-cash workflows. Every billing integration must handle the full
subscription lifecycle, not just the happy-path checkout.

---

## Stripe

### Recommended Object Model

```
Customer (1)
  └── Subscription (1..N)
        └── SubscriptionItem (1..N per Subscription)
              └── Price (references a Product)

Payment Method (1..N per Customer)
Invoice (generated per billing period)
  └── InvoiceItem (1..N per Invoice)
```

**Key principle:** Store Stripe's `customer.id`, `subscription.id`, and
`price.id` in your own database. Never re-derive these from Stripe on every
request — treat them as foreign keys.

### Subscription Lifecycle — Webhook Events to Handle

All of these must be handled idempotently (same event delivered twice = same result):

```
checkout.session.completed      → Create local subscription record; provision access
customer.subscription.created   → Backup event for subscription creation
customer.subscription.updated   → Handle plan changes, quantity changes, status changes
customer.subscription.deleted   → Revoke access; update local record to cancelled
invoice.payment_succeeded       → Extend subscription period; send receipt email
invoice.payment_failed          → Begin dunning sequence; notify user
invoice.finalized               → Record revenue for accounting
payment_method.attached         → Update default payment method in local DB
customer.updated                → Sync billing email/name changes
```

### Dunning (Failed Payment Recovery)

Stripe's Smart Retries handles retry timing. CPQ handles the user communication:

```
Day 0:   Payment fails → email: "Your payment failed. Update your card."
Day 3:   Stripe retries → if fails: email reminder
Day 5:   Stripe retries → if fails: in-app banner
Day 7:   Stripe retries → if fails: email warning ("Access in 3 days")
Day 10:  Grace period ends → downgrade to free / suspend access
Day 30:  Final email before subscription cancellation
```

Configure grace period in CPQ-001. Default: 10 days. Enterprise customers
may warrant longer grace periods (configure per tier).

### Proration

Stripe handles proration calculations automatically. CPQ must configure:

```python
# Upgrade mid-cycle — Stripe prorates automatically
stripe.subscription.modify(
    subscription_id,
    items=[{"id": item_id, "price": new_price_id}],
    proration_behavior="create_prorations"  # Default; creates an invoice item
)

# Downgrade — apply at renewal to avoid charging mid-cycle
stripe.subscription.modify(
    subscription_id,
    items=[{"id": item_id, "price": new_price_id}],
    proration_behavior="none",
    billing_cycle_anchor="unchanged"
)
```

### Stripe Tax (if applicable)

```python
stripe.customer.modify(customer_id, tax={"validate_location": "deferred"})
stripe.subscription.modify(subscription_id, automatic_tax={"enabled": True})
```

Stripe Tax handles EU VAT, US sales tax, and GST automatically based on
customer location. Requires product tax codes to be set in Stripe Dashboard.

---

## Paddle

Paddle acts as Merchant of Record — they handle tax, compliance, and
payment processing globally. Suitable when global tax compliance is a
priority and you want to avoid managing VAT/GST registrations.

### Key Differences from Stripe

| Concern | Stripe | Paddle |
|---------|--------|--------|
| Tax handling | You handle (or use Stripe Tax) | Paddle handles as MoR |
| Payout | You receive full payment; pay taxes separately | Paddle deducts taxes; pays you net |
| Chargebacks | Your responsibility | Paddle absorbs |
| Currency | You choose | Paddle handles conversion |
| Revenue recognition | Full invoice amount | Net amount after Paddle fees/taxes |

### Paddle Webhook Events

```
subscription.created    → Provision access
subscription.updated    → Handle plan/status changes
subscription.canceled   → Revoke access
subscription.payment_succeeded → Extend period
subscription.payment_failed    → Begin dunning
transaction.completed   → Record revenue (note: amount is NET of Paddle fees)
```

### Paddle Passthrough — Linking to Internal Records

Paddle's `passthrough` field carries custom metadata through to webhooks:

```json
{
  "passthrough": "{\"user_id\": \"usr_123\", \"goal_id\": \"G-2026-001\"}"
}
```

Always include your internal `user_id` in passthrough. Paddle's customer ID
is not your customer ID — you need the mapping.

---

## Chargebee

Chargebee is a subscription management layer that sits on top of payment
gateways (Stripe, Braintree, etc.). Best suited for complex billing scenarios:
metered billing, usage-based pricing, multi-currency, or B2B with custom terms.

### When to Choose Chargebee Over Direct Stripe

Use Chargebee when you need:
- Metered / usage-based billing (charge based on API calls, seats used, GB stored)
- Quote-to-cash with CPQ workflow built in
- Revenue recognition (ASC 606) out of the box
- Multi-currency with automatic exchange rate handling
- Complex trial and discount logic across multiple products

### Chargebee Key Concepts

```
Customer
  └── Subscription
        └── Plan (base recurring charge)
        └── Addons (additional recurring items)
        └── Charges (one-time items)

Invoice (generated on billing date or triggered manually)
Credit Note (for refunds and adjustments)
```

### Metered Billing with Chargebee

```python
# Report usage at end of billing period
chargebee.UsageRecord.create(
    subscription_id=sub_id,
    item_price_id="api-calls-usd-monthly",
    usage={
        "quantity": 45231,  # API calls this month
        "usage_date": end_of_month_timestamp
    }
)
```

CPQ must track usage throughout the period and report to Chargebee before
the billing date. Usage not reported before the invoice is generated is not
charged.

---

## Idempotency — Required for All Billing Operations

All billing API calls that create or modify resources must use idempotency keys:

```python
# Stripe
stripe.PaymentIntent.create(
    amount=4999,
    currency="usd",
    idempotency_key=f"create-intent-{order_id}"
)

# Chargebee
chargebee.Subscription.create_with_items(
    customer_id=customer_id,
    idempotency_key=f"sub-create-{user_id}-{plan_id}"
)
```

**Idempotency key format:** `{operation}-{stable-resource-id}`
The key must be deterministic — re-running the same operation with the same
input must produce the same key. Do not use timestamps or random values.

---

## Revenue Reconciliation Checklist (Monthly)

CPQ runs this reconciliation monthly. Discrepancies must be resolved within 48 hours.

```
[ ] Total active subscriptions in billing system == total active subscriptions in CRM
[ ] MRR in billing system == MRR calculated from CRM subscription records
[ ] All invoices marked "paid" in billing have a corresponding payment in bank account
[ ] All failed invoices have a dunning status in CRM
[ ] No subscriptions in "cancelled" state in billing system with "active" in CRM
[ ] No subscriptions in "active" state in billing system with "cancelled" in CRM
[ ] Discount and coupon usage matches approved discount schedule
[ ] No charges above the contracted amount without an amendment signed
```
