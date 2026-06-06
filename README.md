# scaffolding-creator

This repo collects example prompts a developer can use to bootstrap better AI-agent workflows in a git repo. The prompts are intended for humans to read, adapt, and paste into the right AI environment.

Each prompt is **independent**: none refers to another. You paste one into a fresh agent started in your target repo and it stands on its own.

## Prompts

### `1-initial-prompt.md`

Use this prompt in ChatGPT (browser or desktop). It runs a one-question-at-a-time interview then delivers repo scaffolding as a zip download link. ChatGPT is the scaffolding factory only — not an ongoing agent for the target repo.

### `2-plan-estimator-tool.md`

Plan (not implement) a deterministic Python CLI tool (`scripts/estimate_burn.py`) that estimates token burn and recommends the most cost-efficient execution strategy (A/B/C/D/E) across model tiers. This tool's plan is the canonical source for the strategy set and JSON contract that the `multi-model-ai-task` skill (prompt 4) later consumes. Built early. Recommended model: Opus 4.8 or gpt-5.5 high.

### `3-plan-work-item-tool.md`

Plan (not implement) a deterministic Python tool under `tools/` that allocates the next available work-item number (`W-0001` style) by scanning `plans/` and `archive/`, handling concurrent agents safely. Built early; consumed by agents (and the prompt 4 skill) when they create a new work-item plan. Recommended model: Opus 4.8 or gpt-5.5 high.

### `4-plan-multimodel-skill.md`

Plan (not implement) a harness-agnostic `multi-model-ai-task` skill that calls the estimator tool to route each task to the right model tier. **Planned and built last** — it relies on the tools from prompts 2 and 3 already existing in the target repo. Recommended model: Opus 4.8.

## Intended workflow

1. Run `1-initial-prompt.md` in **ChatGPT** to interview and generate repo scaffolding as a zip.
2. Extract the zip into your local git clone and commit.
3. In the target repo, run `2-plan-estimator-tool.md` and `3-plan-work-item-tool.md` to plan the supporting tools. Implement and approve each before moving on.
4. Run `4-plan-multimodel-skill.md` **last** to plan the multi-model skill, which consumes the now-existing tools.

Prompts 2, 3, and 4 are planning prompts: each produces an approved plan that an implementation agent then builds in a separate pass.

## Model routing guide

| Prompt | Recommended model | Why |
|---|---|---|
| 1 — scaffold interview | ChatGPT (required) | Free tokens; zip delivery |
| 2 — plan estimator tool | Opus 4.8 / gpt-5.5 high | Algorithm + contract design; reused everywhere |
| 3 — plan work-item tool | Opus 4.8 / gpt-5.5 high | Concurrency design; correctness matters |
| 4 — plan skill | Opus 4.8 | Plan quality compounds across every use |

Principle: spend the expensive model where output is reused many times (plans, tool contracts). Use a cheaper model for well-specified mechanical implementation passes.
