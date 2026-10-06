---
id: fraud.monitoring.adversarial-drift
title: Fraud adapts to the model; monitor weekly and plan frequent retraining
stage: [deployment, monitoring]
family: [classification, anomaly]
domain: [fraud]
type: checklist
applies_when: "fraud or abuse detection in production"
enforce: advisory
confidence: baseline
sources:
  - "Dal Pozzolo et al. (2018) on concept drift in fraud streams"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Fraud rings probe thresholds and move. Monitor alert volume, precision on matured labels, feature drift (PSI), and score distribution weekly; set a retraining cadence (often monthly) and a trigger (precision drop > X) up front; keep a challenger model in shadow.
