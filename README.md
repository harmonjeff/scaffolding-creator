# scaffolding-creator

This repo collects example prompts a developer can use to bootstrap better AI-agent workflows in a git repo. The prompts are intended for humans to read, adapt, and paste into the right AI environment.

Each prompt is **independent**: none refers to another. You paste one into a fresh agent started in your target repo and it stands on its own.

## Prompts

### `1-initial-prompt.md`

Use this prompt in ChatGPT (browser or desktop). It runs a one-question-at-a-time interview then delivers repo scaffolding as a zip download link. ChatGPT is the scaffolding factory only — not an ongoing agent for the target repo.

### `2-plan-work-item-tool.md`

Plan (not implement) a deterministic Python tool under `tools/` that allocates the next available work-item number (`W-0001` style) by scanning `plans/` and `archive/`, handling concurrent agents safely. Built early; consumed by agents when they create a new work-item plan. Recommended model: Opus 4.8 or gpt-5.5 high.

### `3-plan-multi-model-skill.md`

Plan (not implement) a harness-agnostic `multi-model-ai-task` skill that routes each implementation task to the right model using complexity-based routing. **Planned and built last** — it relies on the work-item tool from prompt 2 already existing in the target repo. Recommended model: Opus 4.8.

## Intended workflow

1. Run `1-initial-prompt.md` in **ChatGPT** to interview and generate repo scaffolding as a zip.
2. Extract the zip into your local git clone and commit.
3. In the target repo, run `2-plan-work-item-tool.md` to plan the work-item allocator tool. Implement and approve it before moving on.
4. Run `3-plan-multi-model-skill.md` **last** to plan the multi-model skill, which uses the now-existing work-item tool.

Prompts 2 and 3 are planning prompts: each produces an approved plan that an implementation agent then builds in a separate pass.

## Model routing guide

| Prompt | Recommended model | Why |
|---|---|---|
| 1 — scaffold interview | ChatGPT (required) | Free tokens; zip delivery |
| 2 — plan work-item tool | Opus 4.8 / gpt-5.5 high | Concurrency design; correctness matters |
| 3 — plan skill | Opus 4.8 | Plan quality compounds across every use |

Principle: spend the expensive model where output is reused many times (plans, tool contracts). Use a cheaper model for well-specified mechanical implementation passes.
