# Agent Skills

A personal collection of skills for AI coding agents. Skills are packaged instructions and references that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Skills

Inspired by [obra/superpowers](https://github.com/obra/superpowers). Two workflows for taking an idea to working code:

| Workflow | When to use | Skills |
|----------|-------------|--------|
| **nash → stoudemire** | Complex tasks that need a full plan | Brainstorm, plan, then execute with reviewed subagents |
| **chuck** | Simpler tasks that just need quick clarification | Clarify, approve, implement in one shot |

---

### `nash` + `stoudemire` — plan then execute

For complex work: multiple components, architectural decisions, or tasks that benefit from a written plan.

**`nash`** brainstorms and plans:
- Asks one question at a time to understand intent, constraints, and success criteria
- Proposes 2-3 approaches with trade-offs and a recommendation
- Writes a plan document to `plans/YYYY-MM-DD-<topic>-plan.md`

**`stoudemire`** executes the plan:
- Dispatches a fresh implementer subagent per task
- Two-stage review per task: spec compliance, then code quality
- Continuous execution — no check-ins between tasks

Flow: `nash` → you approve the plan → `stoudemire` → you review the diff and commit

Skills: `skills/nash/SKILL.md` · `skills/stoudemire/SKILL.md`

---

### `chuck` — clarify and implement

For simpler work: clear scope, handful of files, just needs a few questions before diving in.

- Asks 1-3 targeted questions (or skips if clear)
- Proposes one approach and asks for approval
- Dispatches a fresh implementer subagent
- No plan file, no multi-reviewer pipeline

Flow: `chuck` → you approve the approach → implementer runs → you review the diff and commit

Skill: `skills/chuck/SKILL.md`

## Installation

```bash
npx skills add mattdrose/skills --skill nash
npx skills add mattdrose/skills --skill stoudemire
npx skills add mattdrose/skills --skill chuck
```
