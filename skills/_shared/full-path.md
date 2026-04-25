# Full Path — Complex, Multi-File Changes

Use this path when the change involves new architecture, multiple subsystems, unclear requirements, or >5 tasks.
This path adds brainstorming, detailed planning, and subagent-driven execution with two-stage review.

---

## Specify

Create the initial specification scaffold.

**REQUIRED:** Run `/opsx:propose` with the change description.

- OpenSpec creates: `proposal.md`, `specs/*.md` (delta specs), `design.md`, `tasks.md`
- In Full Path, `design.md` and `tasks.md` are initial scaffolds — Brainstorm and Plan phases will refine and replace them

**IMPORTANT:** `/opsx:propose` may output a generic next-step message (e.g., "run /opsx:apply"). **Ignore it.** In Full Path, the next phase is always **Brainstorm**, not apply. After propose completes, announce: "Specify phase complete. Next: Brainstorm — refining the design through structured discussion."

**Output:** Change directory with proposal and specs as definitive artifacts, design and tasks as starting drafts.

---

## Brainstorm

**Arriving from a Fast Path scope escalation?** The existing `proposal.md` and `specs/*.md` carry over — proceed directly into brainstorming without re-running `/opsx:propose`.

Refine the proposal through structured design discussion.

**REQUIRED SUB-SKILL:** Use `superpowers:brainstorming`

Feed the OpenSpec artifacts as starting context:

- Read `proposal.md` — understand the WHY and WHAT
- Read `specs/*.md` — understand the delta requirements
- Read `design.md` — understand the initial HOW

The brainstorming skill will:

1. Ask clarifying questions (one at a time) to surface blind spots
2. Propose 2-3 approaches with trade-offs
3. Present design in sections for approval
4. Write a validated design document

**Output path override:** Write design output to `openspec/changes/<change-name>/design.md` — NOT to `docs/superpowers/specs/`. This replaces the initial scaffold with the refined version.

**HARD GATE:** Design must be approved by user before proceeding. No code until design is signed off.

---

## Plan

Refine the task list into a detailed, executable implementation plan.

**REQUIRED SUB-SKILL:** Use `superpowers:writing-plans`

**Input:**

- OpenSpec's `tasks.md` — high-level task breakdown
- `design.md` — technical approach and decisions
- `specs/*.md` — detailed requirements for each component

**Output:** Bite-sized implementation plan where each task has:

- Exact file paths (create/modify/test)
- Complete code blocks (no placeholders, no "TBD", no "similar to Task N")
- Verification commands with expected output
- Each step should be atomic and independently verifiable

**Output path override:** Write the detailed plan to `openspec/changes/<change-name>/tasks.md` — NOT to `docs/superpowers/plans/`. This replaces the initial scaffold with the detailed version.

**Self-review checklist:**

- Every spec requirement maps to at least one task
- No placeholders or incomplete code blocks
- Type consistency across tasks
- TDD steps included (failing test before implementation)

**HARD GATE:** Plan must pass self-review AND user approval before execution begins.

> **Execution choice:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task.

---

## Isolate

**REQUIRED SUB-SKILL:** Use `superpowers:using-git-worktrees`

- Creates isolated worktree with new branch
- Auto-detects and runs project setup
- Verifies clean baseline (all tests pass, 0 failures)

**Gate:** Isolated workspace ready with passing tests.

---

## Execute

Execute the plan with quality enforcement at every task.

### Recommended: Subagent-Driven Development

**REQUIRED SUB-SKILL:** Use `superpowers:subagent-driven-development`

Tasks are executed sequentially — each subagent completes and its changes are committed before the next is dispatched. This avoids merge conflicts and ensures each subagent sees the up-to-date working tree.

Per task:

1. **Dispatch implementer subagent** — fresh context, no pollution from prior tasks. Subagents must read the current working tree to discover artifacts produced by prior tasks; do not rely solely on the plan's code blocks, as they were written against the pre-implementation codebase.
2. **Implementer implements with TDD** — follows `superpowers:test-driven-development`
3. **Spec compliance review** — subagent verifies code matches OpenSpec's delta specs exactly (nothing more, nothing less)
4. **Code quality review** — subagent checks patterns, tests, maintainability
5. **Fix issues** → re-review until both stages pass
6. **Mark task complete** in tasks.md

### Alternative: Inline Execution

**REQUIRED SUB-SKILL:** Use `superpowers:executing-plans`

- Single-session execution with checkpoints
- Follow plan steps exactly, verify each
- No subagent isolation (higher context pollution risk)

### During Execution

**If a bug is encountered:**

- **REQUIRED SUB-SKILL:** Use `superpowers:systematic-debugging`
- Four-phase investigation: reproduce → hypothesize → validate → fix
- Write a failing test that captures the bug BEFORE fixing it

**If implementation diverges from spec:**

- STOP implementation
- Update the spec to reflect the new understanding
- Then continue — specs are the source of truth, they must stay accurate

---

## Verify + Finish

After all tasks are complete, continue to the shared Verify and Finish+Archive phases.

**Read the `shared-phases.md` file in this same directory** and follow both phases (Verify → Finish + Archive) to complete the change.
