---
id: forecasting.evaluation.rolling-origin
title: Validate with rolling-origin backtests matched to the real horizon and cadence
stage: [modeling, evaluation]
family: [forecasting]
domain: [any]
type: checklist
applies_when: "any time series target"
enforce: reviewer
confidence: baseline
sources:
  - "Bergmeir & Benítez — On the use of cross-validation for time series predictor evaluation (2012)"
  - "Tashman — Out-of-sample tests of forecasting accuracy (2000)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Evaluate with multiple forecast origins, each forecasting the full business horizon with the same lead time the production system will have, and report error by horizon step.

**Why.** A single split measures one origin's luck. Error grows with horizon; the user decides on month 3, not day 1. Lead time (data available at forecast time) changes which lags are legal.

**How to apply.** Choose ≥ 6 origins spanning at least one seasonal cycle; keep a gap equal to the data latency; report metric per horizon bucket and per segment; use the same origins for every candidate.

**Pitfalls.** Random K-fold on time series; refitting hyperparameters per origin on the test window; letting global models see the future of other series (cross-series leakage in time).
