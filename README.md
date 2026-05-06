# Agent Skills

A personal collection of skills for AI coding agents. Skills are packaged instructions and references that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Nash and Stoudemire

Two skills, inspired by [obra/superpowers](https://github.com/obra/superpowers), that pair to take an idea from rough thought to working code.

### `nash` — brainstorm + plan

Use **before** any creative work.

- Asks one question at a time to understand intent, constraints, and success criteria
- Proposes 2-3 approaches with trade-offs and a recommendation
- Presents the design in sections for incremental approval
- Writes a **single** plan document to `plans/YYYY-MM-DD-<topic>-plan.md` (no separate spec)
- The plan contains both the design and bite-sized TDD tasks ready for execution

Skill: `skills/nash/SKILL.md`

### `stoudemire` — subagent-driven execution

Use **after** nash has produced an approved plan.

- Reads the plan once, extracts all tasks
- Dispatches a fresh implementer subagent per task
- Two-stage review per task: spec compliance, then code quality
- Continuous execution — no "should I continue?" check-ins between tasks
- When all tasks are done, hands the working tree back to you with a `git status` / `git diff --stat` summary so you can review the full diff and commit it however you like

Skill: `skills/stoudemire/SKILL.md`

### Typical flow

1. `nash` → produces `plans/2026-05-06-my-feature-plan.md`
2. You review and approve the plan
3. `stoudemire` → executes every task, leaves all changes uncommitted
4. You review the full diff and commit

## Installation

```bash
npx skills add mattdrose/skills
```
