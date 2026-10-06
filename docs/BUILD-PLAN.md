# Build plan — the `ds` plugin (v0.5 design → working plugin)

**Status:** 2026-10-06. Executable plan. Design sources: `ARCHITECTURE.md` v0.5, `specs/SPECS.md` v2.1, `docs/UX-AND-EXECUTION.md` v0.3, `docs/MODEL-POLICY.md`, `config/policy.yaml`, `docs/EXECUTION-WALKTHROUGH.md`.

## 0. Verified platform facts (Claude Code plugin reference) and what they change

| Fact | Consequence for the build |
|---|---|
| Commands are namespaced `/<plugin-name>:<command>`; the plugin `name` must be kebab-case and must not start with `claude-` or `anthropic-` | Plugin name is **`ds`**, giving `/ds:spec`, `/ds:plan`, `/ds:build`. `ds` passes validation. |
| New plugins should use `skills/<name>/SKILL.md`; `commands/*.md` still work and support `argument-hint` | Each verb is a **skill** (invocable as `/ds:<verb>`); keep `commands/` only if a verb needs a flat file. Decide in the S0 spike. |
| `bin/` files are on the Bash tool's PATH while the plugin is enabled | `dsx` ships as `bin/dsx` (a thin launcher), so skills call plain `dsx …`. |
| `${CLAUDE_PLUGIN_ROOT}` changes on update; `${CLAUDE_PLUGIN_DATA}` persists across updates; neither is exported to Bash-tool commands, only substituted in skill/agent/hook text and exported to hook processes | The engine's virtualenv lives in `${CLAUDE_PLUGIN_DATA}`; `bin/dsx` finds it through a path that skills pass explicitly or a `SessionStart` hook records. Project state never lives under either variable. |
| `agents` must be a list of `.md` files (directories not accepted); components are namespaced `ds:analyst` etc. | Agent files enumerated in `plugin.json`; variants per policy rung generated into `agents/` by `dsx knowledge build`. |
| `hooks/hooks.json` needs the top-level `"hooks"` wrapper; shell-form commands must quote `"${CLAUDE_PLUGIN_ROOT}"` | Hooks call `bin/dsx` through exec form. |
| `claude plugin validate ./ds` and `--plugin-dir` for local testing | CI gate on every milestone. |
| **Not yet verified:** agent frontmatter field names for `model` and `effort`, and whether a skill can pin `argument-hint`/`allowed-tools` | Resolved in the S0 spike before anything depends on them. |

## 1. Ground rules

- **Repo:** `Desktop/ds-expert-agent/` is the plugin root (docs stay in place; code added around them). `git init` at S0.
- **Language and tooling:** Python 3.12, `uv`, `typer`, `pydantic` v2, `ruamel.yaml` (round-trip headers), `pytest`, `ruff`. No network at runtime except model calls made by Claude Code itself.
- **Order of risk:** deterministic engine first (all correctness lives there), then skills and agents on top, then the first full e2e on a toy project with scripted answers. Nothing model-dependent is built before its deterministic scaffolding and fixtures exist.
- **Every milestone ends green:** `pytest`, `ruff`, `claude plugin validate ./ --strict`, and its own acceptance script.
- **Scripted-answers harness from day one:** a `dsx e2e` runner feeds canned user answers to waves and approvals, so skills are testable without a human.

## 2. Milestones

### S0 — Spike and skeleton (2–3 days)
Goal: prove the platform assumptions before committing.
- `git init`; `pyproject.toml` (uv), `ruff`, `pytest`; `.claude-plugin/plugin.json` (`name: ds`, version 0.0.1); empty `skills/ds-spec/SKILL.md`, `agents/analyst.md`, `hooks/hooks.json`, `bin/dsx`.
- Spike checks (each a one-page note in `docs/spikes/`): (1) a skill named `spec` in plugin `ds` is invocable as `/ds:spec` with `$ARGUMENTS` including quoted free text and `--flags`; (2) agent frontmatter accepts `model` and `effort`, and a skill can spawn it via the Agent tool returning JSON; (3) `bin/dsx` resolves on PATH inside a skill's Bash call and can locate its venv; (4) `SessionStart` hook stdout reaches context; `PreToolUse` hook can deny a Write; (5) subagent cannot use AskUserQuestion (confirm); (6) `claude plugin validate` passes.
- **Exit:** all six verified or the design adjusted (record in `docs/spikes/RESULTS.md`). This is the only milestone allowed to change the architecture.

