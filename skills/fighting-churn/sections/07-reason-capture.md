## 7. Reason capture: branching, not dropdowns

The single most common mistake in cancel flow design is the "Other" option. Remove it, and your reason data becomes actionable. Keep it, and you have built a machine that produces noise.

### Why "Other" destroys your data

When "Other" is available, roughly 30-40% of canceling users select it. That makes it, reliably, your largest cancel reason. And it tells you nothing. The free-text field behind it fills with phrases like "not for me," "decided to go a different direction," and "personal reasons." You cannot route an offer to that. You cannot fix a product gap you cannot name. You cannot segment a cohort from it.

The instinct that produces "Other" is reasonable: you do not want users to feel boxed in. But the effect is the opposite of what you want. A user who does not see their reason in your list will pick the closest match if "Other" is not there. That closest match is signal. "Other" absorbs it.

### The fix is not better options. It is branching.

You do not need eight top-level reasons to cover all cases. Five or six is correct. What you need is a second question that branches off each one. The parent question gives you the bucket. The follow-up gives you the cause you can act on.

One screen is fine for this if you reveal the follow-up inline once the parent is selected. Two screens is acceptable. Three is too many.

### The branching tree

| Parent reason | Follow-up question | Follow-up options |
|---|---|---|
| Too expensive | Compared to what? | A specific competitor; the value I'm getting; my budget changed; annual feels like too big a commitment |
| Not using it enough | What got in the way? | Did not have time; forgot it existed; could not figure out a specific feature; project ended |
| Missing a feature | Which feature? | Text field, required |
| Switching tools | Which one? | Text field with autocomplete on common competitors |
| Technical issues | (no follow-up, route to support immediately, skip save offer) | |
| Something else | Tell us more | Single text field |

A few notes on this structure:

"Technical issues" is the one reason where you skip the save offer entirely and route straight to support. A user leaving because your product is broken does not need a discount. They need a fix. Offering a coupon to someone who cannot log in reads as tone-deaf and makes support harder to reach.

"Something else" goes at the bottom of the list, visually separated from the five real reasons. It is not "Other." It does not sit in the grid alongside "Too expensive" and "Not using it enough." It is a small link or a lightly styled row that says "I don't see my reason here." Tucked. The user who genuinely cannot find their reason will find it. Everyone else will pick a real option.

The text field for "Missing a feature" is required because the parent reason already gives you the bucket. You know the user is leaving over a missing feature. The only value is the name of the feature. Leaving the field optional means half your respondents skip it and you get "Missing a feature (no detail)" as a category with no next action.

### What this looks like on one screen

```
Why are you canceling?

  ( ) Too expensive
  ( ) Not using it enough
  ( ) Missing a feature
  ( ) Switching tools
  ( ) Technical issues
      ─────────────────────
  ( ) Something else

[ user selects "Too expensive" ]

  Compared to what?

  ( ) A specific competitor
  ( ) The value I'm getting
  ( ) My budget changed
  ( ) Annual feels like too big a commitment

                              [ Continue ]
```

The follow-up appears below the parent selection, inline, without a page transition. The user sees one decision at a time. The form still fits on one screen.

### What the data looks like before and after

Before branching, your reason report looks like this:

| Reason | Share |
|---|---|
| Other | 34% |
| Too expensive | 28% |
| Not using it enough | 19% |
| Missing a feature | 11% |
| Switching tools | 8% |

The largest category is noise. You make no decisions from it.

After branching, "Too expensive" at 28% becomes:

| Too expensive, sub-reason | Share of parent |
|---|---|
| Switched to a competitor | 23% |
| Budget changed | 18% |
| Annual commitment friction | 11% |
| Perceived value gap | 8% |

Now you have four separate problems with four separate fixes. Competitor-switch gets a retention offer with a direct comparison. Budget-change gets a pause or a downgrade path. Annual-commitment friction is a pricing packaging problem you can address. Perceived value gap tells you the user did not understand what they were getting, which is an onboarding problem.

None of that is visible when 34% of your data is "Other."

### Constraints

Keep the parent list between five and seven options. Fewer than five and you are forcing users to round up too aggressively. More than seven and the cognitive load causes people to stop reading and click whatever is first or nearest the "confirm cancel" button.

Do not add free text at the parent level. Free text at the top of the flow produces the same noise problem as "Other." Reserve it for one place: the follow-up on "Missing a feature," where the parent has already scoped the problem and the text field has a clear job.

The follow-up options for each parent should be exhaustive enough that "Something else" at the follow-up level is almost never needed. If you are seeing high "Something else" volume at the parent level, that is a signal your five top-level reasons are wrong, not that you need a sixth real option.
