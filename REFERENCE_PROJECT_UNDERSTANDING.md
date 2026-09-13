# Reference Project Understanding

## Purpose of This Document

This document summarizes how the two reference GitHub projects will be used in this portfolio project.

The goal is not to copy either project. The goal is to understand what they already cover, what their limitations are, and where this project will add a service design and marketplace recovery layer.

Reference role:

```text
Zeash project = quantitative diagnosis / SQL pipeline / dashboard / seller-region risk reference
Moses project = statistical testing / retention logic reference
My project = data-informed service design + service recovery operating model
```

---

# 1. Reference Project 1: Zeash Olist E-commerce Analysis

## 1.1 Project Positioning

The Zeash project is a relatively complete marketplace operations and CX analysis. Its core question can be summarized as:

> In the Brazilian marketplace, which operational and customer experience issues are damaging satisfaction and repeat purchase, and where should the platform prioritize improvement?

It is close to the quantitative diagnosis part of this project. It is especially useful for:

- late delivery definition;
- data audit process;
- SQL-first project structure;
- seller risk analysis;
- state/geography risk analysis;
- dashboard extracts;
- careful limitations.

## 1.2 Research Questions Covered

| Area | Questions addressed by the reference project |
|---|---|
| Delivery | Which orders arrived late? How many days late? Does lateness affect review score? |
| Seller operations | Which sellers contribute most to late deliveries? |
| Geography | Which Brazilian states have higher delivery risk? |
| Category | Which categories show higher delivery/review risk? |
| Retention | Does first-order delivery experience relate to repeat purchase? |
| Dashboard | How can late delivery and seller/geography risk be turned into stakeholder-facing metrics? |

## 1.3 Data Used

The project loads the nine public Olist tables, but the main analysis relies on seven core tables.

| Table | Use |
|---|---|
| `orders` | order status, purchase timestamp, actual delivery date, estimated delivery date |
| `customers` | `customer_unique_id`, customer state |
| `order_items` | seller ID, product ID, price, freight |
| `order_reviews` | review score and review timestamp |
| `products` | product category |
| `sellers` | seller state |
| `product_category_translation` | English product category names |
| `geolocation` | loaded but not central to the core analysis |
| `order_payments` | loaded but not central to the core analysis |

The main analysis population is:

```text
order_status = delivered
and order_delivered_customer_date is not null
```

The reference material reports **96,470 delivered orders** under this population rule.

## 1.4 Method and Project Structure

The Zeash project is structured as a SQL-first analytics project with a Python statistical notebook and Tableau dashboard output.

| File / module | Role |
|---|---|
| `data/download.py` | downloads Olist data from Kaggle and checks file integrity |
| `sql/01_load.sql` | loads the CSV files into DuckDB with timestamp types |
| `sql/02_audit.sql` | audits IDs, duplicates, missing values, date ranges, translations, repeat purchase key |
| `sql/03_late_delivery.sql` | creates delivery facts and seller-level delivery tables |
| `sql/04_late_reviews.sql` | joins delivery flags with reviews |
| `sql/05_seller_pareto.sql` | identifies sellers contributing disproportionately to late deliveries |
| `sql/06_state_delivery.sql` | analyzes state/geography delivery performance |
| `sql/07_repeat_purchase.sql` | analyzes repeat purchase after first-order experience |
| `sql/08_dashboard_extracts.sql` | exports aggregate CSVs for Tableau |
| `notebooks/statistical_tests.ipynb` | runs Mann-Whitney, Chi-square, Kruskal-Wallis, OLS/logit tests |
| `dashboard/` | contains Tableau workbook and screenshots |

## 1.5 Late Delivery Definition

This is one of the most important parts to reuse.

The reference project recognizes that:

```text
order_estimated_delivery_date is a promised calendar date, not a precise promised timestamp.
```

If late delivery is calculated using raw timestamp comparison:

```sql
actual_delivery_timestamp > estimated_delivery_timestamp
```

then deliveries arriving later on the same promised day may be incorrectly marked as late because the estimated delivery timestamp often appears at midnight.

The better rule is the calendar-date rule:

```sql
CAST(order_delivered_customer_date AS DATE) > CAST(order_estimated_delivery_date AS DATE)
```

The reference material reports:

| Rule | Late rate |
|---|---:|
| Calendar-date rule | 6.77% |
| Naive timestamp rule | 8.11% |

My project should use the calendar-date rule.

## 1.6 Review Handling

The Zeash project does not perform review text coding. It uses review scores.

Its low-rating definition is:

```text
bad review = review_score <= 2
```

