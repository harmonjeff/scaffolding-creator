# Implementation Plan: Update `1-initial-prompt.md` from research recommendations

**Source of truth:** `research/harness-scaffolding-findings.md` (recommendations R1–R22).
**Target file:** `1-initial-prompt.md`.
**Scope approved:** All 22 edits (P1 + P2 + P3). Long-running harness features are **interview-gated, OFF by default**.

## Hard constraints — do NOT violate (preserve these)

1. ChatGPT zip-delivery workflow stays intact.
2. ChatGPT has no local filesystem / git access — never add steps that imply it does.
3. Markdown-first default for generated scaffold.
4. Generated scaffold stays concise.
5. No implementation source files unless the interview explicitly approves them.
6. Line caps (`AGENTS.md` 80 / router 120 / standards 200) remain as **house targets**, not deletions.

Every new capability (nested AGENTS.md, scoped files, long-running state, JSON state, sync scripts, evaluator loops, mechanical checks, skills) MUST be **opt-in / interview-gated / approval-gated** and OFF by default. Do not change the default generation behavior.

## How to apply

Each edit below gives an exact `OLD` (existing text in the file) and `NEW` (replacement) or an `INSERT AFTER` anchor. Use the Edit tool with the literal strings. Strings are unique unless noted. Apply in order. Do not reflow or rewrite unrelated text. Do not echo the full file back in chat.

---

## P1 EDITS

### E1 (R1) — Claude overlay imports `@AGENTS.md`

**Edit 1a — defaults bullet.**
OLD:
```
- Harness-specific files (`CLAUDE.md`, `.cursor/rules/repo.mdc`, `.github/copilot-instructions.md`) are short shims that point back to `AGENTS.md`.
```
NEW:
```
- Harness-specific files are short adapters anchored to `AGENTS.md`, not copies of it. `CLAUDE.md` must import the canonical instructions with `@AGENTS.md` (Claude reads `CLAUDE.md`, not `AGENTS.md`, automatically). `.cursor/rules/repo.mdc` and `.github/copilot-instructions.md` are short pointers/summaries that must not contradict `AGENTS.md`.
```

**Edit 1b — CLAUDE.md section requirements.** In section `3. CLAUDE.md`.
OLD:
```
Requirements:

- Point back to AGENTS.md as canonical.
- State precedence explicitly: AGENTS.md is canonical; if anything here conflicts with AGENTS.md, AGENTS.md wins. This file only adds Claude-specific notes.
- Include only Claude-specific workflow notes, context-loading guidance, and reminders.
- Do not duplicate AGENTS.md.
- Emphasize reading the task router before loading extra files.
- Emphasize avoiding unnecessary context loading.
- Keep this file as a short harness shim only: point to `AGENTS.md`, state that `AGENTS.md` is canonical and wins on conflict, remind agents to use `docs/AGENT_TASK_ROUTER.md`, and avoid duplicating repo rules.
```
NEW:
```
Requirements:

- The file must begin with an `@AGENTS.md` import on its own line so Claude loads the canonical repo instructions at session start. A plain prose "read AGENTS.md" pointer is not sufficient because Claude may not auto-load `AGENTS.md`.
- If symlinks are acceptable and no Claude-specific content is needed, a symlink to `AGENTS.md` is an acceptable alternative; otherwise prefer the `@AGENTS.md` import.
- State that `AGENTS.md` is the canonical source of repo policy; content below the import is additive Claude-specific notes only and must not contradict it.
- Include only Claude-specific workflow notes, context-loading guidance, and reminders below the import.
- Do not duplicate AGENTS.md.
- Emphasize reading the task router (`docs/AGENT_TASK_ROUTER.md`) before loading extra files, and avoiding unnecessary context loading.
- Keep this file short: import line, then only Claude-specific notes.
```

### E2 (R2) — Copilot overlay drops false precedence claim

