# SDD — Shared Rules and Context

This file is read by all three SDD skills (go, fast, full). It contains the shared prerequisites, conventions, and enforcement rules.

## Prerequisites Check

**Note:** The primary prerequisites gate is enforced inline in each SKILL.md (go, fast, full) before this file is read. The checks below serve as a reference and fallback.

**Check 1 — OpenSpec:** Run `ls openspec/ 2>/dev/null && which openspec 2>/dev/null` to check for both the `openspec/` directory and the `openspec` CLI.

**Check 2 — Superpowers:** Look at your available skills list (the skills listed in the system prompt for this session). If any `superpowers:*` skills appear (e.g., `superpowers:using-superpowers`, `superpowers:test-driven-development`, etc.), Superpowers is available. If NO `superpowers:*` skills appear in your available skills list, Superpowers is NOT installed for this project. Do NOT attempt to verify by reading internal files like `installed_plugins.json` — the only reliable check is whether the skills are actually loaded in your current session.

**HARD GATE — Both systems are required.** OpenSpec manages the WHAT (specs, proposals, archival); Superpowers enforces the HOW (TDD, reviews, verification). Without both, the spec-then-implement-then-verify loop cannot close — running with only one would silently skip critical quality gates.

- **Both available** → "Full SDD mode — specs + quality enforcement." Proceed.
- **OpenSpec missing** → STOP. Tell user: "SDD requires OpenSpec for spec management (proposals, delta specs, archival). Install with: `npx openspec init` then run `openspec update` to generate skills. See https://github.com/allaboutai/OpenSpec" Do NOT proceed.
- **Superpowers missing** → STOP. Tell user: "SDD requires Superpowers for quality enforcement (TDD, systematic debugging, verification, code review). Install with: `/plugin install superpowers@claude-plugins-official` in Claude Code. See https://github.com/obra/superpowers" Do NOT proceed.
- **Neither available** → STOP. Tell user both installation instructions above. Do NOT proceed.

## Branch Naming Convention

When creating a branch or worktree for a change, use the pattern `sdd/<change-name>` where `<change-name>` matches the OpenSpec change directory name exactly. This enables resume detection: scan `git worktree list` and `git branch` for `sdd/<change-name>` to find an existing workspace for a change.

## Unified Output Directory

**All artifacts live in the OpenSpec change directory.** No documents should be written to `docs/superpowers/specs/` or `docs/superpowers/plans/`. When invoking Superpowers skills, override their default output paths:

```
openspec/changes/<change-name>/
├── proposal.md          ← Specify: /opsx:propose generates
├── specs/*.md            ← Specify: /opsx:propose generates (delta specs)
├── design.md             ← Brainstorm: superpowers:brainstorming writes HERE (not docs/superpowers/specs/)
├── tasks.md              ← Plan: superpowers:writing-plans writes HERE (not docs/superpowers/plans/)
└── .openspec.yaml        ← OpenSpec metadata (includes sdd_path: fast|full)
```

**Override rules for Superpowers skills:**

- **General rule:** All Superpowers skill file outputs must be redirected to `openspec/changes/<change-name>/`. Do NOT create `docs/superpowers/` directories — the OpenSpec change directory is the single source of truth.
- `superpowers:brainstorming` → write design output to `openspec/changes/<change-name>/design.md`
- `superpowers:writing-plans` → write plan output to `openspec/changes/<change-name>/tasks.md`
- `superpowers:verification-before-completion` → any verification reports go to the change directory
- `superpowers:systematic-debugging` → any investigation notes go to the change directory

This ensures every artifact for a change is in one place, and `/opsx:archive` captures the complete context when archiving.

## Iron Laws

These are non-negotiable. Both paths enforce all five:

1. **Specs before code** — No implementation without a specification first. The spec is the contract.
2. **Tests before implementation** — No production code without a failing test first (RED-GREEN-REFACTOR). **Exempt:** changes that have no testable behavior — pure configuration, static assets, CSS/styling, documentation, dependency version bumps. When exempt, state the reason explicitly.
3. **Root cause before fix** — No fixes without systematic investigation. Random changes create new bugs.
4. **Evidence before claims** — No "should work" or "probably passes." Run the command, read the output, show the proof.
5. **Archive preserves knowledge** — Every completed change merges specs and preserves full context for future reference.