It also handles review duplication by keeping one review record per order, using the latest `review_answer_timestamp` when needed.

This is important because one order may have multiple review rows.

## 1.7 Key Findings from the Reference Project

These numbers are reference findings and will be recalculated in my own pipeline before being used as final project findings.

### Finding 1: Late delivery is strongly associated with lower review score

Reported reference results:

| Metric | On-time / early | Late |
|---|---:|---:|
| Average review score | 4.29 | 2.27 |
| Review score gap | — | -2.02 stars |
| Bad review rate, score 1–2 | 9.27% | 62.42% |
| Bad review risk ratio | — | about 6.7x |

Interpretation:

Late delivery is not only an operational issue. It appears as a major customer satisfaction failure in review scores.

### Finding 2: Delay severity matters

Reported dose-response pattern:

| Lateness band | Average review score |
|---|---:|
| Early | 4.294 |
| On promised day | 4.034 |
| 1–3 days late | 3.291 |
| 4–7 days late | 2.103 |
| 8–14 days late | 1.671 |
| 15–30 days late | 1.614 |
| 30+ days late | 2.058 |

Interpretation:

The later the order arrives, the worse the rating generally becomes. This supports a severity-based recovery system rather than a one-size-fits-all recovery rule.

### Finding 3: Late deliveries are concentrated among a small number of sellers

The reference material reports:

```text
98 sellers, about 3.3% of active sellers, contribute 50% of late shipments.
```

It also notes that high late volume and high late rate are not always the same thing.

This implies two different seller governance paths:

| Seller type | Meaning | Possible platform response |
|---|---|---|
| High late volume | Usually larger sellers with many orders | enablement, capacity planning, logistics support |
| High late rate | Often lower-volume sellers with poor fulfillment reliability | threshold-based monitoring, onboarding support, enforcement |

### Finding 4: Delivery failure can be volatile over time

The reference material reports:

| Month | Late rate |
|---|---:|
| 2018-03 | 18.96% |
| 2018-06 | 1.16% |

Interpretation:

Delivery risk may spike in specific periods rather than remain constant. This supports early warning, capacity monitoring, and escalation planning.

### Finding 5: Geography matters

The reference material highlights Rio de Janeiro as a high-risk geography:

| State | Late rate |
|---|---:|
| RJ | 12.11% |
| SP | 4.49% |

Interpretation:

Regional delivery risk should be part of the intervention prioritization logic.

### Finding 6: Retention signal exists but is weak

The reference material reports first-order delivery experience and repeat purchase under a censored six-month follow-up:

| First-order experience | Repeat rate |
|---|---:|
| First order late | 3.05% |
| First order on-time | 4.13% |

Difference:

```text
-1.08 percentage points
```

Interpretation:

Retention should be treated as supporting evidence, not the main project conclusion.

## 1.8 Recommendations from the Reference Project

| Recommendation | Meaning for my project |
|---|---|
| Set seller late-rate threshold | Can inform seller governance rules |
| Separate high-volume seller enablement from high-rate enforcement | Useful for priority matrix design |
| Investigate Rio de Janeiro | Useful for geography risk discussion |
| Handle split shipment timestamp limitations | Important limitation for order-level analysis |
| Do not overbuild retention strategy from review score alone | Keeps my recommendation honest |

## 1.9 How I Will Use It

I will use the Zeash project as the main reference for quantitative diagnosis.

What I will reuse conceptually:

- delivered-order population rule;
- calendar-date late delivery definition;
- review deduplication concern;
- bad review definition using 1–2 stars;
- seller risk using both late volume and late rate;
- region/state risk logic;
- dashboard-oriented output tables;
- careful limitation writing.

What I will not simply copy:

- final wording;
- full SQL implementation;
- visual design;
- conclusions without recalculation;
- dashboard layout without adapting it to service recovery.

What my project adds beyond Zeash:

- review text coding;
- pain point taxonomy;
- journey map;
- service blueprint;
- opportunity matrix;
- service recovery concept;
- validation / KPI plan;
- China localization memo.

---

# 2. Reference Project 2: Moses Olist Delivery Retention Analysis

## 2.1 Project Positioning

The Moses project is narrower than Zeash. It focuses on whether late delivery affects customer satisfaction and retention.

Its core question can be summarized as:

> Does late delivery hurt customer satisfaction and repeat purchase, and where should a business prioritize improvement?

It is useful for statistical testing and retention logic, but less useful for seller governance, geography operations, dashboard design, or service design.

## 2.2 Research Questions Covered

