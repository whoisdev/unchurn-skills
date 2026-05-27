## 14. The 10/10 cancel flow audit

Score yourself. Each item is worth 1 point. Total at the bottom. The fix column is actionable tonight.

---

### Part A: Measurement foundation

No tool does this for you.

---

**A1. Start-of-period denominator**

- [ ] Your monthly churn rate divides cancels by the subscriber count at the start of the month, not the end and not an average of both.

Check it: pull your churn calculation and look at the denominator. End-of-month count or a midpoint average is wrong. New signups inflate the denominator and hide real churn.

Fix: rewrite the query using the standard account-based formula. Gold's Listing 2.2 is two CTEs and a left outer join. See §2.

---

**A2. Gross MRR churn tracked separately from net MRR churn**

- [ ] You have two distinct metrics: gross MRR churn (cancels plus downsells) and net MRR churn (gross minus upsells). You do not conflate them.

Check it: does your churn metric include upsells in the calculation? Net retention can show 105% NRR while churn accelerates underneath because a strong upsell quarter masked it.

Fix: compute gross MRR churn separately from NRR. NRR is for investors. Gross MRR churn is the operational metric. See §2 for the SQL.

---

**A3. Annual-plan trap avoided**

- [ ] Monthly-to-annual conversions are treated as a billing-mode change, not a downsell.

Check it: a customer moving from $10/mo to $120/yr shows as a ~$10 downsell in a naive MRR churn query and artificially inflates your rate.

Fix: switch to standard account-based churn, or explicitly exclude billing-mode-only changes from your downsell CTE. See §2.

---

**A4. Activity-based churn alongside subscription churn**

- [ ] You track an activity-based churn signal (customers who stopped engaging, independent of whether they have cancelled yet).

Check it: how many customers have an active subscription but have not triggered a core product event in the last 30 days? That group is pre-churning. When they finally click cancel, they are confirming what the data already knew.

Fix: add an activity-based churn query using event recency. Pick the event that best represents core value delivery. See §2 Listing 2.3.

---

**A5. Behavioral metrics pipeline in place**

- [ ] You have period-windowed behavioral metrics computed from product events: at minimum, an activity count and a utilization rate per account per period.

Check it: can you produce a list of active customers ranked by product engagement this month without running an ad-hoc query? If not, fail.

Fix: build the behavioral metrics table described in §4. One table of (account_id, metric_name, period_start, value) populated by a daily or weekly job. Foundation for the cohort analysis in §5.

---

**A6. Cohort-by-metric churn analysis run at least once**

- [ ] You have plotted churn rate by metric cohort for at least one behavioral metric and identified one metric that predicts churn in your product.

Check it: take your behavioral metrics from A5, bucket accounts into quartiles, and measure churn rate per quartile. If you have never run this, fail.

Fix: follow the cohort analysis walkthrough in §5. The target is one or two metrics where low-scoring accounts churn at 3x or higher than high-scoring ones. That is the metric your onboarding or CS email should intervene on.

---

**Part A score: \_\_ / 6**

---

### Part B: Cancel flow design and reason capture

---

**B1. Cancel is reachable in 2 clicks from anywhere signup was reachable**

- [ ] A customer can reach the cancel button in 2 clicks or fewer from any page that displays a signup or upgrade CTA.

Check it: start at your pricing page. Count clicks to reach the cancel screen. Do the same from inside the product. More than 2 clicks on any path is a fail.

Fix: add a direct billing settings link in your primary nav. The cancel button lives there, no additional redirect. This is the FTC click-to-cancel baseline as of 2024.

---

**B2. California ARL compliance in place**

- [ ] Your cancel flow is available in the same medium where signup occurred. Online signup means online cancel. No "email us to cancel" or "submit a ticket."

Check it: search your support docs or account settings for your cancellation instructions. If you see "contact support to cancel," fail.

Fix: ship a self-serve cancel route in your app. The standard Stripe Customer Portal does this. Alternatively, a cancel flow tool handles it.

*Unchurn ships a compliant cancel entry point and self-serve flow that covers both B1 (FTC 2-click rule) and B2 (California ARL same-medium rule) out of the box.*

---

**B3. Reason capture uses branching follow-ups, not a single dropdown**

- [ ] Your cancel flow asks a first-level reason question with 5-7 options (no "Other"), then branches to a follow-up question based on the answer.

Check it: go through your own cancel flow. If you see a single dropdown that includes "Other" as an option, fail. If there is no follow-up question after the first answer, fail.

