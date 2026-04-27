---
name: full
description: Use for complex, multi-subsystem changes — skips path selection and directly executes the SDD Full Path (isolate, specify, brainstorm, plan, execute with subagents, verify, archive)
---

# SDD — Full Path (Direct)

Directly enter the SDD Full Path. Use this when you already know the change is complex or spans multiple subsystems.

**Announce at start:** "I'm using sdd:full for this change."

## Setup — FIRST STEP

**REQUIRED:** Read `${CLAUDE_SKILL_DIR}/../_shared/common.md` now. Follow the **Language Selection** and **Prerequisites Check** sections in that file before continuing. If prerequisites fail, STOP — do not proceed. Apply all rules (iron laws, failure handling, etc.) from that file throughout this workflow.

## Resume or Start

After prerequisites pass, detect whether to resume or start fresh:

1. **Scan `openspec/changes/`** for non-archived change directories (exclude `archive/`) that have `sdd_path: full` in `.openspec.yaml` — these are "in-progress" full-path changes
2. **Scan git** for `sdd/*` branches (`git branch --list 'sdd/*'`) and worktrees (`git worktree list`) that have **no** corresponding `openspec/changes/<name>/` directory — these are "isolated but not yet specified" changes. Note: branches matching `sdd/*` that were not created by SDD may appear; always confirm with the user before resuming.
3. **Present findings:** list in-progress changes and isolated-but-not-specified changes separately; ask user which to resume, or start a new change
4. **If no changes at all** → proceed to start a new change

### Resuming an Active Change

**Determine current phase** by inspecting artifacts top-to-bottom — the first matching condition is where to resume:

| Condition | Resume at | Action |
|-----------|-----------|--------|
| Neither worktree nor branch exists for `sdd/<change-name>` | Isolate | Use `superpowers:using-git-worktrees` to create worktree with branch `sdd/<change-name>` |
| Branch `sdd/<change-name>` exists but no worktree | Isolate | Prompt: "Found branch but no worktree. Full Path requires worktree isolation. Creating a worktree from the existing branch." Use `superpowers:using-git-worktrees` to create a worktree based on the existing branch |
| `proposal.md` missing or empty | Specify | Run `/opsx:propose` in the isolated workspace |
| `design.md` empty/missing | Brainstorm | Run `superpowers:brainstorming` |
| `tasks.md` has only high-level tasks (no bite-sized steps) | Plan | Run `superpowers:writing-plans` |
| `tasks.md` has unchecked tasks (`- [ ]`) | Execute | Resume from first unchecked task |
| All tasks checked (`- [x]`) | Verify | Run verification |
| Verified but not archived | Finish + Archive | Run `superpowers:finishing-a-development-branch` then `/opsx:archive` |

**Workspace detection:** Check both `git worktree list` for a worktree on `sdd/<change-name>` AND `git branch --list 'sdd/<change-name>'` for a plain branch.

**Announce:** "Resuming change `<name>` at <phase name>."

Enter the worktree. Full Path always requires worktree isolation — a plain branch alone is never sufficient (see table above).

## New Change or Execute

**REQUIRED:** Read `${CLAUDE_SKILL_DIR}/../_shared/full-path.md` and follow from the current phase — for a new change start at **Isolate**; for a resumed change skip phases whose gates are already satisfied. After `/opsx:propose` creates `.openspec.yaml`, set `sdd_path: full`.
