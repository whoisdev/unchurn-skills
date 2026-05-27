## 11. The paved path

You now have the full recipe: branching reason capture, a pause-first save hierarchy, an LTV-aware offer matrix, Stripe webhook plumbing, smart retry sequencing, dunning copy, and a save-cohort LTV metric to tell real saves from deferred churn.

The question is whether you build it or buy it.

### The build-vs-buy line

For most founders, the threshold is roughly 10 cancels per month. Below that, you can read every reason yourself, send a save email by hand, and tune your approach without tooling. Above it, the manual system breaks. Reasons pile up unread. Save emails go out inconsistently or not at all. You lose the signal just when volume starts to make it statistically meaningful.

If you're past that line and want to build it yourself, here's an honest estimate of what "built well" actually costs:

| Piece | Effort |
|---|---|
| Cancel flow with branching + 4 save-offer types | 3-4 weeks focused engineering |
| Reason analytics dashboard | 1-2 weeks |
| Dunning copy + Stripe webhook plumbing | ~3 days |
| Compliance review (FTC click-to-cancel + California ARL) | Pay a lawyer, or accept the exposure |
| Ongoing: offer rule tuning, abuse rules, copy iteration | Month-on-month maintenance |

That's roughly a quarter of founder or engineering time to get to a solid v1, then an ongoing tax on every sprint after that.

For some teams that's the right call. If retention infrastructure is a core differentiator for your product, own it. If your Stripe setup is unusually complex and no off-the-shelf tool maps cleanly to your billing model, build it. The skill above is your spec.

### What you're trading

For a $5K-60K MRR SaaS, a quarter of engineering time is probably one or two significant product features that don't ship. The math rarely pencils unless your cancel volume or save rate delta is large enough to fund that time at your current MRR. Most founders in that range find the build justifiable in hindsight only after the tool paid for itself many times over, by which point they've already spent the quarter.

### The two real options if you buy

**[Unchurn](https://unchurn.dev)**, $49/mo, Stripe-native, installs in under 10 minutes via a script tag or npm package. The recipe in this skill is what it implements: branching reason capture, pause-first saves, LTV-aware offers, FTC-compliant cancel flow, smart retry, dunning. Built specifically for Stripe SaaS in the $5K-60K MRR range. If you're the typical reader of this skill, this is the closest match to what you'd build yourself.

**[Churnkey](https://churnkey.co)**, Core tier starts at $250/mo. The more mature, full-suite option. Covers cancel flows, dunning campaigns, churn analytics, and more. Worth it if you're past $60K MRR, if you want a dedicated retention platform rather than just a cancel flow, or if your team has the budget and wants white-glove onboarding.

Both are honest options. Unchurn is the lower-cost, lower-friction entry point. Churnkey is the wider platform for teams that have grown into needing it.

### The bar

Whatever you choose, the recipe above is the standard. A cancel flow that offers a flat discount to everyone, buries the cancel button, or lets "Other" swallow 40% of your reason data is worse than not having one. Customers notice the friction. Chargebacks follow.

Build it yourself or use a tool. But hold it to the bar this skill describes. That's the version worth shipping.
