---
id: imbalanced.modeling.stratified-grouped-cv
title: Stratify folds by class and group by entity; use time-based splits when labels arrive over time
stage: [modeling]
family: [classification]
domain: [any]
type: checklist
applies_when: "rare positives, repeated entities, or time-ordered data"
enforce: reviewer
confidence: baseline
sources:
  - "Kaufman, Rosset, Perlich — Leakage in data mining (2012)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Few positives make fold variance large: stratify. Repeated customers across folds leak identity: group. Labels that mature over time (fraud, churn, default) require a temporal split with a label-maturity gap. Report the metric's spread across folds, not just the mean.
