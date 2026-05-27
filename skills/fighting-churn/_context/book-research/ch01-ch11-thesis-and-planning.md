# Fighting Churn with Data: Ch 1 + Ch 11 Research Notes

Source: Carl Gold, MEAP V08 (Manning). Gold is principal author of Zuora's Subscription Economy Index.

---

## 1. Central Thesis

The book's argument, stated plainly: the data team's main deliverable is not a churn prediction model. It is a **set of well-designed customer metrics** that the whole business can act on.

Key claim: "Predicting churn is not the focus of fighting churn with data. This is one of the most important lessons I had to learn when I started working in this area."

Why prediction alone fails:
- A churn-risk score tells you who might leave, not why. Without why, you cannot target the intervention correctly.
- Churn is rare relative to retention, so false-positive rates are high even in good models.
- Timing of churn is shaped by extraneous factors (busyness, procrastination, competitor pricing) that no model captures.
- There is no one-size-fits-all intervention. You cannot just act on a score.

The replacement: build behavioral metrics from your event data warehouse, run metric cohort analysis, and deliver actionable health benchmarks to product, marketing, and CS teams.

---

## 2. Author Voice and Framing

Gold is a practitioner, not an academic. His tone:
- Direct, self-aware ("I'm going to under-promise and over-deliver")
- Uses "you" throughout, addresses a technical reader embedded inside a business
- Honest about limits ("No silver bullets") before showing what actually works
- Frequently names things non-technical people can use without needing jargon
- Distinguishes "the data person" from "the business people" and positions both as necessary

He is comfortable with quotes from Ben Horowitz ("There are no silver bullets for this, only lead bullets") to make points about difficulty.

Voice principle for the skill: speak like a practitioner to a founder who is also the data person. Do not oversell prediction. Acknowledge the hard parts first.

---

## 3. Contrarian / Surprising Claims

- **Churn prediction is nearly useless as a primary strategy.** This will surprise founders who have seen "build a churn prediction model" as standard advice.
- **Price reduction is a "diamond bullet": it always works but you can't afford it.** Down-sells are still churn by most metrics.
- **Customers who pay more churn less** (especially B2B). Not because loyalty, but because bigger customers use the product more. The metric to watch is unit cost: MRR divided by usage. High cost-per-unit = high churn risk.
- **More detractors is not always bad** without normalization. A raw detractor count correlates negatively with churn because high-review-volume customers get more bad reviews AND more good ones. The correct metric is detractor rate (detractors / total reviews). Raw counts mislead; rates reveal.
- **Discounts do not save churners** (Klipfolio finding): customers who receive discounts tend to churn anyway once the discount expires.
- **License utilization beats active user count.** Active users conflates seat count with engagement. Utilization (active users / licensed seats) is independent of size and actually predicts churn across all cohorts.
- **Half the total benefit comes from just getting churn rate + metrics right** (Ch 11 estimate). Advanced ML adds maybe 10% on top of good metrics.

---

## 4. "Where to Start" Advice from Chapter 11

Gold's explicit framework is three tiers:

**Foundation (delivers ~50% of total benefit):**
1. Calculate churn rate correctly (B2C, B2B, or activity-based, depending on product type)
2. QA your event data, validate daily event counts per account
3. Create standard behavioral metrics from events
4. QA the metrics, agree with the business on time windows
5. Deliver a current customer list with metrics

**Advanced (delivers ~65-70% of total benefit):**
6. Build the analytic dataset with lead time
7. Run metric cohort analyses, plot each metric vs. churn rate
8. Behavioral grouping, correlation analysis
9. Create advanced metrics: utilization, success rates, unit cost
10. Deliver current customer list with advanced metrics

**Extreme / forecasting (delivers ~90% of total benefit):**
- Logistic regression or XGBoost churn forecasts
- Only worth it if you already have a mature analytics practice

**Key quote:** "It's better to start small and deliver something than to try everything and not deliver anything. The benefit of the techniques in the book is front-loaded."

**Communication checklist (Ch 11.1.3):**
- Present churn rate with calculation method. Get business agreement.
- Show daily event count QA plots. Confirm the events represent the business correctly.
- Agree on simple health criteria (e.g., "healthy = above average on the major metrics").
- Deliver the customer list. Review high, typical, and low usage samples together.
- Most important outcome: get business people looking at and using metrics. Not the model.

---

## 5. Quotable Lines and Stats

- "There are no silver bullets to reduce churn."
- "Price reduction is a 'diamond bullet' against churn: it always works, but you can't afford it."
- "The main deliverable to the business from the data analysis project is a set of customer metrics."
- "Churn is effectively a survey of all users, who 'vote with their feet'." (Ch 11)
- Klipfolio: customers with the lowest license utilization churn at 7x the rate of highest-utilization customers.
- Broadly: detractor rate (proportion of bad reviews) predicts churn clearly. Raw detractor count does the opposite.
- Versature: customers in the highest cost-per-call cohort churn at 6x the rate of customers in the lowest.
- "A typical company can get almost half of the benefit from using data to reduce churn if you can just make it through chapter 3."
- "The most important thing is just to get the business people thinking about the metrics and how to improve customer engagement."

---

## 6. The Three Metrics That Work (Ch 1.8 Case Studies)

Gold names three metric types that consistently surface in churn analysis across company types:

1. **Utilization:** Usage as a fraction of what the customer purchased or is allowed. Not raw usage. (Klipfolio: active users / licensed seats)
2. **Success rate:** Proportion of attempts that result in the desired outcome. Rates beat counts. (Broadly: detractor rate vs. raw detractor count)
3. **Unit cost:** MRR divided by usage volume. High cost per unit = poor value = churn risk. (Versature: MRR / calls per month)

These three pattern types apply across nearly all SaaS products.

---

## 7. ICP Relevance Notes (for Unchurn skill application)

Gold's case studies (Klipfolio, Broadly, Versature) are all $5-60K MRR-range B2B SaaS. The techniques are designed for exactly this tier.

Practical minimum requirement to use the methods: "Tracking user events across multiple sessions is the one minimum requirement." Stripe-native SaaS already has subscription data. The gap is usually event data from the product.

Gold's five churn-reduction strategies (product improvement, engagement campaigns, customer success, right-sizing pricing, channel targeting) all have data equivalents. The Unchurn cancel-flow intervention maps most directly to his "right-sizing pricing" and "customer success" categories: the cancel moment is a save tactic, not a root cause fix. Gold's framework makes clear the root cause work (metrics, cohorts) should run in parallel.
