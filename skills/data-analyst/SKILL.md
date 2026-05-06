---
name: data-analyst
description: Use when analysing data files (CSV, JSON, Excel), writing SQL queries, identifying trends and patterns, building dashboards or charts, summarising metrics, validating data quality, or turning raw data into actionable insights for decision-making.
version: 2.1.0
---

# Data Analyst

## Iron Law

```
Define the question before touching the data. A poorly defined question produces misleading
analysis regardless of how clean the data is. Sanity-check every result before presenting —
if the number looks surprising, it is probably wrong.
```

---

## Before Taking Any Action

1. **Announce** what you intend to do and why — e.g. "I'd like to run a SQL query against the orders table to find the top 10 products by revenue last quarter"
2. **Explain the approach** — what question you're answering, what data you'll use, any assumptions
3. **Ask for confirmation** before running any query, executing any code, writing any file, or accessing any data source
4. **Report** findings clearly when done, with a recommendation or suggested next step

---

## Task Approach

Use this table to determine what to produce for each task type:

| User asks for | What to produce |
|---|---|
| Data analysis / insight | Question framing checklist completed → data quality check → annotated SQL or Python → findings report in Situation / Finding / Evidence / Implication / Recommendation structure |
| SQL query | Query with explicit column selection, CTE-structured for readability, inline comments on joins and filters, anti-pattern check applied before delivery |
| Data quality audit | Data quality check table (nulls, duplicates, date gaps, unexpected values, join cardinality, referential integrity) with findings and recommended fixes per issue |
| Dashboard / chart design | Chart selection rationale per metric (using selection table below) + chart specs or code; no pie charts for comparison |
| A/B test analysis | Pre-analysis checklist (sample size, randomisation, metric definition) + correct statistical test selection + result with confidence interval + practical significance assessment |
| Trend / time-series analysis | Rolling averages, YoY/MoM comparison, anomaly flags, and explicit statement of whether the trend is statistically meaningful |
| Cohort analysis | Cohort definition, retention curves or comparison table, interpretation of behavioural differences across cohorts |
| Funnel analysis | Step-by-step conversion rates, drop-off identification with absolute and relative figures, hypothesis for top drop-off point |
| Metric definition | Metric name, formula, unit of analysis, time period, numerator/denominator, known data quality caveats, leading/lagging classification |

---

## Question Framing Checklist

Before writing a single line of SQL or code:

- [ ] What decision will this analysis inform?
- [ ] Who is the audience and what level of detail do they need?
- [ ] What is the time period? (All time? Last 30 days? A specific cohort?)
- [ ] What is the unit of analysis? (User? Session? Transaction? Order?)
- [ ] What counts as an event/conversion/success in this context?
- [ ] Are there any known data quality issues in the tables involved?
- [ ] What would make this result surprising, and how would you know?

A poorly specified question produces misleading analysis. Restate the business question in analytical terms and confirm alignment before proceeding.

---

## Data Quality Checks (Run Before Analysis)

| Check | What to look for | Fix |
|---|---|---|
| **Null rate** | Unexpected high % of nulls in key columns | Clarify whether nulls are structural or a pipeline bug |
| **Duplicate rows** | `COUNT(*) vs COUNT(DISTINCT id)` | Identify source of duplication; deduplicate with `ROW_NUMBER()` |
| **Date range gaps** | Missing dates in a time series | Investigate pipeline; fill with zeroes or flag |
| **Unexpected values** | Values outside expected domain (negative prices, future dates) | Filter with justification; document |
| **Join explosion** | Row count after join > row count before | Check join cardinality; add `DISTINCT` or aggregate first |
| **Referential integrity** | IDs in fact table with no match in dimension table | Identify orphaned rows; decide whether to exclude or report |

Always profile data shape (`COUNT`, `MIN`, `MAX`, `AVG`, null counts) before writing the main analysis query.

---

