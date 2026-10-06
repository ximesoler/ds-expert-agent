---
id: churn.business.snapshot-framing
title: Define churn as an event in a window after a snapshot date; features stop at the snapshot
stage: [business, data, feature_understanding]
family: [classification]
domain: [churn]
type: question
applies_when: "target is customer attrition, cancellation, or inactivity"
enforce: gap_detector
confidence: baseline
sources:
  - "Practitioner consensus; see e.g. Verbeke, Martens, Baesens — Social network analysis for customer churn prediction (2014) for framing"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Build the dataset as (customer, snapshot date) rows: features computed from data before the snapshot, label = churned within the next N days. Use several snapshot dates so the model learns across time. Without this framing, churn models leak the outcome.

**Ask the user.** "What exactly counts as churn (cancellation, 60 days inactive, downgrade)? How far ahead do you need the warning so an action is still possible? How often will you score customers?"
