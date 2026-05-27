# Chapter 3: From Events to Metrics — Research Notes

Source: *Fighting Churn with Data* by Carl Gold (MEAP V08), lines 5177-8142.

---

## 1. The Event-to-Metric Pipeline (conceptual model)

- An **event** is something (event type) that happens to someone (account) at some time (timestamp). Three required fields: `account_id`, `event_type_id`, `event_time`.
- A **metric** summarizes a customer's events over a time window into a single comparable number. Without metrics, accounts are incomparable because events occur at different rates and different times.
- The pipeline is: raw event stream → time-windowed aggregation → metric table → churn analysis dataset.
- Metrics make cohort-level comparison possible. The goal is a behavioral profile per account at each observation date.

---

## 2. Event Data Warehouse Schema (Section 3.2)

Minimum viable event table (Postgres example):

| Column | Type | Required |
|---|---|---|
| `account_id` | integer or char | yes |
| `event_type_id` | integer or char | yes (FK to event_type) |
| `event_time` | timestamp | yes |
| `event_id` | integer or char | optional |
| `user_id` | integer or char | optional |
| `event_data_1..n` | float or char | optional |

Companion lookup table (keeps string names out of the fact table):

| Column | Notes |
|---|---|
| `event_type_name` | unique string |
| `event_type_id` | FK used in events table |

Carl's framing: "An event is something (event type) that happens to someone (account) at some time." Numeric data fields on events are optional, unlike some data warehouse conventions that require them.

---

## 3. Counting Events in One Period (Section 3.3)

Core SQL pattern (Listing 3.1):

```sql
WITH calc_date AS (
  SELECT '2020-05-06'::timestamp AS the_date
)
SELECT account_id, COUNT(*) AS n_like_permonth
FROM event e INNER JOIN calc_date d
  ON e.event_time <= d.the_date
  AND e.event_time > d.the_date - interval '28 day'
INNER JOIN event_type t ON t.event_type_id = e.event_type_id
WHERE t.event_type_name = 'like'
GROUP BY account_id;
```

Key details:
- Use `<=` on end boundary and `>` (not `>=`) on start boundary. This prevents double-counting events that fall exactly on the period boundary when windows slide.
- Do not use SQL `BETWEEN`; it creates double-count risk at boundaries.
- Do not use SQL window functions here. Window functions operate on fixed row counts; event windows are date-bounded, not row-bounded.
- Accounts with zero events in the window produce no row. Carl recommends **not** storing zero-count metrics. Generate zeros at analysis time rather than storing them. At scale, storing zeros is expensive and most rows would be zeros.

---

## 4. Details of the Metric Period (Section 3.4)

**Use multiples of 7 days, never calendar months.**

Reason: human activity follows weekly cycles. B2B products peak Monday-Friday; consumer products peak Friday-Sunday. Calendar months have 4 or 5 weekends, so month-length windows produce ~20% variance from weekend count alone, not from real behavior change. Klipfolio real data showed weekdays average 40% more activity than weekends.

Rule: for windows of ~12 weeks or less, always use multiples of 7 days. For windows over a year, one extra weekend does not matter much.

**Timestamp convention:** stamp the metric with the date immediately after the measurement period ends. If you measure events Jan 1-28, the metric timestamp is Jan 29. Rationale:
- Start-of-period timestamps require you to remember period length when syncing metrics of different durations.
- Last-day-of-period timestamps imply the observation was complete when there was still one day remaining.
- Day-after timestamps allow simple `WHERE metric_time = X` to sync all metrics at a single point.

---

## 5. Making Measurements at Multiple Dates (Section 3.5)

**Overlapping windows.** Non-overlapping 4-week windows update a metric only once a month. That is too slow to detect changing behavior. The solution is to slide the window weekly, producing overlapping 4-week windows.

Core SQL (Listing 3.2):

```sql
WITH date_vals AS (
  SELECT i::timestamp AS metric_date
  FROM generate_series('2020-01-29', '2020-04-16', '7 day'::interval) i
)
SELECT account_id, metric_date, COUNT(*) AS n_like_per_month
FROM event e INNER JOIN date_vals d
  ON e.event_time < metric_date
  AND e.event_time >= metric_date - interval '28 day'
INNER JOIN event_type t ON t.event_type_id = e.event_type_id
WHERE t.event_type_name = 'like'
GROUP BY account_id, metric_date
ORDER BY account_id, metric_date;
```

`generate_series` produces the sequence of weekly calculation dates. Non-Postgres databases: build a permanent table of dates as a one-time load.

**How often to update metrics:**
- Typical SaaS (lifetime measured in months to years): weekly updates are adequate.
- Short lifetime products (few months): daily updates may be needed.
- Very long lifetime products: monthly is acceptable.
- Consumer products: measure at Monday/Tuesday midnight to capture the full previous weekend.
- B2B products: measure on Saturday/Sunday midnight to capture the full previous work week.

