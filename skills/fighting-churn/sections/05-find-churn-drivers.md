## 5. Find your churn drivers (cohort analysis + correlation)

Section 4 gets you clean cancel reasons. This section is about the behavior data that precedes the cancel button: what customers were actually doing (or not doing) in the weeks before they left.

The goal is to replace founder-vibes ("I think users churn because they never connect their third integration") with founder-data: a short ranked list of 3-6 behavior clusters that actually predict churn for your product, with churn rates to back them up. Once you have that list, you know which customers to watch, which save offers to lead with, and where to focus product work.

If you have fewer than 200 active paying subscriptions, skip to the caveat at the end of this section before investing a weekend here. The method still works; it is just noisier.

### What you need going in

One row per subscription-period, with:
- The subscription ID
- Whether the period ended in churn (1) or renewal (0)
- The behavioral metrics you identified in §4 for that period

"One row per subscription-period" means the same customer can appear many times: once per billing cycle. That is intentional. You are not analyzing customers; you are analyzing observations of behavior during a period that ended in churn or renewal.

Strip free trials and comped accounts before you start. Their behavior-churn relationship is different, and mixing them in will blur the signal you are after.

### Step 1: metric cohort analysis

Pick one metric. Sort all your observations by that metric value. Divide them into 5 to 10 equal-size buckets (called cohorts or deciles). For each bucket, compute the churn rate.

Here is a concrete example. Suppose you are a project management tool and you want to test "tasks created in the last 28 days":

| Cohort | Avg tasks created | Churn rate |
|--------|------------------|------------|
| 1 (lowest) | 0 | 18.4% |
| 2 | 2 | 14.1% |
| 3 | 6 | 9.8% |
| 4 | 14 | 7.2% |
| 5 | 28 | 6.9% |
| 6 | 52 | 6.1% |
| 7 | 88 | 5.8% |
| 8 | 145 | 5.5% |
| 9 | 230 | 5.4% |
| 10 (highest) | 420 | 5.2% |

Two things jump out. First, the relationship is real: 18.4% churn in cohort 1 vs. 5.2% in cohort 10 is a 3.5x gap. Second, the churn curve flattens after cohort 4 or 5. Getting users from zero tasks to 14 tasks per period matters enormously. Getting them from 145 to 420 barely moves the needle.

That inflection point is your danger threshold. Customers below cohort 4 (roughly 14 tasks per period) are in a different risk category than customers above it. That is the number you will watch in your retention dashboard.

The SQL to compute this is straightforward. Assuming you have a `subscription_periods` table:

```sql
with ranked as (
  select
    subscription_id,
    tasks_created_28d,
    is_churn,
    ntile(10) over (order by tasks_created_28d) as cohort
  from subscription_periods
  where is_trial = false
),
cohort_stats as (
  select
    cohort,
    round(avg(tasks_created_28d), 1) as avg_metric,
    round(avg(is_churn::numeric) * 100, 1) as churn_rate_pct,
    count(*) as n
  from ranked
  group by cohort
)
select * from cohort_stats order by cohort;
```

Run this for every metric in your set. The ones with the largest top-to-bottom churn gap are your candidates.

A metric is worth keeping if:
- The churn relationship is monotonic (consistently higher metric, consistently lower churn, with normal noise)
- The top-to-bottom ratio is at least 1.5x
- At least 5% of your observations have a non-zero value for this metric

Drop metrics that fail the last test. If 95% of your customers have never triggered an event, that event cannot drive churn.

### Step 2: score skewed metrics before going further

Most behavioral metrics are heavily skewed. A handful of power users create hundreds of tasks; most users create a few. When skew is above 4 (compute it with `SELECT stddev(x)^4 / variance(x)^2 AS approx_skew` or your analysis tool of choice), the raw values compress most cohorts into a tiny range and inflate the ones at the top. The cohort chart looks misleading.

The fix is to apply a log transform before standardizing. The full scoring formula:

1. If skew > 4 and minimum value is 0: replace each value `x` with `ln(1 + x)`. The `+1` keeps zeros from becoming negative infinity.
2. Compute the mean and standard deviation of the transformed values.
3. Subtract the mean, divide by the standard deviation.

