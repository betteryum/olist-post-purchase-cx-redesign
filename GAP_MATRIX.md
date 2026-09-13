# Gap Matrix

## Purpose

This matrix clarifies how my project differs from the two reference projects.

The goal is to avoid building another generic Olist EDA project. My project should use Olist data to diagnose post-purchase CX failures and then convert the diagnosis into a service recovery system.

Core positioning:

```text
Reference projects = data analysis of late delivery, review score, seller/category/region, and retention
My project = data diagnosis + service design synthesis + recovery operating model + validation plan
```

---

# 1. Main Gap Matrix

| Module | Zeash Project | Moses Project | Gap in Existing Projects | My Project Contribution | Olist Data Support | Evidence Type | Final Output |
|---|---|---|---|---|---|---|---|
| Business framing | Strong CX + operations framing | Narrow late-delivery / retention framing | Neither fully frames the problem as a post-purchase service recovery system | Frame late delivery as delivery promise failure and marketplace trust breakdown | Medium | Business synthesis from data problem | README, report intro, deck |
| Data preparation | Strong SQL/DuckDB pipeline and audit | Basic R cleaning workflow | Need a clean project-specific master table | Build order-level master table for all modules | Strong | Direct data processing | `master_order_table.csv`, SQL, notebook |
| Late delivery definition | Strong calendar-date rule | Directionally similar but less explicit | Need a clear and defensible late definition | Use calendar-date comparison to avoid same-day false lateness | Strong | Direct data analysis | analysis plan, SQL logic |
| Delivery failure diagnosis | Strong | Basic | Existing analysis mostly reports rate; opportunity is to connect it to intervention severity | Calculate late rate, delay days, severity bands, time patterns | Strong | Direct data analysis | notebook, dashboard, report finding |
| Review score impact | Strong | Strong | Existing projects show relationship but stop at analytical conclusion | Use as core CX harm evidence and trigger for recovery design | Strong | Direct data analysis + statistical testing | notebook, chart, report finding |
| Bad review definition | Uses 1–2 stars | Uses 1–3 stars | Need consistent main metric and sensitivity metric | Main bad review = 1–2; optional sensitivity = 1–3 | Strong | Direct data analysis | metric definitions, KPI baseline |
| Statistical testing | Mann-Whitney, Chi-square, Kruskal-Wallis, OLS/logit | Wilcoxon, Chi-square | Risk of overloading project with tests | Use only tests that support core business claim | Strong | Statistical association | notebook, methodology section |
| Seller risk | Strong seller Pareto and threshold logic | Missing | Need to translate seller risk into governance actions | Build seller priority matrix using late rate and late volume | Strong | Direct data analysis | `seller_risk_table.csv`, priority matrix |
| Category risk | Present | Present with revenue focus | Existing category work is more ranking than intervention logic | Use category as prioritization layer for recovery and dashboard filters | Strong | Direct data analysis | `category_risk_table.csv`, dashboard |
| Region/state risk | Strong | Missing | Need to connect geography to service reliability and support planning | Build region risk table and use geography as escalation factor | Strong | Direct data analysis | `region_risk_table.csv`, dashboard |
| Repeat purchase | Present but cautious | More central | Effect may be small and non-causal | Use first-order late vs repeat purchase as supporting evidence only | Medium | Observational correlation | retention notebook, limitations |
| Revenue retention | Limited | Present | Not central to service recovery design | Optional context only; avoid making revenue retention the main story | Medium | Descriptive business analysis | optional appendix |
| Review text coding | Missing | Missing | Big gap: no structured analysis of what customers complain about | Sample, translate, and manually code low-score comments | Medium-Strong | Text coding + direct review comments | `review_text_coding.csv`, pain taxonomy |
| Pain point taxonomy | Missing | Missing | Existing projects use scores but not pain types | Convert low-score comments into post-purchase pain point categories | Medium | Data-informed synthesis | `pain_point_taxonomy.md` |
| Journey map | Missing | Missing | No customer-view translation of data findings | Build current-state post-purchase journey from order to review/repeat | Medium | Data-informed service design | `journey_map.md/pdf` |
| Service blueprint | Missing | Missing | No frontstage/backstage view of marketplace delivery failure | Map customer actions, platform touchpoints, seller/logistics backstages, support processes | Medium-Weak | Hypothetical / synthesized service design | `service_blueprint.md/pdf` |
| Opportunity matrix | Missing | Missing | Existing recommendations are not structured by impact/effort/evidence | Prioritize interventions by customer harm, feasibility, and data support | Medium | Business synthesis | `opportunity_matrix.md` |
| Intervention priority logic | Basic seller thresholds | Basic recommendations | No order/seller/category/region-based intervention mechanism | Design rules for proactive recovery list | Medium | Data-informed decision logic | report, dashboard extract |
| Service recovery concept | Basic recommendations | Basic recommendations | No complete operating model | Build three-layer recovery system: delay prediction, proactive notification, escalation/compensation/seller governance | Medium | Proposed intervention | `service_recovery_concept.md` |
| KPI framework | Partial | Partial | No full baseline + future validation separation | Separate historical baseline KPIs from future pilot KPIs | Medium | Direct baseline + future plan | `validation_kpi_plan.md` |
| A/B test / pilot plan | Missing | Missing | No validation design for proposed recovery system | Design control/treatment pilot for delay-risk notification | Medium | Future validation design | KPI plan, report recommendation |
| China localization | Missing | Missing | No transferability discussion | Discuss what transfers and what must change for Chinese marketplaces | Weak from Olist | External research + business reasoning | `china_localization_memo.md` |
| Final report | Business-focused but analysis-centered | Course-style analysis report | Need a stronger service design case study narrative | Write report as diagnosis → synthesis → recovery system → validation | Strong as project output | Communication artifact | `final_case_study.md/pdf` |
| Presentation deck | Some dashboard/reporting material | Basic charts | Need stakeholder story | 10–12 slide executive deck | Strong as communication output | Communication artifact | `olist_cx_redesign_deck.pptx` |
| GitHub repo structure | Strong | Simple | Need clean portfolio repo | Use organized folders for SQL, notebooks, outputs, service design, report, presentation | Strong | Reproducibility / portfolio structure | GitHub repo |

