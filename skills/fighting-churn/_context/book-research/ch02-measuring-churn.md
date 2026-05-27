# Chapter 2: Measuring Churn — Research Notes

Source: "Fighting Churn with Data" by Carl Gold (MEAP V08), lines 2779-5006.

---

## 1. Core Definition of Churn Rate

**Formula (Equation 2.1):**

```
churn_rate = # Churned Customers / # Start Customers
```

The denominator is always the subscriber count (or MRR) at the **start** of the measurement period. Never the end. Never an average of start and end. The acquisition of new subscribers during the period is explicitly excluded from the formula.

**Retention rate (Equation 2.3):**

```
retention_rate = # Retained Customers / # Start Customers
```

And by definition: `churn_rate + retention_rate = 100%`

The time window is flexible (month, quarter, year) but must be stated. The formulas are identical regardless of window; only the dates change.

---

## 2. Net Retention Rate (NRR) vs. Standard Churn vs. MRR Churn

Three distinct measures. Founders typically know one (usually NRR from investor decks) and conflate the other two.

### Net Retention Rate (NRR)

**Formula (Equation 2.8):**

```
NRR = MRR of retained accounts at end / MRR of all accounts at start
```

**Net churn (Equation 2.10):**

```
net_churn = 1.0 - NRR
```

NRR is "net" because it nets upsells against churns and downsells. A customer who upgrades to a higher plan effectively cancels out some churn in the numerator. This can produce **negative churn** (NRR > 100%) when upsell MRR exceeds all losses. Gold calls this "impressive to report to investors" but warns it obscures actual churn.

**Use for:** investor reporting only. Not operationally useful for fighting churn because it conflates three separate business motions.

### Standard (Account-Based) Churn

**Formula (Equation 2.1 applied to accounts):**

```
churn_rate = n_churn / n_start
```

Where `n_churn` counts accounts that were active at the start and not active at the end. Does not count an account that cancelled one subscription but kept another as a churn. Does not include downsells.

**Use for:** fighting churn when subscribers pay roughly similar amounts. The go-to operational metric for most B2C SaaS.

### MRR Churn

**Formula (Equation 2.11):**

```
MRR_churn_rate = (MRR_churned_accounts + MRR_downsell) / MRR_start
```

Includes both complete cancellations AND the revenue lost from accounts that downgraded. Excludes upsells.

**MRR retention (Equation 2.12):**

```
MRR_retention = 1.0 - MRR_churn_rate
```

**Use for:** B2B with wide price variance (e.g. largest accounts pay 10x smallest). Annual plans switching to monthly are a trap: they register as downsells and inflate MRR churn. In those cases, revert to standard churn.

**Ordering across all three measures (consistent empirical pattern):**

```
Standard churn > MRR churn > Net churn
```

This is because large-paying accounts churn less often, so weighting by revenue lowers the rate. And net churn reduces further by counting upsells.

---

## 3. SQL Implementations

### Listing 2.1: Net MRR Retention

```sql
WITH
date_range AS (
  SELECT '2020-03-01'::date AS start_date, '2020-04-01'::date AS end_date
),
start_accounts AS (
  SELECT account_id, SUM(mrr) AS total_mrr
  FROM subscription s INNER JOIN date_range d
    ON s.start_date <= d.start_date
   AND (s.end_date > d.start_date OR s.end_date IS NULL)
  GROUP BY account_id
),
end_accounts AS (
  SELECT account_id, SUM(mrr) AS total_mrr
  FROM subscription s INNER JOIN date_range d
    ON s.start_date <= d.end_date
   AND (s.end_date > d.end_date OR s.end_date IS NULL)
  GROUP BY account_id
),
retained_accounts AS (
  SELECT s.account_id, SUM(e.total_mrr) AS total_mrr
  FROM start_accounts s
  INNER JOIN end_accounts e ON s.account_id = e.account_id
  GROUP BY s.account_id
),
start_mrr AS (SELECT SUM(start_accounts.total_mrr) AS start_mrr FROM start_accounts),
retain_mrr AS (SELECT SUM(retained_accounts.total_mrr) AS retain_mrr FROM retained_accounts)
SELECT
  retain_mrr / start_mrr AS net_mrr_retention_rate,
  1.0 - retain_mrr / start_mrr AS net_mrr_churn_rate,
  start_mrr,
  retain_mrr
FROM start_mrr, retain_mrr
```