The result is a score where 0 means "average customer," positive means "above average," and negative means "below average." The range is roughly -5 to +5.

This is not cosmetic. Every correlation technique in step 3 depends on scores, not raw values. Skewed raw values understate correlation between related behaviors because one outlier can dominate the calculation. Scores fix that.

After scoring, re-run the cohort analysis. The shape of the churn curve will be the same; the x-axis will be easier to read and the inflection point will be clearer.

### Step 3: group correlated metrics into behavior clusters

Your product probably tracks 20-60 behavioral events. Running 40 cohort charts and presenting them individually to yourself is counterproductive. More importantly, correlated behaviors tell the same story about a customer. If logins, dashboard views, and report exports all go up together, averaging them into a single "engagement score" gives you a cleaner churn signal than any one of them alone.

Start with a correlation matrix. Compute pairwise Pearson correlation between all your scored metrics. If you are working in a spreadsheet, export to CSV, color the cells: green above 0.5, yellow between 0.3 and 0.5, gray below 0.3. The block structure you see is your grouping map.

Group metrics that have correlation above 0.5 with each other. For each group, the group score is the simple average of the member metric scores. Because all scores share the same scale (standard deviations from average), averaging is meaningful even when the underlying metrics measure entirely different things.

What this looks like in practice: Klipfolio, a SaaS dashboard tool, had around 70 behavioral metrics. Their correlation matrix revealed 6 clusters. The top group combined viewing and editing behaviors. On any individual metric from that group, the top cohort churned at roughly one-third the rate of the bottom cohort. On the group average score, the top cohort churned at one-tenth the rate. The signal became three times stronger just by combining correlated metrics.

That is the core payoff. Grouped scores surface churn risk more cleanly than individual metrics, and they produce a short list you can actually act on.

### Step 4: rank your behavior clusters

After grouping, you will have 3-6 group scores (plus any solo metrics that did not correlate with anything). Run the cohort analysis one more time on each group score. Rank by the top-to-bottom churn gap.

A typical output for a project management tool might look like:

| Rank | Cluster | Metrics included | Top cohort churn | Bottom cohort churn | Gap |
|------|---------|-----------------|-----------------|--------------------|----|
| 1 | Core task activity | tasks created, tasks completed, tasks assigned | 3.1% | 22.4% | 7.2x |
| 2 | Collaboration | comments, @mentions, invites sent | 3.8% | 17.9% | 4.7x |
| 3 | Integration use | integrations connected, API calls | 4.2% | 15.6% | 3.7x |
| 4 | Session frequency | logins, days active | 4.9% | 14.1% | 2.9x |

That ranked list is the output of this section. Keep it. You will use it in:

- **§6 (save offer matrix):** customers whose group scores are low on cluster 1 but high on cluster 2 have a different problem than customers low on everything. The offer matrix uses these clusters to match save offers to root causes.
- **§8 (metrics dashboard):** the top 2-3 cluster scores become the leading indicators you track weekly. Churn is a lagging signal. These are the early warning system.
- **§9 (audit):** when save rates drop, the first diagnostic is whether a new cohort of customers entered your low-engagement clusters without triggering any intervention.

### The honest caveat on volume

Cohort analysis needs enough churn events to produce stable rates per bin. The practical floor is around 100 total churn events in your dataset, with at least 200-300 observations per cohort. If you are running 10 cohorts and you have 300 total observations, each cohort has 30 rows. A churn rate computed from 30 observations is noisy: a few extra churns in one bucket completely changes the picture.

Below roughly 200 active paying subscriptions, the technique is still useful but should be treated as directional. The gap ratios will swing considerably with small sample changes. What works better at that volume: reading every cancel reason weekly (§4), looking for a pattern by hand, and using that pattern to set your first save offers. Return to the analytical method once you have 6-12 months of churn data and 300+ events.

Above 200-300 active subscriptions with 12+ months of history, the method starts paying dividends that intuition cannot match. The top cluster on the ranked list is almost never what the founder guessed before running the analysis.