---

# 2. Data Feasibility Matrix

This section separates what the Olist dataset can directly prove from what it can only support indirectly.

| Output | Can Olist directly calculate it? | Support Level | How to Write It Honestly |
|---|---:|---|---|
| Late delivery rate | Yes | Strong | Historical delivery delay metric |
| Delay days | Yes | Strong | Historical delay severity metric |
| Late vs on-time review score | Yes | Strong | Association between delivery status and customer rating |
| Bad review rate | Yes | Strong | Review score proxy for dissatisfaction |
| Seller late rate | Yes | Strong | Seller-level operational risk |
| Seller late volume | Yes | Strong | Seller-level impact scale |
| Category late rate | Yes | Strong | Category-level operational risk |
| Region/state late rate | Yes | Strong | Geography-level delivery risk |
| Repeat purchase rate | Yes, using `customer_unique_id` | Medium | Correlation with follow-up purchase behavior |
| First-order late vs repeat purchase | Yes | Medium | Observational association, not causality |
| Review text pain point share | Yes, after coding comments | Medium-Strong | Based on available comment sample, not all customers |
| Pain point taxonomy | Partly | Medium | Data-informed taxonomy from review comments |
| Journey map | No | Medium | Data-informed synthesis using timestamps, reviews, and marketplace logic |
| Service blueprint | No | Medium-Weak | Hypothetical blueprint based on marketplace operating logic |
| Opportunity matrix | No | Medium | Prioritization based on evidence, impact, and feasibility |
| Service recovery concept | No | Medium | Proposed intervention based on diagnosis |
| China localization memo | No | Weak from Olist | Requires external desk research and transferability analysis |
| KPI / validation plan | Partly | Medium | Historical baseline + future pilot design |

