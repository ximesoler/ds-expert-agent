---
id: imbalanced.evaluation.metrics
title: On imbalanced classes report PR-AUC, precision/recall at the operating point, and calibration; never accuracy alone
stage: [modeling, evaluation]
family: [classification]
domain: [any]
type: metric
applies_when: "positive class rate < 20% or user says the event is rare"
enforce: reviewer
confidence: baseline
sources:
  - "Saito & Rehmsmeier — The precision-recall plot is more informative than the ROC plot when evaluating binary classifiers on imbalanced datasets, PLOS ONE (2015)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Headline metrics: PR-AUC (average precision), precision and recall at the chosen threshold, and a calibration measure (Brier score or reliability plot). ROC-AUC is allowed as a secondary number. Accuracy is reported only with the majority-class baseline next to it.

**Why.** With 1% positives, "99% accuracy" is the do-nothing model. ROC-AUC stays optimistic because false positives are diluted by the huge negative class; PR curves expose that.

**How to apply.** Reviewer fails any modeling report whose headline metric is accuracy or whose results table lacks the majority-class baseline and the confusion matrix at the operating threshold.
