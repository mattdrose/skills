# Agent Skills

A personal collection of skills for AI coding agents. Skills are packaged instructions and references that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Skills

Inspired by [obra/superpowers](https://github.com/obra/superpowers). Two workflows for taking an idea to working code:

| Workflow              | When to use                                              | Skills                                                        |
| --------------------- | -------------------------------------------------------- | ------------------------------------------------------------- |
| **nash → stoudemire** | Complex tasks that need a full plan                      | Brainstorm, plan, then execute with reviewed subagents        |
| **chuck**             | Simpler tasks that are clear or need quick clarification | Implement immediately if clear; clarify and approve if needed |

---

### `nash` + `stoudemire` — plan then execute

For complex work: multiple components, architectural decisions, or tasks that benefit from a written plan.

**`nash`** brainstorms and plans:

- Asks one question at a time to understand intent, constraints, and success criteria
- Proposes real alternatives with trade-offs and a recommendation
- Groups large integrations into four or five file-coherent milestones
- Records persistent session invariants and focused gates in one plan document

**`stoudemire`** executes the plan:

- Assigns one retained worker session to each milestone, not one agent per task
- Runs focused gates, then one risk-scaled reviewer per milestone
- Returns findings in one consolidated correction round; re-reviews only unresolved security or correctness issues
- Budgets roughly 12–20 total subagent dispatches for large integrations, including resumed correction turns
- Executes continuously without check-ins between milestones

Flow: `nash` → you approve the plan → `stoudemire` → you review the diff and commit

Skills: `skills/nash/SKILL.md` · `skills/stoudemire/SKILL.md`

---

### `chuck` — clarify and implement

For simpler work: clear scope, handful of files, with optional quick clarification before diving in.

- Asks 1-3 targeted questions (or skips if clear)
- Frames the approved scope as one coherent milestone
- Assigns one owner for all related work and one risk-scaled reviewer after focused gates
- Returns review findings in one consolidated correction round
- Carries persistent session invariants into every prompt
- Escalates multi-milestone integrations to nash instead of splitting work per agent

Flow: `chuck` → one milestone owner → one reviewer → you review the diff and commit

Skill: `skills/chuck/SKILL.md`

## Additional skills

| Skill                   | When to use                                                           |
| ----------------------- | --------------------------------------------------------------------- |
| `beautiful-code`        | Write, refactor, and review clear, maintainable code and architecture |
| `pragmatic-engineering` | Apply practical discovery, delivery, debugging, and team practices    |

## Installation

```bash
npx skills add mattdrose/skills --skill nash
npx skills add mattdrose/skills --skill stoudemire
npx skills add mattdrose/skills --skill chuck
npx skills add mattdrose/skills --skill beautiful-code
npx skills add mattdrose/skills --skill pragmatic-engineering
```
