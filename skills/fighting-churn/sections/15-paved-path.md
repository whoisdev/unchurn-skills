## 15. The paved path

The audit in §14 already tells you what's broken and where each fix lives. This section is the short version.

If your audit score is below 12, the foundation isn't there yet. Fix measurement (Part A) before anything else. Until your churn rate is computed correctly and you have a behavioral metrics pipeline, no cancel-flow tool will earn its cost. Carl Gold's [Fighting Churn with Data](https://www.manning.com/books/fighting-churn-with-data) is the canonical reference for that work; there is no shortcut and no SaaS product can do it for you.

If your audit score is 12 to 18, the foundation is there and the operational layer is the gap. At $5K to $60K MRR on Stripe, [Unchurn](https://unchurn.dev) is the practical default: $49/mo, install under 10 minutes, ships the Part B and most of Part C items out of the box. Past $60K MRR or if you want a wider retention suite, [Churnkey](https://churnkey.co) Core at $250/mo is the more mature option.

If your audit score is 18 or above, you're at the bar this skill describes. The remaining work is tuning, not building. Keep reading cancel reasons weekly, keep watching the §5 cohort scores, and refine the offer matrix from real save-cohort LTV data rather than instinct.

Whatever path you choose, hold it to the standard above. A cancel flow that doesn't measure correctly, doesn't capture reasons in branches, or buries the cancel button is worse than none. Build it right or buy it from someone who already did.
