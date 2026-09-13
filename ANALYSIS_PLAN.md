# Analysis Plan

## Purpose

This document turns the project framing into an executable analysis plan.

The analysis should support this business question:

> Where does the post-purchase experience break down in a marketplace platform, and how should the platform redesign service recovery?

The analysis is not meant to be a random EDA exercise. Every metric should support a decision about customer experience, seller governance, logistics operations, or service recovery.

---

# 1. Analysis Strategy

The project has three layers.

| Layer | Purpose | Methods | Outputs |
|---|---|---|---|
| 1. Direct data diagnosis | Measure delivery failure and customer dissatisfaction | SQL, Python, descriptive stats, statistical tests | master table, metrics, charts |
| 2. Service design synthesis | Translate findings into journey and operating failure points | review text coding, journey mapping, blueprinting | pain taxonomy, journey map, service blueprint |
| 3. Recommendation and validation | Propose and test a recovery system | KPI design, pilot logic, business reasoning | recovery concept, validation plan, localization memo |

---

# 2. Data Sources

Expected Olist CSV files:

| File | Required for v1? | Planned use |
|---|---:|---|
| `olist_orders_dataset.csv` | Yes | order status, delivery timestamps, estimated delivery date |
| `olist_customers_dataset.csv` | Yes | customer ID, unique customer ID, customer state |
| `olist_order_items_dataset.csv` | Yes | seller ID, product ID, price, freight value |
| `olist_order_reviews_dataset.csv` | Yes | review score, review title/comment, review timestamps |
| `olist_products_dataset.csv` | Yes | product category |
| `olist_sellers_dataset.csv` | Yes | seller state |
| `product_category_name_translation.csv` | Yes | English category names |
| `olist_order_payments_dataset.csv` | Optional | payment/revenue context if needed |
| `olist_geolocation_dataset.csv` | Optional | geographic enrichment; not required for v1 |

Raw data should stay local and should not be committed to GitHub.

---

# 3. Analysis Population

Main population for delivery analysis:

```text
order_status = delivered
order_delivered_customer_date is not null
order_estimated_delivery_date is not null
```

Reason:

Late delivery can only be measured reliably when the actual customer delivery date and estimated delivery date are both available.

Optional populations:

| Population | Use |
|---|---|
| All delivered orders | main delivery/review/seller/category/region analysis |
| Delivered first orders by `customer_unique_id` | retention analysis |
| Low-score reviews with non-empty comments | review text coding |
| Orders with valid seller and product category | seller/category risk analysis |

---

# 4. Key Definitions

## 4.1 Late Delivery

Main definition:

```sql
is_late = CAST(order_delivered_customer_date AS DATE) > CAST(order_estimated_delivery_date AS DATE)
```

Why:

`order_estimated_delivery_date` should be treated as a promised calendar date, not a precise timestamp. Comparing full timestamps may wrongly label same-day deliveries as late.

## 4.2 Delay Days

```sql
delay_days = DATE_DIFF('day', CAST(order_estimated_delivery_date AS DATE), CAST(order_delivered_customer_date AS DATE))
```

Interpretation:

| Value | Meaning |
|---:|---|
| `< 0` | delivered early |
| `0` | delivered on promised day |
| `> 0` | delivered late |

## 4.3 Delay Severity Bands

Use these bands for reporting:

| Band | Rule |
|---|---|
| Early | `delay_days < 0` |
| On promised day | `delay_days = 0` |
| 1–3 days late | `delay_days between 1 and 3` |
| 4–7 days late | `delay_days between 4 and 7` |
| 8–14 days late | `delay_days between 8 and 14` |
| 15–30 days late | `delay_days between 15 and 30` |
| 30+ days late | `delay_days > 30` |

## 4.4 Bad Review

Main definition:

```sql
is_bad_review = review_score <= 2
```

Reason:

Review score 1–2 is a stronger signal of dissatisfaction than 1–3.

Optional sensitivity definition:

```sql
is_low_review_sensitivity = review_score <= 3
```

