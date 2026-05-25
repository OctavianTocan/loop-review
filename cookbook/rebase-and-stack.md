# Rebase and Stack Hygiene (Graphite-Aware)

## Why This Matters

Reviewing and iterating on a branch that is behind its base (especially in a stack) produces noisy diffs and fragile fixes.

A "dirty" diff caused by base branch drift makes it hard for reviewers (AI or human) to see what you actually changed.

In stacked PR workflows (Graphite, git-branchless, or manual stacking), rebasing incorrectly can break sibling PRs and create merge conflicts that cascade through the entire stack.

## When to Run

- Automatically as step 3 of `/review` (start) if you're in a stacked workflow
- Manually via `/review stack` before any significant iteration
- When you see unexpected changes in `git diff main...HEAD` that aren't from your work
- When a reviewer comments on code you didn't touch

## Detection

Check if your branch is behind its base:

```bash
git fetch origin
git log --oneline HEAD..origin/$(git rev-parse --abbrev-ref HEAD@{upstream} 2>/dev/null || echo 'main')
```

Or more simply:

```bash
git status
```

If you see "Your branch is behind 'origin/main' by X commits", you have a hygiene problem.

For Graphite users:

```bash
gt status
gt stack
```

If any PR in your stack shows "behind base" or has merge conflicts, fix it.

## Recommended Process

### 1. Fetch the Latest Base

```bash
git fetch origin
# or for Graphite
gt repo sync
```

### 2. Check If You're a Descendant

```bash
git merge-base --is-ancestor origin/main HEAD && echo "clean" || echo "needs rebase"
```

If "clean", you're good. Skip to step 5.

### 3. Rebase (Single Branch)

```bash
git rebase origin/main
```

Resolve conflicts yourself. Do not ask an agent to auto-resolve complex conflicts — you will own the result, and auto-resolution often creates subtle bugs.

After resolving:

```bash
git rebase --continue
git push --force-with-lease
```

### 4. Rebase (Graphite Stack)

If you're in a Graphite stack:

```bash
# From the bottom of the stack upward
gt stack fix

# Or for a specific PR in the stack
gt branch checkout <branch>
gt branch restack
```

Graphite's `restack` command is designed to handle the cascading rebases correctly. Use it.

After restacking:

```bash
gt stack submit --force
```

This updates all PRs in the stack with the new commit SHAs.

### 5. Verify the Diff Is Clean

After rebasing:

```bash
git diff main...HEAD
```

Or:

```bash
gh pr diff
```

The diff should only contain changes from *your* work, not noise from base branch drift.

If you see unexpected changes, you rebased incorrectly or have uncommitted work that conflicted.

### 6. Communicate to Reviewers (If Needed)

If you rebased a PR that already had reviews:

- Add a comment on the PR:
  ```
  Rebased on latest main (commit abc1234). No functional changes — only conflict resolution on src/foo.ts:45.
  ```

- If the rebase was clean (no conflicts), no comment is needed. Reviewers will see the new commit.

## Graphite-Specific Patterns

### Creating a New PR in a Stack

```bash
# Create the branch
gt branch create feat/new-thing

# Make changes, commit, submit
gt submit
```

### Inserting a PR in the Middle of a Stack

```bash
# Checkout the PR that should come *after* the new one
gt branch checkout <existing-pr-branch>

# Create a new branch "below" it
gt branch create feat/intermediate-thing --insert

# Make changes, commit, submit
gt submit
```

### Handling a Base Branch Change Mid-Stack

If `main` moves significantly while you're mid-stack:

1. Restack from the bottom of your stack upward.
2. Use `gt stack fix` to detect and resolve issues.
3. Force-push the updated stack with `gt stack submit --force`.
4. Add a comment on the top PR noting the restack.

### PR Description Stack Notation

Graphite automatically manages stack notation in PR descriptions. Do not manually edit the "Stack" section — let Graphite maintain it.

If you need to manually update (rare):

```markdown
## Stack

- #123 feat/base-change (this PR)
  - #124 feat/builds-on-123
    - #125 feat/builds-on-124
```

## Common Pitfalls

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Rebasing a middle PR without restacking children | Children point to old SHAs, stack breaks | Use `gt stack fix` or restack from bottom up |
| Force-pushing without `--force-with-lease` | Overwrites teammate's work if they pushed to your branch | Always use `--force-with-lease` |
| Asking an agent to "auto-resolve conflicts" | Subtle bugs introduced, you don't understand the resolution | Resolve conflicts yourself |
| Ignoring "dirty diff" warnings | Reviewers see noise, miss real changes | Rebase before asking for review |
| Manually editing Graphite's stack notation | Graphite gets confused on next submit | Let Graphite manage it |

## Pre-PR Hygiene (Branch Not Yet Opened)

Even before opening a PR, keep your branch rebased on the latest base:

```bash
git fetch origin
git rebase origin/main
```

This makes the eventual PR diff clean from day one.

## After Rebasing

Always run the coverage pre-flight again if the rebase touched any logic (rare, but possible with conflict resolution).

Then continue with your normal flow (loop or polish).

## Tooling Recommendations

| Tool | Command | When to Use |
|------|---------|-------------|
| Git native | `git rebase origin/main` | Simple, single-PR workflows |
| Graphite | `gt stack fix` + `gt stack submit --force` | Any stacked PR workflow |
| git-branchless | `git move` + `git submit` | If you use branchless instead of Graphite |

Pick one and be consistent. Mixing tools on the same stack leads to confusion.

## Summary

Rebase hygiene is not optional when you're in a stacked PR workflow.

Dirty diffs and broken stacks waste reviewer time and introduce subtle bugs.

Do the work. Your future self (and your reviewers) will thank you.
