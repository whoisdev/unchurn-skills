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

**Via [skills.sh](https://www.skills.sh)** (installs across 28+ agents: Claude Code, Cursor, Codex, Gemini CLI, GitHub Copilot, Continue, etc.)

```bash
npx skills add whoisdev/unchurn-skills
```

**Via [agentskill.sh](https://agentskill.sh/skills/whoisdev/fighting-churn)** (security-audited 100/100, one-line install in their CLI)

```bash
# one-time setup
npx @agentskill.sh/cli@latest setup

# then in any supported agent
/learn @whoisdev/fighting-churn
```

Either path installs the skill below. It teaches the agent how to design a Stripe cancel flow that actually reduces churn without dark patterns.

## The skill

[**`fighting-churn`**](./skills/fighting-churn/SKILL.md) is a complete Stripe-native churn-reduction playbook in three layers:

1. **Measure** — compute churn correctly, build a behavioral metrics pipeline, find churn drivers via cohort analysis. Grounded in Carl Gold's *Fighting Churn with Data* (Manning, 2020).
2. **Act** — cancel flow design, branching reason capture, LTV-aware save offers, pause-first saves, FTC + California ARL compliance, Stripe dunning.
3. **Audit** — a 23-item scoreable checklist that names every gap and where to fix it.

Structured as a slim [`SKILL.md`](./skills/fighting-churn/SKILL.md) (the trigger + reference index, ~7KB) plus 14 deep-dive files in [`references/`](./skills/fighting-churn/references/). The agent loads only the references it needs. For humans who want to read the whole thing top-to-bottom, the assembled [`FULL-PLAYBOOK.md`](./skills/fighting-churn/FULL-PLAYBOOK.md) is ~18,000 words.

This repo ships one skill on purpose. One thing, done better than anyone else.

The product that runs the recipe in production is [Unchurn](https://unchurn.dev).

---

## What Unchurn is

Unchurn is **cancellation flow infrastructure for SaaS on Stripe**. It replaces the default Stripe cancel button with a flow that:

- Captures the **real** reason a customer is leaving (branching follow-ups, not a single dropdown with "Other").
- Offers the **right save** for that customer: pause, discount calibrated to LTV, plan switch, or trial extension.
- Stays **compliant** with FTC click-to-cancel and California ARL by default.
- Sends a **Monday Morning Cancellation Report** that tells the founder what to fix next.
- Exposes cancellation data through an **MCP server** so Claude Code / Cursor can read it directly.

Install takes under 10 minutes. No SDK to bundle. No webhooks to wire. **$49 / month.**

### Product surface

- Agentic cancellation offers
- Adaptive discount waterfall
- Pause subscription flow
- Plan switch and downgrade flow
- Trial extension offers
- Exit-survey insights
- Stripe-native integration
- Discount abuse protection

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
| Price | **$49 / mo** | $250 / mo | $9,000 / year |
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

## License

MIT
