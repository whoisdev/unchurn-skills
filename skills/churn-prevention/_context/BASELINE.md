# BASELINE: Corey Haines `churn-prevention` skill v2.0.0

Source: https://github.com/coreyhaines31/marketingskills/tree/main/skills/churn-prevention

This is reference material. Do not copy verbatim. Use it to understand the standard structure and to spot the gaps Unchurn's version closes.

---

```yaml
---
name: churn-prevention
description: "When the user wants to reduce churn, build cancellation flows, set up save offers, recover failed payments, or implement retention strategies. Also use when the user mentions 'churn,' 'cancel flow,' 'offboarding,' 'save offer,' 'dunning,' 'failed payment recovery,' 'win-back,' 'retention,' 'exit survey,' 'pause subscription,' 'involuntary churn,' 'people keep canceling,' 'churn rate is too high,' 'how do I keep users,' or 'customers are leaving.' Use this whenever someone is losing subscribers or wants to build systems to prevent it. For post-cancel win-back email sequences, see emails. For in-app upgrade paywalls, see paywalls."
metadata:
  version: 2.0.0
---
```

## Structure

1. Before Starting (4 context blocks: churn situation, billing, product/usage, constraints)
2. How This Skill Works (voluntary vs involuntary table)
3. Cancel Flow Design (structure, exit survey, dynamic offers, save offer types, UI patterns)
4. Churn Prediction & Proactive Retention (risk signals, health score, interventions)
5. Involuntary Churn: Payment Recovery (pre-dunning, smart retry, dunning emails, benchmarks)
6. Metrics & Measurement (churn metrics, cohort analysis, A/B tests)
7. Common Mistakes
8. Tool Integrations (Churnkey, ProsperStack, Raaft, Chargebee Retention; provider table; CLI tools)
9. Related Skills

## Tables worth keeping (adapt, don't copy)

### Reason → offer mapping
- Too expensive → 20-30% discount, fallback downgrade
- Not using it enough → pause, fallback onboarding
- Missing feature → roadmap, fallback workaround
- Switching to competitor → competitive comparison + discount, fallback feedback session
- Technical issues → escalate to support
- Temporary need → pause
- Business closed → respect, no offer

### Retry timing
- Retry 1: 24h after failure
- Retry 2: 3 days
- Retry 3: 5 days
- Retry 4: 7 days (with email escalation)
- After 4: hard cancel + reactivation path

### Health score weights
- Login frequency 30%
- Feature usage 25%
- Support sentiment 15%
- Billing health 15%
- Engagement score 15%

### Dunning email cadence
- Day 0: friendly alert
- Day 3: helpful reminder
- Day 7: urgency (3 day warning)
- Day 10: final warning

### Recovery benchmarks
- Soft decline recovery: <40% poor, 50-60% average, 70%+ good
- Hard decline recovery: <10% poor, 20-30% average, 40%+ good
- Overall payment recovery: <30% poor, 40-50% average, 60%+ good

### Churn metrics
- Monthly churn rate: <5% B2C, <2% B2B
- Save rate: 25-35%
- Offer acceptance rate: 15-25%
- Pause reactivation: 60-80%

## Where the baseline is WEAK (the gaps Unchurn closes)

1. **Treats "Other" as legitimate** in the exit survey. Real cancel data shows ~40% of users click Other when it's offered, destroying signal. Unchurn's version kills Other and branches instead.
2. **Flat 20-30% discount advice** doesn't scale across price points. A $9/mo customer doesn't need 25% off; a $290/mo customer needs more than a coupon. Unchurn uses a 2D matrix (reason × LTV band).
3. **Pause is one option among many**. Pause should be the default save for "not using it enough" reasons because pausers reactivate 60-80% while discount-takers churn at the next renewal.
4. **FTC click-to-cancel and California ARL are listed as a "common mistake" footnote**, not designed in as a compliance baseline. For Stripe SaaS in 2026, these are real legal exposure.
5. **Multi-provider scope** (Stripe, Chargebee, Paddle, Recurly, Braintree) dilutes the Stripe-specific advice. Unchurn's audience is Stripe-only.
6. **No save-cohort LTV tracking**. Baseline mentions the trap ("A 'saved' customer who churns 30 days later wasn't really saved") but doesn't operationalize how to track it.
7. **Reason capture is single-screen**. Should be ask → branch → ask again. The branching is where the real reason emerges.
