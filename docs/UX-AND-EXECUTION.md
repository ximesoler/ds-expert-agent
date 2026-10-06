# UX and execution model — the `/ds:*` plugin (v0.3)

**Status:** v0.3 (2026-10-06). Verb-first grammar, per-stage gap detector, panel at every step. Aligned with ARCHITECTURE.md v0.5, SPECS.md v2.1, MODEL-POLICY.md.

## 1. Principles

1. **Three verbs, every stage.** `/ds:spec` writes the WHAT (functional, via the stage's gap detector) or the HOW (technical, built by the panel); `/ds:plan` writes one bounded iteration; `/ds:build` executes it. `/ds:gate` certifies. `/ds:next` runs whichever of these the board implies.
2. **Specs and deliverables form a dependency graph owned by `dsx`**; headers carry pins only; a change marks dependents stale and never deletes; v1 staleness is stage-level with a cost figure (section-level for the project functional).
3. **Chain what is free, refuse what costs.** Free deterministic prerequisites (skeleton generation, board refresh, validate, pin, impact) are chained automatically; anything that spends tokens or asks the user stops with the exact next command in the footer.
4. **Gates certify evidence** through manifests; **state is snapshots first**, git optional.

## 2. Grammar

```
/ds:spec  [--functional|--technical] [--business|--data|--modeling|--ship] ["text"] [--regen] [--show]
/ds:plan  [--data|--modeling|--ship] ["hypothesis or instruction"] [--regen]
/ds:build [--data|--modeling|--ship] [--task n] [--resume] [--wait]
/ds:gate  [--stage]      /ds:next      /ds:goto <stage>
/ds:status  /ds:history  /ds:impact  /ds:change  /ds:sync  /ds:learn
```

| Rule | Detail |
|---|---|
| Stage names | `business` (`bu`), `data` (`eda`), `modeling` (`experiment`), `ship` (`deploy`), or `--stage N`. Internal folders stay `02-data`, `03-experiment`, `04-ship`. The parser echoes its reading before acting: "→ stage 2 data, functional, gaps". |
| Flags, not positionals | `text` = everything after the first non-flag token or after `--`, verbatim. |
| One parser | Commands are thin: `dsx parse <verb> "$ARGUMENTS" --json` → skill. `/ds:next` emits the same JSON from `dsx board --json`, prints "→ running /ds:spec --functional --data", and batches free steps up to the next question or decision. |
| Cursor | Explicit stage flags never move it; `gate approve`, `build finish`, `/ds:goto` do. |
| Footer | Every verb ends with two dsx-rendered lines: `Done: <artifact> v<n> — <plain clause> (asked a, deferred b, defaulted c; panel: …)` / `Next: <exact command> (<what>, ≈$x) · alt: <exact command>`. |
| Bare `/ds:spec` | Functional if `draft`/missing/stale (runs the gap detector), else technical if missing/stale (panel draft), else both current + text → "Is this about *what* we need, or *how* I do it?", else no-op showing both headers. |
| Refusals | `E_STAGE` (two stage flags), `E_NO_TECH` (`--business --technical`), `E_GATE` (stage ahead of an unpassed gate → `/ds:gate n`), `E_FUNCTIONAL_DRAFT` (technical before the gap detector ran), `E_NO_PLAN`, `E_NO_DATA` (build before files exist), `E_RUNNING` (`--wait`). Always with the exact command. |

Parse output: `{verb, artifact, stage, stage_source: flag|cursor|last_card, iteration, text, mode: create|gaps|revise|regen|resume, prereqs: [{step, kind: free|paid, asks_user, auto, cmd}], persona, refusal}`.

Resolution order: (1) conflicting flags → refuse; (2) stage ← flag, else fresh `last_card.stage`, else cursor; (3) unpassed upstream gate → `E_GATE`; (4) artifact per the bare-spec rule; (5) mode from flags or state; (6) build lock → `E_RUNNING`; (7) prereqs filled, free ones `auto: true`, first paid one is the stop point; (8) commands never write `project.yaml`.

## 3. What each verb does, per stage

### 3.1 `/ds:spec --functional` — the stage gap detector (WHAT)

| Stage | Who proposes the questions | Opening line (business persona) |
|---|---|---|
| 1 Business | analyst (schema slots + business cards); **no data, no files, no profiling** | "Tell me the decision this will help you make. No data needed yet." |
| 2 Data | data-expert + analyst (reviewer only when a gap-detector card flags) | "Now the data. What do you have, where is it, and what does one row mean? A sentence is fine; paths are better." |
| 3 Modeling | developer + evaluator | "What matters most when the model is wrong: being off on big products, or on many small ones? Any method you don't want?" |
| 4 Ship | developer + writer | "Who runs this once it exists, how often, and where should a warning go?" |

Procedure (stages 2–4): `dsx spec gen` skeleton (free) → `dsx schema select --stage` slots + gap-detector cards → `dsx gaps propose` call sheet → one subagent per expert in parallel returns candidate gaps (JSON, ≤ 8 each) → `dsx gaps merge` (schema wins on collision, near-duplicates collapsed, cap 12, persona re-classification) → `analyst.prefill` from the user's text, if any → `dsx wave next` asks ≤ 4 `must_ask` with `needs_data: false` per wave, rewritten per persona, "I don't know" always allowed (→ `safe_default` stated back, or `pending_data`) → answers in `## User knowledge` (hashed) → `functional.md` v1 `approved_by: user`. Stop when no askable `must_ask` remains or the persona wave limit is hit (business 2–3, junior 3, senior 1–2); the rest become `pending_data`, asked after plan 1 profiling.

Text and no text are the same command; text only pre-fills slots. No data yet: the agent asks for a description and a date, records `files: pending`, still drafts the technical from the description, writes plan 1, and `/ds:build` refuses with `E_NO_DATA` until files are in `data/raw/`.

### 3.2 `/ds:spec --technical` — built by the panel (HOW)

Refused while the functional is `draft`. Phase A: contributors in parallel (sonnet/low, ≤ 3, cap ≈ $0.10) return section contributions, cards `{id, hash}` and DoD candidates with `verification`. Phase B: the lead drafts the full file at the oracle rung, records accept/reject in `panel_log`, resolves contradictions in `## Conflicts`; `dsx validate` rejects duplicate DoD ids and missing verification.

| Stage | Lead | Contributors |
|---|---|---|
| Data | data-expert | reviewer, analyst |
| Modeling | developer | evaluator, reviewer |
| Ship | developer | writer, reviewer |

The user is not asked. Senior: confirm / revise / edit file. Business and junior: a persona card ("Drafted by the data expert · checked by reviewer (2 checks added) · evaluator (1 note)") with "Change something? Tell me in words, or /ds:next". An instruction → the lead returns a complete new file → `change_method` (planned plans aborted, deliverables stale, a running build finishes under its pinned version and is marked `superseded_method`). Instructions that change WHAT are refused to `/ds:change`. Protocol P changes after Modeling iteration 1 need explicit confirmation of the ladder rerun cost.

### 3.3 `/ds:plan` — only the experts needed

`dsx plan consult` selects experts deterministically: owners in the technical `§Method` table ∪ owners of the previous build's `next_tasks` ∪ evaluator at the Modeling exit, capped at 2. With a user hypothesis, only owners of the tasks it implies. Each consulted expert returns ≤ 2 `{hypothesis|task, owner, est_usd, inputs, outputs, risks}`; the planner merges and ranks, writes `plans/<n>.md` with an `owner` per task. Senior picks among ≤ 3; business gets the recommendation. Card: hypothesis in plain words · "Who acts: developer writes checks · reviewer verifies · data expert interprets" · cost · time · "Run it? (yes / change / skip)". Reviewer is never consulted at plan time.

### 3.4 `/ds:build` — the plan's tasks, by their owners

Whole plan, one verdict. `dsx build start` returns the task DAG and a lock; tasks dispatch to their `owner` at the routed rung, parallel ≤ 3, `dsx expect` per output; reviewer after code (advisory items), `dsx dod --run` for test items, dispatch / wait / fetch, interpretation (Modeling: dsx precomputes accepted/rejected vs noise), `dsx build finish`. Runs without asking when within budget and no `must_ask` is open; asks for the batched `pending_data` wave and above-budget steps. `E_RUNNING` on a second build (`--wait` attaches); `--task n` / `--resume` skip tasks with evidence. Result card: tried → happened (number vs noise) → business units → recommendation → footer. Plain-word steering: "try another model" → `/ds:plan --modeling "…"`; "I don't trust this" → the agent asks why and maps to `back_to:2`, `change_method`, or the error-analysis view.

### 3.5 `/ds:gate`

`dsx dod` → manifest → writer renders the report → approve / revise WHAT / revise HOW / stop → `dsx gate approve` snapshots, advances the cursor, chains the next stage's free skeleton, and prints the next command.

## 4. The board

`project.yaml` (written only by dsx) adds two fields to the v0.2 board:

```yaml
last_card: {kind: technical|plan|gate, stage: 2, version: 1, hash: 7c…, shown_at: …, session: …}
locks: {build: {stage: 3, iteration: 3, pid: …, since: …}}
```

Bare text with no verb routes to the matching revise only when `last_card` is fresh and its hash matches the file; otherwise the thread asks "WHAT or HOW, which stage?". `SessionStart` re-prints the last card.

## 5. Dependency model and invalidation

Unchanged from v0.2 (edges owned by dsx; project section → stage table; stage-level staleness with cost; `accepted_stale` pins) with one new edge: **stage technical → `## User knowledge` hash** of its functional, so a changed answer makes the technical stale.

## 6. Scenarios

**S1 Business user, full run.** `/ds:spec --functional "estimate inventory for product X"` → 2–3 waves about the decision, horizon, costs → `/ds:gate` → footer "Next: /ds:spec --functional --data (I'll ask what data you have…)" → `/ds:next` repeatedly; each prints the verb it runs and stops at every question or decision.

**S2 Data stage dialogue (business persona).**
```
> /ds:spec --functional --data
Now the data. What do you have, where is it, and what does one row mean?
> data/raw/sales.csv, one line per product per warehouse per day. Promos in promos.xlsx.
Thanks. Three questions:
 1. Are quantities in units or boxes?                                — to compute demand correctly
 2. On a day a product is out of stock, what does the file show: 0, no row, or something else?
 3. Are promo dates known in advance, and how far?                   — decides whether I may use them
> units · I don't know · yes, one quarter ahead
Noted; question 2 I'll answer from the data on first read and confirm with you.
Done: Data functional v1 — demand per SKU×warehouse×week from sales.csv + promos.xlsx (asked 3, deferred 1; panel: data-expert, analyst).
Next: /ds:spec --technical --data (how I'll check it, ≈$0.40) · alt: /ds:next
```

**S3 Change the horizon while modeling.** `/ds:change` → project §2 → stale list with costs → `/ds:next` reruns Data plan 1 (cheap), proposes protocol P v2, asks "rerun champion only ($3) or ladder ($9.5)?". Ship untouched.

**S4 Senior adds an analysis.** `/ds:plan --data "promo weeks explain ≥ 30% of variance in top-20 SKUs"` → `/ds:build` → inventory gains rows → Modeling stale → "use the new columns in features v4?".

**S5 Try another model.** `/ds:plan --modeling "AutoETS within noise of LightGBM at h≤4"` → `/ds:build`.

**S6 Edit the HOW in words.** After the technical card: "actually promotions are planned a quarter ahead, you can use them" → routed via `last_card` to `/ds:spec --technical --data "…"` → v2, diff line shown.

**S7 Come back after two weeks.** `SessionStart` prints board + last card; `/ds:next`.

**S8 New data arrives.** `/ds:sync` → Data plan 1 reruns (cheap) → cascade with cost; the monitoring retrain trigger uses the same path.

## 7. What `/ds:next` does

```
dsx board --json → next = first of:
  stale queue non-empty          → offer reruns (cheap auto, expensive confirm)
  functional draft/missing       → /ds:spec --functional --<stage>   (gap detector: asks the user)
  technical missing/stale        → /ds:spec --technical --<stage>    (panel draft; senior confirms)
  no plan / plan done            → /ds:plan --<stage>                (consult owners; card)
  plan planned                   → /ds:build                         (DAG by owner; reviewer; dod --run; finish)
  build running                  → wait
  DoD met                        → /ds:gate                          (manifest; decision; next skeleton chained)
```
Free steps are batched; `/ds:next` prints the verb before running it.

## 8. Context rebuild after restart or compaction

`dsx board --context` (≤ ~10k tokens): board + stale list + budget + last five decisions + next step + last card → project functional → cursor stage functional + technical → current plan + last build header. Bodies are never loaded; subagents receive paths.

## 9. Invariants enforced in code

- Only dsx writes headers of builds, manifests, `project.yaml`, registry entries.
- The gap detector and any panel call are paid steps: never chained silently.
- Experts run only when a call sheet, a technical contribution list, a plan consult selection, or a task `owner` names them.
- `dsx build finish` is one transaction; `dsx dispatch` refuses training on uncertified features; the evaluator's run id must differ from the developer's.
- Every reviewer-enforced DoD item has `verification: test:<path>`; every model call is routed by `dsx route` and logged to `.dsx/telemetry.jsonl`.
