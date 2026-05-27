# Ch 7: Ratio Metrics

Carl Gold calls ratio metrics "the most important single technique in the book." Raw counts lie because behaviors correlate. A customer who pays more usually uses more, so high MRR looks protective. Split them apart and the signal vanishes. A ratio of MRR to calls reveals the actual unit cost, and unit cost predicts churn cleanly.

---

## 7.1 What ratio metrics are and why raw counts lie

A ratio metric divides one behavioral metric by another, matched account-by-account and date-by-date. The key insight: when two metrics are correlated, each one partially reflects the other. Separating them with a ratio isolates the relationship.

**The Versature example.** Higher MRR correlated with lower churn, but only because high-MRR customers made more calls. When Gold computed MRR per call (effective unit cost), the signal flipped: higher cost per call meant higher churn. MRR per call was nearly uncorrelated with both MRR and calls independently, making it a clean predictor.

**Ratio types from case studies:**

- **Unit cost:** MRR / usage. Customers paying a lot per unit of value are churn risks.
- **Efficiency:** downstream events / upstream events. Transactions per customer for Broadly. Higher efficiency = lower churn.
- **Success rate:** successes / attempts. Broadly's review ask acceptance rate. Counterintuitively, higher success rates raised churn because customers hit their goal (enough reviews) and left.
- **Inefficiency rate:** edits / saves for Klipfolio. High edits-per-save signals friction. More friction = more churn.

**Two gotchas in calculation:**
1. Denominator must be greater than zero. Use a CASE guard.
2. Numerator can be zero but ideally not negative.

SQL pattern: two CTEs select numerator and denominator for the same account and date range, then an INNER JOIN divides them. Accounts missing either metric get zeros in the analysis dataset.

---

## 7.2 Percent of total metrics

Percentage is a special ratio: part divided by whole. Use it when two or more metrics are subcategories of the same total activity.

**When to use percentage vs. plain ratio.** Use a percentage when the two metrics are parts of a whole (likes and dislikes). Use a plain ratio when they are unrelated activities (ads viewed vs. posts). Percentage is more interpretable in the first case.

**Multi-category use.** A streaming service with Action, Comedy, Drama, Romance views will find all four counts highly correlated. Raw counts say "active users churn less." Percentage breakdowns reveal whether skew toward a single category is good or bad. Versature's regional call breakdowns: percentage of calls by region had weak cross-correlation and added information the raw counts could not.

**Broadly detractor case.** Both promoter and detractor count metrics showed lower churn (more engagement = less churn). Detractor percentage showed strong increasing churn. Customers with the lowest detractor percentage (around 2%) had near-zero churn.

**Calculation.** Sum the category metrics into a total metric first, then run the standard ratio SQL against it.

---

## 7.3 Metrics that measure change

### Percentage change

Formula: (end metric / start metric) - 1.0. Computed on a rolling window (e.g., 4 weeks vs. 4 weeks prior).

Why percentage rather than absolute difference: absolute changes are correlated with the starting level. Two customers doubling their logins show the same percentage change despite very different absolute counts.

Percentage change metrics can be negative (activity fell). The minimum is -100% (went to zero). The maximum can be extreme: Versature had a customer at +117,150% because they started from near zero.

**Fat tails transformation.** Percentage change distributions are highly skewed and have negative values, so the standard log-score formula from Ch 5 does not apply. Gold introduces the inverse hyperbolic sine transform: `log(m + sqrt(m^2 + 1))`. Apply this when skew is high AND min is negative. Then subtract mean and divide by std as normal.

### Time since last activity

Counts days from the most recent event before the measurement date to the measurement date. Zero if the customer acted today. Increases every day of inactivity. SQL uses `MAX(event_time)` in a CTE, then subtracts from measurement date.

Klipfolio case: churn risk increased substantially over the first month of inactivity, then leveled off. Gold notes the "let sleeping dogs lie" hypothesis (inactive customers forget they have a subscription) but does not endorse it as a strategy.

---

## 7.4 Scaling metric time periods

