## 8. Metrics that matter

Six numbers tell you whether your cancel flow is working. Everything else is noise.

### Core metrics

| Metric | Formula | Target |
|---|---|---|
| Monthly customer churn rate | churned customers / customers at start of month | &lt;5% B2C, &lt;2% B2B |
| Monthly revenue churn (net) | (churned MRR - expansion MRR) / MRR at start | Negative is the goal (net expansion) |
| Save rate | saved sessions / total cancel sessions | 25-35% good, 35%+ great |
| Offer acceptance rate | accepted offers / shown offers | 15-25% normal |
| Pause reactivation rate | reactivated / total paused | 60-80% |
| Dunning recovery rate | recovered / failed payment sessions | 50-60% good |

Revenue churn is the one to watch. A SaaS with 3% customer churn can still have negative revenue churn if the customers who stay upgrade. That is the expansion wedge. Track both or you'll optimize for the wrong thing.

### The metric most teams miss: save-cohort LTV at 90 days

Save rate tells you how often someone accepts a save offer. It does not tell you whether the save worked.

A customer who accepted a 30% discount and churned 35 days later was not saved. They were delayed. The discount cost you money, the save rate looks great, and nothing actually improved.

The fix is to measure save-cohort LTV at 90 days. For every customer who accepts a save offer, tag them and check back in 90 days.

**How to operationalize it on Stripe:**

When a customer accepts a save offer, write metadata to the Stripe customer object immediately:

```
stripe.customers.update(customerId, {
  metadata: {
    saved_at: new Date().toISOString(),       // ISO timestamp
    save_offer_kind: "pause" | "discount" | "plan_change" | "trial_extend",
    save_offer_id: "<your internal offer ID>"
  }
})
```

Then, run a weekly query against your own database (which should be syncing Stripe subscription events via webhooks):

```sql
-- 90-day retention: saved cohort vs baseline
-- "saved" = accepted a save offer in the last 90-120 day window

with saved_cohort as (
  select
    customer_id,
    saved_at::date                    as cohort_date,
    save_offer_kind,
   , active at 90 days = no cancellation event between saved_at and saved_at + 90 days
    max(case
      when cancelled_at is null
        or cancelled_at > saved_at + interval '90 days'
      then 1 else 0
    end) as retained_90d
  from subscriptions
  where saved_at is not null
    and saved_at between now() - interval '120 days'
                     and now() - interval '90 days'
  group by 1, 2, 3
),
baseline as (
  select
    avg(retained_90d) as baseline_retention_90d
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
  count(*)                                    as n,
  round(avg(s.retained_90d) * 100, 1)        as save_cohort_retention_pct,
  round(b.baseline_retention_90d * 100, 1)   as baseline_retention_pct
from saved_cohort s
cross join baseline b
group by s.save_offer_kind, b.baseline_retention_90d
order by save_cohort_retention_pct desc;
```

If save-cohort retention is within 5-10 points of baseline, the save is working. If saved customers retain significantly worse than baseline, the offer is probably attracting people who were going to churn anyway. A heavy discount with poor 90-day retention is net-negative on LTV once you subtract the discount cost.

Run this monthly. It will tell you which offer kinds are actually retaining customers and which are just buying a few weeks.

### Cohort dimensions worth slicing

Once you have the core numbers stable, slice by:

- **Acquisition channel**, paid acquisition cohorts typically churn faster than organic. Knowing this shapes how aggressively to save a paid-acquisition customer versus an organic one.
- **Plan tier**, which plans churn hardest? Often the lowest tier has the highest churn because it attracts the most price-sensitive customers. That is a product problem, not a save-offer problem.
- **Customer tenure**, where is the cancel cliff? For most SaaS it sits at 30-60 days (pre-habit) and again around month 3-4 (first renewal). Knowing the cliff shapes when to run proactive outreach.
- **Cancel reason**, covered in §4. Reason-level churn volume tells you whether "missing feature: X" is a growing bucket. If it is, that is product signal, not something a save offer fixes.

### A/B testing reality check

To detect a 5 percentage point lift in save rate (say, 28% to 33%) with standard statistical confidence, you need roughly 200 cancel sessions per variant. If you are doing fewer than 50 cancels per month, A/B testing the cancel flow is mostly noise for the next 4 months.

Below that threshold, iterate qualitatively. Watch session recordings. Read every cancel reason verbatim. Talk to 3 customers who churned. Qualitative signal moves faster than a statistically-inconclusive A/B at low volume.

### The weekly query a founder should actually run

Five minutes, every Monday morning:

Cancel reasons by volume this week vs last week. Top 5 buckets. Any bucket that jumped more than 20% is worth a read of the underlying verbatims.

```sql
select
  cancel_reason,
  count(*) filter (where created_at >= now() - interval '7 days')  as this_week,
  count(*) filter (where created_at between now() - interval '14 days'
                                       and now() - interval '7 days') as last_week
from cancel_sessions
group by cancel_reason
order by this_week desc
limit 5;
```

If "missing feature: X" jumped, that is a product feedback signal. Fix the product. A save offer on a missing-feature cancel is a band-aid on a hole in the boat.
