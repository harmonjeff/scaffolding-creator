# Plan: Updating the scaffolding-creator prompts

Status: draft — 2026-06-05

## Decisions locked
- **Remove `CHEATSHEET.md`** from prompt 1's output (redundant with `AGENTS.md` + `STANDARDS_REGISTRY.md`).
- **Renumber to 5 files, tool-first, split plan/implement** so the estimator tool's contract exists before the skill references it, and each phase can be routed to a different model.

## Target file structure & migration

| New file | Source | Role | Run in |
|---|---|---|---|
| `1-initial-prompt.md` | unchanged file, edited | Repo scaffolding interview | **ChatGPT only** (hard rule 1) |
| `2-plan-estimator-tool.md` | split from old `4-…tool.md` (architecture half) | Plan the deterministic token-burn CLI tool | Opus 4.8 / gpt-5.5 high |
| `3-implement-estimator-tool.md` | split from old `4-…tool.md` (implementation half) | Build the CLI tool | gpt-5.5 med / Opus 4.7 |
| `4-plan-multimodel-skill.md` | old `2-plan-multi-model-skill-prompt.md` | Plan the `multi-model-ai-task` skill (references tool contract) | Opus 4.8 |
| `5-implement-multimodel-skill.md` | old `3-implement-multi-model-skill-prompt.md` | Build the skill, wiring in the tool | Opus 4.7 / gpt-5.5 med |

Old `4-plan-implement-token-burn-estimator-tool.md` is consumed by the split into new 2+3.
Update `README.md`'s "Prompts" and "Intended workflow" sections to the new sequence + per-file model recommendation.

---

## Canonical shared contracts (must be identical across files 2–5)

These are the single sources of truth that the tool defines and the skill consumes. Define once, cite verbatim.

### A. Tool invocation
- CLI: `scripts/estimate_burn.py` (Python, no LLM calls), invokable as `python scripts/estimate_burn.py …` or a console entry `estimate-burn`.
- Inputs: task/plan path, scaffolding paths, code-context paths, candidate model tiers, optional price/latency config.
- **Output: JSON to stdout** (machine-parseable — this is what makes the skill deterministic).

### B. JSON output schema (the "report")
Pin these fields so the skill can parse them:
```
{
  "recommended_strategy": "A|B|C|D",
  "confidence_score": 0.0-1.0,
  "estimated_savings_tokens": int,
  "per_strategy": {
    "A": { "token_burn": int, "cost": float, "latency_s": float,
           "success_probability": 0.0-1.0 }, ...
  },
  "inputs_echo": { ... },
  "tokenizer": "tiktoken|anthropic|local",
  "version": "..."
}
```
Optional human-readable report is a *render* of this JSON, not the contract.

### C. Model-tier taxonomy (unify the two vocabularies)
Prompt 4 uses frontier/mid-tier/local; prompts 2/3 use Orchestration/Implementation. Canonical:
- **frontier** — planning, orchestration, review (expensive, high-thinking).
- **mid** — cheaper cloud implementation model.
- **local** — local implementation model.
Skill's "Orchestration Agent" = frontier; "Implementation Agent" = mid | local | frontier-in-thread.

### D. Strategy ↔ routing map (A/B/C/D = the skill's decision tree)
| Strategy | Plan | Implement | Review | Skill routing path |
|---|---|---|---|---|
| A | frontier | frontier | frontier | Orchestration implements in-thread |
| B | frontier | cheap/mid (fresh) | frontier | Handoff to fresh Implementation Agent |
| C | frontier | mid cloud | frontier | Handoff to mid-tier Implementation Agent |
| D | frontier | local | frontier | Handoff to local Implementation Agent |

### E. Division of labor (resolves problems 4 & 5)
- **Tool (deterministic):** token burn, cost, latency, success prob, recommended strategy, savings.
- **Operator (live, unreadable by tool):** current quota % remaining, context-window %.
- **Skill routing = tool recommendation adjusted by live operator metrics.**
- v1 caveat: `success_probability`/`confidence_score` are **heuristic priors** until the telemetry/learning system has data; skill treats them as advisory.

---

## File 1 — `1-initial-prompt.md` (ChatGPT)

### Problem 1 — always dumps 8 questions at once
Root cause: lines 136–163 hand the model a literal `Discovery Questions — Batch 1` block listing Q1–Q8; the model transcribes it. The "ask one at a time" instruction sits *inside* the copied block.
- Delete the multi-question "First Response Format" block (136–163). Replace with an executed protocol: "Output exactly **one** question per turn; after my answer, restate captured decisions + open TODOs in ≤3 lines, run the Confidence Gate, then ask the next single question."
- First literal turn = single question (project name).
- Resolve the batch contradiction (line 116): default one-at-a-time; batches only if user types `batch questions`. Remove "small batches of 5–8" unless opted in.
- Sweep "after each batch" → "after each answer" (124, 167, 755).
- Remove "Batch 1" framing everywhere.

