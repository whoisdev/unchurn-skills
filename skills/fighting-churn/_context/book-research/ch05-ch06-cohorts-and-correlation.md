# Fighting Churn with Data: Chapters 5 and 6
## Cohort Analysis, Metric Scoring, and Behavioral Correlation

---

## Chapter 5: Metric Cohort Analysis and Scoring

### 5.1 Metric Cohort Analysis

Cohort analysis is the foundational method for discovering which behaviors predict churn. The core idea: divide customers into groups by metric value, then measure churn in each group. If behavior predicts retention, the most active cohort should have the lowest churn rate.

**The exact method (5 steps):**

1. Start from a dataset where each row is one customer observation on one date, including metric values and a churn/renew indicator.
2. Extract only the metric of interest and the churn indicator. Ignore account identity and date.
3. Sort by metric value and divide into N equal-size groups using quantile-based discretization (Pandas `qcut`). The default is 10 cohorts (deciles). Cohort boundaries are derived from the data, not chosen in advance.
4. For each cohort, compute: (a) the average metric value, (b) the churn rate (percentage of churns in that cohort).
5. Plot cohort average on the x-axis vs. churn rate on the y-axis.

**Important: one observation is not one customer.** The same customer appears multiple times if they renewed. A cohort is a group of observations, not a group of distinct customers. Carl says not to explain this detail to business colleagues because it creates confusion.

**Python implementation:**
```python
groups = pd.qcut(churn_data[metric_to_plot], ncohort, duplicates='drop')
cohort_means = churn_data.groupby(groups)[metric_to_plot].mean()
cohort_churns = churn_data.groupby(groups)['is_churn'].mean()
```

**What the results look like in practice:** Across Broadly, Klipfolio, and Versature case studies, churn falls sharply in the first few cohorts and then levels off. The inflection point where churn stops declining is the "healthy level" for that metric. Past that point, more of the behavior does not reduce churn further.

**Minimum cohort size:** At least 200 to 300 observations per cohort, preferably thousands. Also aim for at least 100 total churn events in the dataset. If churn rate is below 10%, you need more data before cohort analysis is reliable.

**Significant vs. insignificant results:** A relationship is worth taking seriously if churn rates are consistent across cohorts (monotonically decreasing or increasing) and the top-to-bottom ratio is at least 1.5x. If unsure, focus on metrics with obviously strong relationships first.

**Edge cases:**
- Metrics where most customers have zero: `qcut` will produce fewer cohorts than requested. Remove zero-value observations using `min_valid` filtering before re-running, then use fewer cohorts.
- Free/trial customers: Remove non-paying customers before analysis. They rarely churn regardless of behavior and distort the relationship for paying customers.
- Disengaging behaviors: Rare. When they exist, the churn rate actually rises with more of the behavior. Usually only visible after removing non-users from the cohort.

**Causality warning:** Correlation with churn is not causation. Only behaviors that directly deliver utility or enjoyment to the customer can be considered causal. For associated-but-not-causal behaviors, trying to increase that behavior in customers will not actually reduce churn.

### 5.2 Summarizing Customer Metrics

Before running cohort analyses, compute summary statistics for every metric. This is quality assurance and prevents wasted analysis time.

**Key statistics to compute:**
- Percentage non-zero (how rare the behavior is)
- Mean and standard deviation
- Skew (above 4 or 5 is significantly skewed; behavioral metrics often reach 15+ skew)
- Percentiles: 1%, 5%, 25%, 50% (median), 75%, 99%
- Minimum and maximum

**Screening rare metrics:** Remove metrics with fewer than 5-10% non-zero values. They apply to too few accounts to be practically useful even if they correlate with churn.

**Involve business stakeholders** in reviewing summary stats before sharing cohort results. If data quality issues surface after sharing findings, you lose credibility.

### 5.3 Scoring Metrics

Scoring is Carl's term for normalizing/standardizing metrics. He uses "scoring" because business people find "normalization" intimidating.

**Why score:** Heavily skewed metrics produce cohort plots where most cohorts are compressed in a small range of the x-axis, making the chart hard to read. Scores redistribute the cohorts more evenly and reveal how cohorts compare to the average (score = 0).

