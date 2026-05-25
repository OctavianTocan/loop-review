# Update PR Description

## Context

After making changes (from the feedback loop, polish pass, or rebase), the PR description should accurately reflect the final state.

This is especially important for stacked PRs where Graphite manages stack notation.

## When to Run

- After the polish pass
- After any significant iteration on feedback
- Before requesting final human review
- Before merging (if you want a clean historical record)

## Process

### 1. Read the Current Description

```bash
gh pr view <PR_NUMBER> --json body --jq '.body'
```

Or open the PR in the GitHub UI.

### 2. Identify What Needs Updating

Common updates:

- **New behavior:** If the final implementation differs from the original description, update the "What does this do?" section.
- **Reviewer decisions:** Document any significant pushbacks or alternative approaches taken (see `run-loop.md` step 6).
- **Test coverage:** If the pre-flight identified gaps and you addressed them, note the new coverage.
- **Breaking changes:** If scope changed, update any "Breaking Changes" section.
- **Graphite stack notation:** If Graphite manages it, ensure it's still correct (usually automatic, but verify).

### 3. Update the Description

You can edit directly in the GitHub UI, or use the CLI:

```bash
gh pr edit <PR_NUMBER> --body-file ./pr-description.md
```

Or:

```bash
gh pr edit <PR_NUMBER> --body "updated description here"
```

### 4. Preserve Graphite Stack Notation (If Applicable)

If your PR is part of a Graphite stack, the description will have a section like:

```markdown
## Stack

- #123 feat/base-change (this PR)
  - #124 feat/builds-on-123
```

**Do not manually edit this section.** Graphite maintains it automatically on `gt submit`.

If the stack has changed (you added/removed PRs), run:

```bash
gt stack submit --force
```

This will regenerate the stack notation correctly.

### 5. Verify the Update

```bash
gh pr view <PR_NUMBER>
```

Open the PR in the browser and confirm the description renders correctly (markdown, links, etc.).

## Recommended PR Description Template

If your project doesn't have a standard template, consider this structure:

```markdown
## What does this do?

[One-paragraph summary of the change and its purpose.]

## Why are we doing this?

[Context: what problem this solves, what the previous state was, why this approach.]

## How did we test this?

- Unit tests: [files or coverage notes]
- Integration/E2E: [flows exercised]
- Manual: [what was manually verified]

## Reviewer feedback decisions

[Any non-obvious decisions made in response to reviewer feedback, with links to threads if relevant.]

## Breaking changes

[List any breaking changes, or "None."]

## Stack

[Graphite-managed — do not edit manually]
```

## Anti-Patterns

| Anti-Pattern | Consequence |
|--------------|-------------|
| "I'll update the description later" | Later never comes. The PR becomes a mystery to future readers. |
| Leaving the original "What does this do?" unchanged when implementation diverged | Future readers (and you) will be confused about what actually shipped. |
| Manually editing Graphite stack notation | Graphite overwrites it on next submit; you create merge conflicts in the description. |
| Deleting the entire description and replacing it with a wall of text | Loses historical context. Update incrementally. |

## After Updating

If the PR has active reviewers (AI or human), add a comment:

```
Updated PR description to reflect final implementation and reviewer feedback decisions. No code changes in this update.
```

This prevents reviewers from re-reviewing a description-only change.
