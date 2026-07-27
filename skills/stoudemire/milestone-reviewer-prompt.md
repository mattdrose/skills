# Milestone Reviewer Prompt Template

Use this once after a milestone owner passes every focused gate. The same reviewer covers specification, correctness, scope, tests, and quality at risk-appropriate depth.

```text
Task tool (general-purpose):
  description: "Review Milestone N: [milestone name]"
  prompt: |
    Review Milestone N independently. You did not implement it. Return one consolidated
    findings list; do not modify files and do not create separate specialist reviews.

    ## Overall Design

    [Goal, architecture, and milestone dependencies]

    ## Milestone Specification

    [FULL milestone text: outcome, files, work, acceptance criteria, gates, risk, handoff]

    ## Owner Report and Gate Results

    [Owner's report and confirmation that every focused gate passed]

    ## Session Invariants

    [COPY THE CANONICAL SESSION INVARIANTS BLOCK VERBATIM]

    Treat these as facts. Do not report invariant-consistent state as a problem. For
    example, if new files are intentionally untracked, inspect them but do not flag
    their untracked status. If vendor documentation is read-only, verify it was not
    edited rather than proposing edits to it.

    ## Inspect the Actual Work

    Do not trust the owner report alone.

    - Use `git status --short` to identify modified and untracked files.
    - Use `git diff` and `git diff --stat` for tracked changes.
    - Read every milestone-owned untracked file directly because `git diff` omits it.
    - Read surrounding code only where needed to judge integration and existing patterns.
    - Account for prior milestone work in overlapping files; judge this milestone's
      outcome and whether it preserved the documented handoff.

    ## Review Depth

    Always check:

    - Every acceptance criterion is implemented and meaningfully tested.
    - The integrated outcome works, not merely each work item in isolation.
    - Changes stay within scope and avoid speculative abstractions or unrelated cleanup.
    - Error handling, names, maintainability, and tests fit surrounding code.
    - No commit or history mutation occurred.

    Scale additional attention to the milestone's declared risk:

    - **Low:** obvious regressions, scope, focused test quality, and file fit.
    - **Medium:** integration seams, compatibility, failure paths, public behavior, and
      interactions with earlier milestones.
    - **High:** authorization and trust boundaries, secrets, injection, data loss,
      migrations and rollback, concurrency, public contracts, and other named hazards.

    ## Report Format

    - **Verdict:** APPROVED | CORRECTIONS_REQUIRED
    - **Risk reviewed:** Low | Medium | High — [specific emphasis]
    - **Findings:** one deduplicated list ordered by severity
      - **Security** or **Correctness** findings first
      - Then **Maintainability**, **Testing**, or **Scope**
      - For each: category, severity, `file:line`, evidence, impact, and one concrete correction
    - **Acceptance criteria coverage:** concise pass/gap summary
    - **Gate confidence:** whether the focused gates substantively validate the milestone
    - **Invariant check:** confirm invariant-consistent state was not treated as a finding

    Do not emit optional stylistic preferences as required corrections. The controller
    will send the entire valid findings list to the owner in one correction round.
```
