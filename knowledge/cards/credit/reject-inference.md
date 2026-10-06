---
id: credit.data.reject-inference
title: Training only on approved loans biases the model; account for rejected applicants
stage: [data, modeling]
family: [classification]
domain: [credit]
type: pitfall
applies_when: "historical data contains only funded loans"
enforce: gap_detector
confidence: baseline
sources:
  - "Hand & Henley — Can reject inference ever work? (1993); Siddiqi (2006) ch. on reject inference"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Outcomes exist only for approved applicants, who were pre-selected by the old policy. State this limitation in the brief, use reject-inference techniques if rejected applications are available, and define the population the model is valid for.

**Ask the user.** "Does the data include rejected applications? Do you plan to use the model on a broader population than the historical approvals?"