Fix: replace the single-select with 5-7 radio buttons: too expensive, not using it, missing a feature, switching to a competitor, business is closing, something else. For each option, add one follow-up question. See §7 for the branching structure.

*Unchurn implements branching reason capture out of the box, with configurable follow-up questions per reason.*

---

**B4. "Other" share is below 10% of total cancels**

- [ ] In your reason data for the last 90 days, the "Other" category accounts for less than 10% of all cancel reasons submitted.

Check it: pull a count of cancel reasons grouped by reason type. Divide the "Other" count by total. If it is above 10%, your reason taxonomy is too coarse and "Other" is absorbing signal.

Fix: run the high-"Other" diagnosis from §7. Read every "Other" free-text response from the last 30 days and group them by theme. Those themes are your missing reason options. Add them.

---

**B5. Cancel sets `cancel_at_period_end: true`, not immediate cancel**

- [ ] Your cancel flow calls Stripe with `cancel_at_period_end: true`. Customers keep access until the end of their paid period. They do not lose access the moment they click cancel.

Check it: cancel a test subscription and inspect the Stripe object. If `canceled_at` is in the past and access is cut immediately, fail.

Fix: update your Stripe cancel call. One parameter change. Immediate cancel on click is user-hostile and legally questionable under California ARL.

---

**B6. LTV-aware offer matrix in place**

- [ ] Your save offers are differentiated by both cancel reason AND customer LTV band. You are not offering a flat discount to everyone regardless of how much they pay.

Check it: for two customers, one on your $15/mo plan and one on your $150/mo plan, who both cancel with the reason "too expensive": do they see different offers? If the answer is the same discount percentage for both, fail.

Fix: build the 2D offer matrix from §8. Rows are reason clusters; columns are LTV bands (use plan price as proxy). Each cell maps to an offer type and amount. High-LTV rows warrant more aggressive offers.

*Unchurn ships an LTV-aware offer matrix configured by the merchant.*

---

**B7. Pause is the default save for usage-related cancels**

- [ ] For cancel reasons in the "not using it," "too busy," or "taking a break" cluster, your first save offer is a pause, not a discount.

Check it: go through your cancel flow and select the reason closest to "I'm not using it enough." What is the first save offer shown? If it is a discount, fail.

Fix: route usage-related reasons to pause before any discount. Pausers reactivate at 60-80%. A discount on a usage problem just delays the churn. See §8 for the Stripe `pause_collection` call.

*Unchurn implements pause-first saves as the default routing for usage-related reason codes.*

---

**B8. Repeat-cancelers cannot receive the same offer twice**

- [ ] A customer who accepted a discount at their last cancellation and returns to cancel again is blocked from seeing the same offer.

Check it: does your system record which offers a customer has accepted? Is there logic to exclude them from being shown the same offer on a repeat cancel?

Fix: store accepted offers keyed to customer ID. Add an exclusion check before surfacing each offer type. Below 10 cancels per month you can manage this manually. Above that, it becomes expensive without tooling.

*Unchurn enforces offer-deduplication rules per customer out of the box.*

---

**Part B score: \_\_ / 8**

---

### Part C: Save offers and dunning

---

**C1. Save rate tracked per offer type**

- [ ] You have a save rate metric broken out by offer type: pause, discount, plan-change, commitment. You are not tracking only an aggregate "save rate" across all offers.

Check it: can you tell whether your pause offer has a higher save rate than your discount offer? If not, fail.

Fix: add an `offer_type` column to your cancel event log. Compute save rate as accepted-and-retained / shown, per offer type. Without this split, you cannot tune the matrix from §8.

---

**C2. 90-day save-cohort LTV tracked**

- [ ] For customers saved in the last 6 months, you are tracking their actual revenue collected in the 90 days after the save and comparing it to a cohort of non-saved customers with similar plan values.

Check it: can you pull the average 90-day LTV for saved customers versus the baseline cohort? If no, fail.

Fix: add a `saved_at` timestamp when a save offer is accepted. Weekly: for customers saved 90+ days ago, sum MRR payments since `saved_at` and compare to a baseline cohort of similar customers who never entered the cancel flow. A save that recoups 30 days of MRR then churns is deferred churn. See §11 for the cohort query.

---

**C3. Stripe Smart Retries enabled**

