---
name: go
description: Use when starting any feature, bugfix, refactor, or change that modifies code — auto-selects Fast or Full path based on complexity, orchestrates specification, quality-enforced implementation, and knowledge archival
---

# SDD — Go (Auto-Select Path)

Unify OpenSpec (WHAT to build) with Superpowers (HOW to build). This skill automatically determines whether to use the Fast Path or Full Path based on change complexity.

**Announce at start:** "I'm using sdd:go to orchestrate this change."

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

1. **Scan** `openspec/changes/` for non-archived change directories (exclude `archive/`)
2. **If active change(s) found** → list them, ask user which to resume (or start new)
3. **If no active change** → proceed to Path Selection for a new change

### Resuming an Active Change

**Determine path type:** Read `sdd_path` from the change's `.openspec.yaml`. If present, use it directly (`fast` or `full`). If absent (legacy change created before this field existed), fall back to checking `tasks.md` content — if it contains bite-sized steps with code blocks and verification commands, it was likely refined by `superpowers:writing-plans` (Full Path). Otherwise assume Fast Path. When ambiguous, ask the user, then persist the answer to `.openspec.yaml`.

**Determine current phase** by inspecting artifacts top-to-bottom — the first matching condition is where to resume:

| Condition | Resume at | Action |
|-----------|-----------|--------|
| `proposal.md` missing or empty | Specify | Run `/opsx:propose` |
| `design.md` empty/missing AND path is Full | Brainstorm | Run `superpowers:brainstorming` |
| `tasks.md` has only high-level tasks (no bite-sized steps) AND path is Full | Plan | Run `superpowers:writing-plans` |
| No workspace detected (no worktree or branch for `sdd/<change-name>`) | Isolate | Run `superpowers:using-git-worktrees` (Full Path) or create branch `sdd/<change-name>` (Fast Path) |
| `tasks.md` has unchecked tasks (`- [ ]`) | Execute | Resume from first unchecked task |
| All tasks checked (`- [x]`) | Verify | Run verification |
| Verified but not archived | Finish + Archive | Run `superpowers:finishing-a-development-branch` then `/opsx:archive` |

**Workspace detection:** Check both `git worktree list` for a worktree on `sdd/<change-name>` AND `git branch --list 'sdd/<change-name>'` for a plain branch. Either match means a workspace exists.

**Announce:** "Resuming change `<name>` at <phase name>."

Enter the worktree if one exists, or checkout the existing branch (`git checkout sdd/<change-name>`) if only a branch was found. Then continue from the detected phase — read `${CLAUDE_SKILL_DIR}/../_shared/fast-path.md` or `${CLAUDE_SKILL_DIR}/../_shared/full-path.md` as appropriate.

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

**Persist the choice:** After `/opsx:propose` creates the change directory, write `sdd_path: fast` or `sdd_path: full` into the change's `.openspec.yaml` based on the user's confirmed choice. This enables deterministic resume — no heuristic guessing needed.

## Execute Selected Path

- **Fast Path** → Read `${CLAUDE_SKILL_DIR}/../_shared/fast-path.md` and follow all phases.
- **Full Path** → Read `${CLAUDE_SKILL_DIR}/../_shared/full-path.md` and follow all phases.
