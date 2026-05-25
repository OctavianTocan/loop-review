# Start the Review Optimization Flow

## Context

You have an open PR with feedback from one or more AI reviewers (Devin, Codex, Greptile, etc.). You want a disciplined process that protects quality.

## Pre-requisites

This flow is **hard opinionated** on test coverage. If coverage is weak, the skill will stop you.

## Steps

### 1. Identify the PR and Branch

Confirm you are on the correct branch for the PR you want to improve.

```bash
gh pr view --json number,headRefName,baseRefName
```

### 2. Run the Hard Pre-flight Coverage Check

**This step is mandatory and non-skippable by default.**

Invoke the coverage pre-flight:

Read [cookbook/preflight.md](preflight.md) and execute its process.

Only continue if the pre-flight passes or you have explicitly overridden it after understanding the risk.

### 3. Check Stack / Rebase Hygiene

Read [cookbook/rebase-and-stack.md](rebase-and-stack.md).

Strongly recommended on stacked PRs.

### 4. Enter the Main Loop or Polish Mode

You now have two strong paths:

- **Full loop**: Process reviewer feedback iteratively → [cookbook/run-loop.md](run-loop.md)
- **Direct high-value polish**: Skip straight to the most important step → [cookbook/polish-pass.md](polish-pass.md)

Most high-quality outcomes come from doing **both** (loop first, then polish).

### 5. Final PR Hygiene

After changes, run [cookbook/update-pr.md](update-pr.md) to refresh the description and preserve any stack notation.

## Exit Criteria for a Good Session

- All critical reviewer feedback addressed or explicitly resolved
- Test coverage is strong (or the gaps are understood and documented)
- The self-review polish pass has been completed
- The PR description accurately reflects the final state

## Common Patterns

- Quick polish on a small PR: Go straight to `polish-pass.md`
- Heavy Devin/Codex feedback on a large change: Use the full loop + polish
- Stacked PR work: Always do the rebase/stack check first

Proceed deliberately. This skill exists to prevent "looks good to the reviewer but the code is messy" situations.
