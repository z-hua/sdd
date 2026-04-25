# Shared Phases — Verify and Finish

Both Fast Path and Full Path converge here after their Execute phase completes.

---

## Verify

Two verification systems confirm the change is complete and correct.

**Step 1 — Run full test suite:**

- All tests must pass. No exceptions. No "known failures."

**Step 2 — Evidence-based verification:**

- **REQUIRED SUB-SKILL:** Use `superpowers:verification-before-completion`
- Run every verification command fresh — do not rely on previous runs
- Read actual output. No "should work", no "probably passes."
- Every claim must be backed by command output from after the last code change

**Step 3 — Spec compliance:**

- Run `/opsx:verify` on the change
- Check three dimensions:
  - **Completeness** — All ADDED/MODIFIED specs have corresponding implementation
  - **Correctness** — Implementation behavior matches spec description
  - **Coherence** — No contradictions between specs, design, and code
- Fix any ERROR issues before proceeding. WARNING issues are advisory — review them but they do not block progress.

**Gate:** Tests pass AND verification evidence shown AND spec compliance confirmed (no ERROR issues).

---

## Finish + Archive

Code integrates into the repository; knowledge integrates into the spec library.

**Step 1 — Integrate code:**

- **REQUIRED SUB-SKILL:** Use `superpowers:finishing-a-development-branch`
- Options: merge locally, create PR, keep branch, or discard
- Tests must pass before any integration option

**Step 2 — Archive knowledge:**

- Run `/opsx:archive` on the completed change
- Delta specs merge into the main spec repository (`openspec/specs/`)
- Full change context preserved in archive (`openspec/changes/archive/YYYY-MM-DD-<name>/`)
- Main specs are now the updated source of truth
- **No spec changes?** For infrastructure, tooling, or CI changes that don't modify any specs, suggest `--skip-specs` flag to the user (e.g., `/opsx:archive --skip-specs`)

**Done.** The change is implemented, verified, integrated, and its knowledge is preserved.
