---
name: review
description: Production-quality PR optimization loop for AI-augmented development. Hard opinionated on test coverage, strong emphasis on self-review polish, and works with any AI reviewer (Devin, Codex, Greptile, Cursor, Claude, etc.). Use when you want to turn reviewer feedback into high-quality, well-tested, well-documented code.
---

# Review

A focused, opinionated skill for getting maximum value from AI code reviewers through disciplined iteration and rigorous self-review.

## Core Philosophy

- **Coverage over scores.** A 5/5 reviewer score with weak tests is a liability.
- **Polish > chasing metrics.** The self-review pass is the highest-leverage step.
- **Reviewer-agnostic.** Consumes output from Devin, Codex, Greptile, human comments, etc. Does not spawn reviewer sessions.
- **Stack-aware.** Strong support for Graphite and stacked PR workflows.

## Commands

| Command                  | Purpose |
|--------------------------|---------|
| `/review`                | Start the full guided optimization flow |
| `/review preflight`      | Run the hard coverage gate only |
| `/review polish`         | Execute the high-signal self-review polish pass |
| `/review loop`           | Enter reviewer feedback consumption loop |
| `/review stack`          | Check rebase and Graphite stack hygiene |
| `/review help`           | Show detailed command reference |

## Quick Start

```bash
# On a branch with an open PR
/review
```

The skill will walk you through coverage validation, feedback processing, fixes, and the critical polish pass.

## Structure

This skill follows the standard cookbook-based format:

- `SKILL.md` — This file (lean router)
- `cookbook/` — Detailed imperative workflows
- `references/` — Models, patterns, and guidance
- `examples/` — Realistic, self-contained public examples

## Requirements

- `gh` (GitHub CLI) authenticated
- Willingness to be honest about test coverage

MIT licensed. Public skill.
