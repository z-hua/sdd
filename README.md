# SDD — Spec-Driven Development

A Claude Code plugin that orchestrates specification-driven development — unifying **OpenSpec** (WHAT to build) with **Superpowers** (HOW to build) into a single, quality-enforced workflow.

## Iron Laws

1. **Specs before code** — No implementation without a specification
2. **Tests before implementation** — TDD (RED-GREEN-REFACTOR) for every change
3. **Root cause before fix** — Systematic investigation, not guesswork
4. **Evidence before claims** — Run it, read it, prove it
5. **Archive preserves knowledge** — Every change leaves a trail

## Prerequisites

This plugin requires two companion systems:

- **[OpenSpec](https://github.com/allaboutai/OpenSpec)** — Spec management (proposals, delta specs, archival)
  ```bash
  npx openspec init
  openspec update
  ```

- **[Superpowers](https://github.com/obra/superpowers)** — Quality enforcement (TDD, debugging, verification, code review)
  ```
  /plugin install superpowers@claude-plugins-official
  ```

## Install

```
/plugin marketplace add z-hua/sdd
/plugin install sdd@sdd-dev
```

## Skills

| Skill | Command | When to use |
|-------|---------|-------------|
| **go** | `/sdd:go` | Auto-selects Fast or Full path based on change complexity |
| **fast** | `/sdd:fast` | Directly use Fast Path — small, well-understood changes (≤5 tasks) |
| **full** | `/sdd:full` | Directly use Full Path — complex, multi-subsystem changes |

Every skill starts with **language selection** (English, 简体中文, or other) and a **prerequisites check** before any work begins.

### sdd:go (Auto-Select)

Let the skill analyze the change and recommend the right path. It handles resume detection for both paths.

### sdd:fast (Direct Fast Path)

Phases: **Isolate → Specify → Execute (TDD) → Verify → Finish + Archive**

Skip path selection when you already know the change is small. Supports two execution modes:

- **Inline (default)** — direct TDD in the current session
- **Subagent-driven (optional)** — per-task delegation with review

### sdd:full (Direct Full Path)

Phases: **Isolate → Specify → Brainstorm → Plan → Execute (Subagents + TDD) → Verify → Finish + Archive**

Skip path selection when you already know the change is complex. Includes design brainstorming, detailed planning, and subagent-driven execution with two-stage review.

## How It Works

1. **Language selection** — Choose the language for all communication and generated documents
2. **Prerequisites check** — Verify OpenSpec and Superpowers are both available
3. **Choose change name** — Confirm a kebab-case change name for the OpenSpec change directory and workspace
4. **Isolate** — Create or enter a dedicated workspace (Full Path: worktree required; Fast Path: branch or worktree)
5. **Specify** — Create a change proposal with OpenSpec (`/opsx:propose`)
6. **Brainstorm** *(Full Path)* — Socratic design refinement via `superpowers:brainstorming`
7. **Plan** *(Full Path)* — Detailed, executable task breakdown via `superpowers:writing-plans`
8. **Execute** — TDD implementation via `superpowers:subagent-driven-development` or inline
9. **Verify** — Full test suite + evidence-based verification + spec compliance (`/opsx:verify`)
10. **Finish + Archive** — Integrate code + archive knowledge (`/opsx:archive`)

## Breaking Changes (v1.1)

- **`superpowers:executing-plans` removed.** Full Path now requires `superpowers:subagent-driven-development` for execution (no inline alternative). Fast Path offers both inline and subagent modes.
- **Fast→Full in-place switching removed.** Scope exceeding ≤5 tasks now produces a warning. If the user chooses to switch, all Fast Path artifacts are rolled back (git reset, change directory removed, workspace deleted) and the user restarts with `/sdd:full`. No artifact carry-over between paths.
- **Full Path requires worktree isolation.** Previously worktree was recommended; now it is mandatory for Full Path changes.

## License

MIT