## 4.5 Repeat Purchase

Repeat purchase should use:

```text
customer_unique_id
```

Do not use `customer_id`, because `customer_id` is order-specific in Olist.

---

# 5. Master Order Table

## 5.1 Goal

Create one reusable order-level table that supports all major analysis modules.

Target output:

```text
data/processed/master_order_table.csv
```

## 5.2 Grain

Preferred grain:

```text
one row per order_id
```

Important issue:

`order_items` can contain multiple rows per order, especially when an order has multiple products or sellers. For a clean v1 order-level table, aggregate item-level data before joining to orders.

## 5.3 Planned Fields

| Field | Source | Purpose |
|---|---|---|
| `order_id` | orders | primary order key |
| `customer_id` | orders/customers | join key |
| `customer_unique_id` | customers | repeat purchase analysis |
| `order_status` | orders | delivered-order filter |
| `order_purchase_timestamp` | orders | purchase timing |
| `order_approved_at` | orders | approval timing |
| `order_delivered_carrier_date` | orders | carrier handoff timing |
| `order_delivered_customer_date` | orders | actual delivery date |
| `order_estimated_delivery_date` | orders | promised delivery date |
| `customer_city` | customers | geography context |
| `customer_state` | customers | region risk analysis |
| `seller_id_primary` | aggregated order_items | seller risk; for multi-seller orders use primary seller or flag multi-seller |
| `seller_count` | aggregated order_items | split-seller limitation flag |
| `seller_state_primary` | sellers | seller geography context |
| `product_category_name` | products | category risk |
| `product_category_name_english` | translation | readable category name |
| `item_count` | order_items | order complexity |
| `total_price` | order_items | order value |
| `total_freight_value` | order_items | freight value |
| `review_score` | reviews | satisfaction proxy |
| `review_comment_title` | reviews | review text coding |
| `review_comment_message` | reviews | review text coding |
| `review_creation_date` | reviews | review timing |
| `review_answer_timestamp` | reviews | review deduplication |
| `actual_delivery_date` | derived | calendar actual delivery date |
| `estimated_delivery_date` | derived | calendar promised delivery date |
| `delay_days` | derived | delivery failure severity |
| `is_late` | derived | delivery failure flag |
| `delay_band` | derived | severity reporting |
| `is_bad_review` | derived | score 1–2 flag |
| `is_low_review_sensitivity` | derived | score 1–3 flag |

## 5.4 Join Logic

Conceptual join order:

```text
orders
→ customers on customer_id
→ aggregated order_items on order_id
→ products on product_id or dominant product_id
→ category translation on product_category_name
→ sellers on seller_id
→ deduplicated reviews on order_id
```

Recommended v1 simplification:

1. Create an item aggregation table at `order_id` level.
2. Keep total price and freight.
3. Count unique sellers per order.
4. Choose a primary seller for v1 if needed, but keep `seller_count` to flag multi-seller orders.
5. Choose a primary or most frequent product category for v1, but keep item count and note the limitation.

---

# 6. Module Plan

## Module 1: Data Understanding

Notebook:

```text
notebooks/01_data_understanding.ipynb
```

Questions:

- What files exist?
- How many rows and columns are in each table?
- What are the primary keys and join keys?
- How many missing delivery timestamps exist?
- How many order statuses exist?
- Are there duplicate reviews per order?
- Are there orders with multiple sellers?
- Are there categories missing English translation?

Outputs:

```text
outputs/tables/data_audit_summary.csv
outputs/tables/order_status_summary.csv
outputs/tables/review_duplicate_summary.csv
outputs/tables/multi_seller_order_summary.csv
```

Decision supported:

```text
Can this dataset support the planned post-purchase analysis, and what limitations must be disclosed?
```

## Module 2: Delivery Failure Diagnosis

Notebook:

```text
notebooks/02_delivery_review_analysis.ipynb
```

Questions:

