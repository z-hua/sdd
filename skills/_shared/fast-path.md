# Fast Path — Small, Well-Defined Changes

Use this path when the change is well-understood and has ≤5 tasks.

**If scope exceeds limits at any point** (>5 tasks discovered):
- STOP and announce the scope change to user
- Update `sdd_path` to `full` in the change's `.openspec.yaml`
- All artifacts in `openspec/changes/<name>/` carry over — nothing is lost
- Read the `full-path.md` file in this same directory and resume at the Brainstorm or Plan phase as appropriate

---

## Specify

Create the specification before writing any code.

**REQUIRED:** Run `/opsx:propose` with the change description.

- OpenSpec creates: `proposal.md`, `specs/*.md`, `design.md`, `tasks.md`

**Gate:** Spec exists and tasks are enumerated before proceeding.

### Scope Check

Count the tasks in `tasks.md` immediately after `/opsx:propose` generates it:

- **≤5 tasks** → continue on Fast Path
- **>5 tasks** → STOP and announce: "Scope exceeds Fast Path limits (~N tasks). Recommending switch to Full Path." Update `sdd_path` to `full` in `.openspec.yaml` before switching. All artifacts carry over. Read the `full-path.md` file in this same directory and resume at Brainstorm or Plan as appropriate.

---

## Isolate

Never work directly on main/master.

**RECOMMENDED:** Use `superpowers:using-git-worktrees` for full isolation with a worktree.

**Alternative:** Create a branch named `sdd/<change-name>` and work there directly. This is acceptable for Fast Path changes where worktree overhead is disproportionate to the change size.

- Either way, verify clean baseline (all tests pass) before starting

**Gate:** Working on a dedicated branch (worktree or regular) with passing baseline tests.

---

## Execute

Work through tasks.md one task at a time.

**REQUIRED SUB-SKILL:** `superpowers:test-driven-development`

For each task:

1. **RED** — Write a failing test that defines the expected behavior
2. **Run test** — Verify it fails for the RIGHT reason
3. **GREEN** — Write the minimum code to make the test pass
4. **Run test** — Verify it passes
5. **REFACTOR** — Clean up while tests stay green
6. **Commit** — One commit per task with descriptive message
7. **Mark complete** — Check off the task in tasks.md

**If a bug is encountered or a test won't pass:**

- **REQUIRED SUB-SKILL:** Use `superpowers:systematic-debugging`
- Four-phase investigation: reproduce → hypothesize → validate → fix
- Do NOT guess at fixes. Investigate root cause first.
- Write a failing test that captures the bug BEFORE fixing it

**If scope creep is detected** (task requires changes not in the spec):

- Stop and update the spec first
- Then continue implementation

---

## Verify + Finish

After all tasks are complete, continue to the shared Verify and Finish+Archive phases.

**Read the `shared-phases.md` file in this same directory** and follow both phases (Verify → Finish + Archive) to complete the change.
