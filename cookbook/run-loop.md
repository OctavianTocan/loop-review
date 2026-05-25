# The Reviewer Feedback Loop

## Context

You have feedback from one or more AI reviewers on the current PR.

## Philosophy

- Read what the reviewers actually said.
- Make the code better.
- Resolve what you addressed. Push back (politely) on what doesn't make sense.
- Do not over-optimize for reviewer scores at the expense of code quality.

## Process

### 1. Gather Current Feedback

Collect the latest reviews and unresolved comments from the active reviewers on this PR (Devin, Codex, Greptile, etc.).

Focus on the most recent run on the current commit.

### 2. Triage the Feedback

For each piece of feedback, classify it:

- **Actionable code change** — Make the fix.
- **Informational / style preference** — Decide whether to apply it.
- **False positive or out of scope** — Note it and resolve the thread if appropriate.
- **Requires discussion** — Ask the user or leave a clarifying comment.

### 3. Make Changes Iteratively

Work through the feedback in reasonable batches.

Prefer small, focused commits when it makes sense.

After making changes, push so the reviewers can re-evaluate on the new commit.

### 4. Resolve Threads Thoughtfully

Only resolve threads for feedback you have genuinely addressed or have made a deliberate decision about.

Do not mass-resolve everything just to get a clean review score.

### 5. Know When to Stop

Good stopping signals:
- All critical and high-severity feedback has been addressed.
- Remaining comments are minor style preferences or false positives.
- Further iteration is yielding diminishing returns.
- You have completed (or are about to complete) the polish pass.

## Tips for Different Reviewers

- **Devin**: Often produces long, narrative sessions. Extract the concrete code suggestions.
- **Codex**: Frequently good at spotting edge cases and alternative approaches.
- **Greptile**: Strong on static analysis style issues. Treat its confidence score as one signal, not the only one.

The exact output format of the reviewer does not matter. Read the content, understand the intent, and improve the code.
