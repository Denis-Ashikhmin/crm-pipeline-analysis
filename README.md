# CRM Sales Pipeline Analysis

A SQL-based analysis of 8,800 B2B sales deals to identify pipeline health issues,
performance gaps, and revenue risk across agents, products, and customer segments.

**Tools:** SQLite · DB Browser · Google Sheets · Data Studio (Looker Studio)
**Data:** [Maven Analytics — CRM Sales Opportunities](https://mavenanalytics.io/data-playground/crm-sales-opportunities)

---

## Business Problem

The company scaled rapidly - from 7 deals in October 2016 to 1,198 deals in
July 2017. But as volume grew, win rate declined sharply. Without visibility
into where and why deals were being lost, the business had no way to course-correct.

This project answers three questions:

1. Is the pipeline healthy, and where are deals getting stuck?
2. Which agents and managers are underperforming, and is it a people problem or a systemic one?
3. Which products and customer segments drive the most revenue?

---

## Analytical Approach

### Why cohort analysis instead of stage distribution

A naive count of deals by current stage showed Won > Engaging > Prospecting -
an inverted funnel. This is not a data quality issue: the dataset spans 15 months,
so most early deals have already closed. Counting stages as a snapshot mixes
deals from different time periods and produces meaningless conversion rates
(317% "conversion" from Prospecting to Engaging in this dataset).

**Solution:** cohort-based analysis groups deals by the month they entered the
pipeline (`engage_date`) and measures outcomes within each group. Only "mature"
cohorts are included - deals with `engage_date` before 2017-10-01, giving at
least 90 days to close.

> Average cycle: Won = 51.8 days, Lost = 41.5 days.  
> 90-day buffer was chosen to ensure even the slowest deals had time to resolve.

---

## Key Findings

### 1. Win rate declined steadily as the company scaled

| Period | Cohort Win Rate |
|--------|----------------|
| Oct 2016 | 100% |
| Nov 2016 | 81.4% |
| Apr 2017 | 56.7% |
| Jul 2017 | 55.4% |

Win rate dropped by ~26 percentage points over 9 months while deal volume grew
170x. The data suggests the company prioritized pipeline growth over lead quality -
more deals entered the funnel than the team could effectively close.

### 2. End-of-year stuck deals likely reflect customer decision delays

December 2016 shows an elevated stuck deal rate (17%) despite a high win rate
of 81.6%. This pattern is consistent with customers postponing decisions until
the new year rather than a pipeline quality issue.

### 3. July–August 2017 anomaly points to pipeline oversaturation

July 2017 shows 1,198 deals with 34% stuck — more than double any other month.
August follows with 32% stuck. The simultaneous spike in total deals and stuck
rate suggests the pipeline was flooded with unqualified leads, likely as a result
of aggressive outreach during a rapid expansion phase.

### 4. Revenue growth masked win rate decline — until Q3

| Quarter | Win Rate | Revenue |
|---------|----------|---------|
| 2017-Q1 | 82.1% | $1,134,672 |
| 2017-Q2 | 61.7% | $3,086,111 |
| 2017-Q3 | 61.4% | $2,982,255 |
| 2017-Q4 | 60.3% | $2,802,496 |

In Q2 the company compensated for lower win rate with higher deal volume,
maintaining revenue growth. From Q3 onward both win rate and revenue declined
simultaneously — a signal that volume alone can no longer offset quality loss.

### 5. Manager performance is consistent; agent-level gaps require deeper investigation

Win rates across managers fall within a narrow band, suggesting no single
region or team has a structural advantage. At the agent level, however,
there are meaningful gaps.

**Lajuana Vencill** stands out as the clearest underperformer: lowest win rate
(55%) and lowest revenue ($194,632) in the dataset — both metrics point in
the same direction.

Other agents below team average warrant further analysis before drawing
conclusions. Some may be working larger, longer-cycle deals that compress
win rate while still generating strong revenue. For example, **Donn Cantrell**
has a below-average win rate (-5% vs team) but generates $445,860 in revenue —
suggesting a different deal profile rather than underperformance.

### 6. GTK 500 is the highest-risk product; GTX Plus Pro is the portfolio star

The scatter chart (win rate vs. price) reveals a clear outlier: **GTK 500**
carries the highest price point ($26,768) but the lowest win rate (60.0%).
High-value deals lost on this product represent the largest revenue leakage
in the portfolio.

**GTX Plus Pro** is the opposite: second-highest win rate (64.3%) with a
strong price point ($5,482), making it the most reliable revenue driver.

**MG Advanced** shows both low win rate (60.3%) and low price ($3,393) —
a combination that makes it a candidate for repositioning or discontinuation.

### 7. Sector win rates are narrow; revenue concentration tells the real story

Win rates across sectors cluster between 61–65%, meaning the team closes
at roughly the same rate regardless of customer type. The more actionable
insight is where revenue is concentrated: **Retail** and **Technology**
together account for 34% of total won revenue.

**Marketing** and **Entertainment** show the highest win rates (64.8% and 64.7%)
relative to their deal volume — efficient segments worth expanding into.

**Finance** has the lowest win rate (61.2%) despite the highest average deal
value ($2,536). Improving close rates in this segment would yield
disproportionate revenue gains.

---

## Recommendations

1. **Audit July–August 2017 pipeline.** Investigate what changed in lead sourcing
   or qualification during this period. High stuck deal rates in mature cohorts
   suggest these deals need a resolution - either a follow-up or a formal close-out.

2. **Dig deeper into agent performance before acting.** Flag agents with both
   low win rate and low revenue (like Lajuana Vencill) for coaching review.
   For agents with low win rate but strong revenue, analyze deal size and cycle
   length before drawing conclusions.

3. **Double down on GTX Plus Pro.** Strong win rate at a solid price point -
   this is where sales effort has the highest expected return.

4. **Review GTK 500 qualification process.** At $26,768 per deal, each loss
   is significant. Either the product needs better competitive positioning
   or the team needs stricter qualification criteria before pursuing these deals.

5. **Consider repositioning MG Advanced.** Low price and low win rate make
   this the weakest product in the portfolio on both dimensions.

6. **Track win rate as a leading indicator, not just revenue.** The data shows
   that revenue decline lagged win rate decline by two quarters. A simple monthly
   cohort win rate report would have flagged this trend by February 2017.

---

## Repository Structure

```
crm-pipeline-analysis/
│
├── data/
│   ├── accounts.csv
│   ├── data_dictionary.csv
│   ├── products.csv
│   ├── sales_pipeline.csv
│   └── sales_teams.csv
│
├── sql/
│   ├── 00_filter_view.sql        — enriched view joining all tables
│   ├── 01_cohort_funnel.sql       — cohort win rate and stuck deal detection
│   ├── 02_overall_health.sql      — pipeline scorecard (single row summary)
│   ├── 03_sales_cycle.sql         — Won vs Lost cycle duration
│   ├── 04_agent_performance.sql   — per-agent stats with vs-team comparison
│   ├── 05_sector_analysis.sql     — win rate and revenue by customer sector
│   ├── 06_product_performance.sql — product win rate and price variance
│   └── 07_quarterly_trends.sql    — quarterly revenue and win rate trend
│
├── dashboard/
│   ├── PipeLine Health and Cohort Analysis.jpg
│   └── Agent Performance.jpg
│   └── Win Rate Breakdown.jpg
│
└── README.md
```

---

## Dashboard

[View Live Dashboard →](https://datastudio.google.com/reporting/180e2883-3a5d-4459-a26a-c7bbdedc8476)

---

## Notes on Methodology

- Win rate is calculated only on closed deals (Won + Lost), not total pipeline,
  to avoid distortion from open deals
- Cohort analysis uses `engage_date` as the entry point, since no earlier
  stage timestamp is available in the dataset
- "Mature cohorts" are defined as `engage_date ≤ 2017-10-01`, providing a
  minimum 90-day window for deals to close (based on observed avg cycle of 51.8 days)
- ~10% of closed deals have a cycle of 3 days or less and may reflect data
  entry practices rather than actual fast closes; these were retained in the analysis