**Simulation vs. real data.** Carl uses a simulated social network (events: like, dislike, post, new friend, unfriend, adview, message, reply) throughout. Simulation data is smoother and less variable than real data. All the SQL patterns are identical for both. The simulation is purely pedagogical; you always build the same pipeline on real data.

**Saving metrics:** use `INSERT ... SELECT` to write directly from the calculation query into the metric table. Keep metric names in a separate lookup table (composite PK: `account_id + metric_name_id + metric_time`).

Metric storage schema:

| Column | Type | Notes |
|---|---|---|
| `account_id` | integer or char | Composite PK 1 |
| `metric_name_id` | integer or char | Composite PK 2; FK to metric_name |
| `metric_time` | timestamp | Composite PK 3 |
| `value` | float | always exactly one value per row |

---

## 6. Measuring Totals and Rates (Section 3.6)

When events carry a numeric field (session duration, call length, dollar value), replace `COUNT(*)` with `SUM(field)` or `AVG(field)`. The rest of the query is identical to the count pattern.

```sql
SELECT account_id, metric_date, SUM(duration) AS local_call_duration
FROM event e INNER JOIN date_vals d
  ON e.event_time < metric_date
  AND e.event_time >= metric_date - interval '28 day'
...
```

Common metric types: count, sum, average. All share the same SQL skeleton.

---

## 7. Metric Quality Assurance (Section 3.7)

**Spot-checking a few rows is not enough.** QA must check for missing data in specific accounts, erroneous extreme values, and calculation gaps.

**Time-series stats check (Listing 3.6):** for each metric, compute `AVG`, `MIN`, `MAX`, and `COUNT` per measurement date using a LEFT OUTER JOIN against a generated date series (not a GROUP BY on the metric table alone). The left join ensures zero-count rows appear for dates when no metrics were calculated, making gaps visible.

```sql
WITH
date_range AS (
  SELECT i::timestamp AS calc_date
  FROM generate_series('2020-04-01', '2020-05-06', '7 day'::interval) i
), the_metric AS (
  SELECT * FROM metric m
  INNER JOIN metric_name n ON m.metric_name_id = n.metric_name_id
  WHERE n.metric_name = 'like_per_month'
)
SELECT calc_date, AVG(metric_value), COUNT(the_metric.*) AS n_calc,
       MIN(metric_value), MAX(metric_value)
FROM date_range LEFT OUTER JOIN the_metric ON calc_date = metric_time
GROUP BY calc_date
ORDER BY calc_date;
```

What to look for:
- Missing data: `AVG` and `COUNT` drop together, plateau at zero while window slides past the gap.
- Extreme outlier values: `MAX` spikes sharply; `AVG` rises modestly; `MIN` and `COUNT` are unaffected.
- Negative outliers: `MIN` plunges instead.

**Metric coverage check (Listing 3.8):** what fraction of active accounts have a non-zero value for each metric? Steps: (1) count active accounts in the date range via subscription join, (2) count distinct accounts that have the metric, (3) divide. Low coverage for common events = likely data problem. Low coverage for rare events = may be expected.

---

## 8. Event QA (Section 3.8)

**Correct order of operations:** event QA first, then metric calculation, then metric QA.

**Events per day (Listing 3.9):** count total events per day per event type using a LEFT OUTER JOIN against a generated date series. Missing data shows as zero-count days. Gaps that are shorter than the metric window produce partial metric drops; gaps longer than the window produce zero metrics.

**Events per account per month (Listing 3.11):** divide total events by (1) number of active accounts and (2) number of four-week months in the time range. This gives `events_per_account_per_month`. Sort descending. Example from simulation: ~75 likes per account per month but less than 1 unfriend. Only 59% of accounts had an unfriend metric in any given month, explained by the low event frequency.

If observed event frequency seems wrong and you cannot tell, get a domain expert. Do not proceed to metric calculation until you understand the baseline event volumes.

Carl's stance on automated anomaly detection: for fewer than ~100 events/metrics, manually viewing time-series plots is faster and teaches you the data better than any algorithm. Script the plot generation and flip through them visually. Above ~100 events, automated anomaly detection becomes necessary.

---

## 9. Selecting the Metric Period (Section 3.9)

**Core trade-off:** short period = more responsive to recent behavior changes but misses accounts with rare events. Long period = captures rare events but is slow to reflect behavior changes.

**Rule of thumb:** minimum period = at least twice the average time between events for one account. Never shorter than one week (weekly cycle). Usually never longer than one year.

| Events per account per month | Minimum measurement period |
|---|---|
| >8 | 1 week |
| 4 | 2 weeks |
| 2 | 1 month |
| 1 | 2 months |
| 0.5 | 4 months |
| 0.25 | 8 months |
| 0.17 | 12 months |

For fixed-term subscriptions, align the metric period to the subscription term. Monthly subscription: use ~1 month. Annual subscription: use 3-6 months minimum (recent experience is most salient at renewal). For no-fixed-term products, aim for one-quarter to one-half of the typical customer lifetime.

