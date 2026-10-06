---
id: modeling.interpretation.importance-not-causality
title: Feature importance and SHAP explain the model, not the business; do not recommend interventions from them
stage: [evaluation]
family: [any]
domain: [any]
type: pitfall
applies_when: "stakeholders ask 'what drives X'"
enforce: advisory
confidence: baseline
sources:
  - "Lundberg & Lee — SHAP (2017); Pearl — Causality"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** "Discount is the top feature for churn" does not mean giving discounts reduces churn. Present importances as "what the model uses", flag confounders, and recommend an experiment or a causal design when the user wants to act on a driver.
