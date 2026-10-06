---
id: imbalanced.modeling.resampling-pitfalls
title: Prefer class weights and threshold tuning over SMOTE; never resample the validation set
stage: [modeling]
family: [classification]
domain: [any]
type: pitfall
applies_when: "positive class rate < 20%"
enforce: reviewer
confidence: baseline
sources:
  - "van den Goorbergh, van Smeden, Timmerman, Van Calster — The harm of class imbalance corrections for risk prediction models, JAMIA (2022)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** For tree ensembles, class weighting plus threshold selection matches or beats oversampling, keeps probabilities interpretable, and avoids synthetic-sample artifacts. If resampling is used, it happens inside the training fold only, and evaluation is always on the natural prevalence.

**Why.** Resampling the evaluation data inflates every metric and hides the real precision at deployment prevalence.

**How to apply.** Reviewer checks that the evaluation set prevalence equals the raw prevalence and that resampling, if any, is inside the pipeline (fit on train fold only).