- What share of delivered orders are late?
- How many days early/late are orders delivered?
- Are delays concentrated in severe bands?
- Does late delivery vary by month?

Metrics:

```text
late_delivery_rate
average_delay_days
median_delay_days
share_by_delay_band
monthly_late_rate
```

Outputs:

```text
outputs/tables/late_delivery_summary.csv
outputs/tables/delay_band_summary.csv
outputs/figures/delay_days_distribution.png
outputs/figures/monthly_late_rate.png
```

Decision supported:

```text
Is delivery failure large and variable enough to justify a proactive recovery mechanism?
```

## Module 3: Review Score Impact

Notebook:

```text
notebooks/02_delivery_review_analysis.ipynb
```

Questions:

- Do late orders receive lower review scores?
- Are late orders more likely to receive 1–2 star reviews?
- Does delay severity show a dose-response pattern with review score?

Metrics:

```text
average_review_score_by_delivery_status
bad_review_rate_by_delivery_status
bad_review_rate_ratio
average_review_score_by_delay_band
bad_review_rate_by_delay_band
```

Statistical tests:

| Test | Use |
|---|---|
| Mann-Whitney U / Wilcoxon rank-sum | compare review score distributions between late and on-time orders |
| Chi-square test | compare bad review share between late and on-time orders |
| Optional Kruskal-Wallis | compare review scores across delay bands |

Outputs:

```text
outputs/tables/review_score_by_delivery_status.csv
outputs/tables/bad_review_rate_by_delivery_status.csv
outputs/tables/review_score_by_delay_band.csv
outputs/figures/late_vs_ontime_review_score.png
outputs/figures/bad_review_rate_by_delay_band.png
```

Decision supported:

```text
Should late delivery be treated as a CX early-warning signal?
```

## Module 4: Seller / Category / Region Risk

Notebook:

```text
notebooks/03_seller_category_region_risk.ipynb
```

Questions:

- Which sellers have high late volume?
- Which sellers have high late rate?
- Which sellers combine high volume and high risk?
- Which categories and regions show high late delivery and bad review risk?
- Where should the platform prioritize governance or support?

Metrics:

```text
total_orders
late_orders
late_rate
average_delay_days
bad_review_rate
average_review_score
total_order_value
```

Minimum thresholds:

```text
seller analysis: show all sellers, but create priority groups only for sellers with enough delivered orders
category/state analysis: avoid over-interpreting very small groups
```

Suggested seller priority matrix:

| Segment | Rule concept | Platform action |
|---|---|---|
| High volume + high late rate | many orders and poor reliability | enforcement + operational support |
| High volume + medium late rate | large customer impact | enablement + monitoring |
| Low volume + high late rate | unreliable small seller | onboarding, warning, observation |
| Low volume + low late rate | limited risk | low priority |

Outputs:

```text
outputs/tables/seller_risk_table.csv
outputs/tables/category_risk_table.csv
outputs/tables/region_risk_table.csv
outputs/figures/seller_priority_matrix.png
outputs/figures/category_risk_bar.png
outputs/figures/region_late_rate_bar.png
```

Decision supported:

```text
Which sellers, categories, and regions should be prioritized for service recovery and governance?
```

## Module 5: Retention Analysis

Notebook:

```text
notebooks/04_retention_analysis.ipynb
```

Questions:

- Are customers whose first delivered order was late less likely to purchase again?
- What is repeat purchase within 60 or 90 days after first order?
- Is the effect large enough to be a primary business case?

Metrics:

```text
repeat_purchase_rate
repeat_purchase_within_60_days
repeat_purchase_within_90_days
first_order_late_flag
```

Important caution:

Retention should be treated as supporting evidence because Olist has low repeat purchase overall and the data is observational.

Outputs:

```text
outputs/tables/first_order_late_vs_repeat_purchase.csv
outputs/tables/retention_summary.csv
outputs/figures/repeat_purchase_by_first_delivery_status.png
```

Decision supported:

```text
Does post-purchase failure appear connected to future customer relationship risk?
```

