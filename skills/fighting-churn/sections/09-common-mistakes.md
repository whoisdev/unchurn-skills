## 9. Common mistakes

These are the traps that look reasonable at the time and cost real money.

- **Including "Other" in your exit survey.** When you give people an escape hatch, ~40% of them take it. You end up with a pie chart where "Other" is the biggest slice and you learn nothing actionable. Remove it. If a reason doesn't fit your current options, that's a signal to add a specific option, not to hide behind a catch-all.

- **Cancelling immediately instead of setting `cancel_at_period_end: true`.** When you hard-cancel on click, you lose the final billing period the customer already paid for, you eliminate the win-back window, and you surprise the customer who expected service through the end of their cycle. Always use `cancel_at_period_end: true` on Stripe. The cancel is honored; the relationship isn't severed mid-period.

- **Offering discounts to "not using it enough" customers.** This is the most expensive mistake in the list. A customer who isn't using your product doesn't have a price problem. Discounting them trains the cancel-for-deals behavior in your other customers AND loses this one at the next renewal anyway, now at lower MRR. The right save for non-usage is pause. Pausers reactivate at 60-80%; discount-takers often just churn one cycle later.

- **Discounts above 50%.** Once your save discount crosses 50%, you have created a documented acquisition path: cancel, get 50% off, re-subscribe. You will see repeat offenders. Beyond the loop risk, you are permanently reducing the LTV of your saved cohort. Cap saves at 30% and hold it. If 30% isn't enough to save someone, the problem isn't the percentage.

- **Hiding the cancel button.** This feels like friction engineering. It is FTC click-to-cancel exposure and a reliable way to generate chargeback disputes and one-star reviews. Easy cancellation is the foundation; save offers are the layer above it. Design the cancel button to be reachable in two clicks from anywhere your signup was reachable, then let your exit flow do the work.

- **Ignoring involuntary churn.** Smart Retries off. Automatic Card Updater off. No dunning emails. This is the single highest-ROI fix available to most SaaS founders and the most commonly skipped. Involuntary churn is 20-40% of total churn for most subscription businesses. Soft declines recover at 50-70% with a good retry schedule. You are leaving that money on the floor.

- **Not tracking save-cohort LTV.** A customer who accepts your discount and churns 45 days later was not saved. Your save rate metric will look fine. Your MRR will not. Track saved customers for at least 90 days post-save. If your 90-day retained save rate is below 50%, your offers are delaying churn, not stopping it, and you are paying for the delay with discounts.

- **Running A/B tests on cancel flows below 50 cancels per month.** At 20 cancels per month, a 5-percentage-point difference in save rate is 1 customer. That is noise. You will make confident decisions from coin flips. Either wait until you have the volume (roughly 50+ cancels per month per variant, for 4+ weeks), or skip A/B and roll your best-guess configuration based on the reason-to-offer logic in §5.

- **Treating cancel reasons as analytics instead of product feedback.** The reasons your customers give for canceling are bug reports about your product-market fit. "Too expensive" from 30% of churned customers is a pricing signal. "Missing feature: X" appearing 15 times in a month is a roadmap item. The only correct response to a recurring specific reason is to fix the underlying problem or consciously decide that segment is not your ICP. Do not let cancel data sit in a dashboard no one opens.

- **Offering pauses longer than 3 months.** Pauses up to 3 months see strong reactivation. Beyond 3 months, reactivation rates drop sharply because the customer's context has changed, they've adapted without you, and coming back feels like starting over. Cap your pause duration at 3 months. When the pause period ends, send a reactivation reminder and make it one click to resume.

- **No post-cancel reactivation path.** Some customers cancel for seasonal reasons, budget freezes, or team changes. Six months later, the situation resolves and they want back. If your only path is full sign-up, you will lose a portion of these to friction. A one-click reactivation email with their previous plan pre-filled converts meaningfully better. Build the path; set a win-back sequence at 30, 60, and 90 days post-cancel.

- **Founder stops reading cancel reasons after month 2.** The exit survey exists to route offers automatically, yes. But the highest-signal input your product receives is why paying customers leave. Reading 10 cancel reasons per week takes 5 minutes and will surface things no analytics dashboard surfaces. The moment you delegate this entirely to automation, you stop learning from your most honest customers.
