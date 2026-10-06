---
stage: <n>                      # 2..4
version: 1
status: active                  # active | stale | superseded
depends_on: {functional: 1}
owner_expert: <expert>
applied_cards: []               # [{id, hash}]
pins: {}                        # written by dsx pin: {project_functional: v, user_knowledge_hash: …, deliverables: {id: hash}}
panel: {lead: , contributors: []}
panel_log: []                   # [{expert, section, accepted|rejected, reason}]
dod: []                         # card ids + local.S<n>.<slug>
limits: {max_iterations: 5, max_usd: 5.0, max_minutes: 120}
---

## 1. Definitions
<!-- Operational definitions this stage uses (grain, target rule, protocol P, serving contract ...). See SPECS.md §<n>.T for the mandatory items of this stage. -->

## 2. Design decisions
<!-- Each decision with rationale and the knowledge cards applied (ids). -->

## 3. Method
<!-- Procedure, tools, conventions. Mandatory items per stage in SPECS.md. -->

| method_id | step | owner_expert | call_point |
|---|---|---|---|
<!-- drives /ds:plan consultation and task owners -->

## 3b. Conflicts
<!-- Contradictory contributions and how the lead resolved them (logged to decisions.log). -->

## 4. Verification checks (Definition of Done)
| id | check | verification (test:<path> | reviewer | user) | enforced_by |
|---|---|---|---|
<!-- generated from cards + local items; edit only to add local items -->

## 5. Risks and open questions
<!-- Classified gaps that remain. -->