### M0 — Engine core (~2 weeks) — no models, no ML
Package `dsx/` with these modules, each unit-tested first:
1. `workspace.py` — `dsx init`, project layout, `project.yaml` schema (pydantic), atomic writes, snapshot store (`.dsx/snapshots/`), `decisions.log` appender.
2. `specfiles.py` — header read/write for all templates (round-trip, section hashing), validators per artifact (`dsx validate`), templates loaded from `templates/`.
3. `board.py` — `dsx board --json|--context` (cursor, per-stage status, stale list, budget, `last_card`, next step) as a **pure function** of the workspace; `--context` stays ≤ 10k tokens (test with a size budget).
4. `parse.py` — `dsx parse` implementing SPECS/UX §2 exactly: flags, aliases, `text` rule, resolution order, `prereqs`, all `E_*` refusals. **Parser table test** covers every command line in UX §6 and the plan's §H, quoting, `--`, alias echo.
5. `graph.py` — static edge table (project section → stage; deliverable → consumers; technical → upstream deliverables + `user_knowledge_hash`), registry (`specs/deliverables/`), `dsx pin`, `dsx impact` (stage-level stale with cost estimates from past builds).
6. `guard.py` — `dsx guard <path>` rules (append-only `builds/`, finished `experiments/<n>/`, read-only `data/raw/`, `project.yaml` only via dsx) + Bash deny-list.
7. `spec_gen.py` — `dsx spec gen <stage>` skeleton from `specs/SPECS.md` + cited project sections (deterministic, idempotent, hashed).
8. `gaps.py` — `dsx schema select`, `dsx gaps propose` (call sheet), `dsx gaps merge` (slot-key win, dedup, cap 12, persona reclassification), `dsx wave next`.
9. `build.py` — `dsx expect / check-report`, `build start|abandon` (task DAG from the plan table, lock file), `dsx dod [--run]` (executes `[test]` items, hashes evidence), `dsx build finish` (one transaction: reviewer JSON → evidence → verdict → render → board), `dsx gate approve` (manifest, snapshot, cursor, chain next skeleton).
10. `route.py` + `telemetry.py` — `dsx route` reading `config/policy.yaml` (ladders, triggers, persona/family overrides), `dsx cost add --call` → `.dsx/telemetry.jsonl`.
11. `infra/` — `LocalVenvAdapter` (`submit/wait/fetch`, per-project venv, timeout, cwd pinned) behind an `InfraPort` protocol.
12. `knowledge/` — card loader and validator, `conflicts.yaml` check, stage→plugin-stage map, BM25 index, `dsx knowledge build|search` (playbooks, DoD checklists, question schemas), `applied_cards` `{id, hash}`.
- **Fixtures:** `tests/fixtures/toy-inventory/` (2 years daily sales, stock-out column, promo file), `toy-churn/`, `toy-fraud/` generated by a seeded script.
- **Exit (acceptance script):** on `toy-inventory`, a **scripted run with stub experts** (JSON canned per call point) goes Gate 1 → Gate 4 through `dsx` commands only; board, manifests, stale propagation (change §2 horizon → Data and Modeling stale, Ship untouched), `E_*` refusals and guard denials all behave per the docs.

### M1 — Stage 1 dialogue (~1 week)
- Skill `ds-dialogue` (and the `/ds:spec --functional` entry that runs `dsx init`), agent `analyst`, requirements schemas `schemas/forecasting.yaml` and `schemas/tabular.yaml` (stage_1 sections), business gap-detector cards mapped to stage 1, writer card for "problema vigente", `gate approve 1` handoff text with the free Data skeleton.
- No data access anywhere in this stage (assert in tests that no file path is read).
- **Exit:** scripted e2e produces a complete `project.functional.md` with provenance and `pending_data` for inventory and fraud; question count per persona within the documented range; replaying the same answers yields identical files (determinism of everything except wording).

### M2 — Stage 2 Data (~2 weeks)
- Skills `ds-spec` (functional gap detector with panel proposals + waves; technical lead+contributors), `ds-plan` (consult), `ds-build` (DAG by owner); agents `data-expert`, `reviewer`, `writer`; profiling/target/EDA libraries under `src/data/` with tests; inventory builder and schema; split recommendation; `dsx dod --run` wired; fixtures with scripted answers per persona.
- **Exit:** `toy-inventory` Data stage end to end including the pending-data wave, `change_purpose` back-edge when the target is infeasible, Gate 2 manifest with hashes; no-data-yet path (`E_NO_DATA`); `/ds:next` and explicit verbs produce identical files.

