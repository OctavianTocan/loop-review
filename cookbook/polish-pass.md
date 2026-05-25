# High-Signal Self-Review Polish Pass

## Context

This is the highest-value step in the entire process. Many PRs pass AI reviewers but still contain mediocre code. This pass exists to prevent that.

## When to Run

- After addressing reviewer feedback.
- As a standalone command (`/review polish`) on any PR you care about.

## The Polish Pass (Execute in Order)

### 1. Documentation

For every exported function, class, type, and interface that was added or significantly changed:
- Add high-quality TSDoc / JSDoc if missing.
- The doc should explain *why* it exists and any important invariants, not just repeat the signature.

### 2. Explain the "Why"

For every non-obvious change or decision in the diff, add an inline comment explaining the reasoning.

Good example:
```ts
// We chose a simple LRU cache here instead of a more sophisticated one
// because the working set is small (< 500 items) and we want to avoid
// the complexity and allocation overhead of a more advanced structure.
```

Bad example:
```ts
// cache the result
```

### 3. Code Smell Sweep

Actively look for and fix:
- Dead code and unreachable branches
- Duplication that can be reasonably consolidated
- Misleading or overly generic names
- Complex expressions that deserve extraction
- Leftover debug code, TODOs, or temporary workarounds
- Inconsistent patterns with the rest of the file/module

### 4. Run the Full Check Suite

Detect the package manager and run the project's quality gates:
- Type checking
- Linting
- Unit tests (at minimum the affected packages)
- Any other repo-specific checks (build, etc.)

Fix any failures before considering the polish pass complete.

### 5. Commit the Polish Work

```bash
git add -A
git commit -m "chore: self-review polish pass - docs, clarity, and quality gates"
git push
```

## Mindset

Treat this pass as if you are the last engineer who will ever look at this code before it ships.

The goal is not to make the reviewer happy. The goal is to make the code *good*.
