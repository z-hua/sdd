---
name: go
description: Use when starting any feature, bugfix, refactor, or change that modifies code — auto-selects Fast or Full path based on complexity, orchestrates specification, quality-enforced implementation, and knowledge archival
---

# SDD — Go (Auto-Select Path)

Unify OpenSpec (WHAT to build) with Superpowers (HOW to build). This skill automatically determines whether to use the Fast Path or Full Path based on change complexity.

**Announce at start:** "I'm using sdd:go to orchestrate this change."

## Setup — FIRST STEP

**REQUIRED:** Read `${CLAUDE_SKILL_DIR}/../_shared/common.md` now. Follow the **Language Selection** and **Prerequisites Check** sections in that file before continuing. If prerequisites fail, STOP — do not proceed. Apply all rules (iron laws, failure handling, etc.) from that file throughout this workflow.

## Resume or Start

After prerequisites pass, detect whether to resume or start fresh:

1. **Scan `openspec/changes/`** for non-archived change directories (exclude `archive/`) — these are "in-progress" changes
2. **Scan git** for `sdd/*` branches (`git branch --list 'sdd/*'`) and worktrees (`git worktree list`) that have **no** corresponding `openspec/changes/<name>/` directory — these are "isolated but not yet specified" changes. Note: branches matching `sdd/*` that were not created by SDD may appear; always confirm with the user before resuming.
3. **Present findings:** list in-progress changes and isolated-but-not-specified changes separately; ask user which to resume, or start a new change
4. **If no changes at all** → proceed to Path Selection for a new change

### Resuming an Active Change

**Determine path type:** Read `sdd_path` from the change's `.openspec.yaml`. If present, use it directly (`fast` or `full`). If absent (legacy change created before this field existed), fall back to checking `tasks.md` content — if it contains bite-sized steps with code blocks and verification commands, it was likely refined by `superpowers:writing-plans` (Full Path). Otherwise assume Fast Path. When ambiguous, ask the user, then persist the answer to `.openspec.yaml`.

**Determine current phase** by inspecting artifacts top-to-bottom — the first matching condition is where to resume:

| Condition | Resume at | Action |
|-----------|-----------|--------|
| Neither worktree nor branch exists for `sdd/<change-name>` | Isolate | Full Path: use `superpowers:using-git-worktrees` (required). Fast Path: branch or worktree (ask user) |
| Branch `sdd/<change-name>` exists but no worktree (Full Path) | Isolate | Prompt: "Found branch but no worktree. Full Path requires worktree isolation. Creating a worktree from the existing branch." Use `superpowers:using-git-worktrees` to create a worktree based on the existing branch |
| Branch `sdd/<change-name>` exists but no worktree (Fast Path) | — | Checkout the existing branch (`git checkout sdd/<change-name>`) and resume from the next matching condition below |
| `proposal.md` missing or empty | Specify | Run `/opsx:propose` in the isolated workspace |
| `design.md` empty/missing AND path is Full | Brainstorm | Run `superpowers:brainstorming` |
| `tasks.md` has only high-level tasks (no bite-sized steps) AND path is Full | Plan | Run `superpowers:writing-plans` |
| `tasks.md` has unchecked tasks (`- [ ]`) | Execute | Resume from first unchecked task |
| All tasks checked (`- [x]`) | Verify | Run verification |
| Verified but not archived | Finish + Archive | Run `superpowers:finishing-a-development-branch` then `/opsx:archive` |

**Workspace detection:** Check both `git worktree list` for a worktree on `sdd/<change-name>` AND `git branch --list 'sdd/<change-name>'` for a plain branch. Either match means a workspace exists.

**Announce:** "Resuming change `<name>` at <phase name>."

Enter the worktree if one exists. If only a branch is detected, follow the path-specific action in the table above. Then continue from the detected phase — read `${CLAUDE_SKILL_DIR}/../_shared/fast-path.md` or `${CLAUDE_SKILL_DIR}/../_shared/full-path.md` as appropriate.

## Path Selection (New Change)

| Condition | Path | Rationale |
|-----------|------|-----------|
| Change is well-understood AND ≤5 tasks | **Fast Path** | Minimal ceremony, direct execution |
| New architecture, multiple subsystems, or unclear requirements | **Full Path** | Needs brainstorming + detailed planning + subagent review |
| Borderline complexity but clear requirements | Either — **ask user** | User knows their codebase best |

**Examples to guide judgment:**

- Fast Path: rename a public API method, add a validation rule, fix an off-by-one bug with clear repro
- Full Path: add a new authentication provider, redesign the data pipeline, implement a feature where the user hasn't decided on the UX yet
- Signals that push toward Full Path: cross-module dependencies, no existing tests in the area, conflicting requirements, unfamiliar codebase

Present recommendation to user with a brief scope summary (e.g., "~3 tasks, focused change to one module" or "~8 tasks across 4 subsystems — recommending Full Path"). User can always override.

**Persist the choice:** After the user confirms the path, ask them to provide or confirm a kebab-case `change-name`. Isolate the workspace:

- **Full Path:** Use `superpowers:using-git-worktrees` to create a worktree with branch `sdd/<change-name>` (required)
- **Fast Path:** Ask user: branch (default) or worktree? Create the chosen workspace with branch `sdd/<change-name>`

Then run `/opsx:propose` inside that isolated workspace. After `.openspec.yaml` exists, write `sdd_path: fast` or `sdd_path: full` based on the user's confirmed choice. This keeps resume deterministic while ensuring all OpenSpec artifacts are created inside the isolated workspace.

## Execute Selected Path

- **Fast Path** → Read `${CLAUDE_SKILL_DIR}/../_shared/fast-path.md` and continue from the current phase — skip phases whose gates are already satisfied.
- **Full Path** → Read `${CLAUDE_SKILL_DIR}/../_shared/full-path.md` and continue from the current phase — skip phases whose gates are already satisfied.
