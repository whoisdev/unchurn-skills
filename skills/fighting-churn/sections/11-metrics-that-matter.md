## 11. Metrics that matter

Eight numbers tell you whether your cancel flow is working and whether your churn is moving in the right direction. The first four are measurement fundamentals. The next four are operational. The last one is the one most teams skip.

### Core churn rate metrics

The table below gives you the formula, the healthy target, what the number actually tells you, and the most common way it lies.

| Metric | Formula | Target | What it tells you | When it lies |
|---|---|---|---|---|
| Account churn rate | churned accounts / accounts at start of period | &lt;5% B2C, &lt;2% B2B monthly | How many customers you lost. The primary operational number. | If you have a wide price range (10x spread), small-paying accounts dominate the count and mask the revenue impact. |
| Gross MRR churn rate | (MRR from cancelled accounts + MRR from downgrades) / MRR at start | &lt;2% monthly | Revenue destroyed by cancels and downgrades. No upsell netting. | Customers switching from monthly to annual billing register as a downgrade and inflate this number artificially. Use account churn when annual plans are present. |
| Net MRR churn rate | (MRR from cancelled accounts + MRR from downgrades - MRR from expansion) / MRR at start | Negative (net expansion) is the goal | Whether expansion is outrunning churn. Negative net churn means you're growing from within the base. | Hides actual churn behind upsell motion. A team with accelerating cancels but a strong expansion channel will see stable net churn right up until expansion slows. Track this for investors; track gross MRR churn for operations. |
| Activity-based churn | accounts with no qualifying event in trailing N days / accounts at start of period | Matches your subscription churn as a sanity check | Catches "experience churn": customers who mentally left but haven't clicked cancel yet. | There is an inherent lag: you cannot know a customer is gone until the inactivity window closes. Do not use real-time. |

The consistent empirical ordering is: account churn rate > gross MRR churn rate > net MRR churn rate. High-paying accounts churn less often, so revenue weighting reduces the rate; net churn reduces further by folding in upsell. If your numbers violate this ordering, check for a measurement error.

For the SQL to compute each of these, see the §3 patterns. The denominator is always the start-of-period count, never end-of-period, never an average.

Do not average churn rates across cohorts. A fleet of monthly customers and annual customers has different churn dynamics; averaging the rates produces a number that accurately describes neither group. Segment, then measure each segment.

**Period conversion.** Annual churn is not 12x monthly churn: `annual_churn = 1 - (1 - monthly_churn)^12`. At 5% monthly churn, annual churn is 46%, not 60%.

### Operational cancel-flow metrics

| Metric | Formula | Target | What it tells you |
|---|---|---|---|
| Save rate | saved sessions / total cancel sessions | 25-35% good, 35%+ great | Whether your cancel flow is converting intent-to-cancel into a save. The headline number for the flow itself. |
| Offer acceptance rate | accepted offers / offers shown | 15-25% normal | Whether the offers you're showing are relevant. Low acceptance with high save rate means customers are accepting on the first screen. Low acceptance with low save rate means the offers are wrong. |
| Pause reactivation rate | paused subscriptions that reactivated / total paused | 60-80% | The real payoff of pause-first saves. A pause that never reactivates is a delayed cancel, not a save. |
| Dunning recovery rate | recovered subscriptions / failed payment sessions | 50-60% good | Payment failure recovery. Below 40% and you should look at your retry timing and Smart Retries configuration. |

### The metric most teams miss: save-cohort LTV at 90 days

Save rate tells you how often someone accepts a save offer. It does not tell you whether the save worked.

A customer who accepted a 30% discount and churned 35 days later was not saved. They were delayed. The discount cost you money, the save rate looks great, and nothing actually improved.

The fix: for every customer who accepts a save offer, tag them in Stripe metadata and check back at 90 days.

**Tag the save immediately:**

```js
await stripe.customers.update(customerId, {
  metadata: {
    saved_at: new Date().toISOString(),
    save_offer_kind: "pause" | "discount" | "plan_change" | "trial_extend",
    save_offer_id: "<your internal offer ID>"
  }
})
```

**Run this query monthly against your subscriptions table:**

