# The three kinds of churn

Most retention advice splits churn into two buckets: voluntary (customer chose to leave) and involuntary (payment failed). That split is useful for routing work, but it misses the category that determines whether your cancel flow even gets a chance to run.

| Kind | What it is | Typical share | Where this skill solves it |
|---|---|---|---|
| Voluntary | Customer clicks Cancel and intends to leave | 50–70% of total churn | §3 Cancel flow design, §4 Reason capture, §5 Save offer matrix |
| Involuntary | Payment fails; customer loses access without deciding to leave | 30–50% of total churn | §7 Dunning and payment recovery |
| Experience | Customer has mentally left but has not clicked anything yet | Not a percentage, it's a leading indicator | §6 Churn prediction |

### Voluntary churn

This is the number most founders track and obsess over, and it's the right place to start. The customer made a conscious choice. They reached the cancel button, or found it on their own. Your job is to interrupt that decision with good information: a reason-capture question, a matched save offer, and a frictionless exit if they still want to leave.

The baseline save rate for a well-designed cancel flow runs 20–35%. That number is recoverable from what most DIY flows leave behind.

Voluntary churn is the focus of sections 3 through 5.

### Involuntary churn

A failed payment is not a cancellation decision. The customer still wants your product. They just have a card that expired, a bank that flagged the charge, or a payment method that was never updated after they switched banks.

This makes involuntary churn the easiest kind to recover. There is no objection to overcome, no competing product to beat, no value gap to close. The only problem is a failed transaction. Smart retry logic, Stripe's automatic card updater, and a short dunning sequence recover 40–60% of these customers with almost no friction on their end.

Involuntary churn sits at 30–50% of total churn for most SaaS products. That share is large enough that fixing it without touching your cancel flow can meaningfully move your monthly retention number.

Involuntary churn is covered in §7.

### Experience churn

This is the kind the first two categories miss entirely. The customer is still active. They are still paying. They have not opened the cancel flow. But they have already made the decision.

They stopped logging in daily. They used the core feature less and less over the past three weeks. They visited your billing page twice in the last ten days but did not cancel. They opened two support tickets that went unresolved.

By the time this customer clicks Cancel, the actual decision was made weeks ago. The cancel flow is too late. The save offer will feel hollow because the emotional work of leaving is already done. You are trying to change a mind that has been made up.

Experience churn is not a percentage you can read off a dashboard. It is a signal in your usage data. The customers showing these patterns are in a pre-cancel state, and they are reachable before they reach your cancel button. That is a meaningfully different intervention than anything in sections 3 through 5.

The right response to experience churn is proactive outreach triggered by behavioral signals, not reactive friction at the cancel screen. What signals to watch, how to score them, and what to do when the score drops below a threshold is the subject of §6.

### Why the third category matters

If you only build a cancel flow and a dunning sequence, you are solving for customers who have already decided to leave and customers whose cards failed. You are not touching the customers who are drifting toward the exit in silence.

For most products, experience churn is a larger pool than the cancel-flow numbers suggest, because these customers never surface in your save rate data. They churned before you had a chance to count them.

Section 6 covers the signals, the scoring approach, and the outreach patterns that reach these customers while there is still something to save.