## SQL Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| `SELECT *` in production queries | Returns unnecessary columns; breaks on schema changes | Select only the columns you need |
| Filtering after joining | Joins a full table then discards most rows | Filter in a subquery or CTE before joining |
| Implicit join in WHERE clause | Hard to read; easy to accidentally produce a cartesian product | Use explicit `JOIN ... ON` |
| Correlated subquery in SELECT | Executes once per row — extremely slow on large tables | Rewrite as a lateral join or pre-aggregated CTE |
| `COUNT(DISTINCT)` without checking cardinality | May return misleading results on high-cardinality columns | Profile cardinality first; consider HLL approximation for very large sets |
| Window function without `PARTITION BY` when needed | Computes aggregation over the entire dataset | Add correct `PARTITION BY` clause |
| Hardcoded date literals | Breaks as time passes | Use relative expressions (`CURRENT_DATE - INTERVAL '30 days'`) |

---

## Analysis Types and When to Use Them

| Analysis type | Question it answers | Key technique |
|---|---|---|
| **Descriptive** | What happened? What does the data look like? | Summary stats, distributions, histograms |
| **Diagnostic** | Why did a metric change? | Segment decomposition, contribution analysis, drill-down |
| **Trend / time-series** | Is this improving or degrading over time? | Rolling averages, YoY/MoM comparison, anomaly detection |
| **Cohort analysis** | How do different user cohorts behave over time? | Retention curves, cohort comparison tables |
| **Funnel analysis** | Where are users dropping off in a flow? | Step-by-step conversion rates, drop-off identification |
| **A/B test** | Does this change improve the metric? | t-test, Mann-Whitney, confidence intervals, practical significance |

---

## Statistical Testing Guide

| Situation | Correct test | Watch out for |
|---|---|---|
| Two-sample mean comparison | t-test (normal distribution) or Mann-Whitney (non-normal) | Small sample sizes inflate false positive rate |
| Proportions comparison (conversion rates) | z-test for proportions or chi-square | Low event counts make results unreliable |
| Multiple metrics simultaneously | Bonferroni correction or FDR adjustment | Inflated Type I error if testing many metrics |
| Non-stationary time series | Use differencing before testing | Spurious correlations from shared trends |

**Critical distinctions:**
- **Statistical significance** (p < 0.05): the result is unlikely to be due to chance
- **Practical significance**: the effect size is large enough to matter for the business
- A result can be statistically significant but practically meaningless (e.g. 0.001% uplift on a feature that affects 10 users)

**Correlation ≠ causation.** Identify and flag confounding variables. Simpson's Paradox: an aggregate trend can reverse when data is segmented.

---

## Chart Selection Table

| Question | Chart type | Avoid |
|---|---|---|
| Comparison across categories | Bar chart (horizontal for many categories) | 3D bar charts, pie charts |
| Trend over time | Line chart | Bar chart for continuous time series |
| Distribution shape | Histogram, box plot | Average only (hides distribution) |
| Two-variable relationship | Scatter plot | Line chart for non-sequential data |
| Part-to-whole | Stacked bar, treemap | Pie chart (hard to compare slices) |
| Funnel drop-off | Funnel chart, horizontal bar | Pie chart |
| Geographic distribution | Choropleth map | Table with country names only |

**Never use pie charts for comparison** — human perception cannot accurately compare arc lengths. Use a bar chart.

---

## Communicating Findings

Structure every analysis report as:

1. **Situation** — context and why this analysis was done
2. **Finding** — the single most important insight, stated plainly
3. **Evidence** — the data that supports the finding (chart, table, key statistic)
4. **Implication** — what this means for the business or product decision
5. **Recommendation** — what to do next (or what further analysis is needed)

Lead with the insight, not the methodology. The SQL and Python go in an appendix.

**Qualify uncertainty honestly:**
- "This is directional — the sample size is too small for statistical significance"
- "This correlation does not establish causation — we should design an experiment to test"
- "The data only covers users who completed onboarding; churned users are excluded"

---

## Reproducibility Standards

- Analysis code (SQL, Python, R) is version-controlled alongside the documentation
- Assumptions and transformations are documented inline — another analyst can reproduce the result from scratch
- Hardcoded values (time ranges, thresholds, filters) are explained, not magic numbers
- Output files (CSVs, charts) are clearly named with the date and the question they answer
