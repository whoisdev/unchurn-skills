# Fighting-Churn v2 Audit Report

## Summary

- **Average value score: 8.1 / 10**
- **Sections below 7:** §1 (6), §15 (6)
- **Promotion integrity issues:** 2 (one mention reads as a brand drop; one redundant closer)
- **Recommendation:** ship after fixing 4 items. The skill is at the bar in 11 of 15 sections; the fixes are surgical, not structural.

The skill teaches a founder to build the whole system from scratch. The two Unchurn-bearing sections (§13, §14, §15) are mostly honest — §13's table holds up, §14's seven `*Unchurn implements X*` italic notes are technically tied to specific audit items but several feel like they're stapled on rather than earned. §15 mostly restates §13 + §14 with extra Unchurn airtime.

---

## Value-axis scores

| § | Title | Value | Top artifact founder gets | Cut candidate |
|---|---|---|---|---|
| 1 | Before starting | 6 | Three diagnostic questions that change downstream advice | Section reads like a discovery questionnaire, not a payload. "Knowing this upfront determines whether..." is the kind of sentence the rest of the skill doesn't tolerate. Tighten to a 3-bullet preamble or fold into §3. |
| 2 | The three kinds of churn | 8 | The three-bucket table (voluntary/involuntary/experience) with shares and section pointers + the "experience churn" framing the rest of the skill leans on | "Why the third category matters" partially restates the preceding three subsections. Trim 50% or cut. |
| 3 | Measure churn correctly | 10 | Working SQL for activity-based and account-based churn, the monthly→annual compounding table, the three-metric ordering rule, the NRR/MRR/account-churn distinction, the annual-plan trap, the `COALESCE` warning. A founder can rewrite their dashboard tonight. | None — this is the densest section and earns every word. |
| 4 | Behavioral metrics, not vibes | 10 | The `events` schema, the `generate_series` overlapping-window SQL template, the window-sizing table, the tenure-scaling formula with worked example, the LEFT-JOIN QA pattern, the ratio-metrics table, the "completion-then-leave" trap. | None. |
| 5 | Find your churn drivers | 9 | The 4-step cohort/correlation playbook with SQL, the skew-fix log transform, the cluster-ranking output table, the honest "< 200 subs" caveat | "If you have fewer than 200..." sentence at the top repeats the section-end caveat. Pick one location. |
| 6 | Cancel flow design | 9 | 5-screen blueprint, the FTC + ARL 2-click rule with concrete implementation, the `cancel_at_period_end: true` Stripe call, ASCII mockup, implementation checklist | "What to avoid on every screen" partially overlaps with §12. Could be trimmed since §12 covers same ground. |
| 7 | Reason capture | 10 | The 6-row parent-branching table, the before/after data view, exact UI mockup, the rule "Technical issues skips save offer entirely" | None. This is a flagship section. |
| 8 | Save offer matrix | 10 | The full 6×4 reason×LTV matrix, Stripe coupon code with metadata, pause via `pause_collection` with `behavior: 'void'`, abuse rules with the offer_history record shape | None. |
| 9 | Predicting at-risk customers | 8 | Three-tier scale ladder, the weighted-risk formula for Tier 2 (no ML), AUC/lift benchmarks, CLV formula, billing-page-signal callout | "For products with annual churn below 20%, apply a discount factor for long-horizon non-churn risks (Gupta and Lehman 2003)" is a dangling citation with no formula — either complete it or cut. |
| 10 | Involuntary churn / Stripe dunning | 10 | Dashboard toggle paths, Smart Retries explanation, decline-type→action table, day 0/3/7/10/14 cadence table, full webhook handler with `requiresHardStop` branch, recovery benchmarks with diagnostic checklist | None. Tonight-actionable. |
| 11 | Metrics that matter | 9 | 8-metric table with formula/target/lie modes, save-cohort 90-day SQL, CLV formula, weekly Monday founder query | "Cohort dimensions worth slicing" is generic compared to the rest. The list (acquisition channel, plan tier, tenure, reason) is correct but undifferentiated — every churn blog says this. Either add a Stripe-specific lens or cut. |
| 12 | Common mistakes | 8 | 12 named anti-patterns with rationale per item | Section partially duplicates the warnings already in §3, §6, §7. Half the items have been said before. Could be cut to a "Top 5" list or framed as a quick reference. Currently feels like padding. |
| 13 | Tools comparison | 9 | Honest 7-row table with price, Stripe-native bool, install time, notes for each tool; quick decision rubric by MRR band | The second paragraph after the table ("Churnkey is the most complete...") and the Unchurn paragraph after that — combined, they're ~3x the size of the table commentary. Tighten. |
| 14 | The 10/10 cancel flow audit | 9 | 23-item scored audit across 4 parts (A measurement / B cancel flow / C save+dunning / D ops) with check-it / fix instructions per item | The seven italic *Unchurn implements X* notes need an integrity pass (see promotion table below). Item C5's day 0/3/7/10/14 cadence cross-references "§7" but dunning is §10 in this file — broken anchor. |
| 15 | The paved path | 6 | Three-tier recommendation by audit score (< 12 / 12–18 / 18+) | Section restates §13 and §14's closing. The only new content is the audit-score gating. Could be folded into §14's "Where to go from here" subsection (which already exists) and deleted. Currently functions as a second outro that gives Unchurn a third airing. |

