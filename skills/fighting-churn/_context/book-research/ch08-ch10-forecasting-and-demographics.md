# Chapters 8-10: Forecasting, Model Accuracy, and Demographics

Source: Fighting Churn with Data (MEAP V08) by Carl Gold

---

## Chapter 8: Churn Forecasting with Logistic Regression

### What kind of regression

Carl uses **logistic regression** throughout, specifically sklearn's `LogisticRegression` with L1 penalty and the liblinear solver. He frames the model as predicting **retention probability** (not churn probability directly), because positive weights then map to good outcomes, which is easier to explain to business people. Churn probability = 1 minus the retention probability.

The math is a standard logistic (sigmoid) curve: engagement is estimated as a weighted sum of metric scores, and that engagement feeds the S-curve to produce a retention probability. The model has two outputs: engagement weights per metric, and an offset (intercept) that shifts the S-curve to match the product's actual average churn rate.

### Data preparation requirements

This is the heaviest part of the chapter. Before fitting a regression, a reader needs all of the following:

1. A historical dataset of customer observations with labeled churn outcomes, built per the Chapter 4 observation framework.
2. Summary statistics (mean, standard deviation, skew percentiles) computed on that dataset.
3. Metrics converted from raw counts to **scores** (log-transformed for skew, clipped at 1st and 99th percentile for fat tails, then z-scored).
4. Correlated metrics identified and **averaged into groups** using a loading matrix, so the regression does not receive highly correlated inputs.

The loading matrix and score parameters must be saved and reapplied exactly when scoring current customer data later. Carl is explicit that preparing current-customer data for forecasting requires re-running every transformation step from the historical fit: if the steps diverge, the forecast will drift.

That pipeline is roughly 5-7 Python scripts stacked on each other (Chapters 5-7) before the regression even runs.

### Pitfalls of churn forecasting (8.5)

1. **Correlated metrics break regression.** Logistic regression assumes uncorrelated inputs. Using raw correlated metrics produces unreliable engagement weights and makes interpreting the model misleading. Solution: metric grouping (Chapter 6) must precede regression.

2. **Outliers in current data distort forecasts.** When scoring current customers, extreme outlier values must be clipped to the 1st and 99th percentile thresholds from the historical fit. Without clipping, a single customer with abnormal behavior can receive a near-0% or near-100% forecast, and the average calibration drifts.

3. **Forecast drift.** If the average churn probability on current customers differs noticeably from the historical dataset churn rate, the model may be stale. Carl says to investigate the difference by comparing current vs. historical summary statistics. If product or market conditions have changed materially, a new historical dataset is needed before forecasts are reliable.

4. **Forecast probabilities above 99% are a red flag.** Carl warns (section 8.1 callout) that probabilities near certainty usually indicate a data problem, not a genuine insight.

### Customer lifetime value (8.6)

Carl ties LTV directly to the logistic regression output. The core formula:

- Expected lifetime (months) = 1 / monthly churn probability
- CLV (for retention) = (Margin * Recurring Revenue / Churn Probability) minus one period of revenue and COGS

The model produces an individual churn probability per customer, which means CLV is computed per customer, not just as a fleet average. That is the value: not that the formula is novel, but that per-customer forecasted churn turns a fleet average metric into a per-account number useful for prioritizing interventions.

One important caveat Carl includes: for companies with churn below 20% per year (expected lifetimes over 5 years), a discount factor should be applied to the CLV formula to account for non-churn risks over long horizons. He cites Gupta and Lehman (2003) for the full discounted formula.

Carl also warns: **XGBoost forecasts are not calibrated** and must not be used for LTV calculations. Only the logistic regression produces probability values that match observed churn rates and are safe to plug into the lifetime formula.

### Volume threshold: when does regression pay back?

Carl does not give an MRR number. He gives an **observations-per-cohort** number: every cohort should have at least 200-300 observations, preferably thousands. For a monthly SaaS product, a customer-month is one observation. So 500 customers with 6 months of history = 3,000 observations, which is borderline workable for cohort analysis but tight for a regression with multiple metrics. Carl cites the simulation as using "tens of thousands to around 100,000 customers" when discussing database scale.

