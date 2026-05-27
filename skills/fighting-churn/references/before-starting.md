# Before starting

Three numbers and three constraints. They change the recipe, not just the details.

**1. Monthly churn rate AND active subscriber count.** "3% on 400 subs" means ~12 cancels/month, which is the threshold where save-offer testing produces signal. Below ~10 cancels/month, skip A/B testing and ship a single opinionated flow. (§3 also asks *how* you computed that rate. Most founders get it wrong.)

**2. Stripe setup:** monthly or annual or both? Pause supported in your product or no? Does cancel route through the Stripe portal today (zero friction, zero offer) or a custom flow?

**3. Constraints:** B2B or B2C, self-serve required, and jurisdiction exposure (California ARL + FTC click-to-cancel = cancellation must be at least as easy as signup).

Rough estimates beat waiting for exact figures. "~15 cancels a month, monthly only, Stripe portal today, B2C, US-wide" is enough to start.
