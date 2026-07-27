---
name: stoudemire
description: Executes an approved plan through file-coherent milestone ownership, assigning one worker and one risk-scaled reviewer per milestone with consolidated corrections. Use when the user explicitly invokes stoudemire, when nash has already run in the current session, or when the user asks to execute a plan identified as a nash plan. Do not select it automatically merely because a generic plan exists.
---

# Stoudemire: Milestone-Owned Execution

<INVOCATION-GATE>
Use this skill when the user explicitly invokes `stoudemire`, when `nash` has already run in the current session, or when the user asks to execute a plan explicitly identified as a nash plan. Do not infer it merely from the existence of a generic plan.
</INVOCATION-GATE>

Execute continuously without committing. Assign **one worker session to each milestone**, let that owner retain context across all of the milestone's work items, run focused gates, then assign **one risk-scaled reviewer**. Return all review findings to the owner in one consolidated correction round.

Do not pause between milestones unless execution is blocked, a requirement is genuinely ambiguous, or continuing could lose user data. The human reviews and commits the final working tree.

## Cost and Dispatch Budget

Repository orientation and subagent startup are expensive. Do not dispatch one agent per task, checkbox, test, file, or reviewer specialty.

For a large four- or five-milestone integration:

- Use four or five owner sessions.
- Use one reviewer session per milestone.
- Resume those sessions for questions and corrections instead of creating replacements.
- Budget approximately **12–20 total subagent dispatches**, counting resumed correction and exceptional re-review turns. The normal path still uses only 8–10 unique owner/reviewer sessions; the rest is reserve, not a quota.
- Exceed 20 dispatches only with explicit user approval after explaining the blocker and expected value.

## No Commits

Neither the controller nor any subagent may run `git commit`, amend commits, or otherwise mutate history. Skip commit steps found in older plans. If a subagent commits, determine exactly what it created, soft-reset only those commits while preserving the working tree, and report the incident.

## Setup

Before dispatching:

1. Read the plan once and extract its design, milestones, gates, risks, and full milestone text.
2. Inspect `git status --short` and identify pre-existing modified and untracked files.
3. Build one **Session Invariants** block from the plan, user statements, and known repository state.
4. If the plan is an older task-by-task plan, group its work into four or five file-coherent milestones for a large integration. Do not rewrite the plan merely to create dispatch boundaries.
5. Create one tracking item per milestone, not per work item.
6. Announce the milestone count, ordering, and expected owner/reviewer sessions, then continue without asking “should I continue?”

A dirty working tree is not automatically a blocker. Preserve pre-existing work. Ask once only when unexplained state creates a real risk of overwriting or misattributing changes.

## Persistent Session Invariants

Copy the same invariant block **verbatim** into every owner, reviewer, correction, and exceptional escalation prompt. It must include the no-commit rule and all relevant facts, such as:

```markdown
## Session Invariants

- New files are intentionally untracked. Include them in inspection; do not flag their untracked state.
- Vendor documentation is supplied read-only. It may be consulted but never edited.
- Preserve these pre-existing user changes: [paths].
- Do not commit or mutate git history.
- [Environment, compatibility, generated-file, or scope constraints.]
```

Do not make later subagents rediscover these facts. Do not let reviewers report invariant-consistent state as a finding. If an invariant changes, update the canonical block once and use the new complete block thereafter.

## Milestone Boundaries

A milestone owns a coherent integration outcome and the files or modules needed to deliver it. Its work items may contain several independently testable behaviors; they stay with one owner because they share context.

- Group work with overlapping files or tightly coupled contracts.
- Minimize cross-milestone file sharing. Where sharing is necessary, execute in dependency order and include the earlier handoff in the later owner's prompt.
- Run milestones sequentially in one working tree. Never run owners in parallel when their changes may interact.
- If a planned milestone is too broad for one retained worker context, split it into at most two coherent milestones while staying within the dispatch budget. If that would turn back into task-per-agent execution, stop and ask the user to revise the plan.

