---
id: data.understanding.units-and-semantics
title: Confirm units, currencies, time zones, and what each key column means with the user
stage: [data]
family: [any]
domain: [any]
type: question
applies_when: "always"
enforce: gap_detector
confidence: baseline
sources:
  - "Practitioner standard"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Columns named `amount`, `qty`, `date` hide mixed currencies, units, and time zones. The data profiler lists the ambiguous columns and the orchestrator confirms them in one batch.

**Ask the user.** "Is `amount` in a single currency? Is `date` the order date or the ship date, local time or UTC? Are quantities in units or cases?"
