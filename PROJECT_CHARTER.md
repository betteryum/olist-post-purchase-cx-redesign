# Project Charter

## Project Title

**Post-purchase CX Redesign for Marketplace Delivery Failures**

## One-sentence Summary

This project uses the Olist Brazilian E-Commerce Public Dataset to diagnose marketplace post-purchase experience failures, especially delivery promise failures, and translate the findings into a service recovery system for platform operators.

## Why This Project Exists

Marketplace platforms often treat late delivery as a logistics metric. From the customer perspective, however, late delivery can become a broader service failure:

- the promised delivery date is missed;
- tracking information may be unclear;
- the customer may not know whether the seller, logistics provider, or platform is responsible;
- the platform may not intervene until after the customer complains or leaves a low review.

This project reframes late delivery as an observable signal of post-purchase CX breakdown. The goal is not only to show that late orders receive lower ratings, but to design how a marketplace could detect, communicate, recover, and govern these failures before they become bad reviews or churn risk.

## Business Problem

Olist-style marketplaces depend on sellers, logistics partners, platform rules, and customer-facing support all working together after checkout. When delivery is late or uncertain, customers experience the failure as one platform-level problem, even if the operational cause sits with a seller or carrier.

The business problem is:

> Where does the post-purchase experience break down in a marketplace platform, and how should the platform redesign service recovery around delivery failures?

## Decision Context

This project is written as if it were prepared for a marketplace platform team deciding:

1. whether delivery delay is large enough to deserve CX intervention;
2. whether late delivery is associated with lower customer satisfaction;
3. which sellers, categories, and regions should be prioritized;
4. where the customer journey breaks down after checkout;
5. what service recovery mechanism should be piloted;
6. what KPIs should be used to validate the intervention.

## Target Stakeholders

| Stakeholder | What they care about in this project |
|---|---|
| Head of Customer Experience | Bad review rate, customer trust, recovery experience |
| Marketplace Operations Lead | Operational reliability, late delivery concentration, execution priorities |
| Seller Governance Team | Seller-level risk, thresholds, escalation logic |
| Logistics Operations Manager | Delay patterns, regional risk, delivery volatility |
| Product / Service Design Team | Journey map, touchpoints, proactive notification, support flows |
| Analytics Team | Definitions, master table, reproducible SQL/Python logic, KPI baselines |

## Core Research Questions

### RQ1. Delivery Failure Diagnosis
Where do post-purchase delivery failures occur in the Olist marketplace?

Planned outputs:
- late delivery rate;
- delay days;
- delay severity bands;
- monthly delay pattern;
- delivered-order analysis population.

### RQ2. Review Score Impact
How strongly is delivery failure associated with customer dissatisfaction?

Planned outputs:
- late vs on-time average review score;
- bad review rate for late vs on-time orders;
- delay severity vs review score;
- statistical tests for score/rating differences.

### RQ3. Seller / Category / Region Risk
Which sellers, categories, and regions create the highest operational and CX risk?

Planned outputs:
- seller risk table;
- category risk table;
- region/state risk table;
- priority matrix separating high late-rate from high late-volume cases.

### RQ4. Review Text Pain Points
What pain points appear in low-score customer reviews?

Planned outputs:
- low-score review sample;
- translated review sample;
- manual pain point coding;
- pain point taxonomy;
- pain point distribution.

### RQ5. Service Design Synthesis
Where are the highest-value intervention points in the post-purchase journey?

Planned outputs:
- current-state journey map;
- current-state service blueprint;
- opportunity matrix.

### RQ6. Service Recovery Recommendation
What service recovery system should the platform design and how should it be validated?

Planned outputs:
- three-layer service recovery concept;
- seller-risk-based escalation logic;
- KPI and validation plan;
- China marketplace localization memo.

## Project Framing

This is not only a late-delivery analysis. Late delivery is the measurable entry point into a broader marketplace reliability problem.

The intended framing is:

```text
delivery promise failure
→ customer dissatisfaction
→ seller/category/region operational risk
→ review text pain points
→ post-purchase journey breakdown
→ service blueprint failure points
→ service recovery system
→ KPI validation plan
```

## Data Source

Dataset:

**Olist Brazilian E-Commerce Public Dataset**

Expected tables:

| Table | Planned use |
|---|---|
| `olist_orders_dataset.csv` | order status, purchase timestamp, carrier handoff, actual delivery, estimated delivery |
| `olist_customers_dataset.csv` | customer ID, customer unique ID, customer city/state |
| `olist_order_items_dataset.csv` | seller ID, product ID, price, freight value |
| `olist_order_reviews_dataset.csv` | review score, review comments, review timestamps |
| `olist_products_dataset.csv` | product category, product attributes |
| `olist_sellers_dataset.csv` | seller city/state |
| `product_category_name_translation.csv` | product category English translation |
| `olist_order_payments_dataset.csv` | optional revenue/payment context |
| `olist_geolocation_dataset.csv` | optional geographic enrichment; not required for the first version |

