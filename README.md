# scaffolding-creator

This repo collects example prompts a developer can use to bootstrap better AI-agent workflows in a git repo. The prompts are intended for humans to read, adapt, and paste into the right AI environment.

## Prompts

### `1-initial-prompt.md`

Use this prompt in ChatGPT (browser or desktop). It runs a one-question-at-a-time interview then delivers scaffolding as a zip download link. ChatGPT is the scaffolding factory only — not an ongoing agent for the target repo.

### `2-plan-estimator-tool.md`

Plan a deterministic Python CLI tool (`scripts/estimate_burn.py`) that estimates token burn and recommends the most cost-efficient execution strategy (A/B/C/D) across model tiers. **Run before prompts 4 and 5.** Recommended model: Opus 4.8 or gpt-5.5 high.

### `3-implement-estimator-tool.md`

Implement the approved plan from prompt 2. Builds the CLI tool, tokenizer abstraction (tiktoken / Anthropic API / pluggable), strategy scoring, and telemetry schema. Recommended model: gpt-5.5 medium or Opus 4.7.

### `4-plan-multimodel-skill.md`

Plan a harness-agnostic `multi-model-ai-task` skill that calls the estimator tool to route each task to the right model tier. **Requires prompt 2 and 3 to be complete.** Recommended model: Opus 4.8.

### `5-implement-multimodel-skill.md`

Implement the approved plan from prompt 4. Builds `skills/multi-model-ai-task/SKILL.md`, `reference.md`, harness stubs, and `AGENTS.md` integration. **Requires `scripts/estimate_burn.py` to be present.** Recommended model: Opus 4.7 or gpt-5.5 medium.

## Intended workflow

1. Run `1-initial-prompt.md` in **ChatGPT** to interview and generate repo scaffolding as a zip.
2. Extract the zip into your local git clone and commit.
3. Run `2-plan-estimator-tool.md` in the target repo to plan the estimator tool.
4. After plan approval, run `3-implement-estimator-tool.md` to build it.
5. Run `4-plan-multimodel-skill.md` to plan the multi-model skill (references the tool contract).
6. After plan approval, run `5-implement-multimodel-skill.md` to build the skill.

## Model routing guide

| Prompt | Recommended model | Why |
|---|---|---|
| 1 — scaffold interview | ChatGPT (required) | Free tokens; zip delivery |
| 2 — plan tool | Opus 4.8 / gpt-5.5 high | Algorithm design; one-shot; reused everywhere |
| 3 — implement tool | gpt-5.5 medium / Opus 4.7 | Correctness > open reasoning; plan removes ambiguity |
| 4 — plan skill | Opus 4.8 | Plan quality compounds across every use |
| 5 — implement skill | Opus 4.7 / gpt-5.5 medium | Mechanical given a good plan |

Principle: spend the expensive model where output is reused many times (plans, tool contract). Use the cheaper model for well-specified mechanical passes.
