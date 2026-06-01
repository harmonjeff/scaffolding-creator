# scaffolding-creator

This repo collects example prompts a developer can use to bootstrap better AI-agent workflows in a git repo. The prompts are intended for humans to read, adapt, and paste into the right AI environment.

## Prompts

### `1-initial-prompt.md`

Use this prompt in ChatGPT, either in the browser or desktop app. It is designed to take advantage of the "free" tokens available in ordinary ChatGPT conversations while thinking through repo scaffolding before using a local coding agent.

### `2-plan-multi-model-skill-prompt.md`

Use this prompt in an agent harness that can read and write files in the target repo. It plans a harness-agnostic `multi-model-ai-task` skill for coordinating planning, orchestration, and implementation agents.

### `3-implement-multi-model-skill-prompt.md`

Use this prompt in an agent harness that can read and write files in the target repo. It implements an approved plan for the `multi-model-ai-task` skill, including the canonical skill files, optional templates, and harness integration stubs.

## Intended workflow

1. Start with `1-initial-prompt.md` in ChatGPT to reason about the desired repo scaffolding.
2. Run `2-plan-multi-model-skill-prompt.md` in a local file-system-aware agent harness to create a reviewable implementation plan in the target repo.
3. After approving the plan, run `3-implement-multi-model-skill-prompt.md` in a local file-system-aware agent harness to create the skill and related files.