- [ ] You are using Stripe Smart Retries for failed payments (Stripe's ML-based retry timing), not a fixed retry schedule or no retries at all.

Check it: go to Stripe Dashboard > Settings > Billing > Retry logic. Confirm "Smart Retries" is selected.

Fix: enable it in 30 seconds. Smart Retries uses network-wide payment data to time retries for maximum recovery. Zero engineering cost.

---

**C4. Automatic Card Updater enabled**

- [ ] Stripe Automatic Card Updater is enabled, so when a customer's card is reissued or replaced, the new card details are pulled automatically before a payment attempt fails.

Check it: Stripe Dashboard > Settings > Billing > Card Updates. Confirm it is enabled.

Fix: enable it. Free, no engineering. Reduces involuntary churn from stale card data.

---

**C5. Dunning cadence: day 0 / 3 / 7 / 10 / 14**

- [ ] At least four dunning emails send at those intervals after a first payment failure.

Check it: count your dunning sends and their spacing. One email at day 3 is the most common failure mode.

Fix: build the sequence in §10. Day 0 is the highest-value send. The customer is likely still at their desk.

---

**C6. Dunning copy is neutral**

- [ ] No dunning email contains phrases like "your payment failed," "unable to charge," or "your account will be suspended."

Check it: read your dunning emails out loud. Blame, shame, or urgency-as-threat is a fail.

Fix: "We had trouble processing your payment. Update your card to keep access." The customer usually does not know the card was declined. See §10 for templates.

---

**Part C score: \_\_ / 6**

---

### Part D: Operational hygiene

---

**D1. Founder reading cancel reasons weekly**

- [ ] You read every cancel reason from the past week, every Monday.

Check it: when did you last read your cancel reasons verbatim? More than two weeks ago is a fail.

Fix: block 15 minutes every Monday. Read the raw text, not a pie chart. Product and pricing decisions come from verbatim reasons.

---

**D2. Monday cancel-volume baseline check**

- [ ] You check cancel volume against last week's baseline every Monday and have a threshold above which you investigate (e.g., 25% spike week-over-week).

Check it: do you know your typical weekly cancel volume? Do you know last week's number? If no to either, fail.

Fix: export weekly cancel counts and track them in a spreadsheet. Set a spike threshold (e.g., 25% week-over-week). Any week above it is an investigation trigger. A volume spike combined with a reason shift usually means a product regression, a competitor move, or a pricing change.

---

**D3. Cancel flow A/B tests run only above 200 sessions per variant**

- [ ] Before drawing conclusions from a cancel flow test, you wait for at least 200 sessions in each variant.

Check it: how many cancels per month do you have? If fewer than 200, you should not be A/B testing your cancel flow. The variance in save rates at low volume is too high to distinguish signal from noise.

Fix: below 50 cancels per month, make directional changes from qualitative reason data instead. Read the reasons; iterate on copy and offer logic from what customers say.

---

**Part D score: \_\_ / 3**

---

### Scoring

| Score | Interpretation |
|---|---|
| 0-11 | Foundation missing. Start with A1-A4 and B1-B5 before anything else. |
| 12-17 | Operational gaps. The measurement foundation is in place but the cancel flow or dunning has holes that are losing recoverable revenue. |
| 18-23 | At the 10/10 bar. The gaps are in cohort analysis and save-cohort LTV, not in fundamentals. |

---

### Where to go from here

If your audit reveals that Part B is broken (branching reason capture missing, no LTV-aware offers, no pause-first routing, no offer deduplication), the fastest paths to fix it are:

Build it yourself using the recipe in §6 + §7 + §8 + §10 (estimate: 3-4 weeks of focused engineering). The skill above is your spec.

[Unchurn](https://unchurn.dev) at $49/mo ships Part B and most of Part C out of the box: branching reason capture, pause-first saves, LTV-aware offer matrix, FTC-compliant cancel entry point, and offer deduplication rules. Best fit at $5-60K MRR Stripe SaaS. Installs via script tag or npm package.

[Churnkey](https://churnkey.co) Core tier at $250/mo. The wider retention platform for teams past $60K MRR that want a full suite covering cancel flows, dunning campaigns, and churn analytics under one roof.

For Parts A and D, there is no shortcut. Those are your data work and your operational cadence. Carl Gold's "Fighting Churn with Data" (Manning) is the canonical reference for the measurement and behavioral metrics foundation. The techniques in that book are front-loaded: getting A1 through A4 right delivers roughly half the total benefit of a mature churn-reduction program.
