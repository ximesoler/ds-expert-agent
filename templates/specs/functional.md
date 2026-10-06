---
stage: <n>                      # 2..4 (stage 1 has no stage functional)
name: <data | experiment | ship>
version: 1
status: draft                   # draft | approved | stale | superseded
depends_on:
  project_functional: 1
  upstream: {stage: <n-1>, functional: 1}
owner_expert: <expert>
generated_by: dsx spec gen      # skeleton generated from SPECS.md + cited project sections; completed by the stage gap detector
status: draft                   # draft (skeleton only) | approved (gap detector done)
approved_by:                    # user (after the gap waves)
user_knowledge_hash:            # sha256 of the ## User knowledge section; pinned by the technical
panel: []                       # experts that proposed questions
approved_on:
applied_cards: []
---

## 1. Purpose statement
<!-- What this stage must make true for the project. Cite project functional sections. -->

## 2. Inputs
<!-- Upstream artifacts with versions / hashes. -->

## 3. Deliverables
<!-- Files and reports this stage promises to the next stage. WHAT, not how. -->

## 4. Acceptance criteria
<!-- Verifiable, numeric where possible. These become DoD items. -->

## 5. Questions for the user (gap detector)
<!-- Merged by dsx from schema slots + gap-detector cards + panel proposals. Each row: id · slot_key · question · why_it_matters (1–5) · classification (must_ask | pending_data | safe_default) · needs_data · default · source -->

## 6. Constraints
<!-- From project functional §4 plus stage-specific. -->

## 7. Gaps table
| id | question | why_it_matters | classification | status | answer | decided_by | date |
|---|---|---|---|---|---|---|---|

## 8. User knowledge
<!-- Answers given in the gap waves, verbatim + normalised; hashed separately (user_knowledge_hash). -->

## 9. Change log
| version | date | change | triggered_by |
|---|---|---|---|
