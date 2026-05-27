## 10. Involuntary churn: Stripe dunning

Involuntary churn, subscriptions lost because a payment failed, not because the customer chose to leave, accounts for 30 to 50% of total churn for most SaaS businesses. Most of it is recoverable. Unlike building a cancel flow, fixing involuntary churn is mostly a Stripe configuration problem: two toggles, a short email sequence, and correct handling of decline codes. You can close the gap in a day.

### Why it is the cheap win

When a customer cancels voluntarily, you are fighting their decision. When a payment fails, you are fighting a card network error. The customer still wants the product. The friction is mechanical, not emotional.

That asymmetry makes involuntary churn the highest-ROI retention project for any Stripe SaaS under $100K MRR. Every hour you spend here recovers more revenue than an equivalent hour spent A/B testing save offer copy.

### Pre-dunning: prevent failures before they happen

The best dunning is the dunning you never send.

**Stripe Automatic Card Updater.** Visa and Mastercard run a card-update network that Stripe participates in. When a customer gets a new card (re-issue, number rotation, expiry refresh), Stripe receives the updated details automatically, no customer action required. This catches 30 to 50% of what would otherwise be hard declines due to re-issued cards.

Enable it at: Dashboard > Settings > Subscriptions and emails > Automatic card updates. It is off by default on some accounts. Check now.

**Expiry reminder emails.** Stripe can send these automatically. Enable at Dashboard > Settings > Emails > Send emails when customer cards are expiring. Stripe sends at 30 and 7 days before expiry. If you want 15-day too, add it via your own transactional email.

**Pre-billing notice for annual plans.** Send a plain-text email 3 to 5 days before any annual charge. This is not optional for California subscribers (the ARL requires it for charges over a certain threshold), and it dramatically reduces chargebacks. Chargebacks cost $15 to $35 in fees regardless of outcome. One prevented chargeback pays for the engineering time to ship this email.

**Backup payment method at signup.** Optional, but worth adding to your checkout. A SetupIntent in Stripe lets you collect a second card and store it as a backup. When the primary card fails, you can attempt the backup silently before sending any dunning email at all.

### Stripe Smart Retries

Enable at: Dashboard > Settings > Subscriptions and emails > Smart Retries.

Stripe's machine learning model is trained on millions of declines across its network. Smart Retries chooses retry timing per invoice based on signals like card type, issuer behavior, and time of day. It outperforms fixed retry schedules.

When Smart Retries is on: Stripe attempts up to 4 retries over roughly 3 weeks, timed automatically. When Smart Retries is off: retries fall back to your configured fixed schedule, which recovers meaningfully less.

Turn Smart Retries on. There is no good reason to leave it off.

### Decline type determines the right action

Not all declines are equal. Retrying a hard decline is wasted money and can get your merchant account flagged. The decline code in the Stripe charge object tells you what to do.

| Decline type | Examples | Correct action |
|---|---|---|
| Soft decline | `insufficient_funds`, `do_not_honor`, `processing_error`, `card_velocity_exceeded` | Retry up to 4 times. Smart Retries handles this. |
| Hard decline | `lost_card`, `stolen_card`, `do_not_honor` (permanent), `card_not_supported` | Do not retry beyond one attempt. Switch immediately to "update your payment method" CTA. |
| Expired card | `expired_card` | One retry attempt. Then "update your payment method." Card Updater may have already fixed this silently. |
| Authentication required | `authentication_required` (3DS/SCA) | Do not retry at all. Customer must authenticate. Email with payment update link immediately. SCA errors will never self-resolve. |

For hard declines and SCA errors, skip the retry queue and go straight to the update-payment email. Every extra retry attempt on a stolen card is a failed charge that your issuer notices.

### Dunning email cadence

Stripe has built-in dunning emails. Enable and customize them at Dashboard > Settings > Emails > Failed payments. For most Stripe SaaS founders, these built-in emails are sufficient. Customize the copy, the defaults are generic, but you do not need a third-party dunning service to start.

