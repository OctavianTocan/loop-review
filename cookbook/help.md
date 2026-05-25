# Help and Command Reference

## Context

You want to understand what `/review` can do, or you need a quick reference for a specific subcommand.

## Available Commands

| Command | Alias | Description |
|---------|-------|-------------|
| `/review` | `/review start` | Begin the full review optimization flow on the current branch/PR. Recommended entry point. |
| `/review preflight` | — | Run the hard coverage pre-flight gate only. Non-negotiable gate by default. |
| `/review polish` | — | Execute the self-review polish pass. Highest-leverage step. Always recommended. |
| `/review loop` | — | Enter the reviewer feedback consumption loop. Process Devin/Codex/Greptile/human feedback. |
| `/review stack` | — | Check and fix rebase/stack hygiene. Graphite-aware. |
| `/review update-pr` | — | Update the PR description after changes. |
| `/review help` | `/review --help` | Show this reference. |

## Quick Decision Guide

| I want to... | Run this |
|--------------|----------|
| Start a disciplined review optimization session | `/review` |
| Check if my coverage is strong enough to proceed | `/review preflight` |
| Make the code excellent (docs, clarity, smells, gates) | `/review polish` |
| Work through a pile of reviewer feedback | `/review loop` |
| Fix a dirty diff or broken Graphite stack | `/review stack` |
| Update the PR description after changes | `/review update-pr` |
| See all commands | `/review help` |

## Typical Workflows

### "I have a PR with feedback and want to ship it right"

```bash
/review
# → preflight (mandatory)
# → stack check (if applicable)
# → loop (address feedback)
# → polish (highest value step)
# → update-pr
```

### "I just want to make sure the code is good, no heavy feedback to process"

```bash
/review polish
```

### "My coverage feels weak and I want to check before doing anything else"

```bash
/review preflight
```

### "I'm in a Graphite stack and my diff looks dirty"

```bash
/review stack
```

## Requirements

- GitHub CLI (`gh`) authenticated to the target repo
- A branch with (or ready for) an open PR
- Honest assessment of test coverage

## Philosophy (Reminder)

- **Coverage over reviewer scores.** A clean dashboard with weak coverage is a liability.
- **Polish > chasing metrics.** The self-review polish pass catches what reviewers miss.
- **Reviewer-agnostic.** Works with output from Devin, Codex, Greptile, humans, Cursor, Claude, etc.
- **No reviewer spawning.** This skill consumes feedback; it does not generate reviews or spawn reviewer agents.
- **Graphite-native.** Strong support for stacked PR workflows.

## Getting Help in Context

Each cookbook file contains detailed, imperative steps. Read the relevant cookbook for the subcommand you're using:

- [cookbook/start.md](start.md) — Full flow
- [cookbook/preflight.md](preflight.md) — Coverage gate
- [cookbook/polish-pass.md](polish-pass.md) — Self-review polish
- [cookbook/run-loop.md](run-loop.md) — Feedback consumption
- [cookbook/rebase-and-stack.md](rebase-and-stack.md) — Graphite hygiene
- [cookbook/update-pr.md](update-pr.md) — PR description updates

## Reporting Issues

This is a public skill: https://github.com/OctavianTocan/review

Issues and PRs welcome. If you find a bug or have a suggestion, open an issue with:

- The command you ran
- What you expected
- What actually happened
- Your environment (OS, agent, gh version)

## Contributing

See the README for contribution guidelines.