**The scoring algorithm:**

1. Check if the metric's skew statistic exceeds the threshold (default: 4.0) AND the minimum value is zero. If not skewed, skip to step 4.
2. Add 1 to every metric value (so zeros become 1, enabling log).
3. Take the natural log: `log(1 + metric)`.
4. Calculate mean and standard deviation of the values at this point.
5. Subtract the mean from every value.
6. Divide by the standard deviation.
7. Result is the score.

**Properties of scores:** Mean is always 0. Standard deviation is always 1. Range is roughly -5 to +5. Order of customers is preserved (higher metric always maps to higher score). Cohorts built from scores are identical to cohorts built from original metrics.

**Python:**
```python
skewed_columns = (stats['skew'] > skew_thresh) & (stats['min'] >= 0)
for col in skewed_columns.keys():
    data_scores[col] = np.log(1.0 + data_scores[col])
data_scores = (data_scores - stats['mean']) / stats['std']
```

**Practical result:** In the Broadly case study, a transactions metric with skew = 23 had 7 cohorts compressed into 1/8 of the plot on its natural scale. After scoring, cohorts spread evenly from -1.5 to +2.0, and the churn relationship became clearly readable.

### 5.4 Removing Unwanted Observations

Use `min_valid` and `max_valid` dictionaries to filter the DataFrame before analysis:
```python
clean_data = clean_data[clean_data[metric] > min_valid[metric]]
```

Two main cases: (1) remove free/zero-MRR customers who do not follow the normal behavior-churn relationship, (2) remove zero-value observations when analyzing rare events.

### 5.5 Segmenting Customers

Once cohort analysis reveals which metric levels correspond to higher churn, segment current customers (not the historical dataset) by those thresholds.

**Three segmentation strategies:**
1. Choose the metric level where churn risk exceeds a threshold (for example, where cohort churn rate exceeds 10%).
2. Pick the bottom N customers by metric value when budget constrains outreach volume.
3. Target intermediate-risk customers, not the most disengaged. The most disengaged customers often do not respond to interventions and wasted outreach may disengage them further.

---

## Chapter 6: Correlation Between Behaviors

### 6.1 Correlation Between Metrics

**Why this matters:** Most products generate dozens of correlated behavioral metrics. Looking at individual cohort plots creates information overload without a way to understand how they relate. Groups of correlated behaviors often show a clearer, stronger relationship to churn than any individual metric.

**Correlation coefficient:** Ranges from -1 to +1. Measures how consistently an increase in one metric is associated with an increase (or decrease) in another. Insensitive to scale and units.

**Thresholds Carl uses:**
- Above 0.7: high correlation
- 0.3 to 0.7: moderate correlation
- Below 0.3: weak correlation

**Negative correlation** (one metric goes up when the other goes down) is rare in behavioral count metrics because customers who do more of anything tend to do more of everything. Negative correlations are more common in ratio metrics (chapter 7).

**Scores increase measured correlation:** Metric scores are often significantly more correlated than the same metrics on their natural scale, especially when skewed. This is another reason to always work with scores.

**Python (single pair):**
```python
corr = met1_series.corr(met2_series)  # Pearson correlation
```

**Correlation matrix:** A table of all pairwise correlations. Calculated with `DataFrame.corr()`. For exploration and presentation, use a color-coded heatmap in a spreadsheet (conditional formatting). Static images are impractical for more than 15-20 metrics.

```python
corr_df = churn_data.corr()
corr_df.to_csv(save_name)
```

Klipfolio had around 70 metrics. Their correlation matrix showed 6 distinct groups of highly correlated metrics. This structure is typical.

### 6.2 Averaging Groups of Correlated Metrics

**Core technique:** When multiple metrics are moderately to highly correlated, average their scores together into a single group score. This reduces information overload and produces a cleaner, stronger relationship with churn.

**Why averaging scores works:** Each score measures position relative to the average (score = 0). Because all scores share the same scale and units (standard deviations from average), averaging them is meaningful regardless of what the original metrics measured. A customer who is above average on both logins and edits is fairly described as an above-average user overall.

