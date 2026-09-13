# Post-purchase CX Redesign for Marketplace Delivery Failures

## Overview

This project uses the Olist Brazilian E-Commerce Public Dataset to diagnose post-purchase experience failures in a marketplace platform and translate the findings into a service recovery design.

The project is not only about proving that late delivery affects review scores. Late delivery is treated as an observable signal of a broader marketplace service problem: delivery promise failure, customer uncertainty, seller/logistics risk, weak proactive recovery, and customer dissatisfaction.

## Core Question

> Where does the post-purchase experience break down in a marketplace platform, and how should the platform redesign service recovery?

## Project Positioning

Many Olist projects stop at exploratory analysis, dashboards, or statistical testing. This project extends that work by connecting data analysis with service design.

The project flow is:

```text
Delivery failure diagnosis
→ review score impact
→ seller/category/region risk
→ review text pain point coding
→ journey map
→ service blueprint
→ service recovery concept
→ KPI validation plan
```

## Planned Outputs

| Output | Purpose |
|---|---|
| SQL scripts | Load, audit, join, and aggregate Olist data |
| Python notebooks | Run analysis and create charts/tables |
| Dashboard extracts | Provide stakeholder-facing metrics |
| Final case study report | Explain the business problem, findings, design synthesis, and recommendation |
| Presentation deck | Communicate the project in a concise stakeholder format |
| Journey map | Show the customer post-purchase experience and breakdown points |
| Service blueprint | Connect customer-facing issues with platform, seller, logistics, and support processes |
| Pain point taxonomy | Structure low-score review comments into CX failure categories |
| Service recovery concept | Propose delay prediction, proactive notification, and escalation logic |
| KPI / validation plan | Define how the proposed intervention should be tested |
| China localization memo | Discuss transferability to Chinese marketplace contexts |

## Repository Structure

```text
olist-post-purchase-cx-redesign/
│
├── README.md
├── PROJECT_CHARTER.md
├── REFERENCE_PROJECT_UNDERSTANDING.md
├── GAP_MATRIX.md
├── ANALYSIS_PLAN.md
│
├── data/
│   ├── raw/                  # local only; raw CSV files are not committed
│   └── processed/            # cleaned analysis-ready files
│
├── sql/                      # DuckDB / SQL scripts
├── notebooks/                # Python analysis notebooks
│
├── outputs/
│   ├── figures/              # charts
│   ├── tables/               # summary CSVs
│   └── dashboard_extracts/   # dashboard-ready extracts
│
├── service_design/
│   ├── journey_map.md
│   ├── service_blueprint.md
│   ├── pain_point_taxonomy.md
│   ├── opportunity_matrix.md
│   ├── service_recovery_concept.md
│   ├── validation_kpi_plan.md
│   └── china_localization_memo.md
│
├── report/
│   └── final_case_study.md
│
├── presentation/
│   └── olist_cx_redesign_deck.pptx
│
└── references/               # local reference notes or source project summaries
```

## Data

The raw Olist CSV files are stored locally in `data/raw/` and are not committed to GitHub.

Expected dataset files:

- `olist_orders_dataset.csv`
- `olist_customers_dataset.csv`
- `olist_order_items_dataset.csv`
- `olist_order_reviews_dataset.csv`
- `olist_products_dataset.csv`
- `olist_sellers_dataset.csv`
- `product_category_name_translation.csv`
- `olist_order_payments_dataset.csv`
- `olist_geolocation_dataset.csv`

## Main Metrics

Planned core metrics include:

| Metric | Meaning |
|---|---|
| Late delivery rate | share of delivered orders arriving after the promised delivery date |
| Delay days | actual delivery date minus estimated delivery date |
| Bad review rate | share of orders with review score 1–2 |
| Average review score | average 1–5 star rating |
| Seller late rate | seller-level reliability metric |
| Seller late volume | seller-level customer impact scale |
| Category late rate | product category delivery risk |
| Region late rate | customer-state delivery risk |
| Repeat purchase rate | customer relationship proxy using `customer_unique_id` |
| Pain point share | distribution of coded issues in low-score review comments |

## Key Definitions

### Late Delivery

Late delivery will be measured using calendar dates:

```sql
CAST(order_delivered_customer_date AS DATE) > CAST(order_estimated_delivery_date AS DATE)
```

This avoids incorrectly marking same-day deliveries as late when the estimated delivery timestamp is stored at midnight.

### Bad Review

Main definition:

```text
review_score <= 2
```

Optional sensitivity check:

```text
review_score <= 3
```

### Repeat Purchase

Repeat purchase will be measured with:

```text
customer_unique_id
```

not `customer_id`, because `customer_id` is order-specific in Olist.

## Evidence Levels

This project separates findings into three evidence levels.

| Level | Description | Examples |
|---|---|---|
| Level 1 | Direct data analysis from Olist | late rate, review score impact, seller/category/region risk |
| Level 2 | Data-informed service design synthesis | pain taxonomy, journey map, service blueprint, opportunity matrix |
| Level 3 | Proposed recommendation and future validation | service recovery concept, China localization, KPI pilot plan |

The service design artifacts are not claimed to be internal Olist documents. They are synthesized from delivery timestamps, review scores, review comments, and marketplace operating logic.

## Current Status

Current phase:

```text
Day 1 — project framing and setup
```

Completed / planned Day 1 files:

- `PROJECT_CHARTER.md`
- `REFERENCE_PROJECT_UNDERSTANDING.md`
- `GAP_MATRIX.md`
- `ANALYSIS_PLAN.md`
- `README.md`

Next phase:

```text
Day 2 — data understanding and master order table
```

## Planned Workflow

1. Set up the project repository.
2. Define the business problem and evidence levels.
3. Understand the reference projects.
4. Build the Olist master order table.
5. Analyze delivery failure and review score impact.
6. Segment seller/category/region risk.
7. Analyze repeat purchase as supporting evidence.
8. Code low-score review text into pain point categories.
9. Build journey map and service blueprint.
10. Design service recovery concept and KPI plan.
11. Package final report, deck, dashboard, and GitHub repo.

## Limitations

- The dataset is historical and observational, so it cannot prove causality by itself.
- It does not include customer support tickets, compensation records, notification logs, or internal platform recovery workflows.
- Review comments are available only for a subset of reviews.
- Review comments are in Portuguese and require translation or careful manual coding.
- Multi-item or multi-seller orders require careful aggregation.
- China localization requires additional desk research and should not be treated as a direct Olist data finding.

## Working Principle

Every analysis should answer one of these questions:

```text
What problem does this reveal?
Who should act on it?
Where in the journey does it happen?
What intervention does it support?
How would we measure whether the intervention worked?
```
