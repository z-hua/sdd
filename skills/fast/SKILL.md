---
name: fast
description: Use for well-understood, small changes (≤5 tasks) — skips path selection and directly executes the SDD Fast Path (isolate, specify, TDD execute, verify, archive)
---

# SDD — Fast Path (Direct)

Directly enter the SDD Fast Path. Use this when you already know the change is small and well-understood.

**Announce at start:** "I'm using sdd:fast for this change."

## Setup — FIRST STEP

**REQUIRED:** Read `${CLAUDE_SKILL_DIR}/../_shared/common.md` now. Follow the **Language Selection** and **Prerequisites Check** sections in that file before continuing. If prerequisites fail, STOP — do not proceed. Apply all rules (iron laws, failure handling, etc.) from that file throughout this workflow.

## Resume or Start

After prerequisites pass, detect whether to resume or start fresh:

1. **Scan `openspec/changes/`** for non-archived change directories (exclude `archive/`) that have `sdd_path: fast` in `.openspec.yaml` — these are "in-progress" fast-path changes
2. **Scan git** for `sdd/*` branches (`git branch --list 'sdd/*'`) and worktrees (`git worktree list`) that have **no** corresponding `openspec/changes/<name>/` directory — these are "isolated but not yet specified" changes. Note: branches matching `sdd/*` that were not created by SDD may appear; always confirm with the user before resuming.
3. **Present findings:** list in-progress changes and isolated-but-not-specified changes separately; ask user which to resume, or start a new change
4. **If no changes at all** → proceed to start a new change

### Resuming an Active Change

**Determine current phase** by inspecting artifacts top-to-bottom — the first matching condition is where to resume:

| Condition | Resume at | Action |
|-----------|-----------|--------|
| No workspace detected (no branch or worktree for `sdd/<change-name>`) | Isolate | Create branch or worktree for `sdd/<change-name>` (ask user) |
| `proposal.md` missing or empty | Specify | Run `/opsx:propose` in the isolated workspace |
| `tasks.md` has unchecked tasks (`- [ ]`) | Execute | Resume from first unchecked task |
| All tasks checked (`- [x]`) | Verify | Run verification |
| Verified but not archived | Finish + Archive | Run `superpowers:finishing-a-development-branch` then `/opsx:archive` |

**Workspace detection:** Check both `git worktree list` for a worktree on `sdd/<change-name>` AND `git branch --list 'sdd/<change-name>'` for a plain branch. Either match means a workspace exists.

**Announce:** "Resuming change `<name>` at <phase name>."

Enter the worktree if one exists, or checkout the existing branch if only a branch was found.

## New Change or Execute

**REQUIRED:** Read `${CLAUDE_SKILL_DIR}/../_shared/fast-path.md` and follow from the current phase — for a new change start at **Isolate**; for a resumed change skip phases whose gates are already satisfied. After `/opsx:propose` creates `.openspec.yaml`, set `sdd_path: fast`.
