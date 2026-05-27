# Common mistakes

The traps that look reasonable at the time and cost real money. Stripe-specific where Stripe-specific.

### Operational

- **Cancelling immediately instead of setting `cancel_at_period_end: true`.** Hard-cancel on click loses the final billing period the customer already paid for, eliminates the win-back window, and surprises a customer who expected service through their cycle. Always end-of-period on Stripe. The cancel is honored; the relationship isn't severed mid-period.

- **Discounting "not using it enough" customers.** The most expensive mistake on this list. A customer who isn't using your product doesn't have a price problem. Discounting trains cancel-for-deals behavior in your other customers AND loses this one at next renewal anyway, now at lower MRR. Pause is the right save here. Pausers reactivate at 60-80%; usage-discount-takers churn one cycle later.

- **Save discounts above 50%.** Once your save discount crosses 50%, you have created a documented acquisition path: cancel, get 50% off, re-subscribe. You will see repeat offenders. Beyond the loop risk, you permanently lower the LTV of your saved cohort. Cap at 30% and hold it.

- **Pauses longer than 3 months.** Reactivation drops sharply past 90 days because the customer's context has changed and coming back feels like starting over. Cap pause at 3 months. Send a reactivation reminder when the period ends and make resume one click.

- **No post-cancel reactivation path.** Six months later, the situation that drove the cancel may have resolved. If the only path back is full signup, you lose a portion of these to friction. A one-click reactivation email with the previous plan pre-filled converts meaningfully better. Add a 30/60/90-day win-back sequence post-cancel.

- **Founder stops reading cancel reasons after month 2.** The exit survey routes offers, yes. But the highest-signal input your product receives is why paying customers leave. Reading 10 reasons per week takes 5 minutes and surfaces things no analytics dashboard does. Delegate this entirely to automation and you stop learning from your most honest customers.

### Measurement (the mistakes upstream of everything else)

These make the rest of the skill harder to apply, because the numbers you're optimizing for are wrong.

- **Wrong denominator.** Using end-of-period subscriber count (or an average) is incoherent, not just imprecise. It mixes acquisition signal into the churn rate. The formula requires start-of-period count. See §3.

- **Averaging churn rates across cohorts or months.** Churn rates are not averageable. Two months at 5% and 7% is not 6%. Compute pooled rates from the underlying cancellation and start-of-period counts. Averaging is how founders end up reporting numbers that look stable while underlying churn is moving.

- **Using MRR churn with annual plans.** A monthly-to-annual switch shows up as a downsell in the MRR churn formula, inflating your reported churn rate. If you offer annual billing, report standard account-based churn as the primary number; use MRR churn only as a secondary view.

- **Reporting net revenue retention as "churn".** Net MRR churn (gross MRR churn minus expansion) can look healthy or negative while you're losing customers at a high rate. Companies that quote NRR to their team as the churn metric are hiding cancels behind expansion revenue. Report standard churn, gross MRR churn, and NRR as three separate numbers.
