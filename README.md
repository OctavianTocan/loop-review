# loop-review

Production-quality PR optimization loop for AI-augmented development.

Works with **any** reviewer output (Devin, Codex, Greptile, human comments, Cursor, Claude, etc.).

**Hard opinionated** on test coverage pre-flight gates and self-review polish discipline.

## Install

```bash
npx skills add https://github.com/OctavianTocan/loop-review
```

Then use `/loop-review` (or any subcommand) in your agent.

## Philosophy

| Principle | Why It Matters |
|-----------|----------------|
| **Coverage over scores** | A PR with 100% reviewer approval and 30% test coverage is a future incident |
| **Polish > chasing reviewer metrics** | The self-review polish pass catches what automated reviewers miss |
| **Reviewer-agnostic** | Feedback format is irrelevant. Intent and actionability matter |
| **PR author workflow** | This skill helps you iterate on *your own* PRs. It does not spawn reviewer agents |

## Core Commands

```
/loop-review           Start the full optimization flow (recommended)
 /loop-review preflight Run coverage gate only (hard stop on weak coverage)
/review polish       Execute the highest-value self-review pass
/review loop         Process reviewer feedback iteratively
/review stack        Graphite / stacked PR rebase hygiene check
```

## Typical Flow

1. Open a PR with AI or human reviewer feedback
2. Run `/review`
3. Pass the **mandatory coverage pre-flight** (or explicitly accept risk)
4. Work through feedback via the loop (or jump to polish)
5. Execute the **self-review polish pass** — docs, clarity, smells, quality gates
6. Commit, push, update PR description

## What Makes This Different

- **No subagent spawning for reviews.** This skill consumes reviewer output; it does not generate reviews.
- **Non-negotiable coverage gate.** Most review tools optimize for green dashboards. This one protects long-term code health.
- **Graphite-native.** Explicit support for stacked PR workflows and rebase discipline.
- **Cookbook-driven.** Every major operation has a detailed, imperative guide you can follow step-by-step.

## Repository Structure

```
SKILL.md                 Entry point and command surface
cookbook/
  start.md               Full flow entry point
  preflight.md           Hard coverage gate (opinionated)
  polish-pass.md         Self-review quality pass (highest leverage)
  run-loop.md            Reviewer feedback consumption loop
  rebase-and-stack.md    Graphite / stacked PR hygiene
references/
  unified-feedback-model.md   How feedback from any source is normalized
  graphite-patterns.md        Stacked PR workflows and pitfalls
  reviewer-patterns.md        How to read output from different reviewers
examples/
  *.md                   Self-contained, public examples
```

## Contributing

Issues and PRs welcome at https://github.com/OctavianTocan/review

## License

MIT
