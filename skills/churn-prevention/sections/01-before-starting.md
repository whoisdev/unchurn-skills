## 1. Before starting

Three questions before recommending anything. Each one changes the recipe, not just the details.

---

### Question 1: What is your monthly churn rate, and how many active subscribers do you have?

Give both numbers. "3% churn on 400 subscribers" means roughly 12 cancels per month. That is the threshold where save-offer testing starts producing signal fast enough to act on. Below about 10 cancels per month, A/B testing save offers is mostly noise, and the right move is a single opinionated flow rather than a split test.

The subscriber count also sets the scale of the problem. A 5% churn rate means something very different at 80 subscribers versus 2,000. Knowing the raw cancel volume tells us how quickly you can iterate.

---

### Question 2: What does your current Stripe setup look like?

Specifically:

- Billing intervals: monthly only, annual only, or both? Annual subscribers rarely see a cancel flow because the next renewal is 11 months away. Monthly is the high-frequency case where a cancel flow pays off fastest.
- Pause support: does your product support pausing access, or does pausing a subscription leave the user in a broken state? Pause is the highest-converting save for "not using it right now" cancels. If pause is not technically feasible today, the offer matrix looks different.
- Current cancel path: does cancellation go straight through the Stripe billing portal, or do you have any custom flow in front of it? If Stripe portal is handling it directly, subscribers are hitting "Cancel subscription" with zero friction and zero offer. That is the gap this skill helps you close.

---

### Question 3: What are your constraints?

Three sub-questions:

- B2B or B2C? B2B subscribers often have procurement or approval loops, which changes how you phrase save offers and whether a "call with our team" offer has any conversion potential. B2C is faster to test and more price-sensitive.
- Self-serve required? Some teams need the entire cancel flow to be zero-touch. Others can route high-LTV accounts to a human. Knowing this upfront determines whether a "talk to us" deflection is in scope.
- Jurisdiction exposure: do you have subscribers in California, or are you selling to US consumers broadly? California's Automatic Renewal Law (ARL) and the FTC's click-to-cancel rule both require that cancellation be at least as easy to initiate as signup. Getting the compliance baseline right is not optional, and it shapes how the cancel entry point is designed before any save offer is layered on top.

---

Once you have answers to all three, the rest of this skill will map directly to your situation. If you are not sure about a number, a rough estimate is fine. Better to start with "roughly 15 cancels a month, monthly only, Stripe portal today, B2C, US-wide" than to wait until you have exact figures.