Key detail: a subscription is "active" on a date when `start_date <= that_date AND (end_date > that_date OR end_date IS NULL)`. This handles both termed and evergreen subscriptions.

### Listing 2.2: Standard Account-Based Churn (uses LEFT OUTER JOIN to find churns)

```sql
WITH
date_range AS (
  SELECT '2020-03-01'::date AS start_date, '2020-04-01'::date AS end_date
),
start_accounts AS (
  SELECT DISTINCT account_id
  FROM subscription s INNER JOIN date_range d
    ON s.start_date <= d.start_date
   AND (s.end_date > d.start_date OR s.end_date IS NULL)
),
end_accounts AS (
  SELECT DISTINCT account_id
  FROM subscription s INNER JOIN date_range d
    ON s.start_date <= d.end_date
   AND (s.end_date > d.end_date OR s.end_date IS NULL)
),
churned_accounts AS (
  SELECT s.account_id
  FROM start_accounts s
  LEFT OUTER JOIN end_accounts e ON s.account_id = e.account_id
  WHERE e.account_id IS NULL
),
start_count AS (SELECT COUNT(*) AS n_start FROM start_accounts),
churn_count AS (SELECT COUNT(*) AS n_churn FROM churned_accounts)
SELECT
  n_churn::float / n_start::float AS churn_rate,
  1.0 - n_churn::float / n_start::float AS retention_rate,
  n_start,
  n_churn
FROM start_count, churn_count
```

`DISTINCT` is required because one account can hold multiple subscriptions. The outer join pattern is the canonical way to find "what was present at start but not at end."

### Listing 2.4: MRR Churn (adds downsell CTE)

Key addition over Listing 2.2:

```sql
downsell_accounts AS (
  SELECT s.account_id, s.total_mrr - e.total_mrr AS downsell_amount
  FROM start_accounts s
  INNER JOIN end_accounts e ON s.account_id = e.account_id
  WHERE e.total_mrr < s.total_mrr
),
downsell_mrr AS (
  SELECT COALESCE(SUM(downsell_accounts.downsell_amount), 0.0) AS downsell_mrr
  FROM downsell_accounts
)
SELECT
  (churn_mrr + downsell_mrr) / start_mrr AS mrr_churn_rate,
  ...
```

`COALESCE(..., 0.0)` guards against periods with zero downsells (would produce NULL otherwise).

---

## 4. Activity-Based (Event-Based) Churn

For products without subscriptions (or to supplement subscription data), churn is defined by recency of activity.

**Definition:** A customer is "active" if they had at least one event within a trailing window ending at the measurement date. The window is typically one or two months.

### Listing 2.3: Event-Based Churn

```sql
WITH
date_range AS (
  SELECT '2020-03-01'::TIMESTAMP AS start_date,
         '2020-04-01'::TIMESTAMP AS end_date,
         interval '1 months' AS inactivity_interval
),
start_accounts AS (
  SELECT DISTINCT account_id
  FROM event e INNER JOIN date_range d
    ON e.event_time > start_date - inactivity_interval
   AND e.event_time <= start_date
),
end_accounts AS (
  SELECT DISTINCT account_id
  FROM event e INNER JOIN date_range d
    ON e.event_time > end_date - inactivity_interval
   AND e.event_time <= end_date
),
churned_accounts AS (
  SELECT DISTINCT s.account_id
  FROM start_accounts s
  LEFT OUTER JOIN end_accounts e ON s.account_id = e.account_id
  WHERE e.account_id IS NULL
),
...
```

The logic is identical to subscription churn except the "active" test uses event recency instead of subscription dates. The churn rate formula is unchanged.