Translating to Stripe SaaS reality: a product at $5K-$60K MRR with 100-500 subscribers and 1-3 years of history likely has enough observations to run regression (a few thousand customer-months). But the total time investment (5+ chapters of data prep, Python infrastructure, backtest validation) is substantial. The honest threshold is probably: regression starts to be worth the cost when you have a dedicated analyst and more than 500 active subscribers, or when an expensive intervention (like proactive outreach or dedicated CSM time) is being targeted and you need to rank risk precisely rather than just segmenting by behavioral metrics.

---

## Chapter 9: Forecast Accuracy and Machine Learning

### How to validate churn models

Carl uses two metrics instead of standard classification accuracy (which is misleading for churn because predicting no one churns produces high accuracy by default):

**AUC (Area Under the ROC Curve):** The probability that the model ranks a true churn above a true non-churn in a random pairwise comparison. Healthy range for churn is 0.6-0.8. Below 0.55 means the model is barely better than random. Above 0.85 is suspicious and usually means a data leak (short lookahead period, or future information in the feature set).

**Top Decile Lift:** The churn rate in the top 10% of highest-risk customers divided by the overall churn rate. Healthy range is 2.0-5.0 for low-churn products (under 10% monthly), 1.5-3.0 for high-churn products. Below 1.5 means the model is barely useful for targeting.

Accuracy is measured via **backtesting** (time-series cross-validation), specifically `TimeSeriesSplit` from sklearn. The data is split preserving time order so that earlier periods train the model and later periods test it, mimicking real-world forecasting.

The regression has one key tuning parameter, **C** (controls the number of metrics with nonzero weights). Carl recommends testing C on a log scale (1, 0.1, 0.01, 0.001) rather than a linear one. The optimal C is found by backtesting: reduce weights until accuracy drops, then back off. In practice, he finds you can often zero out 30-50% of metrics with no accuracy loss, and sometimes with a small gain.

### Forecasting churn risk with XGBoost (9.5)

XGBoost is an ensemble of decision trees optimized by gradient boosting. It typically achieves AUC 0.02-0.06 higher than logistic regression in Carl's case studies, and lift improvement of 0.1-0.5. In practical terms, XGBoost using advanced metrics achieves roughly the same AUC as logistic regression, and the additional lift improvement is modest. Carl is measured about the hype: "churn will always be hard to predict due to factors like subjectivity, imperfect information, rarity, and extraneous factors influencing the timing of churn."

Key tradeoff: XGBoost takes roughly 40x longer to train than regression and requires tuning 4 parameters across a grid of 256 combinations. The payoff is real but not transformative for most products.

XGBoost does not require metric scoring or grouping. It uses raw metric values because decision-tree rules operate on actual cutpoints, not z-scores. Correlation between metrics is harmless (and potentially helpful, because diverse features give the tree ensemble more split options).

### Segmenting customers by risk (9.6)

The output of either model is a per-customer churn probability (or rank score for XGBoost). The practical use is to sort customers by this score and select the top N for an intervention. Carl does not recommend a fixed threshold to define "at risk." Instead the business team picks based on available budget or capacity, and may intentionally avoid the most extreme risk customers (who may be unsaveable) in favor of the above-average but reachable segment.

---

## Chapter 10: Demographics and Firmographics

### What predicts churn beyond behavior (10.1)

Demographic and firmographic fields are static facts about a customer: acquisition channel, geography, company size, industry, technology stack, founding date, and similar. Carl is explicit that these are the **weakest** predictors of churn relative to behavioral metrics. In the simulation the strongest demographic signal came from the acquisition channel (app store 1 vs. app store 2 vs. web), with churn rate differences that were statistically significant. Country/region effects were much smaller and often not significant.

For B2B SaaS (which is most of our reader base), useful firmographic fields include: company stage (startup vs. growth vs. established), number of employees, industry vertical, and technology stack. These differ from behavioral metrics in that you cannot influence them with a retention intervention. Their use is for acquisition targeting, not retention tactics.

