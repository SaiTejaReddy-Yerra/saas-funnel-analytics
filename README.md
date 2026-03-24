# SaaS Funnel Analytics — End-to-End Portfolio Project

A complete data analytics project simulating real analytics work
at a B2B SaaS company — from raw data cleaning through SQL analysis
to a four-page Power BI dashboard.

---

## Business Context

A B2B SaaS company with a freemium model wants to understand:
- Where users are dropping off in the conversion funnel
- Which acquisition channels bring the highest quality users
- How monthly cohorts are retaining over time
- How revenue is growing and which plan tier drives the business
- Which geographies represent the best growth opportunity

This project answers all five questions using a structured
analytics pipeline built on 100,000 users over 12 months
of simulated production data.

---

## Project Structure
```
saas_funnel_project/
│
├── 01_data_generation.ipynb     Simulates raw database export
├── 02_data_cleaning.ipynb       Cleans and validates raw data
├── 03_sql_analysis.ipynb        SQL analysis — 8 business questions
│
├── data/
│   ├── raw/                     Dirty source files as exported
│   ├── clean/                   Validated files ready for analysis
│   └── powerbi_exports/         Analytical outputs for dashboard
│
└── dashboard/
    └── saas_funnel_dashboard.pbix   Four-page Power BI dashboard
```

---

## Dataset

Simulates a raw export from a B2B SaaS company's
production PostgreSQL database.

| Table | Rows | Description |
|-------|------|-------------|
| users | 99,832 | One row per user who signed up |
| events | 280,594 | One row per action taken in the product |
| subscriptions | 14,045 | One row per paying customer |

**Time period:** October 2023 — October 2024
**Total ARR:** $10,394,124

---

## Data Cleaning — Notebook 02

Raw data contained 5 categories of quality issues
typical of production database exports.

| Problem | Tables affected | Fix applied |
|---------|----------------|-------------|
| Null values | All 3 tables | Filled with UNKNOWN or dropped |
| Duplicate rows | All 3 tables | drop_duplicates() |
| Inconsistent formatting | users, subscriptions | str.strip().str.lower() |
| Invalid values | All 3 tables | Value range filters |
| Wrong data types | events, subscriptions | pd.to_datetime(), pd.to_numeric() |

**Rows removed across all tables:**

| Table | Raw | Clean | Removed |
|-------|-----|-------|---------|
| users | 102,000 | 99,832 | 2,168 |
| events | 296,547 | 280,594 | 15,953 |
| subscriptions | 14,798 | 14,045 | 753 |

**Funnel integrity enforcement:**

After standard cleaning 1,018 users appeared in the events
table with no corresponding signup event — caused by their
signup events being removed during timestamp validation.

Decision: All events for these users were removed from the
events table. Rationale — a user with no confirmed signup
event cannot be placed at the top of the funnel, making
their conversion rates unmeasurable and misleading.

This guarantees that in all downstream SQL queries:
every user_id in events has exactly one signup event row,
making funnel division mathematically safe without
requiring repeated CTE filters in every query.

---

## SQL Analysis — Notebook 03

8 business questions answered using DuckDB.
SQL concepts progress from basic aggregation to
advanced window functions.

| # | Business Question | SQL Concept | Stakeholder |
|---|------------------|-------------|-------------|
| 1 | Where are users dropping off? | CASE WHEN, aggregation | Product |
| 2 | Which channel converts best? | CTE, JOIN, ratios | Marketing |
| 3 | Which device has highest conversion? | GROUP BY, filtering | Engineering |
| 4 | Which cohort retained best at D30? | CTE, DATEDIFF, DATE_TRUNC | Product |
| 5 | How is MRR growing week over week? | Window functions, LAG | Finance |
| 6 | Which country generates most revenue? | JOIN, GROUP BY | Sales |
| 7 | How long does it take to convert? | Self-join, percentiles | Product |
| 8 | Which plan tier dominates revenue? | Window functions, OVER() | CEO |

---

## Key Findings

### 1. Funnel Analysis

| Step | Users | Step conversion | Overall |
|------|-------|----------------|---------|
| Signup | 98,118 | — | 100% |
| Activation | 74,186 | 75.6% | 75.6% |
| Feature used | 62,926 | 84.8% | 64.1% |
| Upgrade intent | 31,543 | 50.1% | 32.1% |
| Paid converted | 13,821 | 43.8% | 14.1% |