**Important caveat:** with event-based churn you cannot know a customer has churned until the inactivity window has elapsed. With subscriptions, you know on the day after the subscription ends. This lag must be accounted for in real-time dashboards.

---

## 5. Monthly to Annual Conversion Math

**Annual churn is not 12x monthly churn.** This is the most common error.

The correct relationship uses survivor analysis. If monthly retention is `r = (1 - c)`, then after 12 months the fraction of the original pool still retained is `r^12`.

**Annual retention from monthly (Equation 2.14):**

```
R = r^12 = (1 - c)^12
```

**Annual churn from monthly (Equation 2.15):**

```
C = 1 - (1 - c)^12
```

Example: 5% monthly churn. `c = 0.05`, `r = 0.95`. Annual churn = `1 - 0.95^12 = 1 - 0.540 = 46%`. Not 60%.

**Monthly churn from annual (Equation 2.17):**

```
c = 1 - (1 - C)^(1/12)
```

**General conversion for any period p days (Equation 2.18 and 2.19):**

```sql
-- Annual churn from a p-day measurement c':
annual_churn = 1.0 - POWER(1.0 - c', 365.0 / p)

-- Monthly churn from a p-day measurement c':
monthly_churn = 1.0 - POWER(1.0 - c', (365.0/12.0) / p)
```

This is the exact SQL from Listing 2.5. It lets a company with only 90 days of data produce a defensible annualized churn number.

---

## 6. Common Measurement Mistakes

1. **Wrong denominator.** Dividing churns by end-of-period count or by average count mixes acquisition signal into churn signal. Always use start-of-period count.

2. **Annual = 12x monthly.** Always wrong due to compounding. Use the power formula.

3. **Measuring on the wrong time scale.** Annual churn on monthly subscriptions misses accounts that start and churn entirely within the year. Monthly churn on annual subscriptions can be wildly distorted if you happen to pick a renewal-heavy month.

4. **Ignoring seasonality.** Monthly churn rates move up and down through the year. You cannot tell if a change is caused by a churn-reduction initiative or a seasonal pattern unless you compare to the same month one year prior. Need minimum two years of history to detect seasonal patterns.

5. **Reporting NRR to the team.** Net retention hides actual churn by mixing in upsell revenue. A team using NRR as its operational metric can have accelerating churn masked by an expanding upsell motion.

6. **Using MRR churn when annual plans are present.** Switching from monthly to annual billing registers as a downsell and inflates MRR churn artificially. Use standard churn in that scenario.

---

## 7. Picking the Measurement Window

- Match the window to the typical subscription renewal term.
- Consumer/B2C: measure monthly.
- B2B: measure annually.
- If less than a year of data exists, measure whatever is available and convert using the power formula (Equation 2.18).
- To control for seasonality with monthly windows: compare each month's rate to the same calendar month one year earlier. This requires only 13 months of data.

---

## 8. Subscription Data Model Requirements

Minimum required columns for any of these calculations:

| Column | Type | Required |
|---|---|---|
| subscription_id | integer or char | Yes |
| account_id | integer or char | Yes |
| product_id | integer or char | Yes |
| start_date | date | Yes |
| end_date | date | No (null = evergreen) |
| mrr | double precision | Yes for MRR churn |

A subscription is active on a given date when: `start_date <= that_date AND (end_date > that_date OR end_date IS NULL)`.

Gold's repeated emphasis: real subscription databases are messy. Duplicate rows, end dates before start dates, inconsistent terms. The SQL patterns in the book are designed to survive this.

---

## 9. Summary Decision Tree

| Situation | Metric to use |
|---|---|
| Reporting to investors | Net Retention Rate |
| All customers pay same price | Standard churn (or NRR, equivalent) |
| Modest pricing variation, some discounts | Standard churn |
| Wide price range (B2B, 2x+ spread) | MRR churn |
| Annual plans that can be compared to monthly | Standard churn (avoid MRR churn) |
| No subscriptions, only events | Activity-based churn |
