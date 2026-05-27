## 4. Behavioral metrics, not vibes

Most founder dashboards are not analytics. They are event counts with a date filter. "Feature X used 847 times last month" sounds like insight, but it tells you nothing about which accounts are healthy and which are six weeks from canceling. Turning raw events into predictive signals requires one extra step that almost nobody does: aggregating events into per-account, time-windowed metrics.

This section covers that step. Get it right and the cohort analysis and churn-driver work in the next sections become straightforward. Skip it and you are guessing.

---

### The event-metric distinction

An **event** is something that happened to one account at one moment. Three fields, nothing more:

- `account_id` . who
- `event_type` . what
- `occurred_at` . when

Events are append-only. You never update them. When a customer exports a report, you insert a row. When they open a support ticket, you insert a row. The table only grows.

A **metric** is a summary of events for one account over a fixed time window. "Exports in the last 28 days" is a metric. It collapses many event rows into a single number at a single point in time.

The gap between the two is where most founder analytics stop. A Mixpanel funnel or a Stripe dashboard gives you event counts across all accounts. What you need for churn prediction is one number per account per week, so you can compare accounts to each other and track whether individual accounts are improving or declining.

---

### Minimum viable event warehouse

You need one table. Not a data warehouse. Not a Kafka stream. One table:

```sql
CREATE TABLE events (
  id           bigserial PRIMARY KEY,
  account_id   text        NOT NULL,
  event_type   text        NOT NULL,
  occurred_at  timestamptz NOT NULL
);

CREATE INDEX ON events (account_id, event_type, occurred_at);
```

That is the whole schema. Numeric payloads (session duration, export row count) can be a `numeric` column added later. The core pattern works without them.

**Events to capture for a typical Stripe SaaS:**

- `login` . any authenticated session start
- `feature_used:export` . or whatever your core action is; one event per distinct feature
- `feature_used:invite` . if collaboration is part of your value prop
- `feature_used:integration_connected` . third-party integrations signal deeper commitment
- `support_ticket_opened` . friction signal; high frequency predicts churn
- `support_ticket_resolved` . resolution rate matters too
- `billing_portal_visited` . the Stripe one most products miss; a customer who opens the billing portal but does not cancel is reconsidering; this event has strong predictive power
- `password_reset_requested` . friction signal
- `onboarding_step_completed` . critical for new accounts
- `api_call` . if you have a developer tier; raw call volume is a health indicator
- `report_viewed` or equivalent downstream consumption event . use was not capture; consumption was

You do not need all of these on day one. You need the events that map to value delivery in your product. If your product's job is "send cold emails," the load-bearing events are `sequence_created`, `email_sent`, and `reply_received`. Start there.

---

### The `generate_series` overlapping-window pattern

Here is the SQL template that produces one metric row per account per week. Parameterize `'feature_used:export'` for event type, `'28 day'` for window length, and `COUNT(*)` for aggregation:

```sql
WITH date_vals AS (
  SELECT i::timestamp AS metric_date
  FROM generate_series(
    '2024-01-08',  , first metric date (start + window length)
    '2024-06-30',  , last metric date
    '7 day'::interval
  ) i
)
SELECT
  e.account_id,
  d.metric_date,
  COUNT(*) AS feature_export_per_month
FROM events e
INNER JOIN date_vals d
  ON e.occurred_at <  d.metric_date
  AND e.occurred_at >= d.metric_date - interval '28 day'
WHERE e.event_type = 'feature_used:export'
GROUP BY e.account_id, d.metric_date
ORDER BY e.account_id, d.metric_date;
```

A few details that matter:

Use `<` on the end boundary and `>=` on the start boundary. The asymmetry prevents double-counting when windows slide: an event at exactly midnight belongs to exactly one window.

Do not use `BETWEEN`. It is inclusive on both ends and will double-count events that fall on the boundary.

Accounts with zero events in a window produce no row. That is intentional. Generate zeros at analysis time by left-joining to an account list. Storing explicit zero rows wastes space when most accounts have no activity for most event types.

The `generate_series` call produces one row per calculation date. The join does the rest. To add a new event type, copy the query and change the `WHERE` clause. To change the aggregation from count to sum (e.g., total export row count), replace `COUNT(*)` with `SUM(e.row_count)`. The skeleton does not change.

---

### Window sizing rule

The minimum measurement window is twice the average time between events for one account. A window shorter than that will routinely show zero for healthy accounts just because they had a slow stretch.

| Events per account per month | Minimum window |
|---|---|
| More than 8 | 1 week |
| 4 | 2 weeks |
| 2 | 28 days |
| 1 | 8 weeks |
| 0.5 | 4 months |
| 0.25 | 8 months |

Practical translation for a typical Stripe SaaS:

**High-frequency events** (logins, core feature use, API calls): 28 days. These happen multiple times per week for active accounts. A 28-day window smooths the weekly cycle without losing responsiveness.

**Low-frequency events** (plan change, NPS response, billing portal visit, integration connected): 8 to 12 weeks. If an event happens once a month at most, a 28-day window misses it for most accounts on most dates. You need a longer window to get a stable signal.