Solution to having different periods for different events: Chapter 7 shows how to combine metrics with different periods by converting them to rates/averages, making them directly comparable.

---

## 10. Account Tenure (Section 3.10)

Account tenure = length of time a customer has been active in their current uninterrupted sequence of subscriptions, ignoring earlier periods separated by a long gap.

Rules:
- A short gap between subscriptions (up to ~1 month for monthly billing, up to ~4 months for annual) counts as continuous. Covers failed credit card re-sign-ups.
- A long gap resets tenure to zero. The customer is treated as new.
- Tenure is undefined (not zero) during churned periods.

Requires a **recursive CTE** in SQL (Postgres `WITH RECURSIVE`). The recursion walks backward from the current active subscription, finding earlier subscriptions within the allowed gap, until no earlier subscription qualifies.

Core SQL (Listing 3.12, single date):

```sql
WITH RECURSIVE date_range AS (
  SELECT '2020-07-01'::date AS calc_date
), earlier_starts AS (
  SELECT account_id, MIN(start_date) AS start_date
  FROM subscription INNER JOIN date_range
    ON start_date <= calc_date
    AND (end_date > calc_date OR end_date IS NULL)
  GROUP BY account_id
  UNION
  SELECT s.account_id, s.start_date
  FROM subscription s INNER JOIN earlier_starts e
    ON s.account_id = e.account_id
    AND s.start_date < e.start_date
    AND s.end_date >= (e.start_date - 31)
)
SELECT account_id, MIN(start_date) AS earliest_start,
       calc_date - MIN(start_date) AS subscriber_tenure_days
FROM earlier_starts CROSS JOIN date_range
GROUP BY account_id, calc_date
ORDER BY account_id;
```

For multi-date insertion (Listing 3.13): replace `calc_date` CTE with `generate_series`, thread `metric_date` into the recursive SELECT and joins, and wrap with `INSERT INTO metric`.

---

## 11. MRR and Other Dollar Metrics (Section 3.11)

**MRR as a metric.** Treat MRR exactly like any other metric: calculate it on the same weekly sequence of dates and store it in the metric table. This makes it trivially joinable with behavioral metrics for churn analysis. Accounts with multiple simultaneous subscriptions (add-ons, upgrades mid-period) use `SUM(mrr)` across all active subscriptions on each date.

```sql
SELECT account_id, metric_date, SUM(mrr) AS total_mrr
FROM subscription INNER JOIN date_vals
  ON start_date <= metric_date
  AND (end_date > metric_date OR end_date IS NULL)
GROUP BY account_id, metric_date;
```

**Pitfall:** do not treat MRR as a static fact. It changes whenever a subscription starts, ends, or the price changes. The point-in-time calculation is required.

**Subscription unit quantities** (seats, devices, bandwidth). Same pattern, replace `SUM(mrr)` with `SUM(quantity)` and add `WHERE units = 'Seat'`. Quantities are additive across simultaneous subscriptions (add-on seats stack).

**Billing period metric.** Monthly vs. annual billing is predictive of churn (annual customers often churn less, especially when prepaid). Treat billing period as a metric calculated the same way. Aggregation is `MIN(bill_period_months)` across simultaneous subscriptions, not SUM (two annual subscriptions do not equal a 24-month billing period). If a customer has any monthly subscription, they behave like a monthly customer.

**General pitfall note.** Storing a static snapshot of MRR (or any subscription metric) at a single point in time misses plan changes and add-ons between that date and the analysis. Always calculate subscription metrics at the same time granularity as behavioral metrics.

---

## 12. Automation (Sidebar, Section 3.11)

Carl warns explicitly: you will recalculate metrics many times as QA uncovers problems and business priorities shift. A metric calculation framework must:
- Store metric SQL as templates with bind variables for event type and time period.
- Handle inserting metric names into the lookup table.
- Remove old results when a metric is recalculated (idempotent runs).

The book's Python wrapper script covers the first goal. Carl built a separate metric framework (GitHub: `carl24k/fight-churn/metric-framework`) that he used on real client cases.

---

## 3 Patterns to Steal Directly

1. **Overlapping-window metric SQL with `generate_series`.** The `WITH date_vals AS (SELECT i::timestamp FROM generate_series(...))` joined to events via date arithmetic is the exact pattern for our health-score pipeline. It produces one metric row per (account, week) with no extra code per event type. We should build a parameterized version where the event type, window length, and aggregation function are bind variables.

2. **LEFT OUTER JOIN against a generated date series for QA.** The trick of joining metric (or event) data to an independently generated date series ensures missing periods appear as NULL/zero rows rather than simply being absent. This is the correct way to detect calculation gaps. We should apply this to our own metric-freshness checks.

3. **Metric period sizing rule: minimum period = 2x average time between events per account.** This is a concrete, actionable formula we can surface in the skill's guidance for choosing observation windows. For a Stripe-native SaaS founder with monthly billing, the subscription term aligns the window to ~28 days for high-frequency events and 8-12 weeks for low-frequency ones.
