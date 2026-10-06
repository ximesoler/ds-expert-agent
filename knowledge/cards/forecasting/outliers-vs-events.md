---
id: forecasting.eda.outliers-vs-events
title: An outlier is either an error, an event, or a regime change; treat each differently
stage: [eda, feature_understanding]
family: [forecasting]
domain: [any]
type: heuristic
applies_when: "spikes, drops, or level shifts detected in EDA"
enforce: gap_detector
confidence: baseline
sources:
  - "Chen & Liu — Joint estimation of model parameters and outlier effects in time series (1993)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Never silently clip or drop anomalies. Classify each detected anomaly with the user: data error (fix), one-off event (model with an indicator or remove from training with a note), structural break (restrict the training window or add a level-shift regressor).

**Why.** Stock-outs, promotions, and system migrations all look like outliers but require opposite handling. Dropping a promotion spike destroys the one thing the business wants predicted.

**How to apply.** EDA report lists every anomaly with date, magnitude, and a proposed class; the orchestrator confirms the class with the user before features are built.

**Ask the user.** "On these dates demand was far outside normal. Do you know what happened: a promotion, a stock-out, a price change, a data problem?"