Recommended wording for the final report:

```text
This project separates evidence into three levels. Level 1 includes direct analysis from the Olist dataset. Level 2 includes data-informed service design synthesis. Level 3 includes proposed recommendations and future validation plans. The service design artifacts are not claimed as internal Olist documents; they are synthesized from data findings, review text, and marketplace operating logic.
```

---

# 3. Project Contribution by Layer

## Layer 1: Direct Data Analysis

Goal:

```text
Prove that post-purchase delivery failure is measurable and associated with customer dissatisfaction.
```

Modules:

- data audit;
- master order table;
- late delivery rate;
- delay severity;
- review score impact;
- bad review rate;
- seller/category/region risk;
- repeat purchase relationship;
- review text sample and coding.

Outputs:

- SQL scripts;
- notebooks;
- tables;
- charts;
- dashboard extracts.

## Layer 2: Data-informed Service Design

Goal:

```text
Translate the quantitative diagnosis into customer experience and operating-system failure points.
```

Modules:

- pain point taxonomy;
- journey map;
- service blueprint;
- opportunity matrix;
- intervention priority rules.

Outputs:

- `service_design/pain_point_taxonomy.md`;
- `service_design/journey_map.md`;
- `service_design/service_blueprint.md`;
- `service_design/opportunity_matrix.md`.

## Layer 3: Recommendation and Validation

Goal:

```text
Propose a service recovery system and explain how it should be tested.
```

Modules:

- delay prediction concept;
- proactive notification concept;
- escalation and compensation logic;
- seller governance playbook;
- China localization memo;
- KPI and pilot plan.

Outputs:

- `service_design/service_recovery_concept.md`;
- `service_design/china_localization_memo.md`;
- `service_design/validation_kpi_plan.md`;
- final report recommendation section;
- presentation deck.

---

# 4. Originality Statement

This project is original because it does not stop at identifying late delivery as a problem.

The original contribution is:

```text
Using late delivery as an observable signal of post-purchase CX failure, then translating the evidence into a service recovery operating model for marketplace platforms.
```

Compared with the reference projects, the added value is:

1. more explicit service design framing;
2. review text coding rather than only review score analysis;
3. pain point taxonomy;
4. customer journey map;
5. service blueprint;
6. intervention priority logic;
7. recovery concept with trigger/action/KPI structure;
8. China localization and transferability discussion.

---

# 5. What Not to Overclaim

Avoid these claims:

```text
Olist data proves the proposed service recovery system will work.
Olist data directly reveals the internal Olist service blueprint.
Late delivery causes low reviews with full causal certainty.
The same design can be directly applied to China without adaptation.
Retention is the strongest business impact finding.
```

Use these claims instead:

```text
Olist data shows that late delivery is strongly associated with low review scores.
Seller, category, and region risks are unevenly distributed.
Review comments can be coded into post-purchase pain point categories.
The journey map and blueprint are data-informed syntheses, not internal Olist documents.
The service recovery system is a proposed intervention that should be validated through a future pilot.
China localization requires additional desk research and adaptation to local platform rules and user expectations.
```

---

# 6. Priority for Version 1

Version 1 should prioritize completion over complexity.

Must-have modules:

1. master order table;
2. late delivery and review score analysis;
3. seller/category/region risk segmentation;
4. review text coding sample;
5. pain point taxonomy;
6. journey map;
7. service blueprint;
8. recovery concept;
9. KPI plan;
10. final report and deck.

Nice-to-have modules:

- regression model;
- ML prediction model;
- geospatial map;
- advanced NLP topic modeling;
- interactive dashboard polish;
- China localization with deeper legal/platform research.

Do not start nice-to-have modules until the must-have modules are complete.
