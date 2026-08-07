---
name: nash
description: Brainstorm an idea through one-question-at-a-time dialogue, turn the approved design directly into a detailed implementation plan, and hand that plan to stoudemire. Use when the user invokes nash or asks for a collaborative design-and-planning workflow before implementation.
---

# Nash

<INVOCATION-GATE>
Use this skill only when the user explicitly invokes `nash` by name. Do not infer that nash should be used from the size, complexity, or creative nature of a task, and do not invoke it automatically.
</INVOCATION-GATE>

Turn an idea into one implementation plan through collaborative dialogue. Explore the project, clarify the request, compare approaches, get design approval, and write the build-ready plan.

<HARD-GATE>
Do not write code, scaffold a project, or take implementation action while using Nash. Finish and obtain approval for the plan first.
</HARD-GATE>

## Output

Save the only deliverable to `plans/<plan-stub>.md`, where `<plan-stub>` is a concise kebab-case name for the work. Do not add a separate spec or design document.

The plan must be self-contained because Stoudemire gives each implementation task to a fresh subagent with no conversation history.

## Workflow

Follow these steps in order:

1. **Explore project context** — inspect relevant files, documentation, conventions, tests, and recent commits.
2. **Assess scope** — identify whether the request must be split into independently buildable plans.
3. **Ask clarifying questions** — ask one question per message until purpose, constraints, behavior, and success criteria are clear.
4. **Compare approaches** — present 2-3 viable approaches with trade-offs and a recommendation.
5. **Present the design** — explain the proposed architecture, boundaries, data flow, error handling, and testing; obtain user approval.
6. **Write the implementation plan** — save it directly to `plans/<plan-stub>.md`.
7. **Review the plan** — check coverage, consistency, task boundaries, exact interfaces, commands, and placeholders; fix issues inline.
8. **Request plan approval** — ask the user to review the saved plan. Revise it until approved.
9. **Hand off to Stoudemire** — tell the user the approved plan is ready for execution with Stoudemire.

## Understand the Work

Start from the repository rather than assumptions:

- Follow established project structure and conventions.
- Identify the public behavior that must change and the existing boundaries it crosses.
- Look for project-specific testing, validation, accessibility, security, and error-handling requirements.
- Avoid unrelated refactoring. Include a targeted cleanup only when the requested work depends on it.

Before detailed questions, check whether the request contains independent subsystems. If each subsystem could produce useful, testable software on its own, propose separate plans and establish their order. Continue this workflow for the first plan only.

Ask questions one at a time. Prefer multiple-choice questions when useful, but allow open-ended answers. Focus on decisions that affect implementation; do not make the user answer questions the repository already answers.

## Compare Approaches

Offer 2-3 materially different approaches. Lead with the recommendation and explain why it best fits the observed codebase and constraints. Include meaningful trade-offs such as:

- consistency with existing architecture
- implementation and migration risk
- operational or maintenance cost
- testability and reversibility
- unnecessary scope

Apply YAGNI: do not add flexibility, abstractions, or features without a current requirement.

## Present the Design

Scale the design to the task. A small change may need a few paragraphs; a nuanced feature may need several short sections. Cover only applicable topics:

- user-visible behavior and success criteria
- architecture and component responsibilities
- interfaces and data flow
- validation, failures, and recovery
- compatibility or migration concerns
- testing strategy

Present the design in digestible sections and ask whether each section is right before moving on. Resolve feedback before writing the plan.

## Write the Plan

Announce: “I’m using Nash to write the implementation plan.”

Map the files before defining tasks. Each file should have one clear responsibility, and each task should end in an independently testable deliverable that a reviewer could approve or reject on its own. Fold setup, configuration, and documentation into the task whose deliverable needs them.

Use exact repository-relative paths. Include line ranges when they are stable and useful. Define interfaces explicitly so tasks can be implemented independently.

### Required header

Every plan starts with:

```markdown
# [Feature Name] Implementation Plan

**Goal:** [One sentence describing the outcome]

**Architecture:** [Two or three sentences describing the approach]

**Tech Stack:** [Relevant technologies and libraries]

## Global Constraints

- [Exact project-wide requirement]
- [Exact compatibility, dependency, naming, or behavioral constraint]

---
```

Do not invent global constraints merely to fill the section. Write `None beyond existing project conventions.` when there are none.

### Task structure

````markdown
### Task N: [Testable deliverable]

**Files:**

- Create: `exact/path/to/file`
- Modify: `exact/path/to/existing-file:line-range`
- Test: `exact/path/to/integration-test`

**Interfaces:**

- Consumes: [existing or earlier-task interfaces with exact signatures]
- Produces: [new interfaces with exact signatures and behavior]

- [ ] **Step 1: Add the failing behavior test**

```language
[Complete test code or an exact patch-sized excerpt]
```

- [ ] **Step 2: Run the focused test and verify the expected failure**

Run: `[exact command]`
Expected: `[specific failure showing the behavior is missing]`

- [ ] **Step 3: Implement the behavior**

```language
[Complete implementation or an exact patch-sized excerpt]
```

- [ ] **Step 4: Run verification**

Run: `[exact focused and relevant integration commands]`
Expected: `[specific passing result]`

- [ ] **Step 5: Commit the task**

```bash
git add [exact paths]
git commit -m "[specific commit subject]"
```
````

Adapt the test-first sequence when the repository’s testing policy or the nature of the task requires another form of verification. Preserve the same standard: name the observable behavior, exact command, and expected result.

## Plan Quality Rules

Write for a skilled engineer who knows neither this codebase nor the conversation.

- Include actual code or precise patch-sized excerpts for code-writing steps.
- Define types, functions, commands, values, and expected outputs exactly.
- Repeat task-local facts instead of referring vaguely to another task.
- Keep neighboring task interfaces consistent.
- Respect the repository’s existing architecture and testing policy.
- Keep tasks small enough for focused implementation and review, but do not split mechanical setup from the feature that needs it.
- Include validation at trust boundaries, relevant failure handling, security measures, accessibility basics, and data-loss prevention.

Never write placeholders such as `TBD`, `TODO`, “implement later,” “add appropriate error handling,” “write tests for the above,” or “similar to Task N.”

## Self-Review

After saving the plan, review it with fresh eyes:

1. **Requirement coverage:** Every approved requirement maps to at least one task.
2. **Scope:** The plan produces one coherent, independently useful outcome.
3. **Buildability:** Every step contains enough detail to execute without conversation history.
4. **Consistency:** Paths, names, values, signatures, and task-to-task interfaces agree.
5. **Verification:** Tasks test observable behavior using exact commands and expected results.
6. **Placeholder scan:** Remove vague instructions and unfinished content.
7. **Stoudemire readiness:** Tasks have clear boundaries, checkboxes, and no hidden dependencies.

Fix issues inline, then tell the user:

> Plan complete and saved to `plans/<plan-stub>.md`. Please review it and let me know what you want changed. Once approved, Stoudemire can build it task-by-task.

Wait for approval. Do not begin implementation from Nash.
