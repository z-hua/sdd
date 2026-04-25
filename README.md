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
/plugin install https://github.com/z-hua/sdd
```

## Skills

| Skill | Command | When to use |
|-------|---------|-------------|
| **go** | `/sdd:go` | Auto-selects Fast or Full path based on change complexity |
| **fast** | `/sdd:fast` | Directly use Fast Path — small, well-understood changes (≤5 tasks) |
| **full** | `/sdd:full` | Directly use Full Path — complex, multi-subsystem changes |

### sdd:go (Auto-Select)

Let the skill analyze the change and recommend the right path. It handles resume detection for both paths.

### sdd:fast (Direct Fast Path)

Phases: **Specify → Isolate → Execute (TDD) → Verify → Finish + Archive**

Skip path selection when you already know the change is small. Can escalate to Full Path mid-flight if scope grows.

### sdd:full (Direct Full Path)

Phases: **Specify → Brainstorm → Isolate → Plan → Execute (Subagents + TDD) → Verify → Finish + Archive**

Skip path selection when you already know the change is complex. Includes design brainstorming, detailed planning, and subagent-driven execution with two-stage review.

## How It Works

1. **Specify** — Create a change proposal with OpenSpec (`/opsx:propose`)
2. **Brainstorm** *(Full Path)* — Socratic design refinement via `superpowers:brainstorming`
3. **Isolate** — Work on a dedicated branch or git worktree
4. **Plan** *(Full Path)* — Detailed, executable task breakdown via `superpowers:writing-plans`
5. **Execute** — TDD implementation, one task at a time
6. **Verify** — Full test suite + evidence-based verification + spec compliance (`/opsx:verify`)
7. **Finish + Archive** — Integrate code + archive knowledge (`/opsx:archive`)

## License

MIT
