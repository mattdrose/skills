---
name: stoudemire
description: Execute a Nash implementation plan in the current session with a fresh implementer subagent per task, task-scoped review after each task, and a broad final review. Use when the user invokes stoudemire or asks to build a plan created by Nash.
---

# Stoudemire

<INVOCATION-GATE>
Use this skill when the user explicitly invokes `stoudemire` by name. You may also use it when `nash` has already run in the current session or when the user asks to execute a plan explicitly identified as a nash plan. Do not infer that stoudemire should be used merely because a generic task-decomposed plan exists.
</INVOCATION-GATE>

Execute a Nash plan by dispatching a fresh implementer for each task.

**Core principle:** Fresh implementer per task = focused execution with independent quality gates.

**Narration:** Between tool calls, narrate at most one short line. The progress ledger and artifact files carry the detailed record.

**Continuous execution:** Do not pause between tasks. Stop only when blocked by missing information, a plan conflict that requires the user’s decision, a load-bearing defect that cannot be fixed, or completion of all tasks.

## Preconditions

Stoudemire needs a task-decomposed plan, normally `plans/<plan-stub>.md`, whose tasks are sufficiently independent for subagents. If no suitable plan exists, ask the user to run Nash or provide one. If tasks are tightly coupled and cannot be handed off independently, explain why Stoudemire is not a good fit.

Work in the current checkout. Before implementation:

- inspect `git status` and the current branch
- preserve unrelated user changes
- if the current branch is `main` or `master`, obtain explicit permission before creating commits
- record the initial commit for the final review range

## Artifact Workspace

Each plan owns temporary build artifacts at `plans/<plan-stub>/`, next to `plans/<plan-stub>.md`. Use this skill’s `scripts/workspace PLAN_FILE` to resolve and create it.

The workspace contains:

- `progress.md` — recovery ledger
- `task-N-brief.md` — one task’s extracted requirements
- `task-N-report.md` — implementer and fix reports
- `review-<base>..<head>.diff` — task and final review packages

The directory is git-ignored by its own `.gitignore`. Delete this plan’s workspace after finishing the plan. Never read, change, or delete another plan’s workspace.

Conversation memory may not survive compaction, so the ledger is authoritative:

1. Resolve the workspace.
2. If `progress.md` exists and its first line identifies this plan, trust its completion and fix-round entries. Resume at the first incomplete task.
3. Otherwise create it with `# Stoudemire ledger — plan: <plan file path>` as the first line.
4. Cross-check ledgered commits with `git log` before resuming work.

## Setup and Preflight

Read the plan once. Record its goal, architecture, Global Constraints, and task list. Track one todo per task when the harness supports todos.

Before Task 1, scan the complete plan for:

- contradictory tasks or interfaces
- conflicts with Global Constraints
- steps that would overwrite unrelated user work
- requirements that force an obvious quality defect
- missing information that genuinely prevents implementation

Batch any conflicts into one question to the user, quoting the relevant plan text and asking which requirement governs. If the scan is clean, proceed without an extra check-in.

## Per-Task Loop

Never dispatch multiple implementers in parallel; their commits and working-tree edits can conflict.

### 1. Prepare the task

Record `BASE=$(git rev-parse HEAD)` before dispatch.

Run this skill’s helper:

```bash
<stoudemire-skill-directory>/scripts/task-brief PLAN_FILE TASK_NUMBER
```

The command writes the complete task to `plans/<plan-stub>/task-N-brief.md`. Never paste the whole plan or accumulated task history into a subagent prompt.

Create a report path beside the brief: `plans/<plan-stub>/task-N-report.md`.

### 2. Dispatch a fresh implementer

Use [implementer-prompt.md](implementer-prompt.md). Give the implementer:

1. one sentence explaining where the task fits
2. the brief path as the authoritative requirements
3. only the interfaces or decisions from completed tasks that this task needs
4. any ambiguity resolution relevant to this task
5. the report path and report contract
6. the current checkout path

Exact values belong in the task brief, not duplicated in the dispatch. Do not include session history or broad summaries of earlier work.

Choose the least expensive available implementer capable of the task when the harness supports model selection:

- complete, mechanical changes in 1-2 files: fast model
- multi-file integration or debugging: standard model
- architecture or difficult judgment: strongest model

### 3. Handle implementer status

The implementer returns one status:

- **DONE:** Generate a review package and dispatch task review.
- **DONE_WITH_CONCERNS:** Read the concerns. Resolve correctness or scope doubts before review; ledger non-blocking observations.
- **BLOCKED:** Try to unblock them. If you absolutely need to ask the user about a defective plan.

Never repeat the same dispatch unchanged.

For DONE, run:

```bash
<stoudemire-skill-directory>/scripts/review-package PLAN_FILE BASE HEAD
```

Use the recorded task base, never `HEAD~1`; a task may create multiple commits.

### 4. Complete the task

When review is clean, or all residual findings are parked after the cap, append one completion entry:

- `Task <N>: complete (commits <base7>..<head7>, review clean)`
- `Task <N>: complete (commits <base7>..<head7>, <count> parked)`

Mark the task todo complete and immediately continue to the next task.

## Final Review

After all tasks, generate one review package from the initial commit to current HEAD:

```bash
<stoudemire-skill-directory>/scripts/review-package PLAN_FILE INITIAL_COMMIT HEAD
```

Dispatch the strongest suitable reviewer with [final-reviewer-prompt.md](final-reviewer-prompt.md). Give it:

- the complete plan path
- the final review package
- the progress ledger
- the initial and current commits

Point it to deferred-minor and parked entries so it can decide whether any block completion.

If final review finds Critical or Important issues: Dispatch one implementer with the complete findings list.

## Finish

After final review is clean:

1. Verify the working tree does not contain accidental artifact files or unrelated modifications caused by execution.
2. Delete only this plan’s temporary workspace: `rm -rf plans/<plan-stub>/`.
3. Report the implemented plan, commit range, tests run, final-review result, and any parked non-blocking findings.
4. Leave branch integration, merge, push, or pull-request decisions to the user unless they explicitly requested them.