## Analysis Unit

The main analysis unit is the **order level**.

The master table will use delivered orders with valid delivery timestamps wherever late delivery is analyzed.

Important rule:

```sql
is_late = CAST(order_delivered_customer_date AS DATE) > CAST(order_estimated_delivery_date AS DATE)
```

This calendar-date rule avoids incorrectly marking same-day deliveries as late when the estimated delivery timestamp is stored at midnight.

## Evidence Levels

This project separates evidence into three levels so that the report does not overclaim.

### Level 1: Direct Data Analysis
These outputs can be directly calculated from Olist data.

- late delivery rate;
- delay days;
- review score impact;
- bad review rate;
- seller late rate and late volume;
- category late rate and review risk;
- region/state late rate and review risk;
- repeat purchase relationship using `customer_unique_id`;
- review text coding from customer comments.

### Level 2: Data-informed Service Design Synthesis
These outputs are supported by data, review text, and marketplace operating logic, but are not directly generated by the dataset.

- pain point taxonomy;
- current-state journey map;
- current-state service blueprint;
- opportunity matrix;
- intervention priority logic.

### Level 3: Recommendation and Future Validation
These outputs are proposed designs and future validation plans. They are not historical facts proven by the dataset.

- service recovery concept;
- proactive notification design;
- escalation/compensation logic;
- China marketplace localization memo;
- KPI framework and pilot plan.

## Final Deliverables

| Deliverable | Purpose |
|---|---|
| GitHub repo | Show reproducible code, project structure, documentation, and outputs |
| Final case study report | Explain the business problem, data findings, service design synthesis, and recommendations |
| Presentation deck | Communicate the project to stakeholders in 10–12 slides |
| Dashboard | Show delivery failure, review impact, and intervention priorities |
| Journey map | Translate data findings into the customer post-purchase experience |
| Service blueprint | Connect customer-facing failures with seller, logistics, platform, and support processes |
| Pain point taxonomy | Convert low-score review text into structured CX failure categories |
| Service recovery concept | Propose delay prediction, proactive notification, and escalation logic |
| KPI / validation plan | Define how the intervention should be tested |
| China localization memo | Discuss how the design would need to change in a China marketplace context |

## Scope

In scope:

- delivered order analysis;
- delivery delay and review score relationship;
- seller/category/region risk segmentation;
- repeat purchase relationship as supporting evidence;
- low-score review text coding;
- data-informed journey map and service blueprint;
- service recovery recommendation;
- KPI plan and pilot design.

Out of scope for the first version:

- building a production ML prediction model;
- causal proof that late delivery causes low reviews;
- real customer support ticket analysis;
- real compensation/refund data analysis;
- real notification open/click data analysis;
- live integration with platform systems;
- claiming that Olist findings directly represent China marketplaces.

## Key Assumptions

1. Review score is used as a proxy for customer satisfaction.
2. Review score 1–2 is treated as a bad review / strong dissatisfaction signal.
3. Review score 1–3 may be used as a sensitivity check.
4. Late delivery is defined by calendar date, not timestamp comparison.
5. Repeat purchase is measured using `customer_unique_id`, not `customer_id`.
6. Service design outputs are data-informed syntheses, not internal Olist process documents.

## Known Limitations

- The dataset is historical and observational; it supports association, not causality.
- The dataset does not contain customer support tickets, compensation records, refund details, notification logs, or actual service recovery workflows.
- Review comments are not available for every review.
- Review comments are in Portuguese and require translation or careful manual coding.
- Some orders may contain multiple sellers or items, so order-level joins must be handled carefully.
- Customer retention in Olist may be low overall, so retention should be treated as supporting evidence rather than the main project conclusion.
- China localization requires external research and should be presented as a transferability memo, not as a result of the Olist dataset.

## Success Criteria for the Portfolio Version

The project is successful if a reader can understand:

1. what marketplace post-purchase problem is being diagnosed;
2. how late delivery is defined and measured;
3. how delivery failure relates to review scores;
4. where risk is concentrated across sellers, categories, and regions;
5. what customers complain about in low-score reviews;
6. where the journey and service blueprint break down;
7. what intervention the platform should pilot;
8. how success should be measured.

## First Milestone

Day 1 deliverables:

- `PROJECT_CHARTER.md`;
- `REFERENCE_PROJECT_UNDERSTANDING.md`;
- `GAP_MATRIX.md`;
- `ANALYSIS_PLAN.md`;
- updated `README.md`;
- first clean commit and push.

Day 2 starting point:

- create the data folder structure;
- confirm raw CSV files are local but not committed;
- create the first data understanding notebook;
- design and build the master order table.
