## 3. Measure churn correctly

Most founders are computing churn wrong. Not imprecisely wrong. Structurally wrong. The number they track, the benchmark they compare it against, and the metric they report to investors are three different things computed three different ways, and they treat them as interchangeable. This is the section that fixes that.

---

### The denominator is everything

The formula for churn rate is:

```
churn_rate = churned_accounts / start_accounts
```

The denominator is accounts active at the **start** of the period. Not the end. Not an average of start and end.

This is not a stylistic preference. Using end-of-period count as the denominator mixes acquisition signal into a retention metric. If you signed 50 new customers in March and lost 10 existing ones, dividing by the larger end-of-month count makes churn look better than it is. The new customers did nothing to reduce cancels. They just diluted the rate. That is not a measurement. It is noise.

Using the start count is the only coherent definition: of the accounts that were actually at risk of churning this period, what fraction did?

Every churn formula below uses start-of-period as the denominator. Lock this in before you run any numbers.

---

### Three metrics, three different purposes

There are three measures of churn. They answer different questions, they rank predictably relative to each other, and conflating them is one of the most common ways SaaS founders mislead themselves.

**Standard (account-based) churn** counts cancels as accounts, ignoring revenue:

```
churn_rate = n_churn / n_start
```

**MRR churn** weights by revenue and includes downgrades:

```
mrr_churn_rate = (MRR_churned + MRR_downsell) / MRR_start
```

**Net Revenue Retention (NRR)** subtracts expansion revenue from losses:

```
NRR = MRR_retained_accounts_end / MRR_all_accounts_start
net_churn = 1.0 - NRR
```

These three numbers rank in a consistent empirical pattern across B2B SaaS:

```
standard churn > MRR churn > net churn
```

This is not a coincidence of the math. It reflects a real business dynamic: large accounts pay more and churn less. When you weight by revenue, the accounts most likely to cancel are under-weighted. MRR churn is therefore systematically lower than standard churn in any business where pricing varies by account size. Net churn then reduces further by netting upsell revenue against cancels, which can make accelerating cancels invisible if your expansion motion is strong.

**What this means for you:** you almost certainly have all three numbers available. Use standard churn as your operational metric. Use MRR churn as a secondary signal to understand revenue impact. Use NRR for investor reporting only. Never use NRR to tell yourself your retention is improving. It can improve while cancels are accelerating.

---

### The annual-plan trap in MRR churn

If any of your customers are on annual plans, MRR churn has a specific failure mode you need to know about.

When a monthly customer converts to an annual plan, your billing system records the change as a downsell in the month it happens, because the customer's monthly contribution drops from their recurring monthly charge to a prorated fraction of their annual payment. That registers as lost MRR in the `MRR_downsell` bucket.

It is not lost MRR. It is a retention win. But MRR churn counts it as a loss.

The fix: when annual plans are in the mix, revert to standard account-based churn. Count whether the customer kept a subscription, not whether their monthly billing figure went up or down. MRR churn is reliable when your pricing structure is stable across periods. It breaks when billing frequency changes.

---

### Account-based vs. activity-based churn

For subscription products, account-based churn (did they cancel?) is the right primary metric. Stripe tells you exactly when a subscription ends.

Activity-based churn applies when you need to answer a different question: are customers still getting value from the product, regardless of whether they have cancelled? This matters for freemium products, usage-based plans, and seat-based tools where someone can stay subscribed while the team stops using it entirely.

The definition: a customer is "active" if they had at least one qualifying event within a trailing window ending at the measurement date. Common windows are 30 and 60 days.

```sql
-- Activity-based active accounts at a point in time
WITH date_range AS (
  SELECT
    '2024-03-01'::timestamp AS start_date,
    '2024-04-01'::timestamp AS end_date,
    interval '30 days'    AS inactivity_window
),
start_accounts AS (
  SELECT DISTINCT account_id
  FROM events e
  CROSS JOIN date_range d
  WHERE e.event_time > d.start_date - d.inactivity_window
    AND e.event_time <= d.start_date
),
end_accounts AS (
  SELECT DISTINCT account_id
  FROM events e
  CROSS JOIN date_range d
  WHERE e.event_time > d.end_date - d.inactivity_window
    AND e.event_time <= d.end_date
),
churned_accounts AS (
  SELECT s.account_id
  FROM start_accounts s
  LEFT JOIN end_accounts e ON s.account_id = e.account_id
  WHERE e.account_id IS NULL
),
counts AS (
  SELECT
    (SELECT COUNT(*) FROM start_accounts) AS n_start,
    (SELECT COUNT(*) FROM churned_accounts) AS n_churn
)
SELECT
  n_churn::float / n_start::float AS activity_churn_rate,
  n_start,
  n_churn
FROM counts;
```

One important caveat: with activity-based churn, you cannot declare a customer churned until the inactivity window has elapsed. A customer who last logged in on March 28 is still "active" in a 30-day window until April 27. Subscription churn is known the day after the subscription ends. Budget for this lag in any real-time reporting.

For a $5-60K MRR Stripe SaaS with paying subscribers, account-based churn is your operational north star. Activity-based churn is a leading indicator worth tracking alongside it, because customers who stop using the product before they cancel will show up in activity churn first.

---

### Standard account-based churn in SQL

This is the pattern you should run every month. Swap the dates. Store the output.

