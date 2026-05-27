# Cancel flow design

A cancel flow has five screens. Each has a job. Get them in the right order and you cut voluntary churn by 20–35% without dark patterns or discount spam.

---

### The flow shape

```
1. Trigger         → customer clicks cancel
2. Reason capture  → branching follow-up, NOT a single dropdown (see §4)
3. Save offer      → LTV-aware, pause-first (see §5)
4. Confirmation    → cancel-at-period-end, not immediate
5. Post-cancel     → final win-back window + handoff
```

Every screen must show a visible "continue cancelling" path. Never make the user hunt for it.

---

### Easy-cancel is the foundation

The save offer layer only works if the cancel path itself is legally sound. Two rules that are now baseline for any Stripe SaaS in the US:

**FTC click-to-cancel (2024 rule):** cancellation must be at least as easy to initiate as signup was. If signup is a button on your pricing page, cancel must be a button too. Not a support ticket. Not an email.

**California ARL (Business & Professions Code 17600):** cancel must be available in the same medium where signup occurred. Online signup means online cancel. No "call us to cancel."

Concrete implementation rule: the cancel link must be reachable in 2 clicks or fewer from any page that shows a signup CTA. A settings page behind a nav item counts as 2 clicks. A modal behind a gear icon is 2 clicks. Anything deeper is a liability.

This is not anti-conversion. It is the contract that makes your save offer credible. A customer who can see the exit is far more willing to pause and think.

---

### Screen 1: Trigger

The cancel button lives in billing settings (typically a Stripe Customer Portal link or your own billing page). Do not hide it. Do not add friction before the customer reaches the flow . the flow itself is where you engage them.

When they click, open a focused modal or a dedicated route. Do not pop a generic "are you sure?" dialog. That is wasted real estate.

What the trigger screen does: acknowledges the intent without panic, and moves them into reason capture. One sentence is enough.

---

### Screen 2: Reason capture

Covered in detail in §4. The short version: replace the standard single-select dropdown with a short list of radio buttons (5–7 options, no "Other"), and branch to a follow-up question based on the selection. The real reason almost always lives one level deeper than the first answer.

---

### Screen 3: Save offer

Covered in detail in §5. The critical rule here is pause-first bias.

If the reason is any variant of "not using it enough," "too busy," "taking a break," or "temporary," surface a pause offer before any discount. Pausers reactivate at 60–80%. Customers who take a discount churn at the next renewal because the underlying usage problem was never addressed. Discount-first is how you buy 30 days of delay and call it a win.

Auto-select nothing. Show the offer clearly, let the customer accept or decline, and keep "continue cancelling" visible without scrolling.

---

### Screen 4: Confirmation

If the customer declines the offer (or you had no offer to make), confirm the cancellation. Do not make them confirm twice.

The correct Stripe call is:

```typescript
await stripe.subscriptions.update(subscriptionId, {
  cancel_at_period_end: true,
});
```

Not `stripe.subscriptions.cancel()`. Immediate cancellation revokes access right now and forfeits the remaining paid period. That generates chargebacks. `cancel_at_period_end: true` keeps the subscription active until the billing period ends, gives the customer what they paid for, and gives you a final win-back window.

The confirmation screen should show:

- The exact date access ends (format it as a human date, not a Unix timestamp)
- What happens to their data
- A single link to reactivate if they change their mind

---

### Screen 5: Post-cancel handoff

This is the screen most teams skip. It should do three things:

1. Thank the customer without guilt-tripping them.
2. Give a one-click reactivation path visible immediately.
3. Trigger your win-back email sequence starting that day (see §8).

The post-cancel screen is also where you ask the one optional open-text question: "Anything we could have done differently?" This is not a save mechanism. It is product feedback. Keep it optional and do not gate the confirmation on it.

---

### ASCII mockup: the save offer screen

```
+------------------------------------------------+
|  Before you go                                 |
|                                                |
|  You mentioned you're not using it enough.     |
|  A lot of customers hit the same wall in Q1.   |
|                                                |
|  [ Pause for 1 month . resume automatically ] |
|                                                |
|  Pausing keeps your data and settings intact.  |
|  You will not be billed while paused.          |
|                                                |
|  [ Accept pause ]                              |
|                                                |
|  or  continue cancelling >                     |
+------------------------------------------------+
```

Notes on this mockup:

- The offer is specific to the reason (not generic "here's 20% off").
- "Continue cancelling" is plain text but visible without scrolling.
- No countdown timer. No fake urgency. No auto-selected checkbox.
- One offer. Not a menu of three options at different price points.

---

### What to avoid on every screen

**Never auto-select an offer.** If the checkbox for "accept 20% discount" starts checked, you have a dark pattern. It destroys trust and is increasingly cited in FTC enforcement actions.

**Never bury cancel below the fold on an offer screen.** If the save offer is long enough that the customer must scroll to find "continue cancelling," you have an accessibility and legal problem.

**Never use confirmation shaming copy.** "No thanks, I hate saving money" as the decline button reads as contempt for the customer and trains them to associate your brand with manipulation.

**Never skip the period-end flag.** Immediate cancellation on the first click, with no confirmation, is a support ticket waiting to happen. Always use `cancel_at_period_end: true` and always show the access-end date.

---

### Implementation checklist

- [ ] Cancel link reachable in 2 clicks from any page with a signup CTA
- [ ] Flow is a focused modal or route, not a generic browser confirm()
- [ ] Reason capture uses a branching follow-up (§4), not a flat dropdown
- [ ] Save offer is pause-first for low-usage reasons (§5)
- [ ] "Continue cancelling" visible on every screen without scrolling
- [ ] Stripe call uses `cancel_at_period_end: true`, not immediate cancel
- [ ] Confirmation screen shows the exact access-end date
- [ ] Post-cancel screen shows a reactivation link and triggers win-back email
- [ ] No auto-selected offers, no confirmation shaming, no fake timers
