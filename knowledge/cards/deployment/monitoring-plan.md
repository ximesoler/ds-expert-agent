---
id: deployment.monitoring.plan
title: "Every shipped model has a monitoring plan: input drift, prediction drift, delayed performance, and a retrain trigger"
stage: [deployment, monitoring]
family: [any]
domain: [any]
type: checklist
applies_when: "always"
enforce: reviewer
confidence: baseline
sources:
  - "Breck et al. — The ML Test Score (2017)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** The handoff includes: PSI/KS on key features vs training distribution, prediction distribution tracking, performance on matured labels with the label delay stated, alert thresholds, retraining cadence and trigger, an owner, and a rollback path (previous model version in the registry).
