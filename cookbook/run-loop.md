# The Reviewer Feedback Consumption Loop

## Context

You have feedback from one or more reviewers on the current PR — Devin, Codex, Greptile, human comments, Cursor, Claude, or any combination.

Your goal is to **improve the code**, not to optimize for a clean reviewer dashboard.

## Philosophy

- Read what the reviewers actually said.
- Make the code better.
- Resolve what you addressed. Push back (politely, with evidence) on what doesn't make sense.
- Do not over-optimize for reviewer scores at the expense of code quality, coverage, or clarity.

## Process

### 1. Gather Current Feedback

Collect the latest reviews and unresolved comments from all sources on this PR.

```bash
# GitHub-native reviews and comments
gh pr view <PR_NUMBER> --json reviews,comments

# Or use the GitHub UI to see the full thread
```

Focus on the **most recent run** on the **current commit**. Older feedback on previous commits is often stale.

If the feedback exists in a long Devin/Codex session transcript, extract the concrete code suggestions. Ignore narrative fluff.

### 2. Triage the Feedback

For each piece of feedback, classify it into one of four buckets:

| Bucket | Meaning | Action |
|--------|---------|--------|
| **Actionable code change** | Clear, specific improvement that makes the code better | Make the fix. Commit. Push. |
| **Informational / style preference** | Reviewer has an opinion, but it's not a clear improvement | Decide whether to apply. If you disagree, leave a polite note and resolve. |
| **False positive or out of scope** | Reviewer misunderstood the context, flagged something that doesn't apply, or is asking for something outside the PR scope | Note it. Resolve the thread with a brief explanation. |
| **Requires discussion** | You genuinely don't understand the concern, or it needs input from the user or another teammate | Ask the user or leave a clarifying comment on the thread. Do not guess. |

### 3. Make Changes Iteratively

Work through the feedback in reasonable batches:

- **Small batches (3-5 items):** Address, commit, push, let reviewers re-evaluate.
- **Larger batches:** Group related items, address them together, commit with a clear message.

Prefer small, focused commits when it makes sense:

```
fix: validate webhook signature before processing (addresses Codex #3)
refactor: extract payment retry logic (addresses Devin session 2)
test: add coverage for retry edge cases (addresses Greptile finding)
```

After making changes, push so the reviewers can re-evaluate on the new commit.

### 4. Resolve Threads Thoughtfully

Only resolve threads for feedback you have genuinely addressed or have made a deliberate decision about.

**Do not mass-resolve everything just to get a clean review score.** This is the fastest way to ship mediocre code.

Good resolution comment:
```
Fixed in commit abc1234. Added unit test for the null case and an integration test for the retry path.
```

Bad resolution comment:
```
Fixed.
```

If you're pushing back on a piece of feedback, be polite and evidence-based:
```
This suggestion would introduce an N+1 query pattern. The current approach batches the lookup (see src/lib/batch.ts:45). I've added a comment explaining the tradeoff.
```

### 5. Know When to Stop

Good stopping signals:

- All critical and high-severity feedback has been addressed.
- Remaining comments are minor style preferences or false positives.
- Further iteration is yielding diminishing returns (you're spending more time arguing with the reviewer than improving the code).
- You have completed (or are about to complete) the polish pass.

If you're in a loop of "reviewer suggests X → you implement X → reviewer suggests Y on the new code → repeat" with no clear end, step back and ask: "Is this feedback making the code better, or just different?"

### 6. Document Decisions in the PR Description

For any non-obvious decision (especially pushbacks), update the PR description:

```markdown
## Reviewer Feedback Decisions

- **Codex #4 (webhook retry):** Chose exponential backoff with max 3 retries instead of infinite retry. Rationale: prevents runaway costs on persistent failures. See `src/lib/retry.ts:22`.
- **Greptile finding (auth middleware):** This is intentional — the endpoint is public by design (marketing opt-in). Added explicit comment and test case.
```

This helps future reviewers (and future you) understand the context without digging through threads.

## Tips for Different Reviewers

### Devin

- Often produces long, narrative sessions with many suggestions.
- Extract the concrete code suggestions. Ignore the surrounding story unless it contains important context.
- Devin sometimes suggests large refactors that are out of scope. Be disciplined about scope.
- Good at spotting missing error handling and edge cases.

### Codex

- Frequently good at spotting edge cases and alternative approaches.
- Sometimes over-indexes on "clean code" abstractions that add complexity without clear benefit.
- Strong on security and input validation patterns.
- Treat confidence scores as one signal, not gospel.

### Greptile

- Strong on static analysis style issues (unused code, potential bugs, type safety).
- Can produce false positives on generated code or intentional patterns.
- Its confidence score is often well-calibrated for its domain (static analysis), but weak on architectural concerns.
- Excellent for catching "I didn't realize this was a problem" issues.

### Human Reviewers

- Read the actual words. Humans are better than AI at context and intent.
- If a human reviewer is confused, the code is probably unclear — don't just "fix the symptom," improve the clarity.
- Push back respectfully. Humans appreciate evidence and tradeoffs more than AI does.

### Multiple Reviewers (Mixed Signals)

When two reviewers contradict each other:

1. Understand both positions.
2. Make a deliberate choice (don't average them).
3. Document the decision in the PR description.
4. Resolve both threads with a reference to your decision.

Example:
```
Devin suggested approach A. Codex suggested approach B. I chose A because [reason]. See PR description for details.
```

## Anti-Patterns

| Anti-Pattern | Consequence |
|--------------|-------------|
| Mass-resolving threads to make the dashboard green | Ships mediocre code. Reviewers move on; you own the result. |
| Implementing every suggestion without thinking | Code becomes a Frankenstein of reviewer preferences, often inconsistent. |
| Arguing with every piece of feedback | Wastes time. Most feedback is at least directionally useful. |
| Ignoring coverage while chasing reviewer approval | Fastest way to introduce regressions. See `preflight.md`. |
| Not pushing between iterations | Reviewers re-evaluate on stale commits. You get duplicate feedback. |

## After the Loop

Once you've addressed the substantive feedback:

1. Run the **polish pass** ([cookbook/polish-pass.md](polish-pass.md)) — this is non-negotiable.
2. Commit the polish work.
3. Push.
4. Update the PR description.
5. Request final human review (if applicable).

The loop gets the reviewer feedback addressed. The polish pass makes the code *good*.