In section `5. .github/copilot-instructions.md`.
OLD:
```
Requirements:

- Point back to AGENTS.md as canonical.
- Keep it short.
- Focus on Copilot behavior.
- Include guidance to preserve existing patterns.
- Include guidance not to invent dependencies, APIs, commands, or file paths.
- Keep this file as a short harness shim only: point to `AGENTS.md`, state that `AGENTS.md` is canonical and wins on conflict, remind agents to use `docs/AGENT_TASK_ROUTER.md`, and avoid duplicating repo rules.
```
NEW:
```
Requirements:

- Prefer `AGENTS.md` as the cross-agent source where Copilot/VS Code support is sufficient; generate this file only for Copilot-specific, broadly applicable guidance or compatibility.
- State that repo policy is centralized in `AGENTS.md` and this file must not contradict it. Do NOT claim tool-level precedence (e.g. "AGENTS.md wins on conflict"); Copilot/GitHub precedence can place `.github/copilot-instructions.md` ahead of agent instructions, so that claim would be false.
- Keep it short and self-contained (Copilot custom instructions are sent with every chat message). If it must affect Copilot code review, keep the load-bearing content within the first 4,000 characters.
- Focus on Copilot behavior; remind agents to use `docs/AGENT_TASK_ROUTER.md`.
- Include guidance to preserve existing patterns and not invent dependencies, APIs, commands, or file paths.
- Do not duplicate repo rules already in `AGENTS.md`.
```

### E3 (R3) — Optional nested `AGENTS.md` for monorepos (known boundaries only)

In section `Files To Generate`, after the overlay bullet list. 
INSERT AFTER:
```
- Reflect the included overlays in the file tree and manifest; omit the others.
```
INSERT (new block):
```

Nested `AGENTS.md` (optional, monorepos only):

- If the repo has known subprojects/packages with materially different commands, stacks, or safety rules, offer nested `AGENTS.md` files at those subproject roots.
- Generate nested files only when the subproject boundaries and facts are actually known from the interview. Otherwise mark as a TODO/optional in `docs/REPO_MAP.md`; do not invent paths.
- Root `AGENTS.md` remains the global router. Nested files contain only local overrides and must not repeat root rules.
```

### E4 (R4) — Optional native scoped instruction files

In section `10. docs/AGENT_TASK_ROUTER.md`, after the `Additional router requirements:` list (after the final stop/ask sub-bullets) and before the `---` that ends the section.
INSERT AFTER:
```
  - committing or creating PRs
```
INSERT (new block):
```

Optional scoped instruction files (off by default):

- Generate native scoped instruction files only when a named harness supports them, the user wants that harness optimized, and the scopes are known. Otherwise omit.
- `.github/instructions/<scope>.instructions.md` with `applyTo` (and optional `excludeAgent`) for Copilot/VS Code.
- `.claude/rules/<scope>.md` with `paths` for Claude.
- `.cursor/rules/<scope>.mdc` with `globs` or `alwaysApply: false` for Cursor.
- Root `AGENTS.md` remains the cross-agent router; scoped files only narrow rules to known paths.
```

### E5 (R14) — Interview gate for long-running autonomous harness profile

**Edit 5a — Confidence Gate REQUIRED items.** In section `Confidence Gate`.
OLD:
```
- project-specific security, testing, documentation, and architecture constraints when available
```
NEW:
```
- project-specific security, testing, documentation, and architecture constraints when available
- whether the scaffold targets long-running autonomous implementation loops or only ordinary assisted coding (default: ordinary assisted coding)
```

**Edit 5b — Appendix topic.** In section `4. Agent Behavior` of the Appendix, append to its `Capture:` list.
OLD:
```
- whether agents should provide acceptance criteria before implementation
```
NEW:
```
- whether agents should provide acceptance criteria before implementation
- whether the repo needs a long-running autonomous-agent profile; if yes, whether to include approved non-markdown harness state such as `feature_list.json`, `progress.md` (or `docs/PROGRESS.md`), and an `init.sh`/setup-script placeholder. Default: omit all of these unless explicitly approved. These conflict with the markdown-first default, so they are opt-in only.
```

### E6 (R15) — Long-running session lifecycle standard (opt-in)

In section `6. docs/STANDARDS_REGISTRY.md`, in the "Also seed these non-project-specific standards unless overridden" list, after the `S-GIT-001` bullet.
OLD:
```
- S-GIT-001 — Agents must not commit, create PRs, or perform git actions without human approval.
```
NEW:
```
- S-GIT-001 — Agents must not commit, create PRs, or perform git actions without human approval.
- S-SESSION-001 (seed ONLY when the long-running autonomous profile is enabled) — Each session: orient by reading active progress + task/feature state + recent git history; run setup/init; verify the existing baseline before new work; choose one task/feature; implement; verify through the relevant UI/API/tests; update state; leave a clean exit summary. Active progress state lives outside `archive/` (the archive-read restriction in S-ARCHIVE-001 still holds).
```

