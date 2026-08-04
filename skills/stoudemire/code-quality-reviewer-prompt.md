# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent under stoudemire.

**Purpose:** Verify the implementation is well-built (clean, tested, maintainable).

**Only dispatch after spec compliance review passes.**

```
Task tool (general-purpose):
  description: "Code quality review for Task N"
  prompt: |
    You are reviewing the code quality of a recently completed task.

    ## Context

    Plan: [path to plan file]
    Task: [Task N name + summary from implementer's report]

    ## How to Read the Changes

    Stoudemire does NOT commit during execution, so the implementer's changes live
    in the working tree as uncommitted edits. Use:

      git diff               # all unstaged + staged uncommitted changes
      git status             # files touched
      git diff --stat        # summary

    Read the full diff. Then read the modified files in their entirety where needed
    to judge fit with the surrounding code.

    ## What to Evaluate

    Standard code quality concerns:
    - Correctness, edge cases, error handling
    - Test coverage and test quality (do tests verify behavior, not mock interactions?)
    - Naming clarity (do names match what things do?)
    - Readability and maintainability
    - Adherence to existing codebase patterns
    - Security and obvious performance issues

    Stoudemire-specific concerns:
    - Does each file have one clear responsibility with a well-defined interface?
    - Are units decomposed so they can be understood and tested independently?
    - Does the implementation follow the file structure from the plan?
    - Did this change create new files that are already large, or significantly grow
      existing files? (Don't flag pre-existing file sizes — focus on what this change
      contributed.)
    - Did the implementer commit anything? They MUST NOT have. Run
      `git log -5 --oneline` to confirm no new commits exist.

    ## Report Format

    - **Strengths:** what was done well
    - **Issues:**
      - **Critical:** must fix before this task is considered done
      - **Important:** should fix before the human reviews
      - **Minor:** nice to have
    - **Assessment:** ✅ Approved | ❌ Needs fixes
```
