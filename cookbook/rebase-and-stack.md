# Rebase and Stack Hygiene

## Why This Matters

Reviewing and iterating on a branch that is behind its base (especially in a stack) produces noisy diffs and fragile fixes.

## Recommended Process

1. Fetch the latest base branch.
2. Check if your current HEAD is a descendant of the latest base.
3. If not, rebase.
4. If you're in a Graphite stack, consider the impact on sibling PRs.

Resolve any conflicts yourself. Do not ask the agent to auto-resolve complex rebase conflicts.

See the full guidance in the main skill documentation once it is expanded.
