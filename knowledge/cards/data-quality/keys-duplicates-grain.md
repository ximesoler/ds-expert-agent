---
id: data.understanding.keys-and-grain
title: Establish the primary key and the grain of every table before any statistic is trusted
stage: [data]
family: [any]
domain: [any]
type: checklist
applies_when: "always"
enforce: reviewer
confidence: baseline
sources:
  - "Practitioner standard"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** The data card states, per table: candidate key, uniqueness check result, duplicate count, grain in words ("one row per SKU per store per day"), time coverage, and the join keys to other tables with match rates. Duplicates inflate counts and leak across folds; wrong grain makes every aggregate wrong.