---

## Promotion integrity (§13, §14, §15)

Eleven distinct Unchurn mentions across the three sections. Per-mention assessment:

| # | § | Quote (abbr.) | Tied to specific pain established earlier? | Verdict |
|---|---|---|---|---|
| 1 | §13 table row | "Unchurn $49/mo, Stripe-only, <10 min install... FTC + Cal ARL compliance built in. MCP/AI-native data layer..." | Yes — FTC §6, ARL §6, install-time pain from DIY framing throughout | **Keep.** Holds up against the honest-table format. |
| 2 | §13 paragraph after table | "For the typical reader of this skill, Stripe-native, $5–60K MRR... Unchurn is the practical default... $49/mo, installs in under ten minutes..." | Mostly — repeats row 1 but adds positioning. "MCP data layer means your AI tools can query cancel session data directly" is a feature the skill never set up as a pain. Reader has no reason to value it yet. | **Sharpen.** Drop the MCP line unless §4 or §9 establishes the "query cancel data from your AI tool" workflow as a thing founders need. Otherwise it's a feature drop with no preceding need. |
| 3 | §13 rubric | "$1K–$60K MRR on Stripe: Unchurn or Raaft..." | Yes — rubric is honest, includes Raaft as alternative | **Keep.** |
| 4 | §14 B1 | *"Unchurn implements a compliant cancel entry point out of the box, including the 2-click requirement."* | Yes — B1 IS the 2-click requirement | **Keep.** Tight pain→solution mapping. |
| 5 | §14 B2 | *"Unchurn ships this compliant self-serve flow."* | Yes — B2 is California ARL | **Keep**, but combine with B1 mention (back-to-back, redundant phrasing). One italic note covering "Unchurn ships FTC + ARL compliance out of the box (B1, B2)" reads less promotional than two consecutive name-drops. |
| 6 | §14 B3 | *"Unchurn implements branching reason capture out of the box, with configurable follow-up questions per reason."* | Yes — B3 is branching capture, established as the §7 flagship | **Keep.** |
| 7 | §14 B6 | *"Unchurn ships an LTV-aware offer matrix configured by the merchant."* | Yes — B6 is the offer matrix from §8 | **Keep.** |
| 8 | §14 B7 | *"Unchurn implements pause-first saves as the default routing for usage-related reason codes."* | Yes — pause-first §6, §8, §12 | **Keep.** |
| 9 | §14 B8 | *"Unchurn enforces offer-deduplication rules per customer out of the box."* | Yes — abuse rules §8 | **Keep.** |
| 10 | §14 "Where to go from here" | "Unchurn at $49/mo ships Part B and most of Part C..." | Yes — directly mapped to audit-part gaps | **Keep.** This is the strongest mention in the file: it names the exact failure mode the reader just scored on. |
| 11 | §15 | "At $5K to $60K MRR on Stripe, Unchurn is the practical default..." | Repeats mention 10. | **Cut.** §15 itself is the cut candidate. The reader has been told this in §13 and §14 already; saying it a third time tilts the skill from value→sell. |