## Module 6: Review Text Coding

Notebook / workflow:

```text
notebooks/05_review_text_coding.ipynb
service_design/pain_point_taxonomy.md
```

Questions:

- What do low-score customers complain about?
- Are complaints only about lateness, or also about uncertainty, non-delivery, product issues, refund/return issues, and communication?
- Which pain points map to journey stages and service blueprint failure points?

Population:

```text
review_score <= 2
review_comment_message is not null
```

Sampling plan:

```text
sample 300 low-score comments for v1
translate Portuguese comments to English or Chinese
manually code each comment into one primary pain point category
optionally allow one secondary category
```

Initial pain point categories:

| Category | Description |
|---|---|
| Delivery delay | customer says the order arrived late or delivery took too long |
| Item not received | customer says the item never arrived |
| Tracking uncertainty | customer does not know where the package is or status is unclear |
| Wrong / damaged item | item arrived wrong, broken, incomplete, or damaged |
| Product quality mismatch | product does not match expectation or listing |
| Refund / return issue | customer mentions refund, return, cancellation, or money-back problem |
| Seller communication issue | seller does not respond or gives poor information |
| Responsibility unclear | customer is unsure whether seller, platform, or carrier is responsible |
| Other | pain point does not fit above categories |

Outputs:

```text
outputs/tables/low_score_review_sample.csv
outputs/tables/review_text_coding.csv
outputs/tables/pain_point_share.csv
outputs/figures/pain_point_share.png
service_design/pain_point_taxonomy.md
```

Decision supported:

```text
Which customer pain points should the journey map, service blueprint, and recovery concept address?
```

## Module 7: Service Design Synthesis

Files:

```text
service_design/journey_map.md
service_design/service_blueprint.md
service_design/opportunity_matrix.md
service_design/service_recovery_concept.md
```

Inputs:

- late delivery findings;
- review score impact;
- seller/category/region risk;
- review text coding;
- marketplace operating logic.

Journey stages:

```text
Order placed
→ Seller processing
→ Carrier handoff
→ In transit
→ Estimated delivery date approaching
→ Delay / uncertainty
→ Delivered or not received
→ Review submitted
→ Repeat purchase or churn risk
```

Service blueprint layers:

```text
Physical evidence
Customer actions
Frontstage platform touchpoints
Backstage platform operations
Seller operations
Logistics operations
Support processes
Failure points
Intervention opportunities
KPIs
```

Decision supported:

```text
Where should the platform intervene before bad reviews happen?
```

## Module 8: KPI and Validation Plan

File:

```text
service_design/validation_kpi_plan.md
```

Historical baseline KPIs from Olist:

- late delivery rate;
- average delay days;
- bad review rate;
- late-order bad review rate;
- average review score;
- seller late rate;
- seller late volume;
- category late rate;
- region late rate;
- repeat purchase rate;
- review text pain point share.

Future pilot KPIs not available in Olist:

- notification open rate;
- support contact rate;
- complaint rate;
- first response time;
- service recovery completion rate;
- compensation cost;
- refund request rate;
- seller escalation rate;
- customer trust or perceived fairness.

Pilot concept:

| Component | Design |
|---|---|
| Target group | orders predicted or detected as high late-delivery risk |
| Control group | no proactive notification |
| Treatment group | proactive notification + updated ETA + support option |
| Primary KPI | bad review rate, score 1–2 |
| Secondary KPIs | complaint/contact proxy, repeat purchase, refund request, support contact |
| Operational KPIs | recovery completion, seller escalation, delay resolution |
| Guardrails | support workload, compensation cost, seller fairness, abuse risk |

Decision supported:

```text
How should the proposed recovery system be tested before full rollout?
```

---

# 7. Dashboard Plan

Dashboard should answer four business questions:

1. How serious is the delivery failure problem?
2. How strongly is delivery failure associated with bad reviews?
3. Where is risk concentrated by seller, category, and region?
4. Where should the platform intervene first?

Suggested pages:

