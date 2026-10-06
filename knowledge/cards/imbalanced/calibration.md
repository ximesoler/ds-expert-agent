---
id: imbalanced.evaluation.calibration
title: If the probability is used (ranking budgets, expected loss), check and fix calibration
stage: [modeling, evaluation]
family: [classification]
domain: [any]
type: technique
applies_when: "scores are used as probabilities or combined with costs"
enforce: evaluator
confidence: baseline
sources:
  - "Niculescu-Mizil & Caruana — Predicting good probabilities with supervised learning (2005)"
  - "Guo et al. — On calibration of modern neural networks (2017)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** GBMs and resampled models produce distorted probabilities. Use reliability diagrams and Brier score; calibrate with isotonic or Platt scaling on a held-out set; re-check after any resampling or class weighting.

**Pitfalls.** Calibrating on the training folds; forgetting that class weights shift the base rate and must be undone before using scores as probabilities.