### Problem 2 — completed plans not fully moved to archive/
Root cause: ambiguous "Completed **plans/details** move to archive/" (68, 587) lets the model leave the plan in `plans/` and only move output.
- State invariant: **`plans/` holds only draft|approved|in-progress; a `complete` plan must not remain in `plans/`.**
- Define completion as one atomic move: append concise execution summary (what changed, paths, verification) to the plan file, then **move the whole file** to `archive/YYYY-MM-DD-W-0001-slug.md`, delete from `plans/`, remove the `WORK_ITEMS` row. No separate "execution output" location — the moved plan is the record.
- Reinforce read-ban: agents may write archive/ to deposit, never read without approval.
- Edits: tighten 67–71, 559–560, 586–591; strengthen `S-ARCHIVE-001` (485), `S-WORK-001` (484), `S-DECS-001` (486).

### Problem 3 — remove CHEATSHEET.md
- Delete section 11 (650–666), file-tree row (892), reference (729).
- Fold any must-have one-liners into `AGENTS.md` (has headroom under 80 lines).

### Opportunity — existing-repo intake
Prompt 1 assumes new repos and repeatedly stresses "ChatGPT can't see your files," but the goal includes existing repos.
- Add an early interview branch: if existing repo, ask the user to paste the file tree + key stack facts so scaffolding doesn't fight/duplicate what exists.

### Opportunity — trim repetition
- Zip/dot-folder delivery rules restated ~3×; consolidate to reduce mid-generation drift in a browser context window.

---

## Files 2 & 3 — estimator tool (split from old prompt 4)

- **Scope to the TOOL ONLY.** Strip "design a reusable AI Agent Skill" (old line 3, deliverable 2) — the consuming skill is `multi-model-ai-task` (files 4/5). Avoids two competing skills.
- **File 2 (plan):** architecture, repo layout, data model, scoring algorithm, strategy set A/B/C/D, telemetry/learning design — stop at approval gate. Add the **named CLI** and **pinned JSON schema** (contracts A/B above).
- **File 3 (implement):** build `scripts/estimate_burn.py` after plan approval — tiktoken / Anthropic token-counting / pluggable tokenizers; emits the JSON schema; no LLM calls.
- Keep prompt 4's constraints (CLI only, Python, no JS/TS, deps justified, deterministic over LLM).

---

## Files 4 & 5 — multi-model skill (renumbered old 2 & 3)

### Problem 4 — metrics/estimator not consistently used
Root cause: soft language ("if stale", qualitative low/med/high). The tool now supplies deterministic numbers.
- Make the estimator a **required step** per task: (a) invoke `scripts/estimate_burn.py`, (b) ask operator for live metrics, (c) apply routing rule, (d) record numeric estimate + decision in the plan.
- Define hard gate points: orchestration kickoff, before **any** handoff, and when an estimate crosses a threshold. Replace "if stale" with these named triggers.
- Pin "key model usage metrics" to a concrete askable list: context-window %, quota % remaining, implementation-agent context %.
- Replace qualitative low/med/high with tool output + live metrics.

### Problem 5 — tool not wired into skill
- File 4 (plan): prerequisite line — "the estimator tool contract in files 2/3 is required; reference its CLI and JSON schema; do not invent your own."
- File 5 (implement): `SKILL.md` must invoke the tool and parse its JSON deterministically, with a worked example.
- Align skill routing decision tree to strategy map (contract D); unify model tiers (contract C).
- Update acceptance criteria: "estimator was invoked and its JSON output recorded in the plan."

---

## Model routing (hard rule 2)

| File | Work | Model | Why |
|---|---|---|---|
| 2 plan-tool | high-thinking, one-shot | Opus 4.8 / gpt-5.5 high | algorithm design reused everywhere |
| 3 impl-tool | deterministic code, well-specified | gpt-5.5 med / Opus 4.7 | correctness > open reasoning |
| 4 plan-skill | high-thinking, one-shot | Opus 4.8 | plan quality compounds |
| 5 impl-skill | mechanical given a good plan | Opus 4.7 / gpt-5.5 med | ambiguity removed by plan |

Principle: spend the expensive model where output is reused many times (plans, tool contract); cheaper model for well-specified mechanical passes. The split-file structure is what lets you route per phase.

---

## Execution order
1. Edit `1-initial-prompt.md` (problems 1/2/3 + existing-repo intake + trim).
2. Create `2-plan-estimator-tool.md` and `3-implement-estimator-tool.md` from old prompt 4; add CLI name + JSON schema; strip the skill deliverable.
3. `git mv` old 2→`4-plan-multimodel-skill.md`, old 3→`5-implement-multimodel-skill.md`; rewrite for tool wiring + hard metric gates + taxonomy/strategy alignment.
4. Delete old `4-plan-implement-token-burn-estimator-tool.md`.
5. Update `README.md` to new sequence + model table.

## Open items / TODOs
- Finalize CLI name (`scripts/estimate_burn.py` vs console entry `estimate-burn`).
- Confirm v1 cost/latency config source (static price table vs operator-supplied).
- Decide whether the optional human-readable report ships in v1 or telemetry-later.
