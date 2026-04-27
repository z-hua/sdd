# SDD — Shared Rules and Context

This file is read at the start of every SDD skill, before any project files are touched.

## Language Selection

**This is the first step.** Ask the user which language to use for all subsequent communication and generated documents (proposals, specs, design docs, task lists, commit messages, etc.). Present the options:

- **English**
- **简体中文**
- **Other** (let user specify)

Use the chosen language consistently throughout the entire workflow. This includes announcements, questions, gate messages, failure reports, and all files written to `openspec/changes/`.

## Prerequisites Check — HARD GATE

**After language selection is complete, you MUST pass these checks before doing ANYTHING else. If either check fails, STOP IMMEDIATELY — do not read any other files, do not start any work.**

**Check 1 — OpenSpec:** Run two checks independently:

- **Directory:** `ls openspec/ 2>/dev/null` — does the `openspec/` directory exist?
- **CLI:** `which openspec 2>/dev/null` — is the `openspec` CLI on PATH?

Both are required. If only one is present, give a targeted message:

- Directory exists but CLI missing → "Found `openspec/` directory but `openspec` CLI is not in PATH. Run `npm install -g openspec` or check your PATH."
- CLI exists but directory missing → "Found `openspec` CLI but no `openspec/` directory. Run `npx openspec init` to initialize."
- Neither exists → Show the full installation instructions (see Results below).

**Check 2 — Superpowers:** Look at your available skills list (the skills listed in the system prompt for this session). If any `superpowers:*` skills appear (e.g., `superpowers:using-superpowers`, `superpowers:test-driven-development`, etc.), Superpowers is available. If NO `superpowers:*` skills appear in your available skills list, Superpowers is NOT installed for this project. Do NOT attempt to verify by reading internal files like `installed_plugins.json` — the only reliable check is whether the skills are actually loaded in your current session.

Both systems are required. OpenSpec manages the WHAT (specs, proposals, archival); Superpowers enforces the HOW (TDD, reviews, verification). Without both, the spec-then-implement-then-verify loop cannot close — running with only one would silently skip critical quality gates.

**Check 3 — OpenSpec skills:** Look at your available skills list for the three required OpenSpec skills: `opsx:propose`, `opsx:verify`, `opsx:archive`. All three must be present. If any are missing, tell the user which ones are missing and instruct: "Run `openspec update` to generate the missing OpenSpec skills." — then **END. Do NOT continue.**

**Results:**

- **All checks pass** → Announce "Full SDD mode — specs + quality enforcement." and continue.
- **OpenSpec missing (partially or fully)** → STOP. Show the targeted message from Check 1 above (directory-only, CLI-only, or neither). If neither exists, also show the full installation reference: "Install with: `npx openspec init` then run `openspec update` to generate skills. See https://github.com/allaboutai/OpenSpec" — then **END. Do NOT continue.**
- **OpenSpec skills missing** → STOP. List the missing skills (e.g., "`opsx:verify` not found"). Tell user: "Run `openspec update` to generate the missing OpenSpec skills." — then **END. Do NOT continue.**
- **Superpowers missing** → STOP. Tell user: "SDD requires Superpowers for quality enforcement (TDD, systematic debugging, verification, code review). Install with: `/plugin install superpowers@claude-plugins-official` in Claude Code. See https://github.com/obra/superpowers" — then **END. Do NOT continue.**
- **Multiple checks fail** → STOP. Show all relevant installation instructions above — then **END. Do NOT continue.**

## Branch Naming Convention

When creating a branch or worktree for a change, use the pattern `sdd/<change-name>` where `<change-name>` matches the OpenSpec change directory name exactly. This enables resume detection: scan `git worktree list` and `git branch` for `sdd/<change-name>` to find an existing workspace for a change.

Collect or confirm the kebab-case `change-name` before isolation so the workspace can be created first and `/opsx:propose` can later generate the matching OpenSpec change directory inside that isolated workspace.

## Unified Output Directory

**All artifacts live in the OpenSpec change directory.** No documents should be written to `docs/superpowers/specs/` or `docs/superpowers/plans/`. When invoking Superpowers skills, override their default output paths:

```
openspec/changes/<change-name>/
├── proposal.md          ← Specify: /opsx:propose generates
├── specs/*.md            ← Specify: /opsx:propose generates (delta specs)
├── design.md             ← Brainstorm: superpowers:brainstorming writes HERE (not docs/superpowers/specs/)
├── tasks.md              ← Plan: superpowers:writing-plans writes HERE (not docs/superpowers/plans/)
└── .openspec.yaml        ← OpenSpec metadata (sdd_path: fast|full, sdd_execution_mode: inline|subagent, sdd_scope_override: true if user overrode scope warning)
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

**Clarification:** Creating or entering an isolated workspace before writing the spec is allowed and encouraged. The prohibition is against writing implementation code before the spec/proposal exists, not against doing isolation setup first.

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
- Use `superpowers:finishing-a-development-branch` → choose "Discard work" (handles both branches and worktrees)
- The OpenSpec change directory remains in `openspec/changes/` — user can delete manually or resume later
- Do NOT run `/opsx:archive` on incomplete work

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

- `superpowers:using-git-worktrees` — Isolated development workspace (Full Path: required, Fast Path: optional)
- `superpowers:test-driven-development` — RED-GREEN-REFACTOR cycle
- `superpowers:subagent-driven-development` — Per-task delegation with two-stage review (Full Path: required, Fast Path: optional)
- `superpowers:verification-before-completion` — Evidence-based validation
- `superpowers:finishing-a-development-branch` — Merge/PR workflow

**Full Path additionally requires:**

- `superpowers:brainstorming` — Socratic design refinement
- `superpowers:writing-plans` — Detailed implementation planning

**When debugging:**

- `superpowers:systematic-debugging` — Four-phase root cause investigation
