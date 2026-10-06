---
id: <stage>-<iteration>
stage: <n>
iteration: <k>
depends_on: {technical: 1}
status: planned                 # planned | running | done | aborted
limits: {max_turns: 40, max_usd: 2.0, max_minutes: 45}
approved_by: auto               # auto | user
consulted: []                   # experts consulted by dsx plan consult (owners in technical §Method, cap 2)
created_on:
---

## 1. Hypothesis
<!-- One falsifiable sentence with a number. -->

## 2. Inputs
<!-- Exact paths + versions / hashes / run ids consumed. -->

## 3. Tasks
| # | owner | action / command | expected output path |
|---|---|---|---|

## 4. Expected outputs
<!-- Artifacts, metrics, report sections. -->

## 5. Acceptance rule
<!-- Numeric criterion the build result is judged against. -->

## 6. Stop conditions
<!-- Limits above + task timeouts. -->

## 7. Not in scope

## 8. Decision table
| outcome | next |
|---|---|
| accepted | |
| rejected | |
| inconclusive | |
