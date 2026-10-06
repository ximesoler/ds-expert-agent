---
id: forecasting.modeling.beat-the-naive
title: Seasonal naive is the bar; a model that does not beat it is not shipped
stage: [modeling, evaluation]
family: [forecasting]
domain: [any]
type: checklist
applies_when: "any time series target"
enforce: reviewer
confidence: baseline
sources:
  - "Hyndman & Athanasopoulos — FPP3, ch. 5.2 simple forecasting methods"
  - "Makridakis, Spiliotis, Assimakopoulos — M5 accuracy competition results, IJF 2022"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Always report naive, seasonal naive, and a simple ETS next to every candidate, on the same backtest, in the same metric. Ship the simplest model within noise of the best.

**Why.** In the M-competitions a large share of sophisticated models lost to seasonal naive on some horizons. Stakeholders cannot judge "MAE 42" without "seasonal naive MAE 47".

**How to apply.** Reviewer must see a results table with the benchmarks as rows and the skill score (1 − model/naive) per horizon bucket.

**Pitfalls.** Comparing on a single train/test split; comparing models fitted on different windows; using a benchmark with a different seasonal period than the data.
