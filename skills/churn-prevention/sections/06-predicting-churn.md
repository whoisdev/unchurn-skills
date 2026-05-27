## 6. Predicting churn before the click

By the time someone clicks cancel, they have already mentally left. The click is a formality. Your job is to catch the signals that precede it.

### Leading signals

Not all signals carry equal weight. This table orders them by reliability, with approximate lead time before the cancel event.

| Signal | Risk level | Typical lead time | Notes |
|--------|-----------|-------------------|-------|
| Billing page visits increase | Critical | Days | Highest-reliability signal. Details below. |
| Data export initiated | Critical | Days | User is extracting what they need before leaving. |
| Support tickets spike, then go silent | High | 1-2 weeks | The spike is frustration; the silence is resignation. |
| Key feature usage stops | High | 1-3 weeks | Define "key feature" as the one action correlated with retention in your cohort data. |
| Login frequency drops 50%+ | High | 2-4 weeks | Reliable lagging indicator; not fast enough alone. |
| NPS detractor score submitted | Medium | 1-3 months | Long lead time. Useful for early-stage intervention, not crisis response. |

### The free Stripe signal most founders miss

When a customer opens your billing page, that is expressed intent to cancel, not idle curiosity. Two flavors to instrument:

**Stripe Customer Portal sessions.** Every time you create a portal session via `stripe.billingPortal.sessions.create(...)`, log the customer ID and timestamp. The portal is where Stripe-native cancel buttons live. A customer who has visited your billing portal twice in a week but not engaged with the product is a Tier 1 churn signal.

**In-app billing page visits.** If you have a `/settings/billing` route, fire an analytics event on page load. This costs nothing to add and gives you the same signal without requiring a portal session.

Treat any billing-page visit from a customer with otherwise declining engagement as an immediate action trigger. You do not need a health score to act on this. Email them the same day.

### Health score model

For founders with more than a few hundred active subscribers, a simple weighted score helps prioritize who to contact first. This is not machine learning. It is a weighted sum on a 0-100 scale.

| Component | Weight | How to measure |
|-----------|--------|----------------|
| Login frequency | 30% | Logins in the last 14 days vs. their 90-day average |
| Feature usage | 25% | Key-feature events in last 14 days vs. 90-day average |
| Billing-page-visit recency | 20% | Days since last billing-page visit; higher recency = lower score |
| Support sentiment | 15% | Recent ticket tone; no tickets recently (after a history of tickets) scores low |
| Engagement breadth | 10% | Number of distinct features touched in last 30 days |

Normalize each component to 0-100, multiply by its weight, and sum. A customer who logged in yesterday, used three features, and has not visited billing scores near 100. A customer who visited billing twice this week, last logged in 12 days ago, and opened a frustrated ticket 10 days ago scores near 20.

### Score to action

| Score range | Status | Action |
|-------------|--------|--------|
| 80-100 | Healthy | Upsell window. This customer is primed to expand. |
| 60-79 | Needs attention | Trigger an in-app nudge or usage tip email within 3 days. |
| 40-59 | At risk | Outbound email or in-app intervention within 48 hours. |
| 0-39 | Critical | Personal founder email within 24 hours, no automation. |

The 0-39 bucket should be short. If you have 40 customers there, your product has a problem a save email will not fix.

### Proactive intervention triggers

| Trigger | Intervention |
|---------|-------------|
| Key-feature usage drops to zero for 7 days | "Noticed you haven't used X recently, anything I can help with?" email from founder address |
| Billing-page visit + login frequency down 50%+ in same week | Founder reaches out personally within 24 hours, no automation, no template |
| NPS detractor score submitted | Personal follow-up within 48 hours asking one specific question about what broke |
| Annual renewal in 30 days | Value recap email citing their own usage data and the outcomes they have gotten |
| Data export initiated | Immediate personal email; do not wait for cancel event |

Keep the intervention copy short. One sentence on what you noticed, one question. The goal is to open a conversation, not deliver a pitch.

### The honest caveat

Below roughly 500 active subscribers, this scoring model is noisy. The variance in any individual week is high enough to generate false positives and miss real churn. At under $10K MRR, you are probably better served by the founder reading the last 30 cancel-reason responses every Friday morning and personally emailing anyone who fits a pattern. That takes 20 minutes and beats an automated system built on thin data.

Above $10K MRR and growing, the manual read becomes unsustainable. That is when you instrument the billing-page signal, wire up the health score, and set automated triggers for the 60-79 and 40-59 bands. Keep the 0-39 band as a personal email, always. Automation degrades trust precisely when trust matters most.