| Area | Questions addressed |
|---|---|
| Delivery delay | What share of delivered orders are late? |
| Review score | Do late orders receive lower scores? |
| Statistical testing | Are differences statistically significant? |
| Repeat purchase | Are customers with late first orders less likely to repeat? |
| Revenue retention | How does cohort revenue behave over time? |
| Category | Which product categories matter from a revenue perspective? |

## 2.3 Data Used

The Moses project uses six main Olist tables.

| Table | Use |
|---|---|
| `orders` | delivery status and timestamps |
| `order_items` | revenue, price, freight, product ID |
| `customers` | customer unique ID |
| `reviews` | review score |
| `products` | product category |
| `product_category_translation` | category translation |

It does not use:

- `sellers`;
- `geolocation`;
- `payments`.

This means it does not support detailed seller governance or regional operations analysis.

## 2.4 Method

The project is implemented mainly as a single R script:

```text
olist_delivery_retention_analysis.R
```

The script follows a procedural tidyverse workflow:

| Step | Action |
|---|---|
| 1 | load packages such as tidyverse, lubridate, scales |
| 2 | read CSV files |
| 3 | inspect structures with `glimpse()` |
| 4 | check missing values |
| 5 | filter delivered orders |
| 6 | calculate delivery delay days |
| 7 | label orders as late or on-time/early |
| 8 | join review scores |
| 9 | compare review score descriptive statistics |
| 10 | run Wilcoxon rank-sum test |
| 11 | run Chi-square test |
| 12 | analyze repeat purchase |
| 13 | analyze cohort revenue retention |
| 14 | analyze category revenue, late rate, and review score |
| 15 | create ggplot charts |

## 2.5 Late Delivery Definition

The Moses project calculates:

```text
delivery_delay_days = actual_delivery_date - estimated_delivery_date
late_delivery = delivery_delay_days > 0
```

This is directionally aligned with the calendar-date rule, but it does not explain the timestamp problem as carefully as Zeash.

For my project, Zeash's calendar-date definition should be treated as the main rule.

## 2.6 Review Score Coding

The Moses project uses two levels of review coding.

### Level 1: Raw review score

```text
review_score = 1 to 5 stars
```

### Level 2: Satisfaction binary label

```text
Low = review_score 1–3
High = review_score 4–5
```

This differs from Zeash.

| Project | Low / bad review definition |
|---|---|
| Zeash | 1–2 stars |
| Moses | 1–3 stars |

My project should use 1–2 stars as the main bad review definition and optionally use 1–3 stars as a sensitivity check.

## 2.7 Findings and Cautions

The reference notes contain several important cautions about Moses.

### Caution 1: README and code-derived numbers may not fully match

The reference material notes that Moses's README reports:

```text
6.8% delivered orders late
7,826 of 96,470 delivered orders late
```

But:

```text
7,826 / 96,470 = 8.11%
```

The code-derived calendar-style late count appears closer to:

```text
6,534 late orders
late rate = 6.77%
```

This aligns with the Zeash calendar-date result.

Project implication:

Do not directly quote Moses's headline numbers without recalculating them.

### Caution 2: Late review score estimate differs across reported materials

The reference notes indicate Moses's README reports approximately:

```text
on-time average review score = 4.29
late average review score = 2.57
```

But recalculation from the available code/data appears closer to:

```text
on-time average review score ≈ 4.29
late average review score ≈ 2.27
```

This again aligns more closely with Zeash.

Project implication:

Use my own pipeline to recalculate the final numbers.

## 2.8 Useful Results from Moses

### Satisfaction coding result

Using Moses's 1–3 low-score definition, late orders are more likely to be in the low-review group.

Reference table:

| Delivery status | High, 4–5 | Low, 1–3 |
|---|---:|---:|
| Late | 1,706 | 4,675 |
| On time / early | 73,944 | 15,499 |

Interpretation:

Late delivery is associated with lower satisfaction under both raw score and binary satisfaction logic.

### Retention result

Reference material reports:

| First-order delivery experience | Repeat purchase rate |
|---|---:|
| Late first delivered order | about 2.55% |
| On-time first delivered order | about 3.03% |

Interpretation:

Directionally, first-order late delivery is associated with lower repeat purchase. The absolute difference is small, so this should be supporting evidence.

### Revenue retention result

The reference project analyzes cohort revenue retention and suggests:

```text
most revenue occurs in the first purchase month, and later-month revenue retention drops quickly.
```

This may help frame why post-purchase failures matter early in the customer relationship, but it should not dominate my project.

### Category result

The project highlights high-revenue categories such as:

- health_beauty;
- watches_gifts;
- bed_bath_table;
- sports_leisure;
- computers_accessories.

