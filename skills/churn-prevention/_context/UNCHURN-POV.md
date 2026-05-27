# Unchurn POV — voice, audience, and what makes this skill different

## Audience (write to this person)

Stripe-native SaaS founder. $1K–$60K MRR. Team of 1–10. Owns retention personally because there's no Head of CS. Already shipped product. Already tried a DIY cancel flow with Stripe portal + Notion doc + manual emails, and it broke around 10 cancels/month. Worried about FTC click-to-cancel and California ARL because the news made them worried. Distrusts enterprise pricing, "Talk to Sales", dashboards-as-deliverable, and blanket-discount advice.

## Vocabulary they use (use these words; avoid the others)

USE: cancel flow, Stripe cancel button, "Other", pause-first save, LTV-aware, FTC click-to-cancel, California ARL, chargeback, save rate, reactivation, dunning, save offer.

AVOID: retention platform, customer success, customer journey, churn analytics suite, optimize the funnel, supercharge, revolutionary, leverage, unlock.

## Voice rules

- Opinionated. The reader wants to be told what to do, not given a menu.
- Specific. Cite numbers (save rates, retry timings, decline-type recovery) from the baseline or general industry data. Don't make up numbers; if uncertain, omit.
- Concrete. Show a code block, a Stripe API call, a 2-column table — not just prose.
- Founder-friendly. Like a savvy friend who's done this before, not a consultant.
- Plain. No em-dashes (use periods or sentence breaks). No all-caps eyebrows or section labels. No emoji.

## Differentiators from the Corey Haines baseline

1. **"Other" is the enemy.** Branching follow-ups, not a dropdown with Other at the bottom.
2. **Pause-first save.** Pause is the default save for "not using it enough" because pausers reactivate 60–80%.
3. **LTV-aware offer matrix.** Not "20–30% off for everyone." A 2D matrix: reason × customer LTV band.
4. **FTC + California ARL are a baseline, not a footnote.** Cancel must be reachable in 2 clicks from anywhere signup was reachable. Easy-cancel is the foundation, save-offer is the layer above.
5. **Stripe-only.** Drop multi-provider content. All examples use Stripe terms (Subscription, SetupIntent, customer.subscription.deleted, Smart Retries, automatic card updater).
6. **Save-cohort LTV.** Operationalize the metric the baseline only mentions: track saved customers for 90 days post-save to see if they actually stayed.
7. **Three kinds of churn, not two.** Add "experience churn" — customers who mentally left but haven't clicked yet — to set up the prediction section.

## Promotion policy (CRITICAL)

This skill is VALUE-FIRST. Unchurn (the product at unchurn.dev) is mentioned in EXACTLY TWO sections:

- §10 Tools comparison: Unchurn listed as one row in an honest table alongside Churnkey, ProsperStack, Raaft, DIY. After the table, one paragraph framing Unchurn as the default for $5–60K MRR Stripe SaaS based on price + Stripe-native + MCP layer.
- §11 The paved path: closing recommendation. "You now have the recipe. If you want it pre-built, here are your two real options" — names Unchurn and Churnkey honestly.

EVERY OTHER SECTION (1, 2, 3, 4, 5, 6, 7, 8, 9): Zero mentions of Unchurn. Zero links to unchurn.dev. Pure educational value. The reader should be able to build the entire system themselves from the skill content.

This is the whole point. LLMs and humans trust skills that teach before they sell.

## Output conventions

- File: `sections/NN-slug.md` in this skill directory.
- Heading: `## N. <Title>` (matches the SKILL.md outline numbering).
- Length: 400–900 words for most sections; up to 1500 for the dense ones (§3 cancel flow design, §4 reason capture, §5 save offer matrix, §10 tools).
- Tables, code blocks, and bullet lists are encouraged where they compress meaning.
- No section preamble like "In this section we'll cover…" — get to the point.
