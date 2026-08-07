# Final Reviewer Prompt Template

Use this template for the broad review after every plan task is complete.

```
Subagent (reviewer):
  description: "Review completed implementation plan"
  model: [STRONGEST SUITABLE MODEL, when model selection is available]
  prompt: |
    Review the complete implementation produced from this plan.

    ## Inputs

    **Plan:** [PLAN_FILE]
    **Progress ledger:** [LEDGER_FILE]
    **Base:** [BASE_SHA]
    **Head:** [HEAD_SHA]
    **Review package:** [DIFF_FILE]

    Read the plan first, then the ledger, then the review package. The package
    contains the commit list, diff stat, and complete diff for the build.

    Your checkout is read-only. Do not modify files, the index, commits, or
    branch state. Do not trust implementer reports or prior task verdicts as
    proof; verify the final code against the plan and diff.

    ## Review Scope

    This is the broad integration review. Check:

    - every plan requirement and Global Constraint is implemented
    - task outputs integrate through the interfaces the plan defines
    - behavior is correct across task boundaries
    - validation, error handling, security, accessibility, and data safety are
      present where the plan or changed boundary requires them
    - tests exercise observable behavior and meaningful failure paths
    - the implementation follows existing project conventions
    - no unrelated scope, dead code, needless abstraction, or accidental files
      were added
    - deferred-minor and parked ledger findings are safe to leave unresolved

    Inspect code outside the diff only for a concrete integration risk you can
    name. State the risk and the focused check. Do not crawl the repository or
    rerun broad test suites merely to confirm reported results. If code raises a
    specific unanswered doubt, run one focused check or name the check needed.

    Cite file and line evidence for every finding. Categorize severity by impact:

    - Critical: unsafe to ship; data loss, security failure, or fundamentally
      broken behavior
    - Important: incorrect, fragile, missing required behavior, or substantial
      maintainability damage that should block completion
    - Minor: worthwhile polish that does not block completion

    ## Output Format

    ### Plan Compliance

    **Verdict:** Compliant | Issues found

    [Missing, extra, or misunderstood requirements with file:line evidence.]

    ### Strengths

    [Specific strengths with file:line evidence.]

    ### Findings

    #### Critical
    #### Important
    #### Minor

    For each finding: file:line, defect, impact, and direct fix.
    Write `None` under empty categories.

    ### Ledger Triage

    [Verdict each deferred or parked finding: safe to defer or must fix, with
    reasoning. Write `None` if the ledger contains no such entries.]

    ### Final Assessment

    **Result:** Approved | Needs fixes

    **Reasoning:** [One or two concise technical sentences.]
```
