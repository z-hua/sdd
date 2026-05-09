# SDD — Spec-Driven Development

A Claude Code plugin that orchestrates specification-driven development — unifying **OpenSpec** (WHAT to build) with **Superpowers** (HOW to build) into a single, quality-enforced workflow.

## Iron Laws

1. **Specs before code** — No implementation without a specification first.
2. **Tests before implementation** — RED-GREEN-REFACTOR for every change.
3. **Root cause before fix** — No fixes without systematic investigation.
4. **Evidence before claims** — Run it, read it, prove it. No "should work."
5. **Archive preserves knowledge** — Every completed change archives full context.

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
/plugin install sdd@z-hua
```

## Skills

| Skill | Command | When to use |
|-------|---------|-------------|
| **go** | `/sdd:go` | Complex, multi-subsystem changes (isolate, specify, brainstorm, plan, execute with subagents, verify, archive) |

Every skill starts with **language selection** (English, 简体中文, or other) and a **prerequisites check** before any work begins.

### sdd:go

Phases: **Isolate → Specify → Brainstorm → Plan → Execute (Subagents + TDD) → Verify → Finish + Archive**

Use for complex changes spanning multiple subsystems. Includes design brainstorming, detailed planning, and subagent-driven execution with two-stage review.

## How It Works

1. **Language selection** — Choose the language for all communication and generated documents
2. **Prerequisites check** — Verify OpenSpec and Superpowers are both available
3. **Choose change name** — Confirm a kebab-case change name for the OpenSpec change directory and workspace
4. **Isolate** — Create or enter a dedicated worktree (required)
5. **Specify** — Create a change proposal with OpenSpec (`/opsx:propose`)
6. **Brainstorm** — Socratic design refinement via `superpowers:brainstorming`
7. **Plan** — Detailed, executable task breakdown via `superpowers:writing-plans`
8. **Execute** — TDD implementation via `superpowers:subagent-driven-development`
9. **Verify** — Full test suite + evidence-based verification + spec compliance (`/opsx:verify`)
10. **Finish + Archive** — Integrate code + archive knowledge (`/opsx:archive`)

## Breaking Changes (v1.1)

- **`superpowers:executing-plans` removed.** Now requires `superpowers:subagent-driven-development` for execution.
- **Worktree isolation is mandatory.** Previously worktree was recommended; now it is required.

## License

MIT
