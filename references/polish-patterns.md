# Self-Review Polish Pass Patterns

## Purpose

This reference documents high-signal patterns for the self-review polish pass — the highest-leverage step in the `/review` workflow.

The polish pass is where you catch what reviewers (AI or human) miss. It is not optional.

## Documentation Patterns

### Exported Functions and Types

Every exported function, class, type, and interface should have a doc comment that answers:

1. **What does it do?** (one sentence)
2. **Why does it exist?** (the intent)
3. **What are the invariants?** (what must be true before/after)
4. **What are the gotchas?** (edge cases, performance considerations, non-obvious behavior)

Good:
```ts
/**
 * Validates a webhook signature and returns the parsed event.
 *
 * This is the security boundary for all incoming Stripe webhooks.
 * It throws WebhookValidationError on any failure (invalid signature,
 * malformed payload, unknown event type).
 *
 * Never returns a partial result. Either the event is fully validated,
 * or an error is thrown.
 */
export function validateWebhookSignature(
  payload: string,
  signature: string
): Stripe.Event {
```

Bad:
```ts
/**
 * Validates a webhook signature.
 */
export function validateWebhookSignature(...) {
```

### Complex Internal Functions

For non-exported but non-obvious functions, add a brief block comment:

```ts
// We intentionally use a simple string comparison here instead of
// timingSafeEqual because:
// 1. The input is not user-controlled (it's from our own DB)
// 2. timingSafeEqual would require converting to Buffers, adding allocation
// 3. The performance difference is negligible for our scale (< 1000 checks/sec)
function isTokenValid(token: string, expected: string): boolean {
```

### When *Not* to Add Docs

- Getters/setters that are self-explanatory from the property name
- One-line utility functions with obvious names (`isEmpty`, `toString`)
- Test code (unless the test itself is testing documentation behavior)

## "Why" Comment Patterns

### Tradeoff Explanations

```ts
// We chose a simple LRU cache (max 500) instead of a more sophisticated
// solution (e.g., SWR, React Query) because:
// - The working set is small and bounded
// - We want to avoid the complexity of cache invalidation
// - The data is read-heavy and staleness is acceptable (5 min TTL)
const cache = new LRUCache<string, User>({ max: 500, ttl: 1000 * 60 * 5 });
```

### Intentional Deviations

```ts
// Intentionally NOT using the shared validateEmail helper.
// The shared helper rejects +aliases (user+test@domain.com), but this
// flow explicitly needs to support them for the marketing integration.
// See marketing/aliases.md for context.
if (!email.includes('@')) {
  throw new Error('Invalid email');
}
```

### "Why This, Not That"

```ts
// We use a for-loop instead of Array.map + filter because:
// - We need early exit on first invalid item (performance)
// - We want to collect errors, not throw on first failure
// - The resulting code is more readable than a 3-line chain
for (const item of items) {
```

## Code Smell Sweep Checklist

### High Priority (Always Fix)

- [ ] Dead code (unused functions, unreachable branches, commented-out code)
- [ ] Leftover debug code (`console.log`, `debugger`, `TODO: remove before merge`)
- [ ] Secrets or tokens in code or test fixtures
- [ ] Duplicate logic that appears 3+ times (extract a helper)

### Medium Priority (Fix If Easy)

- [ ] Misleading names (a variable named `data` that is actually a `User`)
- [ ] Complex expressions that deserve extraction:
  ```ts
  // Bad
  if (user.age > 18 && user.verified && !user.suspended) {

  // Better
  if (isEligibleToPurchase(user)) {
  ```
- [ ] Inconsistent patterns within the same file (pick one style, make it consistent)
- [ ] Magic numbers (extract to named constants)

### Low Priority (Fix If Trivial, Otherwise Note)

- [ ] Long functions (> 50 lines) — consider splitting if the logic has clear boundaries
- [ ] Missing error handling on non-critical paths
- [ ] Minor duplication (2 occurrences) that would require a complex abstraction

## Quality Gate Patterns

### Running the Right Checks

Always run the project's quality gates after polish work:

```bash
# Detect package manager and run appropriate checks
if [ -f package.json ]; then
  if [ -f bun.lockb ]; then bun run check && bun test; fi
  if [ -f pnpm-lock.yaml ]; then pnpm check && pnpm test; fi
  if [ -f yarn.lock ]; then yarn check && yarn test; fi
  if [ -f package-lock.json ]; then npm run check && npm test; fi
fi
```

### What to Do With Failures

- **Type errors:** Fix before committing. Types are part of the contract.
- **Lint errors:** Fix before committing. Don't argue with the linter in a polish pass.
- **Test failures:** Fix before committing. If a test is flaky, document it — don't ignore it.
- **Known-flaky checks:** If a check is intentionally skipped in CI, document that in the PR description. Do not silently ignore failures.

## Commit Message Patterns

### Single Concern

```
chore: self-review polish pass — docs, clarity, and quality gates
```

### Split by Concern (Preferred for Larger Polish Work)

```
docs: add TSDoc to payment webhook helpers

refactor: extract webhook signature validation into shared helper

test: add coverage for retry edge cases in payment flows

chore: run full lint + typecheck pass (no errors)
```

## Re-Reading the Diff

Before marking the polish pass complete, re-read the entire diff:

```bash
git diff main...HEAD
# or
gh pr diff
```

Ask yourself:

1. Would I be proud to maintain this code in 6 months?
2. Would a new engineer on the team understand the "why" behind key decisions?
3. Are there any comments or docs I would add if I were reviewing someone else's PR?
4. Is there any dead code, debug code, or TODO that I would be embarrassed to ship?

If the answer to any of these is "no," continue polishing.

## Time Expectations

| PR Size | Expected Polish Time | Notes |
|---------|---------------------|-------|
| < 100 lines, low complexity | 5-10 minutes | Mostly docs + quick smell sweep |
| 100-400 lines, moderate complexity | 15-25 minutes | Docs + smells + quality gates |
| 400+ lines or high complexity | 30-60 minutes | Consider splitting the PR |

If polish takes significantly longer than expected, the PR may be too large or doing too many things. Consider splitting.

## Anti-Patterns

### "I'll add docs later"

Later never comes. Documentation is part of the change, not a follow-up task.

### "The reviewer didn't complain about X"

Reviewers are not a substitute for your own standards. The polish pass is *your* process.

### "This is just a small change"

Small changes compound. Polish is cheap insurance.

### Skipping the quality gate run because "it passed earlier"

Your polish changes may have introduced new issues. Run it again.

### Mass-resolving threads to make the dashboard green

You are optimizing for the wrong metric. The polish pass is about code quality, not reviewer scores.

## The Mindset

You are the last engineer who will deeply understand this change before it ships.

Reviewers (AI or human) will move on. Future you, or future teammates, will live with the result.

Act like it.
