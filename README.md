# Heterogeneous Effects of State Repression on Social Movement Mobilization

This repository contains code and analysis for a final project on how state repression reshapes protest mobilization in Indonesia. Using an admin1-week panel built from ACLED event data and merged with V-Dem country-year indicators, the project predicts next-week protest counts from repression, protest history, and regime context. The modeling pipeline compares lasso, random forest, and XGBoost, and interprets results using coefficients, variable importance, partial dependence plots, and SHAP values. 

## Research Question

This project asks: under what conditions does state repression reshape the trajectory of contentious politics, and what do these heterogeneous effects reveal about movement durability under varying regime constraints? More specifically, the project focuses on three questions:

1. **Mobilization (short-run):** How does repression at time *t* affect protest activity at time *t + 1*?
2. **Durability:** How does repression affect the speed at which areas enter quiescence?
3. **Heterogeneity:** How do regime characteristics and baseline movement capacity moderate these effects?

The current repository implements the short-run predictive component as a supervised regression task: predicting next-week protest counts from repression and contextual features. 

## Why This Project Matters

The repression–mobilization relationship is not mechanically one-way. Repression can deter protest, provoke backlash, or reconfigure contention in more complicated ways depending on its form and context. This matters substantively for understanding authoritarian governance, protest durability, and the conditions under which state coercion escalates rather than stabilizes conflict. In this project, the goal is not only to ask whether repression “works” on average, but to identify which features matter most for future modeling of durability and heterogeneous effects. 

## Data

The analysis combines two main sources:

- **ACLED (Armed Conflict Location & Event Data Project)** for event-level protest and repression data in Indonesia, covering 2019–2025.
- **V-Dem (Varieties of Democracy, v15)** for country-year indicators of electoral democracy, liberal democracy, civil society repression, freedom of expression, and censorship. 

### Unit of analysis

The unit of analysis is **admin1-week**: one Indonesian province in one week. A balanced panel is constructed so that province-weeks with no recorded events are coded as zero rather than dropped. This is substantively appropriate here, because “no event” is an observed zero, not missing data.

### Key variables

**Outcome**
- `protest_t1`: protest count in the next week for the same province

**Main treatment**
- `repression_events`: count of protest-related repression sub-events
- `repression_fatalities`: fatalities occurring in repression events

**Alternative treatment**
- `repression_broad`: protest-linked repression plus violence against civilians

**Temporal features**
- `protest_lag1`, `protest_lag2`
- `repress_lag1`
- `protest_roll4`, `repress_roll4`
- `cum_repress_year`
- `week_of_year`, `month_num`

**Contextual features**
- `v2x_polyarchy`
- `v2x_libdem`
- `v2csreprss`
- `v2x_freexp_altinf`
- `v2mecenefm` :contentReference[oaicite:5]{index=5}

## Analytical Strategy

The project compares three supervised learning models:

- **Lasso regression** for feature selection and interpretable coefficients
- **Random forest** for nonlinear relationships and feature importance
- **XGBoost (Poisson objective)** for count-aware boosting and SHAP-based interpretation

All models are evaluated using a **time-based 80/20 train/test split**, so the models learn from earlier weeks and predict later weeks. For lasso, lambda is tuned using **time-ordered folds** rather than random cross-validation. This is an attempt to respect the temporal structure of the panel, though the data remain non-IID because observations are correlated both within province over time and across provinces within the same week. 

## Main Descriptive Patterns

Several descriptive patterns shape the modeling problem:

- Protest counts are zero-heavy and right-skewed.
- Repression is rare: among protest events, **96.2%** are coded as peaceful protest, **3.8%** as protest with intervention, and **0.1%** as excessive force against protesters.
- Protest momentum is the strongest raw signal in the data: past protest variables are much more strongly correlated with next-week protest than repression variables are. 

This means the models are trying to detect a relatively sparse repression signal on top of a much louder background pattern of protest persistence. Tiny statistical goblin, large substantive implications. 

## Main Findings

Across models, **protest momentum** is the dominant predictor of next-week protest activity. The strongest features are lagged protest measures and especially the 4-week protest rolling mean. Repression adds a weaker but still interpretable signal. In the lasso, repression shows a split pattern: **repression fatalities** are positively associated with next-week protest, while **repression events** are negatively associated with it, suggesting a distinction between backlash and deterrence.

In model comparison, performance is similar across the linear and tree-based baselines:

- **Lasso (lambda.min):** RMSE = 1.82, MAE = 1.20
- **Random Forest:** RMSE = 1.81, MAE = 1.22
- **XGBoost (Poisson):** RMSE = 2.03, MAE = 1.29

The tuned XGBoost improves over its default configuration but still trails lasso and random forest on this dataset. Its main value in this project is interpretive rather than predictive, because it enables SHAP-based decomposition of observation-level heterogeneity. 

## Repository Contents

```text
.
├── README.md
├── 03-Final-report.qmd
├── 03-Final-report.pdf
├── data_raw/
│   └── acled_idn_2019_2025.csv
└── outputs/
    ├── figures/
    └── tables/