If you catch yourself thinking "just this once" or "it's too simple to need specs" — **STOP. The iron laws apply especially when you think they don't.** The TDD exemption above is narrow — if the change affects runtime behavior in any way, TDD is required.

## Failure Handling

**Verification repeatedly fails (`/opsx:verify` reports ERROR issues):**
- Re-read the failing spec requirements carefully
- Fix implementation to match spec, or update spec if the requirement was wrong
- Re-run verification — max 3 attempts before escalating to user
- After 3 failures: present the full error context (which specs fail, what the code does vs. what the spec requires), then suggest options: (1) update the spec if the requirement was wrong, (2) revert to last passing state and re-approach — this means `git reset --hard` to the last per-task commit where all tests passed; spec changes made after that commit are also reverted, (3) manual investigation by the user

**Tests keep failing during execution:**
- **REQUIRED SUB-SKILL:** Use `superpowers:systematic-debugging`
- If root cause is outside the change scope, STOP and report to user
- Do NOT work around test failures with skips or mocks

**User wants to abandon a change mid-way:**
- If in a worktree: use `superpowers:finishing-a-development-branch` → choose "Discard work"
- The OpenSpec change directory remains in `openspec/changes/` — user can delete manually or resume later
- Do NOT run `/opsx:archive` on incomplete work

**Fast Path scope exceeds limits (>5 tasks discovered mid-execution):**
- STOP and announce: "Scope exceeds Fast Path limits. Recommending switch to Full Path."
- Update `sdd_path` to `full` in the change's `.openspec.yaml`
- All existing artifacts in `openspec/changes/<name>/` carry over — nothing is lost
- Resume at the appropriate Full Path phase (usually Brainstorm or Plan)
- Read the `full-path.md` file in this same `_shared/` directory and continue from there

**OpenSpec command failures (`/opsx:*`):**
- `/opsx:propose` fails → Common causes: invalid change name (must be kebab-case), schema not found, directory already exists. Check the error message, fix, and retry.
- `/opsx:verify` reports issues → Issues are ERROR (blocking) or WARNING (informative). Fix ERROR-level issues before proceeding; WARNINGs are advisory.
- `/opsx:archive` fails validation → Delta specs must be valid before archiving. Fix spec validation errors first. For infrastructure/tooling changes with no spec changes, suggest `--skip-specs` flag to user.
- Any command gives an unexpected error → Show full error output to user. Do not retry blindly.

## Red Flags

**STOP immediately if:**

- Writing code before the spec/proposal exists
- Skipping TDD because "it's just a small change" (unless the change qualifies for a TDD exemption — see Iron Laws)
- Claiming tests pass without running them since the last code change
- Proceeding after a verification failure
- Skipping archive because "the change is done"

## Integration

**OpenSpec commands** (invoked via `/opsx:*` slash commands — these are generated by `openspec init`/`openspec update`):

- `/opsx:propose` — Create change proposal with specs, design, and tasks
- `/opsx:verify` — Validate implementation against specifications
- `/opsx:archive` — Merge delta specs and archive completed change

**Superpowers skills** (invoked via `superpowers:*` skill references):

- `superpowers:using-git-worktrees` — Isolated development workspace
- `superpowers:test-driven-development` — RED-GREEN-REFACTOR cycle
- `superpowers:verification-before-completion` — Evidence-based validation
- `superpowers:finishing-a-development-branch` — Merge/PR workflow
- `superpowers:executing-plans` — Inline single-session execution with checkpoints

**Full path additionally requires:**

- `superpowers:brainstorming` — Socratic design refinement
- `superpowers:writing-plans` — Detailed implementation planning
- `superpowers:subagent-driven-development` — Per-task delegation with two-stage review

**When debugging:**

- `superpowers:systematic-debugging` — Four-phase root cause investigation
