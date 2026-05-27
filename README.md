<p align="center">
  <a href="https://unchurn.dev">
    <img src="./assets/banner.jpg" alt="Unchurn — cancel-flow infrastructure for Stripe SaaS" width="100%" />
  </a>
</p>

<h1 align="center">Unchurn Skills</h1>

<p align="center">
  Agent skills for Stripe SaaS founders fighting churn. Maintained by <a href="https://unchurn.dev">Unchurn</a>.
</p>

<p align="center">
  <a href="https://unchurn.dev">Website</a> ·
  <a href="https://app.unchurn.dev">App</a> ·
  <a href="https://docs.unchurn.dev">Docs</a> ·
  <a href="https://www.skills.sh">skills.sh</a>
</p>

---

## Install

```bash
npx skills add whoisdev/unchurn-skills
```

This installs the skills below into your Claude Code, Cursor, or any skills.sh-compatible agent. The skills teach the agent how to design a Stripe cancel flow that actually reduces churn without dark patterns.

## Skills in this repo

| Skill | What it does |
| --- | --- |
| `cancel-flow` *(coming)* | Design a Stripe-native cancel flow: branching reason capture, LTV-aware save offers, pause-first saves, FTC click-to-cancel + California ARL compliance. |
| `onboarding-to-prevent-churn` *(coming)* | Map first-week activation milestones to cancellation reasons. Fix churn upstream by fixing onboarding. |

Each skill is a recipe. The product that runs the recipe in production is [Unchurn](https://unchurn.dev).

---

## What Unchurn is

Unchurn is **cancellation flow infrastructure for SaaS on Stripe**. It replaces the default Stripe cancel button with a flow that:

- Captures the **real** reason a customer is leaving (branching follow-ups, not a single dropdown with "Other").
- Offers the **right save** for that customer: pause, discount calibrated to LTV, plan switch, or trial extension.
- Stays **compliant** with FTC click-to-cancel and California ARL by default.
- Sends a **Monday Morning Cancellation Report** that tells the founder what to fix next.
- Exposes cancellation data through an **MCP server** so Claude Code / Cursor can read it directly.

Install takes under 10 minutes. No SDK to bundle. No webhooks to wire.

### Product surface

- Agentic cancellation offers
- Adaptive discount waterfall
- Pause subscription flow
- Plan switch and downgrade flow
- Trial extension offers
- Exit-survey insights
- Stripe-native integration
- Discount abuse protection

### Pricing

Flat **$79 / month**. No seat fees. No usage caps. No "Talk to Sales". 30-day money-back guarantee.

---

## Who it's for

| | Fit |
| --- | --- |
| Stripe-native SaaS at **$1K–$60K MRR** | ✅ |
| Dev tools, B2B SaaS, prosumer software, AI-native tools, agency software | ✅ |
| 1–10 person team, founder owns retention | ✅ |
| Pre-$1K MRR (not enough cancellations yet to measure) | ❌ |
| Billing on Paddle, LemonSqueezy, or Chargebee | ❌ |
| Enterprise / sales-led / Gainsight territory | ❌ |

---

## How Unchurn compares

### vs Churnkey

| | Unchurn | Churnkey Core | Churnkey Intelligence |
| --- | --- | --- | --- |
| Price | **$79 / mo flat** | $250 / mo | $9,000 / year |
| Minimum churn requirement | None | None | $10,000+ monthly churn |
| FTC / ARL compliance routing | Included | Higher tier | Higher tier |
| Branching reason capture | Included | Limited | Included |
| LTV-aware save offers | Included | Limited | Included |
| MCP / AI-native data layer | Included | Not available | Not available |
| Install time | Under 10 minutes | Under 10 minutes | Sales-assisted |

Unchurn is the cheaper, simpler, AI-native cancel-flow layer. Churnkey is a larger retention suite.

### vs DIY (Stripe portal + spreadsheets + manual emails)

DIY works at the very start. It breaks when:

- Customers click "Other" and you stop reading reasons by week 3.
- You need to prove FTC click-to-cancel / California ARL compliance.
- You want to offer different saves to a $9/mo user and a $290/mo team plan.
- You need cancellation data structured enough to feed back into product decisions.

Unchurn replaces that DIY surface with a production cancel flow in under 10 minutes.

### vs analytics / CS / dunning tools

Unchurn is **not**:

- A churn analytics suite (that's Baremetrics or ChartMogul)
- A customer success platform (that's Gainsight or ChurnZero)
- A failed-payment recovery tool (that's Stripe Smart Retries or FlyCode)
- A billing platform (that's Paddle, LemonSqueezy, or Chargebee)

Unchurn owns one surface: **the cancel flow**.

---

## The problem these skills exist to solve

Founders running Stripe-billed SaaS lose customers through a cancel surface they never designed. The default Stripe portal collects no reason, offers no save, and produces no learning. Founders patch it with a Notion doc, a spreadsheet, and manual save emails. That works until cancellation volume passes ~10/month, at which point the manual loop collapses and the reason data becomes garbage ("Other", "Other", "Other").

The skills in this repo encode the playbook for fixing this. The product that runs the playbook is Unchurn.

---

## Links

- Product: https://unchurn.dev
- App: https://app.unchurn.dev
- Docs: https://docs.unchurn.dev
- Pricing: https://unchurn.dev/pricing
- Compare: https://unchurn.dev/vs/churnkey

## License

MIT
