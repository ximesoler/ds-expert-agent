---
id: forecasting.modeling.intermittent-demand
title: Intermittent demand needs its own methods and its own metrics
stage: [eda, modeling, evaluation]
family: [forecasting]
domain: [demand]
type: technique
applies_when: "share of zero periods > 30% or ADI > 1.32"
enforce: reviewer
confidence: baseline
sources:
  - "Croston (1972); Syntetos & Boylan (2005) classification by ADI and CV²"
  - "Teunter, Syntetos, Babai — TSB method (2011)"
  - "Kolassa — Evaluating predictive count data distributions in retail sales forecasting (2016)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Classify each series by average demand interval (ADI) and CV² of non-zero sizes into smooth / erratic / intermittent / lumpy. Intermittent and lumpy series should be modeled with Croston/SBA/TSB or a global GBM with zero-inflation handling, and evaluated with scaled metrics (RMSSE, MASE) or quantile loss, never MAPE.

**Why.** MAPE is undefined at zero and explodes near it; point forecasts for intermittent series are often legitimately "0.3 units", which is useless as a point but useful as a distribution for reorder decisions.

**How to apply.** Add the classification table to the EDA report; choose metric per class; for inventory decisions forecast quantiles (service-level quantile) not the mean.

**Pitfalls.** Aggregating intermittent SKUs to weekly/monthly often removes intermittency and is the better decision grain; check with the user whether the decision is really made at the daily level.

**Ask the user.** "What service level (e.g., 95%) do you plan for? Decisions are made per day, week, or month?"
