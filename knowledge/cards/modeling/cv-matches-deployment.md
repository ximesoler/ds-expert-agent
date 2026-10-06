---
id: modeling.validation.cv-matches-deployment
title: The validation scheme must imitate how the model will be used
stage: [modeling]
family: [any]
domain: [any]
type: principle
applies_when: "always"
enforce: reviewer
confidence: baseline
sources:
  - "Kaufman et al. (2012); Bergmeir & Benítez (2012)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** New customers in production → group K-fold by customer. Future periods → time-based split. New stores → leave-store-out. Rare events → stratified. If the validation design does not match the deployment question, the number is meaningless no matter how good it looks.

**Ask the user.** "When the model runs, will it see entities it was trained on, or new ones? Will it see later time periods?"