| Page | Purpose | Key visuals |
|---|---|---|
| Executive Overview | summarize problem size and CX impact | late rate, average score gap, bad review gap |
| Delivery Failure Diagnosis | show delay severity and time pattern | delay bands, monthly late rate |
| Risk Segmentation | show sellers/categories/regions | seller matrix, category bar, region bar |
| Intervention Priority | connect risk to action | priority table, recovery trigger logic |

---

# 8. Report Plan

Final report structure:

```text
1. Executive Summary
2. Business Problem
3. Dataset and Method
4. Reference Project Gap Analysis
5. Delivery Failure Diagnosis
6. Review Score Impact
7. Seller / Category / Region Risk
8. Retention Analysis
9. Review Text Coding and Pain Point Taxonomy
10. Current-state Journey Map
11. Current-state Service Blueprint
12. Opportunity Matrix
13. Future-state Service Recovery Concept
14. China Marketplace Localization
15. Validation and KPI Plan
16. Limitations
17. Final Recommendation
```

The report should not read like a code notebook. It should read like a business and service design case study.

---

# 9. Day-by-Day Execution Plan

## Day 1: Project framing

Outputs:

```text
PROJECT_CHARTER.md
REFERENCE_PROJECT_UNDERSTANDING.md
GAP_MATRIX.md
ANALYSIS_PLAN.md
README.md
```

Goal:

```text
Lock the project direction before writing code.
```

## Day 2: Data understanding and master table design

Outputs:

```text
notebooks/01_data_understanding.ipynb
sql/01_load_data.sql
sql/02_data_audit.sql
sql/03_create_master_table.sql
data/processed/master_order_table.csv
```

Goal:

```text
Create a reliable order-level dataset.
```

## Day 3: Delivery and review score analysis

Outputs:

```text
notebooks/02_delivery_review_analysis.ipynb
outputs/tables/late_delivery_summary.csv
outputs/tables/review_score_by_delivery_status.csv
outputs/figures/late_vs_ontime_review_score.png
```

Goal:

```text
Prove whether late delivery is associated with customer dissatisfaction.
```

## Day 4: Seller/category/region risk

Outputs:

```text
notebooks/03_seller_category_region_risk.ipynb
outputs/tables/seller_risk_table.csv
outputs/tables/category_risk_table.csv
outputs/tables/region_risk_table.csv
outputs/figures/seller_priority_matrix.png
```

Goal:

```text
Turn analysis into platform prioritization.
```

## Day 5: Retention and review text coding

Outputs:

```text
notebooks/04_retention_analysis.ipynb
notebooks/05_review_text_coding.ipynb
outputs/tables/review_text_coding.csv
service_design/pain_point_taxonomy.md
```

Goal:

```text
Connect delivery failure to customer relationship risk and actual complaint themes.
```

## Day 6: Dashboard extracts and first dashboard

Outputs:

```text
outputs/dashboard_extracts/*.csv
dashboard screenshots
```

Goal:

```text
Make the analysis usable for stakeholder decisions.
```

## Day 7–8: Service design deliverables

Outputs:

```text
service_design/journey_map.md
service_design/service_blueprint.md
service_design/opportunity_matrix.md
service_design/service_recovery_concept.md
service_design/validation_kpi_plan.md
service_design/china_localization_memo.md
```

Goal:

```text
Translate data findings into intervention design.
```

## Day 9–10: Final report, deck, GitHub cleanup

Outputs:

```text
report/final_case_study.md
report/final_case_study.pdf
presentation/olist_cx_redesign_deck.pptx
final README update
```

Goal:

```text
Package the project as a portfolio case study.
```

---

# 10. Immediate Next Step After Day 1

After committing these planning files, start Day 2 with:

1. place Olist CSVs in `data/raw/` locally;
2. confirm `.gitignore` excludes raw files;
3. create `notebooks/01_data_understanding.ipynb`;
4. inspect all tables;
5. create a draft SQL join plan;
6. build the first version of `master_order_table.csv`.
