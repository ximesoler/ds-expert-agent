---
id: fraud.features.time-patterns
title: "Encode unusual timing: hour-of-day vs the entity's habit, night and weekend activity, bursts"
stage: [eda, feature_engineering]
family: [classification, anomaly]
domain: [fraud]
type: heuristic
applies_when: "event timestamps present"
enforce: advisory
confidence: baseline
sources:
  - "Le Borgne et al. — Fraud Detection Handbook (2022), ch. 3"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Night-time and weekend transactions have higher fraud rates in most card datasets, but the stronger signal is the deviation from the entity's own hourly profile. Encode hour cyclically (sin/cos), add "share of this entity's past activity in this hour bucket", "seconds since previous event", and burst counts (events in 10 min / 1 h).

**EDA to show.** Fraud rate by hour of day and by day of week, overall and per segment; distribution of inter-event times for fraud vs legit.