Its recommendation is to prioritize operational improvements in high-revenue categories when late-rate differences across categories are not strongly concentrated.

## 2.9 Recommendations from Moses

| Recommendation | Meaning for my project |
|---|---|
| Protect first-time customer delivery experience | Useful for recovery priority logic |
| Prioritize high-revenue categories | Useful for business prioritization |
| Treat late delivery as a CX early-warning signal | Directly supports my project framing |
| Add late delivery rate to CX KPI | Useful for KPI plan |

## 2.10 How I Will Use It

I will use Moses as a statistical and retention reference.

What I will reuse conceptually:

- Wilcoxon/Mann-Whitney style comparison for review scores;
- Chi-square test for bad review distribution;
- first-order late vs repeat purchase logic;
- category revenue prioritization logic;
- warning that retention effects may be small.

What I will not copy:

- inconsistent headline numbers;
- one-script repo structure;
- lack of seller/geography analysis;
- lack of service design deliverables;
- any conclusion not recalculated in my own pipeline.

---

# 3. Comparison of the Two Reference Projects

| Dimension | Zeash project | Moses project | How my project will use it |
|---|---|---|---|
| Project type | CX + operations analytics | Statistical delivery/retention analysis | Combine quantitative rigor with service design |
| Business framing | Strong marketplace operations framing | Narrower late delivery/retention framing | Upgrade framing to post-purchase recovery system |
| Data pipeline | SQL-first, audit-heavy, reproducible | R script, simpler pipeline | Use Zeash-style structure |
| Late delivery definition | Strong calendar-date rule | Directionally similar, less explicit | Use calendar-date rule |
| Review score analysis | Strong, uses 1–2 bad review | Uses 1–3 low satisfaction | Main metric = 1–2; sensitivity = 1–3 |
| Statistical tests | Multiple tests and models | Wilcoxon and Chi-square | Use only necessary tests |
| Seller analysis | Strong | Missing | Build seller governance priority map |
| Region analysis | Strong | Missing | Build state/region risk layer |
| Category analysis | Present | Present | Use for prioritization, not only ranking |
| Retention analysis | Present but cautious | More central | Treat as supporting evidence |
| Review text coding | Missing | Missing | Add as original contribution |
| Journey map | Missing | Missing | Add service design synthesis |
| Service blueprint | Missing | Missing | Add frontstage/backstage diagnosis |
| Service recovery concept | Basic recommendations | Basic recommendations | Build three-layer recovery system |
| China localization | Missing | Missing | Add transferability memo |
| KPI validation plan | Partial | Partial | Build full pilot and KPI plan |

---

# 4. Main Gap Identified

Both reference projects prove that Olist can support a strong quantitative analysis of late delivery, review scores, seller risk, and retention.

However, both projects stop mostly at analysis and recommendations. They do not fully answer:

```text
What should the marketplace actually redesign in the post-purchase service system?
```

The main gap is therefore:

> Existing projects analyze delivery failure, but they do not convert the findings into a service design operating model for proactive recovery.

## My Project Contribution

My project will add:

1. review text coding of low-score comments;
2. post-purchase pain point taxonomy;
3. journey map from order placement to review / repeat purchase;
4. service blueprint connecting customers, sellers, logistics, platform systems, and support;
5. opportunity matrix for intervention points;
6. three-layer service recovery concept:
   - delay prediction;
   - proactive notification;
   - escalation / compensation / seller governance;
7. validation plan with primary, secondary, operational, and guardrail KPIs;
8. China marketplace localization memo.

---

# 5. Practical Rules for My Own Work

## Do

- Recalculate all final numbers in my own pipeline.
- Use delivered orders with valid delivery timestamps for late delivery analysis.
- Use calendar-date comparison for late delivery.
- Use `customer_unique_id` for repeat purchase.
- Use review score 1–2 as the main bad review indicator.
- Treat review score 1–3 as optional sensitivity analysis.
- Separate seller late volume from seller late rate.
- Treat retention as supporting evidence.
- Be explicit about which outputs are directly data-backed and which are data-informed synthesis.

## Do Not

- Do not frame the project as only “late delivery affects review score.”
- Do not copy reference project code without understanding it.
- Do not quote reference project numbers as final findings before recalculation.
- Do not claim that Olist data proves the service blueprint.
- Do not claim that Olist directly proves China marketplace behavior.
- Do not overstate retention results.
- Do not build a complex ML model unless the core analytics and service design outputs are already complete.

---

# 6. One-line Takeaway

The reference projects give the quantitative foundation. My project should turn that foundation into a marketplace post-purchase service recovery case study.
