---
id: imbalanced.evaluation.threshold-by-cost
title: Choose the decision threshold from the cost matrix or the capacity, never 0.5
stage: [business, evaluation]
family: [classification]
domain: [any]
type: question
applies_when: "a classifier output drives an action with asymmetric costs or limited capacity"
enforce: gap_detector
confidence: baseline
sources:
  - "Elkan — The foundations of cost-sensitive learning, IJCAI (2001)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** The threshold is a business parameter. Derive it either from the cost of a false positive vs a false negative (expected-cost minimization on validation) or from capacity (top-k where k is what the team can act on). Report the metric at that threshold and the sensitivity to it.

**Ask the user.** "What happens when the model flags a case: who acts, how much does it cost, how many can you handle per day? What does a missed case cost?"