**Averaging often reveals stronger churn signal:** In the Klipfolio case study, the average score for the main group of viewing and editing behaviors showed the top cohort churning at less than one-tenth the rate of the bottom cohort. Individual metrics from the same group had shown less separation (roughly one-third to one-half difference from top to bottom cohort).

**Loading matrix:** A table of weights for forming the averages. Metrics are in rows, groups are in columns. The weight for each metric in a group is approximately 1/N, but slightly higher (see equation 6.3 below).

**Matrix multiplication:**
```python
grouped_ndarray = np.matmul(ndarray_2group, load_mat_ndarray)
```

**Why weights are higher than 1/N:** Averaging N scores with 1/N weights produces a result with a standard deviation less than 1. To keep the average score as a proper score (standard deviation = 1), the weights are adjusted by multiplying 1/N by the factor 1/sqrt(correlation_threshold). With a threshold of 0.5, the factor is sqrt(2) = 1.41. With 0.6, it is sqrt(1/0.6) = 1.29.

### 6.3 Discovering Groups of Correlated Metrics

**Algorithm: hierarchical agglomerative clustering on the correlation matrix.**

The algorithm works bottom-up, greedily combining the most similar items:

1. Find the highest correlation in the current correlation matrix.
2. Update the loading matrix to group those two metrics together.
3. Apply the loading matrix to create a new dataset where those two are averaged.
4. Recalculate the correlation matrix on the new dataset.
5. Repeat steps 1-4 until all remaining correlations fall below the threshold.

**Dissimilarity conversion:** The Scipy `linkage` function works on dissimilarity, not correlation. Convert: `dissimilarity = 1.0 - correlation`. The threshold is also converted: `diss_thresh = 1.0 - corr_thresh`.

**Python implementation:**
```python
from scipy.cluster.hierarchy import linkage, fcluster
from scipy.spatial.distance import squareform

dissimilarity = 1.0 - corr
hierarchy = linkage(squareform(dissimilarity), method='single')
labels = fcluster(hierarchy, diss_thresh, criterion='distance')
```

**Setting the threshold:** Carl recommends:
- Stay in the range 0.4 to 0.7. Never above 0.8 or below 0.3.
- Default starting point: 0.5.
- Use binary search: if 0.5 gives one big group, try 0.6; if 0.6 still seems too low, try 0.65.
- Evaluate by looking at whether the grouped metrics make business sense, not by an algorithmic criterion.
- Small changes (0.01 to 0.02 in threshold) can have large effects on groupings; expect this.

**Comparing HC to PCA:** Hierarchical clustering produces similar block structure to PCA but with only positive weights, making it directly interpretable as an average. PCA produces negative weights (differences between behaviors) that are hard to explain to business stakeholders.

### 6.4 Explaining Correlations to the Business

Carl's recommended presentation sequence:

1. Ask how much statistics the audience knows and calibrate your language.
2. Show scatter plots of individual metric pairs from their own data. Use the term "correlation" but drop "coefficient." People intuitively understand which behaviors go together.
3. Show the heatmap (use full color, not grayscale). Describe it as a "correlation heatmap," not a "correlation matrix."
4. Outline the groups in the heatmap and provide a list of which metrics are in each group. Explain the groups came from the data automatically, not from your judgment.
5. Compare the cohort churn analysis on grouped vs. individual metrics.

**Terms to avoid with business stakeholders:** matrix, loading, matrix multiplication, hierarchical agglomerative clustering.

### What Counts as a Useful Correlation vs. Noise

Carl does not specify a hard cutoff for "useful" but the practical criteria he uses:
- Correlation above 0.5 is actionable for grouping purposes.
- Correlation above 0.7 is high enough that the metrics are measuring essentially the same behavior.
- Weak correlations below 0.3 do not warrant grouping.
- Business sense matters as much as the number: metrics grouped by the algorithm should correspond to behaviors that a product person would intuitively expect to go together.
- After forming groups, the grouped cohort analysis should show a stronger churn signal than individual metrics. If it does not, the grouping may be wrong.