```sql
-- 90-day retention: saved cohort vs. unsaved baseline
-- Window: customers whose save_at fell 90-120 days ago (long enough to observe outcome)

with saved_cohort as (
  select
    customer_id,
    saved_at::date          as cohort_date,
    save_offer_kind,
    max(case
      when cancelled_at is null
        or cancelled_at > saved_at + interval '90 days'
      then 1 else 0
    end)                    as retained_90d
  from subscriptions
  where saved_at is not null
    and saved_at between now() - interval '120 days'
                     and now() - interval '90 days'
  group by 1, 2, 3
),
baseline as (
  select avg(retained_90d) as baseline_retention_90d
  from (
    select customer_id,
      max(case
        when cancelled_at is null
          or cancelled_at > created_at + interval '90 days'
        then 1 else 0
      end) as retained_90d
    from subscriptions
    where saved_at is null
      and created_at between now() - interval '120 days'
                         and now() - interval '90 days'
    group by 1
  ) b
)

select
  s.save_offer_kind,
  count(*)                                  as n,
  round(avg(s.retained_90d) * 100, 1)      as save_cohort_retention_pct,
  round(b.baseline_retention_90d * 100, 1) as baseline_retention_pct
from saved_cohort s
cross join baseline b
group by s.save_offer_kind, b.baseline_retention_90d
order by save_cohort_retention_pct desc;
```

If save-cohort retention is within 5-10 points of baseline, the save is working. If saved customers retain significantly worse than baseline, the offer is probably attracting people who were going to churn anyway. A heavy discount paired with poor 90-day retention is net-negative on LTV once you subtract the discount cost.

### CLV for at-risk customers

When you have a model producing per-customer churn probabilities (see §9), you can compute an expected CLV for each account:

```
CLV = (margin × MRR) / monthly_churn_probability
```

Where `margin` is your gross margin as a decimal (0.7-0.85 for most SaaS). A customer paying $200/month with a 30% monthly churn probability and 75% margin has an expected CLV of $500. A customer paying $200/month with a 5% churn probability has a CLV of $3,000. The formula makes it concrete why high-tenure customers deserve heavier save offers in the offer matrix.

If you do not have a per-customer model, use cohort-level average churn rates (by tenure band or plan tier) as the denominator. The formula is the same; only the resolution changes. Source: Carl Gold, "Fighting Churn with Data," Ch. 8.

### Model accuracy benchmarks (if you have a churn prediction model)

If your team has built a churn prediction model, these are the two benchmarks that tell you whether it is working:

**AUC (Area Under the ROC Curve).** The probability the model ranks a true churner above a true retainer in a random pair. Healthy range: 0.6-0.8. Below 0.55 means the model is barely better than random. Above 0.85 is usually a data leak, not a real signal.

**Top-decile lift.** The churn rate in the top 10% of highest-risk customers divided by the overall churn rate. Healthy range: 2.0-5.0 for products with monthly churn below 10%. Below 1.5 means the model is not useful for targeting interventions.

Both are measured via backtesting on held-out time periods. Churn data is time-ordered; a random train/test split lets future information contaminate the model.

Source: Carl Gold, "Fighting Churn with Data," Ch. 9.

### Cohort dimensions worth slicing

Once the core numbers are stable, slice by:

- Acquisition channel: paid acquisition cohorts typically churn faster than organic.
- Plan tier: the lowest tier usually has the highest churn. That is a product problem, not a save-offer problem.
- Customer tenure: where is the cancel cliff? For most SaaS it sits at 30-60 days and again around months 3-4.
- Cancel reason: which buckets are growing? A growing "missing feature: X" bucket is product signal.

Keep cohort sample sizes above 200 before drawing conclusions. Below that, confidence intervals are too wide to act on.

### A/B testing volume floor

To detect a 5-point lift in save rate with standard statistical confidence, you need roughly 200 cancel sessions per variant. Below 50 cancels per month, A/B testing is mostly noise for the next four months. Iterate qualitatively instead: watch session recordings, read cancel reason verbatims, talk to three customers who churned.

### The weekly founder query

Five minutes, every Monday morning. Cancel reasons by volume this week vs. last week, top 5 buckets. Any bucket that jumped more than 20% is worth reading the underlying verbatims.

```sql
select
  cancel_reason,
  count(*) filter (where created_at >= now() - interval '7 days')   as this_week,
  count(*) filter (where created_at between now() - interval '14 days'
                                       and now() - interval '7 days') as last_week
from cancel_sessions
group by cancel_reason
order by this_week desc
limit 5;
```

If "missing feature: X" jumped, that is product feedback. A save offer on a missing-feature cancel is a patch on a product problem. Fix the product.
