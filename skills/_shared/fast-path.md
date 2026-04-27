# Fast Path — Small, Well-Defined Changes

Use this path when the change is well-understood and has ≤5 tasks.

---

## Isolate

Never work directly on main/master.

Confirm or collect a kebab-case `change-name` first so resume detection can find it later. Then ask the user which isolation method to use:

- **Branch (default):** Create a branch named `sdd/<change-name>` and check it out
- **Worktree (optional):** Use `superpowers:using-git-worktrees` to create an isolated worktree with branch `sdd/<change-name>`

Verify clean baseline (all tests pass) before starting implementation.

**Gate:** Working on a dedicated `sdd/<change-name>` workspace (branch or worktree) before running `/opsx:propose`.

---

## Specify

Create the specification inside the isolated workspace before writing any code.

**REQUIRED:** Run `/opsx:propose` with the change description.

- OpenSpec creates: `proposal.md`, `specs/*.md`, `design.md`, `tasks.md`

**Gate:** Spec exists and tasks are enumerated before proceeding.

### Scope Check

Count the tasks in `tasks.md` immediately after `/opsx:propose` generates it. If >5 tasks are discovered, warn the user: "This change has ~N tasks, which exceeds the Fast Path threshold (≤5). Recommend switching to `sdd:full` — Fast Path skips brainstorming and detailed planning, which increases the risk of integration issues and rework for changes of this size."

**If the user chooses to switch to Full Path:**

Fast Path artifacts are not compatible with Full Path (no brainstorming or detailed planning was done), so a clean restart is required:

1. Roll back all changes: `git reset --hard` to the commit before isolation began
2. Remove the OpenSpec change directory: `rm -rf openspec/changes/<change-name>/`
3. Return to the main branch: if in a worktree, exit it first; otherwise `git checkout main` (or `master`)
4. Remove the workspace: delete the `sdd/<change-name>` branch (`git branch -D`) or worktree (`git worktree remove`)
5. Tell the user: "Workspace cleaned up. Run `/sdd:full` to restart this change on the Full Path."
6. **END. Do NOT continue.**

**If the user chooses to continue on Fast Path:** Record `sdd_scope_override: true` in `.openspec.yaml` to document the decision, then proceed.

---

## Execute

Work through tasks.md one task at a time. Check `.openspec.yaml` for `sdd_execution_mode`. If present, use it. If absent, ask the user which execution mode to use and persist the choice as `sdd_execution_mode: inline` or `sdd_execution_mode: subagent` in `.openspec.yaml`:

### Mode A — Inline Execution (default)

**REQUIRED SUB-SKILL:** `superpowers:test-driven-development`

For each task:

1. **RED** — Write a failing test that defines the expected behavior
2. **Run test** — Verify it fails for the RIGHT reason
3. **GREEN** — Write the minimum code to make the test pass
4. **Run test** — Verify it passes
5. **REFACTOR** — Clean up while tests stay green
6. **Commit** — One commit per task with descriptive message
7. **Mark complete** — Check off the task in tasks.md

### Mode B — Subagent-Driven Execution (optional)

**REQUIRED SUB-SKILL:** `superpowers:subagent-driven-development`

Tasks are executed sequentially — each subagent completes and its changes are committed before the next is dispatched.

Per task:

1. **Dispatch implementer subagent** — fresh context, no pollution from prior tasks. Subagents must read the current working tree to discover artifacts produced by prior tasks; do not rely solely on the plan's code blocks.
2. **Implementer implements with TDD** — follows `superpowers:test-driven-development`
3. **Review** — subagent verifies code matches spec and checks quality
4. **Fix issues** → re-review until passing
5. **Mark task complete** in tasks.md

### During Execution (both modes)

**If a bug is encountered or a test won't pass:**

- **REQUIRED SUB-SKILL:** Use `superpowers:systematic-debugging`
- Four-phase investigation: reproduce → hypothesize → validate → fix
- Do NOT guess at fixes. Investigate root cause first.
- Write a failing test that captures the bug BEFORE fixing it

**If scope creep is detected** (task requires changes not in the spec):

- Stop and update the spec first
- Then continue implementation

**If task count grows significantly** (new tasks added during execution push total well beyond 5):

- STOP and warn the user: "Task count has grown to ~N, well beyond the Fast Path threshold (≤5). Fast Path skips brainstorming and detailed planning — continuing without them increases the risk of integration issues and rework. Recommend switching to Full Path."
- **If the user chooses to switch:** Roll back all changes (`git reset --hard` to the commit before isolation), remove `openspec/changes/<change-name>/`, return to the main branch (exit worktree or `git checkout main`/`master`), then remove the `sdd/<change-name>` branch (`git branch -D`) or worktree (`git worktree remove`). Tell the user: "Workspace cleaned up. Run `/sdd:full` to restart this change on the Full Path." **END. Do NOT continue.**
- **If the user chooses to continue on Fast Path:** Record `sdd_scope_override: true` in `.openspec.yaml` to document the decision, then proceed.

---

## Verify + Finish

After all tasks are complete, continue to the shared Verify and Finish+Archive phases.

**Read the `shared-phases.md` file in this same directory** and follow both phases (Verify → Finish + Archive) to complete the change.