### E7 (R16) — Constrained JSON feature/test state (long-running mode only)

In section `9. docs/WORK_ITEMS.md`, in `Requirements:`, after the first bullet.
OLD:
```
- Start empty unless project work items were provided.
```
NEW:
```
- Start empty unless project work items were provided.
- Markdown remains the default tracker. Only when the long-running autonomous profile is enabled may feature-completion state that agents update repeatedly use a constrained JSON file (e.g. `feature_list.json` or `tests.json`) where agents may change only status/pass-fail fields unless explicitly approved. Narrative plans and progress stay in markdown.
```

### E8 (R21) — Optional generated harness-adapter sync policy (documented, not default file)

In section `Files To Generate`, immediately after the nested-`AGENTS.md` block added in E3.
INSERT AFTER (the E3 block ending):
```
- Root `AGENTS.md` remains the global router. Nested files contain only local overrides and must not repeat root rules.
```
INSERT (new block):
```

Harness-adapter sync (documented policy; generate scripts only if approved):

- Root `AGENTS.md` is canonical. Do not blind-copy its full contents into every harness-specific file (risks stale, conflicting, or oversized adapters).
- If multiple harness-specific files are generated and the user approves executable tooling, optionally create `scripts/sync-agent-instructions.sh` (or the repo's native task runner equivalent) that renders tool-specific adapters from `AGENTS.md` and fails when adapters drift. Adapters preserve native shape: `CLAUDE.md` starts with `@AGENTS.md`; the Copilot adapter stays concise and within its limits; the Cursor adapter keeps MDC frontmatter.
- Do not generate any shell/script file unless the interview approves executable tooling. By default, describe this policy in docs only.
```

---

## P2 EDITS

### E9 (R5) — Cursor overlay only when Cursor-specific behavior is needed

