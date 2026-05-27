# Save offer matrix (reason x LTV)

The baseline advice is "show a 20–30% discount." That works if all your customers pay the same amount. They don't.

A 25% discount on a $9/mo plan saves the customer $2.25/mo. That is not a meaningful number to anyone. You just trained a customer to cancel annually and collect a coupon for the cost of a coffee. On a $290/mo plan, that same 25% is $72.50/mo. You may not need to spend it at all if a plan switch or a pause would have kept them.

The fix is a two-dimensional save decision: what the customer said (reason) crossed with what they pay (LTV band).

### LTV bands

These are starting points. Adjust the cutoffs to fit your pricing.

| Band | Monthly MRR from this customer |
|------|-------------------------------|
| Low  | under $15/mo                  |
| Mid  | $15–$99/mo                    |
| High | $100–$499/mo                  |
| Top  | $500+/mo                      |

### The matrix

Rows are the reasons from your exit survey (see §4). Columns are LTV bands. Each cell is the recommended save offer. "Fallback" means the customer declined the primary offer and you are down to one last option before confirming cancel.

| Reason | Low (< $15) | Mid ($15–99) | High ($100–499) | Top ($500+) |
|---|---|---|---|---|
| Too expensive | Downgrade to lower tier | 20% off for 3 months, downgrade fallback | 30% off for 3 months, or annual switch (~17% equivalent) | Personal founder email within 24h, custom offer |
| Not using enough | Pause 1–3 months | Pause 1–3 months | Pause 1–3 months | Pause 1–3 months |
| Missing feature | Honest roadmap timeline (or honest no) + downgrade fallback | Honest roadmap + Mid discount fallback | Honest roadmap + High discount fallback | Founder call to scope the gap; roadmap commit or honest no |
| Switching tools | Fair-fight comparison, no discount | Fair-fight comparison, no discount | Personal call + extended trial of the gap feature | Personal call + extended trial of the gap feature |
| Don't need it right now | Pause 1–3 months | Pause 1–3 months | Pause 1–3 months | Pause 1–3 months |
| Too complex / hard to use | Offer a 30-min setup call, no discount | Offer a setup call + CSM follow-up | Dedicated onboarding session | Dedicated onboarding session + founder check-in |

Two rules apply to every cell before you render an offer:

1. If the reason is "not using enough" or "don't need it right now," offer pause. Never a discount. Customers who pause reactivate at 60–80%. Customers who discount-then-churn cost you money twice.
2. If the reason is "switching tools" at Low or Mid, do not offer a discount. They have already decided. A discount delays the inevitable by one billing cycle and trains cancel-for-deals behavior.

### Offer mechanics

**Discount.** Cap at 30% for 2–3 months. Never 50%+. A 50% discount is a signal that your price was wrong, and it invites repeat cancellers to harvest coupons. Always show the dollar amount alongside the percentage: "Save $29/mo for the next 3 months" lands harder than "30% off." The dollar amount makes the offer real.

**Pause.** Offer 1, 2, or 3 months. Auto-reactivation fires automatically at the end of the pause period. Send a heads-up email 7 days before reactivation so the customer is not surprised by a charge. Keep all data intact during the pause. The customer left because of a timing problem, not a product problem. Make it easy to return.

**Plan switch.** Position as right-sizing, not a downgrade. Show two columns: what they keep and what they lose. Be specific. "You keep unlimited seats and the API. You lose the white-label domain and priority support." Customers respect honesty. Vague "some features may be limited" language erodes trust.

**Personal outreach.** Route the top 10–20% of MRR (roughly your Top band and some of High) to a real reply from a founder or senior team member within 24 hours. Not a template. Not a bot. One paragraph acknowledging what they said in the exit survey, and a direct question about what would make staying worth it. This is the highest-leverage intervention you have and it does not cost you anything except time.

### Issuing a discount via Stripe

When a customer accepts a discount offer, create a coupon and apply it to their subscription. Do not use invoice credits or manual adjustments. Coupons are auditable, attach cleanly to subscriptions, and expire automatically.

```typescript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

// Create a coupon: 30% off for 3 months
const coupon = await stripe.coupons.create({
  percent_off: 30,
  duration: 'repeating',
  duration_in_months: 3,
  name: 'Save offer: 30% for 3 months',
  max_redemptions: 1, // one-time use
  metadata: {
    save_reason: 'too_expensive',
    ltv_band: 'high',
    offered_at: new Date().toISOString(),
  },
});

// Apply it to the subscription instead of cancelling
await stripe.subscriptions.update(subscriptionId, {
  coupon: coupon.id,
});
```

The `max_redemptions: 1` plus per-customer tracking (see abuse rules below) prevents the same customer from collecting the same discount twice. Store `coupon.id` and `customer.id` together in your database when you apply it.

For a pause, you have two options: use Stripe's built-in subscription pause (sets `pause_collection` on the subscription) or cancel and create a new subscription on reactivation. Pause collection is simpler and keeps the subscription object intact.

```typescript
// Pause billing for 2 months
const resumeAt = Math.floor(Date.now() / 1000) + 60 * 60 * 24 * 60; // 60 days

await stripe.subscriptions.update(subscriptionId, {
  pause_collection: {
    behavior: 'void',
    resumes_at: resumeAt,
  },
});
```

Use `behavior: 'void'` to void invoices during the pause rather than marking them unpaid. The customer should not see a bill they owe but cannot pay.

### Abuse rules

Track every save offer shown, accepted, and rejected at the customer level. Store this in your own database alongside the Stripe customer ID.

- Never show the same coupon percent twice to the same customer. If they cancelled six months after the last save and want another discount, show a lower offer (or none at all).
- Customers who have cancelled and been saved more than twice are likely deal-hunters. Show them the pause option only. No discounts.
- Flag any customer who cancels within 7 days of a renewal, accepts a discount, and then cancels again within 30 days. That pattern is coupon harvesting. Stop the sequence.
- Keep an `offer_history` record: `{ customer_id, offered_at, reason, offer_type, offer_value, accepted }`. Query it before every save flow to decide what to show.

The matrix is a default. Override it when you have data. If your "too expensive" + Mid customers consistently reject the 20%-for-3-months offer and cancel anyway, the price gap is real and a plan restructure will do more than a better coupon. The save flow surfaces the signal. You have to act on it.
