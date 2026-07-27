---
name: chuck
description: Clarifies a focused task with up to 3 questions, proposes an approach, then assigns one milestone owner and one risk-scaled reviewer without a plan file. Use only when the user explicitly invokes chuck by name (for example, "use chuck", "/chuck", or "$chuck"). Never select this skill automatically based on the task.
---

# Chuck: Clarify and Implement One Milestone

<INVOCATION-GATE>
Use this skill only when the user explicitly invokes `chuck` by name. Do not infer it from task scope or need for clarification.
</INVOCATION-GATE>

Chuck is the lightweight path: clarify briefly, frame the approved scope as **one coherent milestone**, assign one worker who owns all its work, run focused gates, then assign one reviewer. There is no plan file and no task-per-agent execution.

<HARD-GATE>
Do not implement until necessary clarification is complete and the user has approved any meaningful assumptions or proposed approach.
</HARD-GATE>

## When to Use

Use chuck when the request can be delivered by one owner retaining context across a coherent set of related files and behaviors.

Use nash + stoudemire when the request needs several architectural outcomes, four or five file-coherent milestones, meaningful cross-milestone handoffs, or more than three clarification questions. Do not split a large integration into a parade of chuck workers.

## Checklist

1. **Quick context check** — inspect relevant structure and current working-tree state
2. **Clarify** — ask up to three questions, one at a time; skip only for mechanical, unambiguous work
3. **Approve approach** — state one recommendation and get approval when assumptions matter
4. **Frame one milestone** — outcome, files, work, acceptance criteria, focused gates, risk, and invariants
5. **Dispatch one owner** — the same worker handles all work items and gate failures
6. **Dispatch one reviewer** — combined, risk-scaled review after gates pass
7. **Consolidate corrections** — return all valid findings to the owner once
8. **Report** — hand back an uncommitted diff

## Clarify and Approve

Glance at relevant files, patterns, dependencies, and `git status --short`. Keep this quick.

Ask up to three questions, one per message, only when the answers affect behavior, UX/API shape, compatibility, scope, or validation. Prefer multiple choice where useful. If the request is precise and mechanical, say why it is unambiguous and proceed without ceremonial questions.

When clarification or assumptions matter, present one recommended approach covering the outcome, likely files, validation, and trade-offs. Ask for approval before dispatching.

## Milestone Fit Gate

Treat the approved scope as one milestone containing multiple related work items. It fits chuck when:

- One worker can retain the necessary context and deliver the integrated outcome in one session.
- Files and modules are related enough that one owner avoids repeated repository orientation.
- Focused tests and checks can validate the outcome.
- The work does not require unresolved architectural choices or several independent handoffs.

Do not create a worker per behavior, file, or checkbox. If the scope does not fit one coherent milestone, tell the user:

> "This needs multiple milestone owners and explicit handoffs. I recommend switching to nash so we can group it into a small number of file-coherent milestones instead of paying for task-per-agent execution."

Wait for the user's choice. Do not silently fragment the work.

## Frame the Milestone

Before dispatching, synthesize:

- **Outcome:** the integrated state the owner must deliver
- **Files:** exact create/modify paths and pre-existing files to preserve
- **Work:** related implementation, test, fixture, documentation, or packaging items
- **Acceptance criteria:** observable results
- **Focused gates:** exact narrow test, typecheck, lint, build, or integration commands
- **Risk:** Low, Medium, or High with named hazards
- **Session Invariants:** persistent repository and user constraints

Capture invariants once and copy them verbatim into every owner, reviewer, and correction prompt. Include facts such as:

- `New files are intentionally untracked.`
- `Vendor documentation is supplied read-only and must not be edited.`
- Pre-existing user changes and files that must be preserved.
- No commits or history mutation.
- Environment, compatibility, generated-file, or scope constraints.

A dirty tree is not automatically a blocker. Ask only if unexplained state creates overwrite or attribution risk.

## Dispatch Budget

Normal chuck execution uses **two subagent sessions**: one milestone owner and one reviewer. Resume them for questions and corrections. Do not create new agents to hit a quota. A second review is a resumed reviewer turn, and only for unresolved security or correctness findings.

If execution appears to need more than four unique sessions, stop and recommend nash rather than allowing chuck to grow into task-per-agent orchestration.

## Owner Dispatch

Dispatch one worker with this structure:

```text
You own this complete milestone. Implement and validate all related work items in one
retained context; do not delegate them to separate agents.

## Outcome
[Integrated result]

## Files
[Exact paths, including pre-existing work to preserve]

## Work
[Concrete related work items]

## Acceptance Criteria
[Observable outcomes]

## Focused Gates
[Exact commands]

## Risk
[Low/Medium/High and named hazards]

## Session Invariants
[Copy canonical block verbatim]

Deliver the integrated outcome, preserve invariant-listed state, and do not commit.
If new files are intentionally untracked, leave them that way and include them in your
self-review. If vendor documentation is read-only, consult but never edit it.

Run every focused gate and fix failures in this same session. Do not report DONE while
a gate is red. Self-review the complete result, including untracked files.

Report:
- Status: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
- Outcome and acceptance criteria delivered
- Exact gate results
- Files changed, including untracked files
- Concerns and confirmation that no commit occurred
```

Resume the same owner for questions, missing context, or failed gates. Review starts only after every focused gate passes.

## Risk-Scaled Review

Dispatch one fresh reviewer with the full milestone, owner report, gate results, and canonical Session Invariants block. Tell it to inspect actual tracked and untracked files and to return one consolidated findings list without modifying anything.

One reviewer covers spec, correctness, scope, tests, and quality. Scale depth rather than agent count:

- **Low:** acceptance criteria, scope, tests, obvious regressions, and code fit.
- **Medium:** low-risk checks plus integration seams, compatibility, and failure paths.
- **High:** medium-risk checks plus security boundaries, authorization, data loss, migrations, concurrency, public contracts, rollback, and named hazards.

The reviewer must not flag invariant-consistent state. In particular, intentional untracked files are inspected rather than reported merely for being untracked, and read-only vendor documentation is checked for unauthorized edits rather than targeted for changes.

Require this format:

```text
- Verdict: APPROVED | CORRECTIONS_REQUIRED
- Risk reviewed and emphasis
- One deduplicated findings list ordered by severity
- Each finding: category, severity, file:line, evidence, impact, concrete correction
- Acceptance criteria and focused-gate coverage
- Confirmation that Session Invariants were respected
```

## One Correction Round

If corrections are required:

1. Remove duplicate, invalid, and invariant-conflicting findings.
2. Send the entire remaining list to the same owner in one prompt.
3. Have the owner fix all accepted findings and rerun affected focused gates.
4. Do not dispatch a generic fixer or drip findings across prompts.

Do not run a routine second review. Resume the same reviewer only if a security or correctness finding remains unresolved or the correction materially changes a security boundary or core behavior. Scope it to those findings. If they remain after that, escalate to the user instead of looping.

## No Commits and Final Report

Neither controller, owner, nor reviewer commits or mutates history. If an agent commits, soft-reset only its commits while preserving the working tree and tell the user.

At the end, run `git status --short` and `git diff --stat`, account for intentional untracked files, and report the outcome, gates, corrections, residual concerns, and uncommitted files.

## Key Principles

- Brief clarification, not ceremony
- One milestone owner, not one agent per task
- Focused gates before one combined review
- Risk changes review depth, not reviewer count
- One consolidated correction round
- Second review only for unresolved security or correctness
- Persistent invariants in every prompt
- Two sessions by default; no uncontrolled dispatch growth
- Human-owned commit history