| Day | Message | Goal |
|---|---|---|
| Day 0 | "Your payment did not go through. Here is your payment update link." | Inform, remove friction. Link directly to Customer Portal payment screen, no login required. |
| Day 3 | "Still having trouble? Here is the link again." | Catch the people who missed day 0. |
| Day 7 | "Your account will pause in 3 days if payment is not updated." | Urgency. Be specific about what pausing means for their data. |
| Day 10 | "Final notice: update payment today to keep access." | Last chance before action. |
| Day 14 | Account pauses or cancels. Send reactivation link. | Close the loop. Make reactivating one click. |

Tone rule: write "your payment did not go through," not "you failed to pay." The failure is the card network's, not the customer's. Plain text outperforms HTML for dunning emails. Your goal is to look like a person, not a billing system.

The Customer Portal payment update URL can be pre-seeded with the customer's email so they land directly on the payment form without a login prompt. Generate it with `stripe.billingPortal.sessions.create` with `return_url` set.

### Webhook handling: tracking recovery state

Listen to these two events to know whether an invoice recovered:

```typescript
import Stripe from "stripe";

export async function handleStripeWebhook(event: Stripe.Event) {
  switch (event.type) {
    case "invoice.payment_failed": {
      const invoice = event.data.object as Stripe.Invoice;
      const declineCode =
        invoice.last_finalization_error?.decline_code ?? null;

      const hardDeclineCodes = new Set([
        "lost_card", "stolen_card", "card_not_supported",
      ]);
      const requiresHardStop =
        declineCode !== null &&
        (hardDeclineCodes.has(declineCode) ||
          declineCode === "authentication_required");

      await db.invoiceRecovery.upsert({
        where: { stripeInvoiceId: invoice.id },
        create: {
          stripeInvoiceId: invoice.id,
          customerId: invoice.customer as string,
          subscriptionId: invoice.subscription as string,
          status: "failed",
          attemptCount: invoice.attempt_count,
          declineCode,
          requiresHardStop,
          failedAt: new Date(),
        },
        update: {
          attemptCount: invoice.attempt_count,
          declineCode,
          requiresHardStop,
          lastFailedAt: new Date(),
        },
      });
      break;
    }

    case "invoice.paid": {
      const invoice = event.data.object as Stripe.Invoice;
      await db.invoiceRecovery.updateMany({
        where: { stripeInvoiceId: invoice.id, status: "failed" },
        data: { status: "recovered", recoveredAt: new Date() },
      });
      break;
    }

    case "customer.subscription.updated": {
      const sub = event.data.object as Stripe.Subscription;
      if (sub.status === "active") {
        await db.subscription.update({
          where: { stripeSubscriptionId: sub.id },
          data: { status: "active", pastDueResolvedAt: new Date() },
        });
      }
      break;
    }
  }
}
```

The `requiresHardStop` flag is your branch point: if true, skip Smart Retries and send the payment update email immediately. If false, let Smart Retries handle timing and only escalate if the invoice is still open after 10 days.

### Recovery benchmarks

Use these to diagnose your dunning setup. If you are below the "needs work" column, check the three most common culprits first: Smart Retries off, Automatic Card Updater off, or dunning emails disabled entirely.

| Metric | Needs work | On track | Good |
|---|---|---|---|
| Soft decline recovery rate | below 40% | 40 to 55% | 55 to 70% |
| Hard decline recovery rate | below 15% | 20 to 30% | 35 to 45% |
| Overall payment recovery | below 30% | 40 to 50% | 55 to 65% |

If overall recovery is below 30%, your three-step audit is: (1) confirm Smart Retries is enabled in the Stripe dashboard, (2) confirm Automatic Card Updater is enabled, (3) confirm at least a Day 0 and Day 7 dunning email are going out. Fix those before touching anything else.

If you are above 50% overall recovery but want to push higher, the next lever is the backup payment method at checkout and pre-billing notices for annual plans. Those two changes alone can move the needle another 5 to 10 points on hard declines.
