# review

Production-quality PR optimization loop for AI-augmented development.

**Install:**
```bash
npx skills add https://github.com/OctavianTocan/review
```

Then invoke with `/review` in your agent.

## What This Skill Does

`/review` helps you ship higher-quality pull requests by enforcing a disciplined optimization process:

1. **Hard pre-flight coverage gate** — Test coverage is non-negotiable. Weak coverage stops the process by default.
2. **Reviewer feedback consumption** — Works with output from *any* reviewer (Devin, Codex, Greptile, human comments, Cursor, Claude, etc.). No spawning of review sessions.
3. **Self-review polish pass** — The highest-leverage step. Many PRs pass reviewers but still contain mediocre code. This pass prevents that.
4. **Stacked PR / Graphite hygiene** — Strong awareness of rebase and stack discipline.

## Core Philosophy

- **Coverage over reviewer scores.** A clean reviewer dashboard with 40% test coverage is a liability, not a win.
- **Polish is more important than chasing scores.** Self-review discipline beats automated reviewer approval.
- **Reviewer-agnostic by design.** The skill consumes feedback; it does not produce reviews or spawn reviewer agents.
- **PR author workflow, not reviewer workflow.** This is for engineers iterating on their own PRs, not for autonomous review systems.

## Commands

| Command | Purpose |
|---------|---------|
| `/review` or `/review start` | Begin the full review optimization flow on current branch/PR |
| `/review preflight` | Run the hard coverage pre-flight gate only |
| `/review polish` | Execute the self-review polish pass (highest value step) |
| `/review loop` | Enter the reviewer feedback consumption loop |
| `/review stack` | Check and fix rebase/stack hygiene (Graphite-aware) |
| `/review help` | Show usage and subcommand reference |

## Quick Start

```bash
# On a branch with an open PR that has reviewer feedback:
/review

# Or jump straight to the most important step:
/review polish
```

## Structure

This skill follows the standard create-skill cookbook format:

- **SKILL.md** (this file) — Entry point, philosophy, command surface
- **cookbook/** — Detailed, imperative workflows for each sub-operation
- **references/** — Supporting models, patterns, and cross-cutting guidance
- **examples/** — Concrete, self-contained examples (no private references)

## Requirements

- GitHub CLI (`gh`) authenticated
- A clean working tree (or willingness to commit changes)
- Honest assessment of test coverage — this skill will challenge you

## License

MIT. Public skill. Use it. Improve it. Share it.