## Execution Loop

For each milestone:

### 1. Dispatch one owner

Use `./milestone-owner-prompt.md`. Give the owner:

- The full milestone text, not just a summary
- Overall architecture and dependency context
- Exact files and cross-milestone handoffs
- Acceptance criteria and focused gates
- Risk classification
- The complete Session Invariants block

The owner implements all work items, tests, self-reviews, and runs every focused gate. Resume the same owner for questions, missing context, or gate failures.

### 2. Require focused gates

Do not dispatch review while a milestone gate is red. The owner fixes failures in the same session. Use the plan's focused tests, typecheck, lint, build, migration check, or integration command. Avoid the full repository suite unless the milestone explicitly requires it.

### 3. Dispatch one risk-scaled reviewer

After gates pass, use `./milestone-reviewer-prompt.md`. One reviewer checks specification, correctness, scope, test quality, and maintainability together. Scale review depth, not reviewer count:

- **Low risk:** acceptance criteria, changed files, tests, scope, obvious regressions.
- **Medium risk:** low-risk checks plus integration seams, compatibility, error handling, and cross-milestone interactions.
- **High risk:** medium-risk checks plus security boundaries, authorization, data loss, migrations, concurrency, public contracts, rollback, or other named hazards.

Reviewers inspect untracked files listed in the milestone or status; `git diff` alone does not show them.

### 4. Consolidate one correction round

If the reviewer requests changes:

1. Normalize its findings into one deduplicated list ordered by severity.
2. Send the complete list in one prompt to the same milestone owner with the Session Invariants block.
3. Ask the owner to fix all accepted findings and rerun affected focused gates.
4. Resolve invalid or invariant-conflicting findings in the controller instead of sending them back as work.

Do not drip findings across several prompts and do not dispatch a new fixer.

### 5. Re-review only unresolved security or correctness findings

After corrections and passing gates, do not perform a routine second review. Resume the same reviewer only when a specific security or correctness finding remains unresolved or the correction materially changes a security boundary or core behavior. Scope that review to the unresolved findings and affected diff.

Do not re-review style, naming, documentation, or already resolved maintainability findings. If a second review still finds the same security or correctness issue, stop and escalate to the human rather than starting an open-ended loop.

### 6. Close the milestone

Record the owner status, gate results, reviewer verdict, corrections, and any accepted residual observations. Then proceed immediately to the next milestone.

## Handling Owner Status

- **DONE:** all focused gates pass; dispatch review.
- **DONE_WITH_CONCERNS:** resolve correctness or scope concerns before review; carry observational concerns into reviewer context.
- **NEEDS_CONTEXT:** answer and resume the same owner.
- **BLOCKED:** provide context, narrow the milestone, or escalate. Create a replacement owner only if the original session is unavailable or demonstrably cannot proceed.

## Final Gates and Handoff

After every milestone closes:

1. Run only the plan's final integration gates, if any. Fix failures with the owner whose milestone caused them; do not create a generic cleanup agent by default.
2. Run `git status --short` and `git diff --stat`. Explicitly include intentional untracked files in the summary.
3. Report milestones completed, gates run, reviewer corrections, residual concerns, and actual subagent sessions and dispatches against budget.
4. Hand the uncommitted working tree to the human for review and commit.

## Red Flags

Never:

- Dispatch one worker per task, checkbox, file, or test
- Split spec, quality, security, and documentation review across separate agents for the same milestone
- Start review before focused gates pass
- Send reviewer findings back one at a time
- Create a fresh fixer when the milestone owner can be resumed
- Perform a routine second review after corrections
- Let “new files are intentionally untracked” or another persistent invariant become a repeated finding
- Edit read-only vendor material
- Run overlapping milestone owners in parallel
- Commit or mutate history

## Prompt Templates

- `./milestone-owner-prompt.md` — one retained implementation session per milestone
- `./milestone-reviewer-prompt.md` — one combined, risk-scaled review per milestone
