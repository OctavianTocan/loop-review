# Graphite and Stacked PR Patterns

## Purpose

This reference documents common patterns, pitfalls, and hygiene practices for working with stacked PRs — primarily via Graphite, but the principles apply to any stacked workflow (git-branchless, manual stacking, etc.).

## Core Concepts

### What Is a Stack?

A stack is a linear sequence of branches where each branch's base is the branch below it, not `main`:

```
main
  └── feat/base-change (PR #123)
        └── feat/builds-on-123 (PR #124)
              └── feat/builds-on-124 (PR #125)
```

Each PR in the stack depends on the one below it. When you merge a PR, its children are automatically rebased onto the new base.

### Why Stacks Exist

- **Incremental review:** Smaller, focused PRs are easier to review than one massive diff.
- **Parallel work:** Teammates can review and merge lower PRs while you continue working on higher ones.
- **Reduced blast radius:** A bug in PR #125 doesn't block the merge of PR #123.

### Tradeoffs

- **Complexity:** You must maintain the stack. Rebase mistakes cascade.
- **Review overhead:** More PRs = more context switching for reviewers.
- **Merge order rigidity:** You generally must merge from the bottom up.

## Graphite Workflow Patterns

### Creating a New Stack

```bash
# Start from main
git checkout main
git pull

# Create the first PR in the stack
gt branch create feat/payment-validation
# ... make changes, commit ...
gt submit

# Create the second PR on top
gt branch create feat/payment-ui --parent feat/payment-validation
# ... make changes, commit ...
gt submit
```

### Viewing the Stack

```bash
gt stack
gt status
```

Example output:
```
📚 feat/payment-ui (current)
   └── feat/payment-validation
         └── main
```

### Making Changes Mid-Stack

If you need to change a lower PR in the stack:

```bash
# Checkout the PR you want to modify
gt branch checkout feat/payment-validation

# Make your changes
# ... edit, commit ...

# Restack everything above it
gt stack fix

# Force-push the updated stack
gt stack submit --force
```

### Inserting a New PR in the Middle

```bash
# Checkout the PR that should come *after* the new one
gt branch checkout feat/payment-ui

# Create a new branch "below" it (inserts in the stack)
gt branch create feat/payment-logging --insert

# Make changes, commit, submit
gt submit
```

### Handling Main Moving Under the Stack

When `main` receives new commits while you're mid-stack:

```bash
# From any branch in the stack
gt repo sync

# This fetches latest main and restacks your entire stack
# If conflicts arise, resolve them bottom-up
```

### Force-Pushing a Stack

```bash
gt stack submit --force
```

This updates all PRs in the stack with new commit SHAs. Use after restacking.

**Always use `--force` with Graphite stacks.** Regular `git push --force-with-lease` will not update the PR metadata correctly.

## PR Description Stack Notation

Graphite automatically manages a "Stack" section in PR descriptions:

```markdown
## Stack

- #123 feat/payment-validation (this PR)
  - #124 feat/payment-ui
```

**Do not manually edit this section.** Graphite regenerates it on every `gt submit`.

If you manually edit it, Graphite will overwrite your changes on the next submit, and you may create merge conflicts in the description.

## Common Pitfalls

### Pitfall 1: Rebasing a Middle PR Without Restacking Children

**Symptom:** Children show as "behind" or have merge conflicts after you rebase a parent.

**Fix:** Always use `gt stack fix` or restack from the bottom of the stack upward. Never rebase a single middle PR in isolation.

### Pitfall 2: Using `git push --force` Instead of `gt stack submit --force`

**Symptom:** PRs in the stack show old commit SHAs. The GitHub UI shows "out of date" even after you pushed.

**Fix:** Use `gt stack submit --force` for any force-push in a Graphite stack. This updates PR metadata correctly.

### Pitfall 3: Asking an Agent to "Auto-Resolve" Rebase Conflicts

**Symptom:** Subtle bugs introduced during conflict resolution. You don't understand why the code looks the way it does.

**Fix:** Resolve conflicts yourself. You own the result. Auto-resolution is fast but dangerous for anything non-trivial.

### Pitfall 4: Ignoring Dirty Diffs

**Symptom:** Reviewers (AI or human) comment on code you didn't change. The diff is full of noise from base branch drift.

**Fix:** Rebase before asking for review. A clean diff is a courtesy to your reviewers.

### Pitfall 5: Manually Editing Stack Notation in PR Descriptions

**Symptom:** Graphite overwrites your edits. You create merge conflicts in the description.

**Fix:** Let Graphite manage stack notation. If the stack has genuinely changed, run `gt stack submit --force` to regenerate it.

### Pitfall 6: Merging Out of Order

**Symptom:** You merge a high PR before its dependencies. The stack breaks. CI fails on dependent PRs.

**Fix:** Merge from the bottom up. Graphite will prompt you if you try to merge out of order.

## Hygiene Checklist

Before asking for review on any PR in a stack:

- [ ] `gt stack fix` (or equivalent) has been run
- [ ] No PR in the stack shows "behind base" or merge conflicts
- [ ] `git diff main...HEAD` (or `gh pr diff`) is clean — no noise from base drift
- [ ] All commits are focused and well-described
- [ ] Stack notation in the PR description is correct (auto-managed by Graphite)

## When to *Not* Use a Stack

Stacks add overhead. Don't use them for:

- Trivial changes (< 50 lines, no dependencies)
- Urgent hotfixes (merge fast, stack later if needed)
- Experimental spikes where you expect to throw away the work
- Changes that are genuinely atomic and benefit from being reviewed together

## Tooling Alternatives

| Tool | Command | Notes |
|------|---------|-------|
| Graphite | `gt stack fix` + `gt stack submit --force` | Most popular, excellent UX |
| git-branchless | `git move` + `git submit` | Powerful, steeper learning curve |
| Manual (git) | `git rebase` + careful force-pushing | Error-prone, not recommended for >2 PR stacks |

Pick one and be consistent within a project.

## Further Reading

- Graphite documentation: https://graphite.dev/docs
- "Stacked PRs" on GitHub blog (general concepts)
- Your project's CONTRIBUTING.md (may have stack-specific guidelines)
