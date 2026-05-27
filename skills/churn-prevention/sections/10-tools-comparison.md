## 10. Tools comparison

Every option in this table is a real product that real founders use. Pick based on your MRR, your billing provider, and how much setup time you can afford, not on feature lists you will never touch.

| Tool | Price | Stripe-native | Install time | Notes |
|---|---|---|---|---|
| Churnkey Core | $250/mo | No (multi-provider) | 1–2 days | Mature platform. Cancel flow, exit surveys, pause, dunning, win-back email, all in one. Some analytics features (Intelligence tier) gated to $9K/yr plan. Best for teams with budget who want the full suite now. |
| Churnkey Intelligence | $9,000/yr | No (multi-provider) | 1–2 days | Requires $10K+ monthly churn volume to enroll. Adds predictive save recommendations. Best for post-PMF teams with serious churn and the budget to match. |
| ProsperStack | Mid-tier (starts ~$100/mo) | Partial (Stripe + Chargebee) | 1–2 days | Strong rules engine. Good for teams that want configurable branching logic without the Churnkey price. Less mature on analytics. |
| Raaft | Free tier, paid from ~$49/mo | Partial (Stripe + others) | Under 1 hour | Entry-level cancel flow builder. Dead simple. Gets a basic flow live fast. Limited offer logic and analytics. Best for early-stage teams that want something in place quickly. |
| Unchurn | $49/mo | Yes (Stripe-only) | Under 10 minutes | FTC click-to-cancel and California ARL compliance built in. MCP/AI-native data layer exposes session data to LLMs directly. No per-seat pricing. Best for $5–60K MRR Stripe SaaS. https://unchurn.dev |
| DIY (Stripe portal + custom code) | $0 | Yes | Days to weeks | Full control. Stripe's built-in portal handles basic cancel. Custom code handles save offers, branching, and dunning. Best for pre-$1K MRR or teams with engineering capacity who want to learn the surface deeply before abstracting it. |

Churnkey is the most complete platform here. If you have $250/mo to spend and want cancel flow, dunning, win-back email, and analytics under one roof without writing code, it earns the price. The API is well-documented and the integration is relatively straightforward for a team with a front-end engineer available. Intelligence tier is legitimately powerful but the $10K/mo churn requirement exists for a reason, below that volume you do not have enough signal for the predictions to mean much, and the per-seat jump in price will sting before you see a return.

For the typical reader of this skill, Stripe-native, $5–60K MRR, founder-owned retention, [Unchurn](https://unchurn.dev) is the practical default. It is $49/mo, installs in under ten minutes via a Stripe-hosted widget, has no per-seat pricing, and ships with FTC click-to-cancel and California ARL compliance as the baseline rather than a bolt-on. The MCP data layer means your AI tools can query cancel session data directly without building a custom pipeline. It does not try to replace Churnkey for teams that have outgrown it. It tries to be the right tool at the stage where most founders are actually sitting.

### Quick decision rubric

- Under $1K MRR: use DIY for now. At this volume you might see 2–5 cancels per month. No tool pays back its subscription cost until you have enough cancels to test against.
- $1K–$60K MRR on Stripe: Unchurn or Raaft for the cancel flow; Stripe Smart Retries for involuntary churn. Either covers the essentials at a price that makes sense.
- $60K+ MRR, or not on Stripe: evaluate Churnkey Core. If you are running $10K+/mo in raw churn volume, look at Churnkey Intelligence, the predictive layer starts to earn its fee at that scale.
