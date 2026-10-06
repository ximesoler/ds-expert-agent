---
id: modeling.process.baseline-first
title: Train the dumbest reasonable model first and keep it in every results table
stage: [modeling]
family: [any]
domain: [any]
type: checklist
applies_when: "always"
enforce: reviewer
confidence: baseline
sources:
  - "Practitioner standard; Hyndman FPP3 ch. 5 for forecasting benchmarks"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Classification: majority class and logistic regression. Regression: mean/median and ridge. Forecasting: naive and seasonal naive. They set the floor, catch leakage (a baseline that scores 0.99 is a leak), and tell the user what the complexity buys.
