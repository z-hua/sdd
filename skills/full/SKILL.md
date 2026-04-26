---
name: full
description: Use for complex, multi-subsystem changes — skips path selection and directly executes the SDD Full Path (isolate, specify, brainstorm, plan, execute with subagents, verify, archive)
---

# SDD — Full Path (Direct)

Directly enter the SDD Full Path. Use this when you already know the change is complex or spans multiple subsystems.

**Announce at start:** "I'm using sdd:full for this change."

## Language Selection

**This is the first step.** Ask the user which language to use for all subsequent communication and generated documents (proposals, specs, design docs, task lists, commit messages, etc.). Present the options:

- **English**
- **简体中文**
- **Other** (let user specify)

Use the chosen language consistently throughout the entire workflow. This includes announcements, questions, gate messages, failure reports, and all files written to `openspec/changes/`.

## Prerequisites Check — HARD GATE

**You MUST complete these checks before doing ANYTHING else after language selection. If either check fails, STOP IMMEDIATELY — do not read any other files, do not start any work.**

**Check 1 — OpenSpec:** Run `ls openspec/ 2>/dev/null && which openspec 2>/dev/null` to check for both the `openspec/` directory and the `openspec` CLI.

**Check 2 — Superpowers:** Look at your available skills list (the skills listed in the system prompt for this session). If any `superpowers:*` skills appear (e.g., `superpowers:using-superpowers`, `superpowers:test-driven-development`, etc.), Superpowers is available. If NO `superpowers:*` skills appear in your available skills list, Superpowers is NOT installed for this project. Do NOT attempt to verify by reading internal files like `installed_plugins.json` — the only reliable check is whether the skills are actually loaded in your current session.

**Results:**

- **Both available** → Announce "Full SDD mode — specs + quality enforcement." and continue.
- **OpenSpec missing** → STOP. Tell user: "SDD requires OpenSpec for spec management (proposals, delta specs, archival). Install with: `npx openspec init` then run `openspec update` to generate skills. See https://github.com/allaboutai/OpenSpec" — then **END. Do NOT continue.**
- **Superpowers missing** → STOP. Tell user: "SDD requires Superpowers for quality enforcement (TDD, systematic debugging, verification, code review). Install with: `/plugin install superpowers@claude-plugins-official` in Claude Code. See https://github.com/obra/superpowers" — then **END. Do NOT continue.**
- **Neither available** → STOP. Show both installation instructions above — then **END. Do NOT continue.**

## Load Shared Rules

**REQUIRED:** Read `${CLAUDE_SKILL_DIR}/../_shared/common.md` and apply all rules (prerequisites, iron laws, failure handling, etc.) before proceeding.

## Resume or Start

After prerequisites pass, detect whether to resume or start fresh:

1. **Scan** `openspec/changes/` for non-archived change directories (exclude `archive/`) that have `sdd_path: full` in `.openspec.yaml`
2. **If active full-path change(s) found** → list them, ask user which to resume (or start new)
3. **If no active full-path change** → proceed to start a new change
4. **If the user already isolated a change but `/opsx:propose` has not run yet** → ask for the `change-name`, enter the existing `sdd/<change-name>` workspace, and resume at **Specify**

### Resuming an Active Change

**Determine current phase** by inspecting artifacts top-to-bottom — the first matching condition is where to resume:

| Condition | Resume at | Action |
|-----------|-----------|--------|
| No workspace detected (no worktree or branch for `sdd/<change-name>`) | Isolate | Run `superpowers:using-git-worktrees` or recreate the `sdd/<change-name>` workspace |
| `proposal.md` missing or empty | Specify | Run `/opsx:propose` in the isolated workspace |
| `design.md` empty/missing | Brainstorm | Run `superpowers:brainstorming` |
| `tasks.md` has only high-level tasks (no bite-sized steps) | Plan | Run `superpowers:writing-plans` |
| `tasks.md` has unchecked tasks (`- [ ]`) | Execute | Resume from first unchecked task |
| All tasks checked (`- [x]`) | Verify | Run verification |
| Verified but not archived | Finish + Archive | Run `superpowers:finishing-a-development-branch` then `/opsx:archive` |

**Workspace detection:** Check both `git worktree list` for a worktree on `sdd/<change-name>` AND `git branch --list 'sdd/<change-name>'` for a plain branch. Either match means a workspace exists.

**Announce:** "Resuming change `<name>` at <phase name>."

Enter the worktree if one exists, or checkout the existing branch if only a branch was found.

## New Change

For a new change:

1. Ask the user to provide or confirm a kebab-case `change-name`
2. Create and enter a dedicated branch or worktree named `sdd/<change-name>`
3. Run `/opsx:propose` inside that isolated workspace
4. After `/opsx:propose` creates `.openspec.yaml`, set `sdd_path: full`

## Execute

**REQUIRED:** Read `${CLAUDE_SKILL_DIR}/../_shared/full-path.md` and follow all phases (**Isolate → Specify → Brainstorm → Plan → Execute → Verify + Finish**).
