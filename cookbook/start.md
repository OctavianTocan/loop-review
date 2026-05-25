# Start the Review Optimization Flow

## Context

You have an open PR (or a branch ready for PR) with feedback from one or more reviewers — Devin, Codex, Greptile, human comments, Cursor, Claude, or any combination.

You want a disciplined process that protects long-term code quality instead of optimizing for a clean reviewer dashboard.

## Pre-requisites

- GitHub CLI (`gh`) authenticated to the repo
- Clean or intentionally-dirty working tree (you will commit changes)
- Willingness to be honest about test coverage

This flow is **hard opinionated** on test coverage. If coverage is weak, the skill will stop you by default. You can override, but you must explicitly accept the risk.

## Steps

### 1. Confirm You're on the Right Branch

```bash
git branch --show-current
gh pr view --json number,headRefName,baseRefName,title,author
```

If no PR exists yet for this branch, create one first or use `/review` in "pre-PR" mode (the flow will guide you).

### 2. Run the Hard Pre-flight Coverage Check

**This step is mandatory and non-skippable by default.**

Read and execute: [cookbook/preflight.md](preflight.md)

The pre-flight launches a structured analysis (fast model recommended) that answers:

- Which new/modified logic paths have direct unit test coverage?
- Which user-facing flows touched by this change have E2E/integration coverage?
- What is the highest-risk area lacking test coverage?

**Green light:** Reasonable coverage on new public APIs and critical paths. Proceed.

**Yellow/Red light:** New complex logic with zero tests, user flows with no coverage, or high-risk areas (auth, payments, data boundaries, security) untested.

Default behavior: **Stop and ask the user** before proceeding with heavy iteration. The user must explicitly say "accept risk and continue" or "add tests first."

### 3. Check Rebase and Stack Hygiene

Read and execute guidance from: [cookbook/rebase-and-stack.md](rebase-and-stack.md)

This is **strongly recommended** if:
- You're in a Graphite stack (or any stacked PR workflow)
- Your branch is behind its base
- You see "dirty" diffs caused by base branch drift

Resolve conflicts yourself. Do not ask an agent to auto-resolve complex rebase conflicts — you will own the result.

### 4. Choose Your Path

You now have two strong options. Most high-quality outcomes come from doing **both**.

**A. Full feedback loop** — Process reviewer comments iteratively

Read: [cookbook/run-loop.md](run-loop.md)

Use this when you have substantial feedback from Devin, Codex, Greptile, or humans that needs to be triaged, addressed, and resolved thoughtfully.

**B. Direct high-value polish** — Skip straight to the most important step

Read: [cookbook/polish-pass.md](polish-pass.md)

Use this when:
- Feedback is minor or already addressed
- You want to ensure the code is *actually good*, not just reviewer-approved
- You're shipping something you care about

**Recommendation:** Run the loop first (if needed), then always run the polish pass. The polish pass catches what reviewers miss.

### 5. Final PR Hygiene

After changes:

- Commit with clear messages (consider conventional commits)
- Push
- Update the PR description to reflect the final state (use `/review update-pr` if available, or do it manually)
- Preserve any Graphite stack notation in the description if applicable

### 6. Verify Locally

Before asking for final review or merging:

```bash
# Run the project's full quality suite
npm run typecheck && npm run lint && npm test
# or equivalent for your package manager / framework
```

If the project has a `ci` or `check` script, run that.

## Exit Criteria for a Good Session

- All critical and high-severity reviewer feedback has been addressed or explicitly resolved with a comment
- Test coverage is strong on new logic and critical paths (or gaps are understood, documented, and risk-accepted)
- The self-review polish pass has been completed (docs, clarity, smells, quality gates)
- The PR description accurately reflects the final state
- Local quality gates pass

## Common Patterns

| Situation | Recommended Path |
|-----------|------------------|
| Small PR, minor feedback | Jump straight to `polish-pass.md` |
| Large change + heavy Devin/Codex feedback | Full loop → polish pass |
| Stacked PR (Graphite or similar) | `rebase-and-stack.md` first, then loop or polish |
| Pre-PR (branch not yet opened) | Run preflight + polish, *then* open PR |
| Reviewer score is green but you feel uneasy | Always run polish pass |

## Mindset

This skill exists to prevent the situation where:

> "The reviewer is happy, the dashboard is green, but the code is mediocre and the tests are thin."

You are the last engineer who will deeply understand this change before it ships. Act like it.

Proceed deliberately.