### M3 — Stage 3 Modeling (~3 weeks)
- Agents `developer`, `evaluator` (+ policy variants by rung); protocol P deliverable and its freeze; feature-set versioning and **point-in-time certification** (`tests/test_point_in_time.py` template, `dsx dispatch` refusal on uncertified sets); backtest engine implementing P once; benchmark ladder (naive, seasonal naive, AutoETS, LightGBM global; majority, logistic, ridge, GBM for tabular); MLflow wiring; engine-computed noise rule; business translation scripts; **`dsx mutate`** (target shifted, future rolling mean, entity baseline including current row, random-split P, wrong translation sign) and **`dsx shadow`**.
- **Exit:** champion on `toy-inventory` with a business number from an independent evaluator run; mutation suite catches 100% of injected leaks against the generated tests; `toy-churn` passes through the tabular path; Gate 3 manifest.

### M4 — Stage 4 Ship (~1.5 weeks)
- `templates/deploy/` (FastAPI app, batch script, Dockerfile, tests), packaging and monitoring skills/builds, replay test, drift job, rollback script, handoff docs rendered from manifests.
- **Exit:** replay within tolerance, container builds (when Docker is present; otherwise the test is marked and skipped with a recorded reason), model card before `prod`, Gate 4.

### M5 — Hardening and policy evidence (~2 weeks)
- `/ds:change`, `/ds:sync`, `/ds:impact` flows incl. new-data refresh; cost caps and nested budgets; `knowledge` regression tests (fraud asks alert capacity; imbalanced never headlines accuracy; intermittent never uses MAPE); policy evaluation arms (A all-Opus/high, B policy, C one notch up) on the three toys × three personas; `dsx route tune` proposing a `policy.yaml` diff; docs sync; packaging for a marketplace.
- **Exit:** the evaluation report in `docs/POLICY-EVAL.md`; all e2e fixtures green; `claude plugin validate --strict` clean.

Total ≈ 11–13 weeks for one engineer; M0 is the critical path, and M1–M2 give a usable business-understanding-plus-data assistant at about week 5.

## 3. Cross-cutting work packages

| Package | Lands in | Notes |
|---|---|---|
| Header/section hashing library | M0.2 | Used by guard, graph, board; one implementation, property-tested. |
| Stub expert harness (canned JSON per call point, keyed by `call_point` + stage + attempt) | M0 | Lets every skill be tested offline; real agents swap in per milestone. |
| Persona fixtures (business, junior, senior answer scripts) | M1 | Reused in M2–M5. |
| Docs sync | each milestone | Any behavior change updates SPECS/UX in the same PR. |
| CI | S0 | `uv run pytest`, `ruff`, `claude plugin validate --strict`, template/YAML parse, policy schema check. |

## 4. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Plugin platform behaves differently from assumptions (arguments, agent frontmatter, hook output) | S0 spike gates everything; results recorded; only S0 may change the architecture. |
| Skill procedures drift from engine rules | Skills are thin; every rule is a `dsx` command with tests; e2e diffs `/ds:next` against explicit verbs. |
| Silent model errors in oracle call points | Mutation tests (M3) before any downgrade; Opus floors in `policy.yaml`; shadow runs. |
| Panel calls inflate cost | Caps in `policy.yaml` (`max_consulted`, `max_contributors`, cost cap); telemetry from M0; tuning in M5. |
| Business users never reach the end | Scripted-persona e2e measures question counts and approvals per persona; limits tested, not assumed. |
| Docker or MLflow unavailable on a machine | Tests detect and skip with a recorded reason; local file store default. |

## 5. What I need from you, and what I will assume otherwise

Nothing blocks starting S0. Defaults I will use unless you say otherwise:
1. **Repo location:** `Desktop/ds-expert-agent/` as plugin root, `git init` there. Alternative: a separate code repo with the docs as a submodule.
2. **Plugin name `ds`** (so commands read `/ds:spec`). Alternative names change every command.
3. **First target:** `toy-inventory` forecasting, then churn; fraud as the third e2e fixture in M5.
4. **Pace:** one milestone at a time with a green acceptance script before the next, reporting after each.
5. **Models in CI:** none. Real-model runs happen only in the M5 policy evaluation and by hand.
