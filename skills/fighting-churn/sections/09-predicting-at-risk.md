## 9. Predicting at-risk customers

Most founders are pitched ML-based churn prediction within a year of launching. Carl Gold, who spent years building these systems, has a sobering take: pure churn prediction is overrated. The behavioral metrics themselves, correctly measured and grouped as in §4 and §5, are what product and CS teams can actually act on. A regression model trained on those same metrics does not change what you do, it just re-sorts the queue. At small scale, that sorting is not worth the pipeline cost.

Scale changes the economics. Here is where each approach starts to pay back.

### The three-tier scale ladder

| Tier | Subscriber count | History needed | Recommended approach |
|------|-----------------|----------------|----------------------|
| 1 | Under 200 | Any | Manual: read cancel reasons weekly, watch cohort scores, call bottom decile |
| 2 | 200-1,000 | 3+ months | Weighted score built from §5 group metrics |
| 3 | 1,000+ | 6+ months | Logistic regression with L1 penalty (Carl Gold's pipeline) |

---

### Tier 1: under 200 subscribers

Do not build a model. The data is too thin, and the noise in any weekly slice is high enough to generate false positives that waste your time and miss real churn.

What works instead:

- Every Friday, read the last 30 cancel-reason responses from your cancel flow (§4). Look for a new pattern, a specific complaint that appeared more than twice, or a competitor mention you have not heard before. This takes 20 minutes.
- Watch the §5 cohort scores. If your "usage frequency" group score has slid two standard deviations in the past month, that is a product signal, not a customer-by-customer problem.
- Pull the bottom-decile accounts by engagement each week and email them personally. One sentence: what you noticed, one question. Not a template.

Personal outreach at this scale outperforms any automated system. Your sample is small enough that a single saved customer materially changes the numbers.

---

### Tier 2: 200-1,000 subscribers

A weighted risk score makes sense here. No historical dataset, no Python pipeline. Just the grouped metric scores from §5.

**Worked formula.** The §5 pipeline produces a score for each metric group. Common groups for SaaS products are: feature engagement, login frequency, support activity, and billing attention. Assign weights that reflect how strongly each group correlates with churn in your product. As a starting point before you have correlation data:

```
risk_score = 100
  - (0.35 × feature_engagement_score)
  - (0.25 × login_frequency_score)
  - (0.20 × billing_attention_score)
  - (0.20 × support_health_score)
```

Each group score is a normalized 0-100 value from §5 (higher = healthier). The risk score starts at 100 and subtracts each group's contribution. A customer who is engaging heavily, logging in regularly, has not visited billing, and has no recent frustrated tickets scores near zero risk. A customer who has dropped feature usage, dropped logins, visited billing twice this week, and opened a complaint ticket scores near 80-100.

After three to four months, compare the group scores of churned customers in the two weeks before cancel against those who stayed. The groups with the largest pre-cancel drops are the ones whose weights to increase. This is manual calibration, not regression, and it is sufficient for 200-1,000 subscribers. Recalibrate quarterly; keep the weights in a config file, not hardcoded.

---

### Tier 3: 1,000+ subscribers with 6+ months of history

Now regression pays back. The full pipeline is described in Carl Gold's book (chapters 4-9), but the outline is:

1. Build a historical observation dataset: one row per customer per period, labeled with whether they churned that period.
2. Score each behavioral metric: log-transform for skew, clip at the 1st and 99th percentile, then z-score.
3. Group correlated metrics into composite scores (the §5 approach). Do not feed correlated raw metrics into regression; it breaks the weight interpretability.
4. Fit logistic regression with an L1 penalty (`sklearn.LogisticRegression`, solver `liblinear`). L1 zeroes out weak predictors automatically.
5. Validate with time-series backtesting (`sklearn.TimeSeriesSplit`). Earlier periods train; later periods test. Do not shuffle the data.

**Accuracy benchmarks to expect:**

| Metric | Healthy range | Red flag |
|--------|--------------|----------|
| AUC (Area Under ROC Curve) | 0.60-0.80 | Below 0.55 (barely better than random) or above 0.85 (likely a data leak) |
| Top-decile lift | 2.0-5.0x | Below 1.5x (model not useful for targeting) |

XGBoost typically achieves AUC 0.02-0.06 higher and top-decile lift 0.1-0.5 better than logistic regression. Real, but not transformative. It also takes roughly 40 times longer to train. More importantly: XGBoost does not produce calibrated probabilities. Do not use it for CLV calculations. Use logistic regression for anything that feeds a dollar estimate.

**CLV formula from the regression output:**

```
monthly_churn_prob = 1 - retention_probability  (from logistic regression)
expected_lifetime_months = 1 / monthly_churn_prob
CLV = (gross_margin × MRR) / monthly_churn_prob
```

Each subscriber gets their own churn probability and therefore their own CLV estimate. A customer at $200 MRR with 3% monthly churn probability has a CLV of roughly $3,000-$4,000 (at 50% margin). The same $200 MRR customer at 15% monthly churn has CLV under $700. That difference is worth knowing before you decide how much time to spend on an outreach call.

For products with annual churn below 20%, apply a discount factor for long-horizon non-churn risks (Gupta and Lehman 2003).

---

### The billing-page signal applies at every tier

Regardless of which tier you are in, instrument this one signal immediately. When a customer visits your Stripe billing portal or your in-app billing page, log the customer ID and timestamp. A billing-page visit from a customer with otherwise declining engagement is the highest-reliability leading indicator available, and it costs nothing to capture.

```js
// Next.js route handler: log every portal session creation
const session = await stripe.billingPortal.sessions.create({ customer: customerId, return_url: returnUrl });
await db.insert(billingPageVisits).values({ customerId, visitedAt: new Date() });
```

Any customer who has visited billing twice in seven days while their feature engagement score is falling is a same-day outreach target. No regression model needed.

---

### Risk score to action

This table applies to Tier 2 and Tier 3. Tier 1 founders should skip the score and act on the pattern read directly.

| Score range | Status | Action |
|-------------|--------|--------|
| 80-100 | Healthy | Upsell window. This customer is primed to expand. |
| 60-79 | Needs attention | In-app nudge or usage tip email within three days. |
| 40-59 | At risk | Outbound email or in-app intervention within 48 hours. |
| 0-39 | Critical | Personal founder email within 24 hours, no automation. |

The 0-39 bucket should stay small. If you have 40 customers there simultaneously, you have a product problem, not a retention workflow problem.

---

### The demographics caveat

Carl Gold's data is clear: demographics are the weakest churn lever. In his simulations, demographics alone achieved AUC around 0.56 and top-decile lift around 1.5x, barely above random.

The one demographic worth tracking at small scale is acquisition channel. If customers from one channel churn at 3x the rate of customers from another, that is an acquisition problem masquerading as a retention problem. Fix the acquisition. Every other demographic cut is noise until you have thousands of subscribers per cohort.