### Measurement window vs. description period

You measure over a long window (84 days) for rare events, but you describe the result as a monthly average. Divide the raw count by the ratio of measurement period to description period: `count * (28 / 84)`. All metrics can then be described on a common scale.

Do NOT scale metrics that are averages of event properties (e.g., average purchase amount). Only counts and sums.

### Estimating metrics for new accounts

A new account with 2 weeks of data and 5 events should not be compared directly to a mature account's 28-day count. Gold's equation (Eq 7.4) handles three cases:

- Tenure below Tmin: no metric calculated.
- Tenure between Tmin and Tdescribe: scale up by `Tdescribe / tenure` (estimate forward).
- Tenure above Tdescribe: scale down by `Tdescribe / Tmeasure` (average backward).

SQL uses the `LEAST(Tmeasure, tenure)` trick in the denominator rather than a CASE branch.

**Gold's standard parameters for monthly subscriptions:**
- Tmin = 14 days
- Tdescribe = 28 days
- Tmeasure = 84 days

For annual subscriptions: Tmin = 28 days, Tdescribe = 28-84 days, Tmeasure = 365 days.

Gold notes that every case study in the book uses this scaling pattern. He never uses plain 28-day counts in production work.

---

## 7.5 User metrics

For multi-seat products, churn is still measured at the account level. Individual user inactivity is not churn.

**Active users:** count distinct user IDs in a time window. SQL replaces `COUNT(*)` with `COUNT(DISTINCT user_id)`.

Critical: do NOT apply tenure scaling to active user counts. DISTINCT is not additive. 2 active users in 2 weeks does not imply 4 in 4 weeks.

**License utilization:** active users / licensed seats. Klipfolio case showed this was a stronger churn predictor than active users alone. Continuous relationship: every cohort step up in utilization reduced churn.

**Per-user ratios:** divide any account-level behavior metric by active users to get a per-user rate. Dashboard views per user was a useful secondary predictor for Klipfolio.

---

## 7.6 Which ratios to use

**Why not interaction terms (multiplication)?** Interaction metrics have unintuitive units. "Dollar-calls" means nothing. Ratios produce interpretable units (dollars per call, transactions per customer). Gold's rule: use multiplicative interactions only when the business already has a name for the combined unit (e.g., kilowatt-hours).

**Why not score differences?** Score subtraction can approximate a ratio but is harder to explain to business stakeholders. Ratios remain the only option for broadly understandable relationship metrics.

**How to pick ratios:**
1. Only use ratios where both metrics are non-zero for most customers.
2. Do not try every possible pair. Spurious relationships appear at scale.
3. Prefer ratios that have intuitive business meaning before you run the numbers.
4. Look within a correlated group (Ch 6) for the most common metric, then try ratios across groups.

**Summary table (Table 7.2):**

| Type | Ratio | Signal |
|---|---|---|
| Unit cost | MRR / use | High cost per unit = churn risk |
| Unit value | Use / MRR | Low value per dollar = churn risk |
| Utilization | Use / allowance | Low utilization = churn risk |
| Success rate | Successes / attempts | Depends on product; sometimes more success = goal achieved = churn |
| Percent of total | Part / whole | Reveals balance across categories |
| Percentage change | (End / start) - 1 | Falling usage predicts churn |

---

## Key SQL patterns

**Ratio metric (Listing 7.1):** two CTEs for numerator and denominator, INNER JOIN on account and date, CASE guard against zero denominator.

**Total of metrics (Listing 7.3):** single query, `SUM(metric_value)` with `IN (name_list)`, grouped by account and date.

**Percentage change (Listing 7.4):** same metric twice, denominator CTE offset by change period, `end/start - 1.0`, LEFT OUTER JOIN so null end = -100%.

**Time since last event (Listing 7.6):** `MAX(event_time)` CTE, then `metric_date - last_date`.

**Scaled count for new accounts (Listing 7.8):** `(28 / LEAST(84, tenure)) * COUNT(*)`, WHERE tenure >= 14.
