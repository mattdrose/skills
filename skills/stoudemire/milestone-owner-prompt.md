# Milestone Owner Prompt Template

Use this template for the single worker session that owns an entire milestone.

```text
Task tool (general-purpose):
  description: "Own Milestone N: [milestone name]"
  prompt: |
    You own Milestone N from implementation through focused validation. Retain context
    across every work item; do not delegate individual tasks to new agents.

    ## Overall Design

    [Goal, architecture, and where this milestone fits]

    ## Milestone

    [FULL milestone text from the plan]

    ## Dependencies and Handoffs

    [What prior milestones established, especially contracts and shared-file behavior
    that must be preserved; what later milestones will rely on]

    ## Session Invariants

    [COPY THE CANONICAL SESSION INVARIANTS BLOCK VERBATIM. Include intentional
    untracked files, read-only vendor documentation, pre-existing user changes,
    environment constraints, and the no-commit rule.]

    ## Ownership

    Deliver the milestone's integrated outcome, not isolated checklist completions.
    Implement all listed work, tests, fixtures, and packaging changes that belong to
    this milestone. Preserve earlier work in overlapping files.

    You may inspect nearby code needed to follow existing patterns. Stay within the
    listed file/module scope unless a small unavoidable integration edit is required;
    report any such edit. Stop with NEEDS_CONTEXT before broadening architecture or
    overwriting unexplained user work.

    If Session Invariants say new files are intentionally untracked, leave them
    untracked and include them in your own inspection. If vendor documentation is
    read-only, consult it without changing it.

    ## Focused Gates

    Run every gate listed in the milestone. Fix failures in this same session. Do not
    substitute the whole repository suite unless the milestone requires it. Do not
    report DONE while a focused gate is red.

    ## Self-Review

    Before reporting:

    - Check every acceptance criterion against actual behavior.
    - Read the complete resulting changes in milestone-owned files, including untracked files.
    - Check integration with prior milestone handoffs.
    - Remove scope creep, dead code, debug output, and speculative abstractions.
    - Check error handling, compatibility, security, and data safety at the stated risk level.
    - Confirm no commits or history mutations occurred.

    ## Corrections

    A reviewer may return one consolidated findings list. When resumed, address all
    accepted findings together, explain any finding that conflicts with the spec or
    Session Invariants, and rerun affected focused gates. Do not wait for findings one
    at a time.

    ## Report Format

    - **Status:** DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
    - Integrated outcome delivered
    - Acceptance criteria satisfied
    - Focused gates and exact pass/fail results
    - Files changed, including intentional untracked files and any out-of-scope edit
    - Cross-milestone handoff for the next owner
    - Concerns or missing context
    - Confirmation that no commit occurred

    Keep the report concise. Do not narrate repository exploration.
```
