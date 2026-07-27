---
name: nash
description: Brainstorms intent, requirements, and design through one-question-at-a-time dialogue, then writes a milestone-based implementation plan. Use only when the user explicitly invokes nash by name (for example, "use nash", "/nash", or "$nash"). Never select this skill automatically based on the task.
---

# Nash: Brainstorm and Plan

<INVOCATION-GATE>
Use this skill only when the user explicitly invokes `nash` by name. Do not infer that nash should be used from the size, complexity, or creative nature of a task, and do not invoke it automatically.
</INVOCATION-GATE>

Turn an idea into one approved implementation plan. Nash plans around **milestone ownership**, not task-per-agent execution: a milestone groups related work that benefits from one worker retaining repository and file context.

<HARD-GATE>
Do NOT write code, scaffold a project, or take implementation action until you have presented a design, the user has approved it, and you have written the plan document.
</HARD-GATE>

## Output

Write one plan document at `plans/YYYY-MM-DD-<topic>-plan.md`. This is the only artifact. Stoudemire or a human executes it milestone by milestone.

## Checklist

Complete these in order:

1. **Explore project context** — check relevant files, docs, and recent commits
2. **Ask clarifying questions** — one at a time; understand purpose, constraints, and success criteria
3. **Propose approaches** — compare real alternatives and recommend one
4. **Present design** — scale detail and approval rounds to complexity
5. **Write the plan** — organize the work into file-coherent milestones
6. **Review the plan once** — one read-only subagent checks completeness and execution readiness; fix its findings in one pass
7. **User review** — wait for approval before handing off

## Brainstorming

### Understand the idea

- Inspect the current project before proposing changes. Follow existing patterns.
- If the request contains several independent products or subsystems, help the user choose a smaller integration to plan first.
- Ask questions one at a time and prefer multiple choice when useful.
- Ask only questions whose answers affect behavior, architecture, compatibility, scope, or validation.

### Explore approaches

- Lead with the recommended approach and explain the trade-offs.
- Offer 2–3 alternatives only when meaningful alternatives exist. If constraints leave one sensible path, say so and move on.
- Avoid unrelated refactors and speculative infrastructure.

### Present the design

Cover architecture, components, data flow, error handling, compatibility, and testing. For nuanced designs, seek approval section by section. For straightforward designs, present the design once and ask once.

## Plan Around Milestones

A milestone is a coherent integration outcome owned by one worker session. It may contain several implementation and test work items. Do not turn every checkbox, behavior, or file into a separate dispatch.

For a large integration, default to **four or five milestones**. Use fewer for smaller plans; add a sixth only when a genuinely separate deployment or migration boundary makes five incoherent. The milestone names must come from the actual design. Examples such as contracts, runtime, public API, fixtures, and release are illustrations, not a template.

### Group by file context

- Put work items that touch the same files or tightly coupled modules in the same milestone.
- Minimize files revisited by later milestones. When overlap is unavoidable, order milestones by dependency and state what the later owner must preserve.
- Give one milestone ownership of a cross-cutting concern where practical rather than making every worker rediscover it.
- Keep a milestone large enough to produce an integration outcome, but small enough for one worker to implement, validate, and self-review in one retained context.
- Order milestones so each leaves the working tree in a usable, verifiable state.

### Dispatch budget

For a four- or five-milestone integration, set a default budget of approximately **12–20 total subagent dispatches**, including resumed correction and exceptional re-review turns. The normal path uses only four or five owner sessions and four or five reviewer sessions; the remaining budget is reserve, not a quota. Prefer fewer when sufficient and never create agents merely to spend the budget.

### Persistent session invariants

Add a `Session Invariants` section containing facts every later prompt must preserve. Capture known repository state and external constraints, for example:

- `New files are intentionally untracked.`
- `Vendor documentation is supplied read-only and must not be edited.`
- Pre-existing modified files belong to the user and must be preserved.
- No worker or reviewer commits.
- Environment, compatibility, generated-file, or validation constraints.

Use the user's wording where possible. Stoudemire will copy this block verbatim into every owner, reviewer, and correction prompt so known conditions do not become repeated false alarms.

## Plan Format

