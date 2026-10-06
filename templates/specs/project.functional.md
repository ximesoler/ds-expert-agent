---
version: 1
status: draft                   # draft | approved | superseded
approved_by:
approved_on:
persona: <business | junior_ds | senior_ds>
family: <classification | regression | forecasting | anomaly>
domain: <demand | fraud | churn | credit | pricing | maintenance | marketing | any>
applied_cards: []            # [{id, hash}]
supersedes:
provenance:
  requirements_schema: {id: , hash: }
  cards_set_hash:
  persona:
  waves: []                   # decisions.log line ranges per wave
pending_data: []              # gap ids deferred: asked by the Data gap detector, or closed by Data plan 1 profiling
---

## 1. Decision
## 2. Unit of prediction and target
## 3. Success criteria
### a. Business metric and threshold
### b. Technical proxy metric and the translation formula to business units
### c. Baseline to beat
### d. "Ship" and "stop" definitions
## 4. Costs and constraints
## 5. Scope
## 6. Data expected (prose: sources the user believes exist, grain, period, owner, caveats; no paths, nothing verified)
## 7. Assumptions
| assumption | tag (confirmed_by_user | safe_default) |
|---|---|
## 8. Gaps table
| id | question | why_it_matters | classification | status | answer | decided_by | date |
|---|---|---|---|---|---|---|---|
## 9. Consumers and deployment intent
## 10. Change log