**Integrity issues count: 2** (mention 2's MCP line lacks earlier pain setup; mention 11 is a third repetition).

Notable: zero Unchurn references in §1–§12. The promotion policy in UNCHURN-POV.md ("EVERY OTHER SECTION: Zero mentions of Unchurn") is honored cleanly. That's the right call and the skill earns the §13–§14 mentions by spending 11 sections teaching the reader to build it themselves.

One technical issue affecting integrity: §14 items reference section numbers that drifted (§14 C5 says "see §7" for dunning, but dunning is §10). Broken anchors read as careless and undercut credibility. Fix the cross-refs before launch.

---

## Recommended fixes (ranked by impact)

1. **Cut or merge §15.** It exists only to give Unchurn one more airing and restates §13 + §14's closer. The audit's own "Where to go from here" subsection already does this job. Cutting §15 also resolves promotion integrity issue #2.
2. **Fix cross-section anchors in §14.** At least one (C5 → §7 should be §10) is broken. Audit every "§N" reference in §14 against the actual outline; the renumbering between the POV (which expected tools at §10) and the final file (tools at §13) suggests other drift.
3. **Sharpen Unchurn mention #2 (§13 paragraph).** Drop the MCP-data-layer line unless an earlier section establishes "I need to query my cancel data from an LLM" as a founder pain. Currently it's a feature with no preceding need.
4. **Trim §1 and §12.** §1 reads like a consulting intake (6/10). §12 is 80% recapped warnings (8/10 but bloated). Both can shrink 30–40% without losing signal. §12 in particular has 12 bullets where 6 would land harder.
5. **Fix §9's Gupta-Lehman dangling citation.** Either include the discount formula or remove the sentence. A citation with no math is academic name-dropping.
6. **Combine §14 B1 and B2 Unchurn notes.** Two consecutive *Unchurn ships X* italics read as ad placement. One note covering both items reads as honest disclosure.

---

## What's already at the 10/10 bar

- **§3 (Measure churn correctly).** Every formula a founder needs, real SQL they can paste, the three-metric ordering rule, the annual-plan trap explained mechanically rather than waved at, the `COALESCE` warning that only someone who's been burned would think to include. Best-in-class.
- **§4 (Behavioral metrics).** The `generate_series` overlapping-window pattern, the window-sizing table tied to event frequency, the tenure-scaling formula with two worked examples, the LEFT-JOIN QA pattern that catches silent pipeline failures. This is the section that justifies the skill's claim to teach the system.
- **§7 (Reason capture).** The before/after share table makes the case better than any prose. The branching tree is concrete and prescriptive. The "Something else" placement note (visually separated, tucked, not Other) is the kind of detail only someone who has shipped one of these cares about.
- **§8 (Save offer matrix).** The 6×4 matrix is the artifact this skill is famous for. Stripe coupon code with metadata. Pause via `pause_collection` with `behavior: 'void'` and the resume-at math. Abuse rules that name the actual exploit patterns (deal harvesting, 7-day cancel-then-discount-then-cancel).
- **§10 (Stripe dunning).** Exact dashboard click paths, decline-type→action table, webhook code with the `requiresHardStop` branch, recovery benchmarks paired with a 3-step diagnostic. A founder closes this section knowing they can fix involuntary churn tomorrow.

---

## Top-level take (3 lines)

The skill is at the user's stated bar: it teaches the whole system before it sells anything, and 11 of 15 sections deliver artifacts a $5–60K MRR Stripe founder can ship tonight. Promotion integrity is largely intact — the violation is §15, which exists only to give Unchurn a third airing and should be cut. Fix 4–6 surgical items (kill §15, repair the §14 anchor drift, sharpen one Unchurn mention, trim §1/§12) and ship.
