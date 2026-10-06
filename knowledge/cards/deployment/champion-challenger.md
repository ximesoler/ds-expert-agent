---
id: deployment.release.champion-challenger
title: Ship in shadow or champion/challenger before replacing a decision system
stage: [deployment]
family: [any]
domain: [any]
type: technique
applies_when: "a model replaces an existing rule set, model, or human decision"
enforce: advisory
confidence: baseline
sources:
  - "Industry practice (banking model risk management, SR 11-7)"
added_by: baseline
added_on: 2026-10-05
---
**Claim.** Run the new model alongside the current decision maker on live traffic without acting on it, compare on matured outcomes, then switch with a documented acceptance criterion. Keep the old system available for rollback.

**Ask the user.** "Is anything deciding this today (rules, a model, a person)? Can we run side by side for a few weeks?"