One rule that overrides the table: always use multiples of 7 days, never calendar months. Human behavior follows weekly cycles. A 30-day window has 4 or 5 weekends depending on the month. That alone introduces roughly 20% variance in activity metrics that has nothing to do with the customer's health. 28 days, 56 days, 84 days. Not 30, 60, 90.

---

### Tenure-scaled metrics

Naive 28-day metrics systematically misclassify new accounts. An account that signed up 10 days ago has had 10 days to accumulate events. Compared against a 180-day-old account's 28-day count, they look inactive. They are not inactive; they are new.

Carl Gold's solution, which he uses in every case study: measure over a long window (84 days for monthly subscriptions), then scale the result to a 28-day equivalent. The formula handles three cases:

```
Tmin     = 14 days  (minimum tenure before any metric is calculated)
Tdescribe = 28 days  (the "per month" unit you report)
Tmeasure  = 84 days  (the actual observation window)

scaled_count = raw_count * (Tdescribe / LEAST(Tmeasure, tenure))
```

Worked example. An account with 60 days of tenure has 12 exports in the 60-day window. Plug in:

```
scaled = 12 * (28 / LEAST(84, 60))
       = 12 * (28 / 60)
       = 12 * 0.467
       = 5.6 exports per 28-day equivalent
```

A mature account (tenure > 84 days) with 18 exports in the full 84-day window:

```
scaled = 18 * (28 / LEAST(84, 84))
       = 18 * (28 / 84)
       = 18 * 0.333
       = 6.0 exports per 28-day equivalent
```

Now the two accounts are on the same scale and you can compare them. The SQL implementation uses `LEAST(Tmeasure, tenure)` in the denominator, which covers both cases without a CASE branch. Accounts with tenure below `Tmin` are excluded entirely. There is not enough data to say anything useful about them yet.

---

### Metric QA via left join to a generated date series

The most common metric pipeline failure is silent: a cron job skips a week, an event source goes dark, a migration drops rows. Your metrics table has gaps, but nothing errors. The averages look fine because missing rows are absent, not zero.

The fix is the same `generate_series` trick, applied to the metric table itself:

```sql
WITH date_range AS (
  SELECT i::timestamp AS calc_date
  FROM generate_series('2024-01-08', '2024-06-30', '7 day'::interval) i
),
the_metric AS (
  SELECT * FROM metrics
  WHERE metric_type = 'feature_export_per_month'
)
SELECT
  d.calc_date,
  AVG(m.value)        AS avg_value,
  COUNT(m.account_id) AS n_accounts,
  MIN(m.value)        AS min_value,
  MAX(m.value)        AS max_value
FROM date_range d
LEFT OUTER JOIN the_metric m ON m.metric_date = d.calc_date
GROUP BY d.calc_date
ORDER BY d.calc_date;
```

The `LEFT OUTER JOIN` ensures every date in the range appears in the output. A missing calculation week shows up as a row with `avg_value = NULL` and `n_accounts = 0`. An inner join or a plain `GROUP BY` on the metric table would simply omit that week and you would never know.

Run this check after any pipeline change. A sudden drop in `n_accounts` with unchanged `avg_value` means accounts are falling out of the calculation. An `avg_value` spike with stable `n_accounts` means an outlier snuck in.

---

### Ratio metrics beat raw counts

Raw event counts have a structural problem: they correlate with account size. Larger accounts log in more, use features more, and generate more support tickets. Raw counts will tell you big accounts are healthy when some of them are paying a lot for a tool they no longer use.

Ratio metrics break that correlation. The pattern from the Versature case study: MRR per call (unit cost) was nearly uncorrelated with both MRR and call count independently. It was a clean predictor of churn. Neither input was.

The ratios worth trying for most SaaS products:

| Ratio | What it measures | Signal direction |
|---|---|---|
| Core feature use / logins | Engagement per session | Low = customer shows up but does not get value |
| Support tickets / active months | Support load rate | High = friction accumulating |
| Features used / features available | Breadth of adoption | Low = shallow footprint, easy to cancel |
| Active users / licensed seats | License utilization | Low = paying for seats nobody uses |
| Core action / MRR | Value per dollar | Low = customer perceives poor ROI |

The SQL pattern is two CTEs joined on `account_id` and `metric_date`, with a `CASE WHEN denominator > 0 THEN numerator / denominator ELSE NULL END` guard. The CASE guard matters. Division by zero silently crashes a pipeline and drops the account from your analysis dataset.

---

### The completion-then-leave trap

One ratio signal deserves a specific warning: success rate.

For most products, higher success rates mean lower churn. More transactions completed, less churn. More reports generated, less churn. That relationship holds until your product has a fixed completion goal.

The Broadly case study: Broadly helps small businesses collect customer reviews. Their review acceptance rate (reviews accepted / review requests sent) showed a counterintuitive pattern. Accounts with very high acceptance rates churned more, not less. They had collected enough reviews. The job was done. Goodbye.

Ask yourself: does my product have a "we did the job, goodbye" trap? Common examples:

- A migration tool. Once the data is moved, there is no ongoing job.
- A compliance documentation tool. Once the audit passes, urgency drops.
- A one-time growth campaign tool. One successful launch and the need dissolves.

If your product fits that pattern, high success rates in your metrics should not be read as health. They should trigger a check: has this account hit their goal and stopped needing you? The answer is often: build the next job, fast.