```sql
WITH date_range AS (
  SELECT
    '2024-03-01'::date AS start_date,
    '2024-04-01'::date AS end_date
),
start_accounts AS (
  SELECT DISTINCT account_id
  FROM subscriptions s
  CROSS JOIN date_range d
  WHERE s.start_date <= d.start_date
    AND (s.end_date > d.start_date OR s.end_date IS NULL)
),
end_accounts AS (
  SELECT DISTINCT account_id
  FROM subscriptions s
  CROSS JOIN date_range d
  WHERE s.start_date <= d.end_date
    AND (s.end_date > d.end_date OR s.end_date IS NULL)
),
churned_accounts AS (
  SELECT s.account_id
  FROM start_accounts s
  LEFT JOIN end_accounts e ON s.account_id = e.account_id
  WHERE e.account_id IS NULL
),
counts AS (
  SELECT
    (SELECT COUNT(*) FROM start_accounts)  AS n_start,
    (SELECT COUNT(*) FROM churned_accounts) AS n_churn
)
SELECT
  n_churn::float / n_start::float         AS churn_rate,
  1.0 - n_churn::float / n_start::float   AS retention_rate,
  n_start,
  n_churn
FROM counts;
```

The `DISTINCT` on `account_id` is required because one customer can hold multiple subscriptions. The LEFT JOIN pattern is the canonical way to find "present at start, missing at end." A subscription is active on a given date when `start_date <= that_date AND (end_date > that_date OR end_date IS NULL)`. This handles both fixed-term and evergreen subscriptions.

For MRR churn, add the revenue weight and a downsell CTE:

```sql
-- Extend the pattern above: replace start_accounts and end_accounts
-- with MRR-summed versions, then add:
downsell_accounts AS (
  SELECT
    s.account_id,
    s.total_mrr - e.total_mrr AS downsell_amount
  FROM start_accounts_mrr s
  INNER JOIN end_accounts_mrr e ON s.account_id = e.account_id
  WHERE e.total_mrr < s.total_mrr
),
downsell_mrr AS (
  SELECT COALESCE(SUM(downsell_amount), 0.0) AS downsell_mrr
  FROM downsell_accounts
)
SELECT
  (churn_mrr + downsell_mrr) / start_mrr AS mrr_churn_rate,
  start_mrr,
  churn_mrr,
  downsell_mrr
FROM start_mrr_cte, churn_mrr_cte, downsell_mrr;
```

The `COALESCE(..., 0.0)` guard is not optional. In months with zero downsells, the SUM returns NULL and the division produces NULL, silently dropping your result.

---

### Monthly to annual: the conversion table founders skip

Quoting a monthly churn rate without the annual equivalent is how you avoid a hard truth. Annual churn is not 12 times monthly churn. Subscriptions compound. The correct formula is:

```
annual_churn = 1 - (1 - monthly_churn)^12
```

| Monthly churn | Annual churn |
|---------------|--------------|
| 1%            | 11.4%        |
| 2%            | 21.5%        |
| 3%            | 30.6%        |
| 5%            | 46.0%        |
| 7%            | 57.8%        |
| 10%           | 71.8%        |

A 5% monthly churn rate means you replace nearly half your customer base every year. Founders who report "only 5% monthly churn" to investors who are thinking in annual terms are presenting a number that is roughly 2.5x more flattering than reality. Make sure you know which frame you are in before any conversation where this number matters.

To convert in the other direction (annual to monthly):

```
monthly_churn = 1 - (1 - annual_churn)^(1/12)
```

If you only have 90 days of data and need an annualized figure, the general form is:

```sql
SELECT 1.0 - POWER(1.0 - churn_rate_over_period, 365.0 / period_days)
  AS annualized_churn_rate;
```

---

### Measurement interval guidance

Measure monthly for active monitoring. Monthly gives you enough data points to detect whether interventions are working, without waiting a quarter to learn you moved in the wrong direction.

Measure quarterly for trend analysis. Monthly rates bounce around. A three-month rolling average smooths noise without hiding signal.

Never average churn rates across cohorts. If January had 4% churn and February had 6%, the combined rate is not 5%. Compute the pooled rate from the underlying counts: total churns divided by total start accounts across both months. Averaging rates of rates produces a number that corresponds to no actual business reality.

The "industry benchmarks" you read about in SaaS newsletters are almost never computed against the same denominator, time window, or account definition you are using. A B2B SaaS benchmark built on annual contracts compared against your monthly subscription churn is not a benchmark. It is a different measurement. Spend less time benchmarking and more time comparing your own number, computed consistently, month over month.

---

> **Common measurement mistakes**
>
> - **Wrong denominator.** Dividing by end-of-period count or an average mixes new acquisition into a retention signal. Use start-of-period count, always.
> - **NRR reported as churn.** Net Revenue Retention hides actual cancels behind expansion revenue. A company growing through upsells can have accelerating churn and improving NRR at the same time. NRR belongs in investor decks, not operational dashboards.
> - **Annual-plan downsell inflation.** Converting monthly customers to annual registers as a downsell in MRR churn. When annual plans are present, use account-based churn instead.
> - **Averaging rates across cohorts.** "Our average monthly churn is X%" computed by averaging monthly rates is mathematically wrong. Sum the raw counts and divide once.
> - **12x monthly = annual.** This underestimates annual churn significantly due to compounding. Use `1 - (1 - monthly)^12`.