In section `4. .cursor/rules/repo.mdc`, in `Requirements:`. 
OLD:
```
- Default for initial scaffolding: `alwaysApply: true` unless interview answers specify path-scoped rules.
```
NEW:
```
- Default for initial scaffolding: `alwaysApply: true` unless interview answers specify path-scoped rules.
- Prefer root `AGENTS.md` for simple cross-agent instructions (Cursor supports `AGENTS.md` as a simple alternative to `.cursor/rules`). Generate `.cursor/rules/*.mdc` only when the user wants Cursor-specific behavior, path scoping, or reusable Cursor workflows.
- Never generate `.cursorrules`; it is legacy/deprecated. Mention it only as a migration note if relevant.
```

### E10 (R6) — Relax the absolute README read-ban

**Edit 10a — defaults bullet.**
OLD:
```
- `README.md` is human-facing only. Agents must not read it unless the task is to update human-facing documentation.
```
NEW:
```
- `README.md` is human-facing and is not part of the default agent read order; agents start at `AGENTS.md` and routed docs. Agents may read `README.md` when the task is to update human-facing docs, when a routed doc explicitly points there, or when setup/usage facts are missing from agent docs and `README.md` is the known canonical source.
```

**Edit 10b — S-READ-001 standard wording.** In section `6. docs/STANDARDS_REGISTRY.md`.
OLD:
```
- S-READ-001 — Agents read `AGENTS.md`, then routed docs only. Do not read `README.md` unless updating human docs. Do not read `archive/` without explicit approval.
```
NEW:
```
- S-READ-001 — Agents read `AGENTS.md`, then routed docs. `README.md` is not in the default read order; read it when updating human docs, when a routed doc points there, or when canonical setup facts are missing from agent docs. Do not read `archive/` without explicit approval.
```

### E11 (R7) — Reusable-skills interview topic

In the Appendix, in section `6. Documentation and Token Controls`, append to its `Only ask about:` list.
OLD:
```
- whether project-specific generated outputs should be ignored, tracked, or human-owned
```
NEW:
```
- whether project-specific generated outputs should be ignored, tracked, or human-owned
- whether the repo should include repo-scoped reusable skills (`.agents/skills/<name>/SKILL.md`) or harness-specific commands/rules for repeatable workflows (release, security review, performance review, evals). Default: do not generate skills unless explicitly requested.
```

### E12 (R8) — Vendor-specific size caps alongside house line caps

In `### Default doc size targets`.
OLD:
```
- Agents must ask permission before exceeding a size target and briefly explain why expansion is needed.
```
NEW:
```
- Agents must ask permission before exceeding a size target and briefly explain why expansion is needed.
- These line caps are house targets, not ecosystem standards. Where a tool enforces its own limit, respect it too: keep Copilot custom-instruction files within the first ~4,000 characters when they must affect Copilot code review; target `CLAUDE.md` under ~200 lines when it has substantive content; keep Codex combined project instructions under the ~32 KiB default unless deliberately reconfigured.
```

### E13 (R9) — Conflict/staleness review across instruction files

In section `Global Requirements`, append a bullet.
OLD:
```
- Use markdown only unless a requested file format requires otherwise.
```
NEW:
```
- Use markdown only unless a requested file format requires otherwise.
- Avoid conflicting rules across root, nested, scoped, user, and harness-specific instruction files. When updating any harness doc, review adjacent instruction files and remove or narrow stale or contradictory guidance (contradictory rules may be applied arbitrarily by agents).
```

### E14 (R10) — "Markdown guides, settings enforce" distinction

In section `Agent Safety Requirements`, after the `Dependency and command defaults:` list's final bullet.
OLD:
```
- Commands that modify files, create many files, create large outputs, or are destructive require approval unless already covered by an approved plan.
```
NEW:
```
- Commands that modify files, create many files, create large outputs, or are destructive require approval unless already covered by an approved plan.

Enforcement vs guidance:

- Markdown agent docs are behavioral guidance, not hard enforcement. For hard command/file/tool restrictions, use the harness's enforced settings, permissions, hooks, or rules where available, and document those surfaces separately from `AGENTS.md`.
```

### E15 (R11) — Post-extract instruction-load verification (human checklist only)

In section `Human Checklist (output after generation)`, after step 7.
OLD:
```
7. If I use Cursor, confirm `.cursor/rules/repo.mdc` is present and applies. Skip overlay checks for agents I don't use.
8. Use my ongoing coding agents for repo work; entrypoint is `AGENTS.md`.
```
NEW:
```
7. If I use Cursor, confirm `.cursor/rules/repo.mdc` is present and applies. Skip overlay checks for agents I don't use.
8. Verify each agent actually loads its instructions: for Codex, ask it to list loaded instruction sources; for Copilot, check response references include `.github/copilot-instructions.md` when applicable; for Claude, start a new session (or use the documented load check) after editing `CLAUDE.md`; for Cursor, confirm active rules in the Agent sidebar. (These run on my machine; ChatGPT cannot perform them.)
9. Use my ongoing coding agents for repo work; entrypoint is `AGENTS.md`.
```

### E16 (R17) — Optional evaluator/QA loop standard

In section `10. docs/AGENT_TASK_ROUTER.md`, inside the E4 "Optional scoped instruction files" block area — add as a separate block right after the E4 block.
INSERT AFTER (E4 block's last line):
```
- Root `AGENTS.md` remains the cross-agent router; scoped files only narrow rules to known paths.
```
INSERT (new block):
```

Optional evaluator/QA loop (complex, UI, design, or long-running work only):

- Before implementation, define gradable acceptance criteria.
- For complex work, separate generator and evaluator roles: the evaluator reviews running behavior, code review, screenshots, or tests and returns concrete feedback. Do not rely solely on the implementing agent's self-assessment.
- Add this only as a routed/optional standard or task-router row; do not expand always-on docs.
```

### E17 (R18) — Agent-legible verification surfaces (interview capture)

In the Appendix, in section `5. Architecture and Standards`, append to its `Capture:` list.
OLD:
```
- privacy or compliance constraints, if applicable
```
NEW:
```
- privacy or compliance constraints, if applicable
- for web/UI/service repos, agent-legible verification surfaces that already exist or are wanted: dev-server command, browser automation (Playwright/Puppeteer), screenshots/DOM snapshots, logs, metrics, traces, seeded data, and whether per-worktree isolated app instances are supported. Capture only known facts or TODOs; do not invent commands. Route these in `docs/AGENT_TASK_ROUTER.md` only when known.
```

### E18 (R19) — Prefer mechanical checks for enforceable rules

In section `Agent Safety Requirements`, append after the E14 "Enforcement vs guidance" block.
INSERT AFTER:
```
- Markdown agent docs are behavioral guidance, not hard enforcement. For hard command/file/tool restrictions, use the harness's enforced settings, permissions, hooks, or rules where available, and document those surfaces separately from `AGENTS.md`.
```
INSERT (new block):
```

Mechanical checks (documented by default):

- When the repo has enforceable architecture or doc-freshness requirements, recommend mechanical checks (linters, structural tests, CI, doc-index validation) over prose-only rules, and have check failure messages tell agents how to remediate.
- Recommend these in docs by default; generate actual lint/CI/config files only when the interview explicitly approves them.
```

### E19 (R22) — Cross-harness skill sync policy

In the Appendix, in section `6. Documentation and Token Controls`, append after the skills bullet added in E11.
OLD:
```
- whether the repo should include repo-scoped reusable skills (`.agents/skills/<name>/SKILL.md`) or harness-specific commands/rules for repeatable workflows (release, security review, performance review, evals). Default: do not generate skills unless explicitly requested.
```
NEW:
```
- whether the repo should include repo-scoped reusable skills (`.agents/skills/<name>/SKILL.md`) or harness-specific commands/rules for repeatable workflows (release, security review, performance review, evals). Default: do not generate skills unless explicitly requested.
- if skills are approved: choose one canonical skill source (prefer `.agents/skills/<name>/SKILL.md`). If other harnesses need equivalents and executable tooling is approved, optionally create `scripts/sync-agent-skills.sh` to generate harness-specific adapters that preserve each target's metadata and invocation rules. Never paste large skill bodies into always-on files (`AGENTS.md`, `CLAUDE.md`, Copilot instructions, Cursor alwaysApply rules); that defeats progressive disclosure.
```

---

## P3 EDITS

### E20 (R12) — Label stable IDs as a local convention

In `### Standards defaults`.
OLD:
```
- Standard IDs may be topic-prefixed, such as `S-PLAN-001`, `S-READ-001`, `S-DOCS-001`, `S-DEPS-001`.
```
NEW:
```
- Standard IDs may be topic-prefixed, such as `S-PLAN-001`, `S-READ-001`, `S-DOCS-001`, `S-DEPS-001`.
- Stable IDs are this repo's compression and traceability convention; they are not required by `AGENTS.md` or any harness vendor. Use them where they reduce repetition; do not invent low-value IDs for trivial facts.
```

### E21 (R13) — Exclude local/private instruction files from the zip

In section `Files To Generate`, after the line listing overlay generation rules (after E3/E8 blocks). Place near the overlay rules, right before `1. README.md`.
OLD:
```
1. README.md

Purpose: human-facing project overview.
```
NEW:
```
Local/private instruction files:

- Do not include local/private memory or personal instruction files in the zip by default (e.g. `CLAUDE.local.md`, local memory directories, personal config). If mentioned, document them as local setup notes and add ignore guidance when needed. Checked-in team guidance (`AGENTS.md` and docs) remains the durable source.

1. README.md

Purpose: human-facing project overview.
```

### E22 (R20) — Harness-maintenance note for model upgrades

In section `Global Requirements`, append after the E13 conflict/staleness bullet.
OLD:
```
- Avoid conflicting rules across root, nested, scoped, user, and harness-specific instruction files. When updating any harness doc, review adjacent instruction files and remove or narrow stale or contradictory guidance (contradictory rules may be applied arbitrarily by agents).
```
NEW:
```
- Avoid conflicting rules across root, nested, scoped, user, and harness-specific instruction files. When updating any harness doc, review adjacent instruction files and remove or narrow stale or contradictory guidance (contradictory rules may be applied arbitrarily by agents).
- Include a short maintenance note in the scaffold: review this harness after major model or agent-tool upgrades; remove scaffolding that no longer improves outcomes, and add new harness surfaces only when they unlock measured capability or reliability.
```

---

## Post-implementation verification

- Confirm no edit removed or weakened the six hard constraints above.
- Confirm all new capabilities default OFF / approval-gated.
- Confirm `<!-- PASTE START -->` / `<!-- PASTE END -->` markers are unchanged and all new prompt text sits between them (except human-facing checklist items, which stay where their section already is).
- Do not commit unless the user asks.
