You are planning the implementation of a Python tool in this repository. Do not implement anything yet.

**Design principle:** Design this tool as a standalone, scriptable command with a stable invocation and output contract so any agent or future workflow can consume it **without changes**.

**Prerequisite — expected repo scaffolding:** This prompt assumes the repo already contains standard AI-agent scaffolding, specifically: `plans/` (active work item plans), `archive/` (completed plans and superseded decisions), `AGENTS.md` (cross-agent entrypoint), `docs/STANDARDS_REGISTRY.md`, `docs/REPO_MAP.md`, and `docs/WORK_ITEMS.md`. If these are not present, scaffold the repo before running this prompt.

Goal:
Create a plan for adding a deterministic, scriptable Python tool under `tools/`. The tool will be used by AI coding agents while they are generating a plan for a new work item.

The tool’s purpose is to allocate the next available work item number in the format:

W-0001

The tool must determine the next number by inspecting:

- active work items in the `plans/` folder
- previously used work item numbers in the `archive/` folder

The tool must account for multiple agents operating in parallel and asking for a work item number at the same time. The implementation plan must address race conditions and propose a safe allocation strategy.

The allocation rule is:

- Always reserve `max(existing_work_item_number) + 1`.
- Do not reuse gaps.
- For example, if `W-0001`, `W-0002`, and `W-0004` exist, the next allocated work item number must be `W-0005`, not `W-0003`.
- This rule applies across both `plans/` and `archive/`.

The tool must be usable by AI coding agents operating in this repo through:

- Claude Code CLI
- Codex CLI
- GitHub Copilot CLI
- Cursor IDE

Before proposing changes:
1. Inspect the repository structure.
2. Identify existing conventions for:
   - Python scripts
   - `tools/`
   - `plans/`
   - `archive/`
   - work item naming
   - tests
   - README files
   - AGENTS.md / CLAUDE.md / other agent scaffolding
   - command-line invocation patterns
3. Preserve existing repo conventions wherever possible.

Hard constraints:
- The tool must be written in Python.
- The tool must live under `tools/`.
- The tool must be invokable from the repo root.
- Prefer simple invocation such as:
  - `python tools/<tool_name>.py ...`
  - or `python -m tools.<tool_name> ...`
- Do not add dependencies unless clearly justified.
- Do not add a web UI.
- Do not add a service, daemon, API server, database, JavaScript, TypeScript, or other non-Python technology unless explicitly approved.
- Do not implement the tool until the plan is approved.
- Do not modify files during the planning phase.
- Do not dump large file contents or large diffs into the conversation unless needed for the plan.

Functional requirements to plan:
1. The tool must scan `plans/` for active work items.
2. The tool must scan `archive/` for previously used work item numbers.
3. The tool must recognize work item numbers matching the format `W-0001`, `W-0002`, etc.
4. The tool must determine the next work item number using `max(existing_work_item_number) + 1`.
5. The tool must not reuse gaps in numbering.
6. The tool must prevent duplicate allocation when multiple agents run the tool concurrently.
7. The tool must produce output that is easy for agents to consume.
8. The tool should fail clearly if required folders are missing or if the repo is not in an expected state.
9. The tool should avoid relying on model memory or agent judgment for numbering.

Concurrency requirements to plan:
- Explain the race condition that exists if agents only scan existing files and then independently choose the next number.
- Propose a safe reservation/allocation mechanism.
- Prefer a repo-local mechanism that works from the command line.
- Consider whether the tool should create a reservation file, lock file, placeholder work item file, or other durable artifact.
- Explain how stale reservations should be handled, if applicable.
- Explain what should happen if a lock cannot be acquired.
- Explain how the approach works when multiple local agents operate in the same git working tree.
- Call out limitations if multiple agents operate in different clones of the repository.

The plan must include:

1. Repository findings
   - Relevant existing files and conventions.
   - Existing patterns in `plans/` and `archive/`.
   - Any existing work item number format or naming convention.
   - Any existing agent scaffolding that should be updated.
   - Any existing test conventions.

2. Proposed file changes
   - New files.
   - Modified files.
   - Purpose of each file.
   - Why each change is needed.

3. Tool interface design
   - Proposed command name/path.
   - CLI arguments.
   - Default behavior.
   - Whether the tool only prints the next number or also reserves it.
   - Input assumptions.
   - Output format.
   - Exit codes.
   - Error message style.
   - Examples of successful and failed invocations.

4. Number allocation design
   - How the tool scans `plans/`.
   - How the tool scans `archive/`.
   - Regex or parsing strategy for `W-0001` style identifiers.
   - The rule that the tool must always allocate `max(existing_work_item_number) + 1`.
   - Why gaps must not be reused.
   - How malformed work item references are handled.
   - How duplicate existing work item numbers are detected and reported.

5. Concurrency design
   - How duplicate allocation is prevented.
   - What lock/reservation mechanism is recommended.
   - What file or directory is used for locking/reservation.
   - Whether the approach is atomic on macOS/Linux filesystems.
   - How the tool behaves when another agent is allocating a number.
   - How stale locks or reservations are detected.
   - Limitations of the approach across separate git clones.

6. Agent accessibility design
   Explain how the tool will be discoverable and usable by:
   - Claude Code CLI
   - Codex CLI
   - GitHub Copilot CLI
   - Cursor IDE

   Include where usage instructions should be documented so these agents can find them without relying on vendor-specific behavior only.

7. Documentation/scaffolding plan
   - Where to document the tool.
   - What to add to AGENTS.md, CLAUDE.md, README.md, or tool-specific markdown files if they exist.
   - The exact agent-facing rule that should tell agents to use this tool before creating a new work item plan.
   - How to keep the guidance concise enough to avoid unnecessary token burn.

8. Validation strategy
   - Unit tests, smoke tests, or deterministic examples.
   - How to test basic number allocation.
   - How to test scanning `plans/`.
   - How to test scanning `archive/`.
   - How to test duplicate detection.
   - How to test malformed identifiers.
   - How to test gap behavior, confirming gaps are not reused.
   - How to test concurrent invocations.
   - How to run validation from the repo root.
   - Expected outputs.
   - Failure cases to test.

9. Risk and assumptions
   - Assumptions you made.
   - Open questions.
   - Any tradeoffs.
   - Any places where approval is needed before proceeding.

Output format:
Produce a clear implementation plan with headings. End with a short approval gate that asks whether to proceed with implementation.

Do not write code.
Do not modify files.
Stop after the plan.