---
name: git-town
description: Use this skill whenever the user asks about Git Town, git-town commands, stacked branches, branch lineage, syncing Git branches, proposing or shipping branches with Git Town, configuring git-town.toml, troubleshooting interrupted Git Town operations, or deciding which Git Town workflow/command to use. This skill gives a practical overview of Git Town and points to the bundled docs for exact command options and deeper context.
---

# Git Town workflow assistant

Use this skill to help users understand and operate [Git Town](https://www.git-town.com/), a high-level CLI that automates common Git workflows. The bundled `docs/` directory is the source of truth. Do not edit files in `docs/`; read the relevant docs and cite the path(s) you used when giving detailed guidance.

## What Git Town is

Git Town adds workflow-oriented commands on top of Git. It helps create feature branches, keep branches and stacks synced, propose pull requests/merge requests, ship completed work, manage branch lineage, and recover from interrupted operations. It is compatible with common workflows such as GitHub Flow, Git Flow, GitLab Flow, trunk-based development, monorepos, and stacked changes.

Read `docs/introduction.md` when the user needs a conceptual overview or is new to Git Town.

## Start with the docs map

When you need exact details, first consult `docs/SUMMARY.md`, then read the specific page(s) below. Prefer targeted reading over loading many docs at once.

### Core references

| User need | Start here |
| --- | --- |
| What Git Town does | `docs/introduction.md` |
| Install Git Town | `docs/install.md` |
| Initial setup / interactive config | `docs/configuration.md`, `docs/commands/init.md` |
| Full command list | `docs/all-commands.md` |
| Basic workflow | `docs/basic-commands.md` |
| Branch types and sync behavior | `docs/branch-types.md` |
| Stacked branches / stacked changes | `docs/stacked-changes.md` |
| Config file format | `docs/configuration-file.md` |
| Preferences and precedence | `docs/preferences.md` |
| Handling errors or conflicts | `docs/error-commands.md`, then `docs/commands/continue.md`, `docs/commands/skip.md`, `docs/commands/undo.md`, or `docs/commands/status.md` |
| Practical recipes | `docs/how-tos.md` and the linked `docs/how-to/*.md` files |

### Command details

For exact syntax, flags, configuration interactions, and see-also links, read the command's page in `docs/commands/` before recommending nontrivial command usage.

Common commands:

- `git town hack` — create/start a feature branch. See `docs/commands/hack.md`.
- `git town sync` — update the current branch, stack, or all local branches. See `docs/commands/sync.md`.
- `git town switch` — visually switch branches. See `docs/commands/switch.md`.
- `git town propose` — create/open a pull request or merge request. See `docs/commands/propose.md`.
- `git town ship` — deliver a completed branch, mostly for edge cases/offline/stacked workflows. See `docs/commands/ship.md`.
- `git town branch` — display branch hierarchy and types. See `docs/commands/branch.md`.
- `git town append`, `prepend`, `set-parent`, `detach`, `swap`, `up`, `down`, `walk` — manage/navigate stacks. See their pages under `docs/commands/`.
- `git town continue`, `skip`, `undo`, `status` — recover from interrupted operations. See `docs/error-commands.md` and their command pages.

## Practical mental model

Explain Git Town in terms of branch roles and lineage:

- **Main/perennial branches** are long-lived roots like `main`, `master`, `develop`, `staging`, or `production`.
- **Feature branches** are normal work branches. They sync with their parent and tracking branch and can be proposed or shipped.
- **Prototype branches** are local/private work-in-progress branches that sync from their parent but do not push unless they already have a tracking branch.
- **Parked branches** are intentionally inactive and do not sync automatically.
- **Contribution/observed branches** are somebody else's branches, with different push/pull behavior.
- **Branch lineage** is the parent/child relationship Git Town uses to keep stacked branches organized and to target PRs correctly.

Use `docs/branch-types.md` for branch-type specifics and `docs/preferences/parent.md` or `docs/commands/set-parent.md` when lineage is the focus.

## Typical workflow to recommend

For normal feature work:

```sh
git town hack my-feature
# make changes and commit
git town sync
git town propose
# after review/merge through the forge, or for appropriate edge cases:
git town sync --all
```

Notes:

- `git town hack` starts a new feature branch, usually from the main branch.
- `git town sync` should be run frequently; it can be undone with `git town undo` if a completed sync produced the wrong result.
- `git town propose` opens a PR/MR flow and pre-populates branch/target information.
- Most users merge in their forge UI or merge queue. `git town ship` is mainly for offline mode, specific ship strategies, or stacked-change workflows; read `docs/commands/ship.md` before recommending it.

## Stacked changes workflow

For dependent changes that should be reviewed separately, explain stacks as a chain of branches where each child builds on its parent.

Common stack commands:

```sh
git town hack 1-refactor       # start stack from main
git town append 2-rename-foo   # add child branch
git town append 3-feature      # add another child
git town sync --stack          # update the current stack
git town propose --stack       # propose all branches in the stack
```

Guidance:

- Use stacks only when changes depend on each other; independent work can be separate top-level branches.
- Keep one focused change per branch.
- Ship/merge the oldest branch first.
- Run `git town sync --stack` or `git town sync --all` regularly to avoid drift and reduce conflicts.
- For phantom conflicts, consult `docs/stacked-changes.md` and the sync/ship strategy docs.

## Configuration guidance

If the repo has `git-town.toml`, `.git-town.toml`, or `.git-branches.toml`, treat it as team-wide configuration. If no config exists or behavior is unclear, recommend:

```sh
git town init
```

Configuration precedence from highest to lower priority:

1. CLI flags
2. Git metadata
3. configuration file
4. environment variables
5. system-specific settings
6. defaults

Use these references:

- `docs/configuration.md` for setup and forge API access.
- `docs/configuration-file.md` for TOML shape and default settings.
- `docs/preferences.md` for preference precedence.
- Individual `docs/preferences/*.md` files for exact preference meanings.

## Safety and troubleshooting

Before running Git Town commands for a user, inspect the repository state with normal Git commands such as `git status` and, if helpful, `git branch`/`git log --oneline --graph --decorate --all`. Explain what you are about to do.

Use safety-oriented options and recovery commands:

- Use `--dry-run` when the user wants to preview what Git Town will do.
- Use `git town status` to inspect an interrupted Git Town command.
- After conflicts or required manual intervention, resolve the underlying Git issue, then use `git town continue`.
- Use `git town skip` only when syncing multiple branches and the user wants to skip the problematic branch.
- Use `git town undo` to abort an interrupted operation or undo the last completed Git Town command.

Avoid destructive or history-changing actions without explicit confirmation, especially deleting branches, shipping branches, changing branch parents, swapping stack order, force-like operations, or changing shared configuration.

## Response style

When answering Git Town questions:

1. Give the shortest useful answer first: the command(s) or decision.
2. Add a brief explanation of why that command fits the branch/stack state.
3. Mention important safety/recovery notes when relevant.
4. Cite the docs path(s) for exact details, especially for flags or advanced workflows.

For command recommendations, prefer copy-pastable shell snippets and include placeholders clearly, for example:

```sh
git town hack <branch-name>
git town sync --stack
git town propose --stack
```