Every plan starts with:

```markdown
# [Feature Name] Plan

> **Execution:** Use `stoudemire` milestone by milestone. Assign one worker session to each milestone and one risk-scaled reviewer after its focused gates pass. Do not dispatch one agent per work item. Do not commit during execution.

**Goal:** [One sentence]

**Architecture:** [2–3 sentences]

**Tech Stack:** [Key technologies and libraries]

**Dispatch budget:** Approximately 12–20 total subagent dispatches for a large integration, including resumed correction turns; fewer is better when sufficient.

## Design

[Approved architecture, components, data flow, error handling, compatibility, and test strategy]

## Session Invariants

- [Persistent repository or user constraint]
- No worker or reviewer commits; the human owns commit history.

## Milestone Map

| Milestone        | Outcome             | Primary files/modules | Depends on | Risk                       |
| ---------------- | ------------------- | --------------------- | ---------- | -------------------------- |
| 1. [Unique name] | [Integrated result] | [Paths/modules]       | None       | Low/Medium/High — [reason] |

---
```

Use this structure for each milestone:

```markdown
## Milestone N: [Outcome-oriented name]

**Outcome:** [The integrated state this milestone makes true]

**Owner context:** [How this fits the architecture, what earlier work exists, and what later milestones depend on]

**Files:**

- Create: `exact/path`
- Modify: `exact/path`
- Preserve from earlier milestones: `exact/path` — [contract or behavior]

**Work:**

- [ ] [Concrete implementation work item with enough detail to act]
- [ ] [Related implementation or integration work item]
- [ ] [Tests or fixtures that belong with this outcome]

**Acceptance criteria:**

- [Observable behavior]
- [Observable integration result]

**Focused gates:**

- `exact command for focused tests`
- `exact command for relevant typecheck/lint/build if needed`

**Risk:** Low | Medium | High — [why, and what the reviewer should emphasize]

**Handoff:** [What the next milestone can rely on; include file-overlap constraints]
```

### Plan quality rules

- Give exact paths and executable gate commands.
- Include implementation detail that resolves ambiguity, but avoid microsteps whose only purpose is creating more dispatches.
- Every type, endpoint, command, fixture, and package referenced must be defined in this milestone or an earlier one.
- Acceptance criteria must be observable rather than restating the milestone name.
- Focused gates should validate the milestone without defaulting to the entire suite. Reserve broad final gates for the end when they are genuinely needed.
- Do not include commit steps.
- Do not include placeholders such as `TBD`, `TODO`, “appropriate error handling,” “similar to above,” or unspecified test commands.

## One Plan Review

After writing the plan, dispatch one read-only review subagent. Give it the absolute plan path and the known session invariants. Ask it to report, not fix, issues against this checklist:

1. The plan reflects the approved design without contradictions or placeholders.
2. A large integration has four or five coherent, uniquely named milestones unless the plan explains why not.
3. Work sharing file context is grouped together; unavoidable cross-milestone overlap has an explicit handoff.
4. Each milestone has one integrated outcome, exact files, observable acceptance criteria, focused gates, risk, and dependencies.
5. Types, interfaces, commands, and file names remain consistent across milestones.
6. The plan uses one owner per milestone and does not imply one agent per work item.
7. Session invariants are concrete enough to copy into every execution prompt.
8. There are no commit steps.

Ask for one consolidated list with locations and suggested corrections. Apply the corrections in one pass. Do not dispatch a second plan review unless a reported correctness contradiction remains unresolved.

## User Review and Handoff

Ask the user to review the saved plan. If they request changes, update it and only repeat plan review when the changes affect architecture, security, or cross-milestone correctness.

After approval, offer:

> "Plan approved. Ready to execute with stoudemire? It will assign one worker and one risk-scaled reviewer per milestone, keep correction findings consolidated, and leave the full diff uncommitted for you."

## Key Principles

- One question at a time
- Real alternatives, not ceremonial alternatives
- Milestone ownership over task-per-agent execution
- Group by shared file and repository context
- Persistent invariants in every downstream prompt
- Focused gates before review
- One consolidated correction round
- Fewest subagents that preserve correctness and security
- Human-controlled commit history