### Cohorts with demographic data (10.2)

Because demographic categories (strings) do not have a natural order, cohort analysis for these fields requires **confidence intervals** around the churn rate, not just the measured rate. Carl uses Wilson confidence intervals (`statsmodels.stats.proportion.proportion_confint`). Two categories have a statistically significant difference only if their confidence intervals do not overlap.

Critical practical note: confidence intervals narrow as sample size grows. Carl's 200-300 observation minimum per cohort applies here too. Sparse demographic categories (a country with 30 customers) will have very wide confidence intervals and provide no actionable signal. The solution is grouping: combine similar countries into regions, group similar industry verticals, etc.

Carl explicitly notes that in his experience, demographic cohorts show weaker churn relationships than behavioral cohorts. This is a key caveat for skill readers.

### Date-based churn analysis (10.4)

Date-type demographic fields (date of birth, company founding date) must first be converted to numeric intervals by subtracting the date from the observation date. This produces a numeric field (customer age in years, company tenure in years) that is then analyzed exactly like a behavioral metric using standard cohort plots. No special technique is needed once the date becomes a number.

In the simulation, older customers had higher churn, but the effect was weak relative to behavioral metrics. Carl suggests tenure at the time of observation is the most practically useful date-based signal for SaaS.

### Demographic forecasting (10.5)

To use string demographics in a regression or XGBoost model, they must be converted to **dummy variables** (one-hot encoding): one binary column per category, with 1 for membership and 0 otherwise. Carl uses Pandas `get_dummies`. Sparse categories should be grouped first.

Dummy variables must be kept separate from behavioral metrics in the pipeline: metrics go through scoring and grouping, then the processed metric scores and dummy variables are merged for the final dataset. The metric grouping algorithm must not be run on dummy variables because categories within the same field are negatively correlated by definition (belonging to one category excludes all others), which would cause the grouping algorithm to treat them incorrectly.

In the simulation, demographics alone achieved AUC of only ~0.56 and lift ~1.5. Combined with behavioral metrics, AUC rose to the same range as metrics alone (0.65-0.75). The demographic contribution was small. This finding likely generalizes: for most SaaS products, the improvement in forecast accuracy from adding demographic/firmographic data is modest.

---

## Honest Take for Our Reader

**Regression forecasting** is a Chapter 8-9 project, not a Chapter 8 task. It requires the full data pipeline from Chapters 4-7 as a prerequisite. A Stripe SaaS founder at $5K-$60K MRR almost certainly does not have a Python data pipeline producing scored, grouped customer metrics on a historical dataset. Building that pipeline from scratch is a multi-week project requiring a data analyst or engineer.

**When it starts to pay back.** The quantitative answer from the book: you need hundreds of observations per cohort (200-300 minimum), meaning roughly 500+ active subscribers with 6+ months of history. More importantly, regression only pays back if you have an expensive intervention to target (personalized outreach, custom offers, high-touch retention sequences) where ranking risk precisely matters. If you're using Unchurn's cancel-flow widget, the segmentation happens at the moment of cancellation intent and does not benefit from pre-computed churn scores.

**The skill implication.** These three chapters are best served as a **pointer with caveats**, not full integration. The signal worth surfacing to readers is:

1. If you have 500+ subscribers and a dedicated analyst, regression-based risk scoring is real and achievable. Carl's book is the right reference for the implementation.
2. AUC 0.6-0.8 and top-decile lift 2-4 are the realistic benchmarks. Anything outside that range means a data problem.
3. CLV = (margin * MRR) / monthly churn probability is the useful practical formula, and you do not need regression to use it (just use the fleet average or segment averages).
4. Demographics are the weakest lever. Acquisition channel is usually the strongest demographic signal, and even that is more useful for fixing acquisition than for retention.
5. XGBoost is more accurate than regression but does not produce calibrated probabilities, so do not use it for CLV calculations.

Full integration into the skill (worked examples, code walkthroughs) would be overkill for the $5K-$60K MRR reader. A clear pointer to Chapter 8 plus the CLV formula and the AUC/lift benchmarks is the right scope.