**Biggest drop:** Signup to activation loses 24.4% of all users.
This is the single highest-priority area for product improvement.
A 5% improvement in activation rate would add approximately
690 additional paying customers at current downstream rates.

### 2. Channel Performance

Referral is the highest converting channel.
Paid social is the lowest.

| Channel | Conversion rate | Estimated LTV | Recommendation |
|---------|----------------|---------------|----------------|
| Referral | Highest | Highest | Invest in formal referral program |
| Email | Strong | Strong | Increase email capture investment |
| Organic | Solid | Solid | Invest in SEO and content |
| Paid search | Average | Average | Monitor CAC vs LTV |
| Paid social | Lowest | Lowest | Review spend — risk of negative ROI |

### 3. Time to Convert

| Metric | Value |
|--------|-------|
| Average days to convert | 5.8 days |
| 25th percentile | 3 days |
| Median | 5 days |
| 75th percentile | 8 days |
| 90th percentile | 11 days |

90% of all paying customers convert within 11 days of signup.
Peak conversion happens on Days 2, 3 and 4.

**Implication:** The product experience in the first 3 days
is the single biggest lever for revenue growth.
Upgrade prompts should trigger on Day 2-3, not Day 7+.
Users who have not converted by Day 11 are unlikely to ever pay.

### 4. Plan Tier Revenue

| Plan | Subscribers | % of base | Total MRR | % of MRR |
|------|-------------|-----------|-----------|----------|
| Starter | 7,644 | 55.5% | 221,485 | 25.6% |
| Pro | 4,776 | 34.7% | 377,192 | 43.5% |
| Business | 1,344 | 9.8% | 267,500 | 30.9% |

Starter customers represent 55.5% of the base
but only 25.6% of revenue — a 29.9pp gap.

A 10% starter-to-pro upgrade campaign would add
approximately 764 upgrades — generating
38,200 in additional monthly MRR (458,400 ARR).

### 5. Country Revenue

Best markets by MRR per signup:

| Country | MRR per signup | Recommendation |
|---------|---------------|----------------|
| Canada | 9.64 | Increase acquisition investment |
| Australia | 9.25 | Increase acquisition investment |
| Germany | 9.13 | Increase acquisition investment |
| India | 8.77 | Maintain — volume compensates |
| Brazil | 8.43 | Review acquisition cost |
| France | 8.40 | Review acquisition cost |

All countries show similar avg MRR (61-65).
Differences come entirely from conversion rates —
not plan preferences. Localised pricing would not help.
Better localised onboarding would.

---

## Dashboard

Four-page Power BI dashboard covering:

| Page | Content | Audience |
|------|---------|----------|
| 1. Funnel Overview | Waterfall chart, drop-off rates, device breakdown | Product, CEO |
| 2. Retention & Cohorts | Heatmap, D7/D14/D30 trends, cohort sizes | Product, CS |
| 3. Revenue & Plans | MRR growth, plan donut charts, ARR breakdown | Finance, CEO |
| 4. Channel & Geography | Channel table, country revenue, time to convert | Marketing, Sales |

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python 3.11 | Data generation and cleaning |
| Pandas | Data manipulation and cleaning |
| NumPy | Statistical distributions |
| DuckDB | In-process analytical SQL engine |
| Matplotlib / Seaborn | Exploratory visualisation |
| Power BI Desktop | Final dashboard |
| Jupyter Notebook | Analysis environment |

---

## What I Learned

Working through this project reinforced several
analytical principles that do not appear in tutorials:

**On data quality:**
Funnel denominators must come from confirmed event rows —
not from a users registration table. A user with no signup
event in the events table cannot be placed at the funnel top.
Cleaning this at the data layer eliminates the need for
defensive CTEs in every downstream query.

**On SQL:**
Window functions and GROUP BY should not sit in the same
SELECT level. Always aggregate first in a CTE, then apply
window functions on top. Cleaner, easier to debug, and
avoids subtle ordering issues.

**On funnel analysis:**
Step-over-step conversion rates require sequential filtering —
each step's denominator must be the confirmed population
from the previous step. Independent counts divided together
are not conversion rates — they are coincidental ratios.

**On business communication:**
The SQL query is 30% of the value.
The business recommendation is 70%.
Every analysis should answer: so what should we do differently?

---

## Contact

Your Name
LinkedIn: linkedin.com/in/yourprofile
Email: your@email.com
