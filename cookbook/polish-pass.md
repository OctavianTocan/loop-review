# High-Signal Self-Review Polish Pass

## Context

This is the **highest-value step** in the entire process.

Many PRs pass AI reviewers (Devin, Codex, Greptile) and even human review, but still contain mediocre code. The polish pass exists to prevent shipping that.

Reviewers are optimized for *finding problems in the diff*. They are not optimized for *making the code excellent*.

You are.

## When to Run

- After addressing reviewer feedback (as the final step before "ready for final review")
- As a standalone command (`/review polish`) on any PR you care about
- Before opening a PR (pre-PR polish)
- When you feel uneasy about a change even though reviewers are green

**Recommended:** Always run this pass, even on small PRs. It takes 10-20 minutes and dramatically improves quality.

## The Polish Pass (Execute in Strict Order)

### 1. Documentation (Exported + Complex Internal)

For every exported function, class, type, and interface that was added or significantly changed:

- Add high-quality TSDoc / JSDoc if missing.
- The doc should explain *why* it exists, important invariants, and gotchas — not just repeat the signature.

Good example:
```ts
/**
 * Creates a payment intent and returns the client secret.
 *
 * This is the primary entry point for the checkout flow. It:
 * - Validates the cart contents and pricing
 * - Creates or retrieves a Stripe customer
 * - Creates a PaymentIntent with the correct metadata
 *
 * Throws PaymentError on validation failure or upstream error.
 * Never returns a partial result.
 */
export async function createPaymentIntent(input: CreatePaymentInput): Promise<PaymentIntentResult> {
```

Bad example:
```ts
/**
 * Creates a payment intent.
 */
export async function createPaymentIntent(input: CreatePaymentInput) {
```

For complex internal functions (not exported but non-obvious), add a brief comment block explaining the "why."

### 2. Explain the "Why" (Inline Comments for Non-Obvious Decisions)

For every non-obvious change or decision in the diff, add an inline comment explaining the reasoning.

Good examples:
```ts
// We chose a simple LRU cache here instead of a more sophisticated one
// because the working set is small (< 500 items) and we want to avoid
// the complexity and allocation overhead of a more advanced structure.
const cache = new LRUCache<string, User>({ max: 500 });
```

```ts
// Intentionally NOT using the shared validateEmail helper here.
// The shared helper rejects +aliases (user+test@domain.com), but this
// flow explicitly needs to support them for the marketing integration.
if (!email.includes('@')) {
```

Bad examples:
```ts
// cache the result
const cache = new LRUCache({ max: 500 });
```

```ts
// validate
if (!email.includes('@')) {
```

If the "why" is obvious from the code or surrounding context, you don't need a comment. Add comments for *decisions*, not for *what the code does*.

### 3. Code Smell Sweep

Actively look for and fix (in priority order):

**High priority:**
- Dead code and unreachable branches (including in tests)
- Leftover debug code, `console.log`, TODOs, or temporary workarounds
- Secrets, tokens, or sensitive data in code or test fixtures

**Medium priority:**
- Duplication that can be reasonably consolidated (3+ occurrences of the same pattern)
- Misleading or overly generic names (variables, functions, files)
- Complex expressions that deserve extraction into a well-named helper
- Inconsistent patterns with the rest of the file/module (pick one style, make it consistent)

**Low priority (fix if easy, otherwise note):**
- Magic numbers (extract to named constants)
- Long functions that could be split (subjective — use judgment)
- Missing error handling on non-critical paths

Do not over-refactor. The goal is to make the code *clear and safe*, not to achieve some abstract notion of perfection.

### 4. Run the Full Quality Gate Suite

Detect the package manager and run the project's quality gates:

```bash
# Common patterns — adapt to your project
npm run typecheck && npm run lint && npm test
# or
bun run check && bun test
# or
pnpm typecheck && pnpm lint && pnpm test
# or (Python)
ruff check && mypy . && pytest
```

Run **at minimum** the checks that affect the changed files. Ideally, run the full suite.

Fix any failures before considering the polish pass complete.

If a check is known-flaky or intentionally skipped in CI, document that in the PR description — do not silently ignore it.

### 5. Commit the Polish Work

```bash
git add -A
git commit -m "chore: self-review polish pass — docs, clarity, and quality gates"
git push
```

If the polish work is substantial or touches multiple concerns, split into focused commits:

```
chore: add TSDoc to new payment helpers
refactor: extract webhook signature validation
test: add coverage for retry edge cases
chore: run full lint + typecheck pass
```

### 6. Re-Read the Diff

Before marking the polish pass complete:

```bash
git diff main...HEAD  # or gh pr diff
```

Read the entire diff one more time, top to bottom.

Ask yourself:
- Would I be proud to maintain this code in 6 months?
- Would a new engineer on the team understand the "why" behind key decisions?
- Are there any comments or docs I would add if I were reviewing someone else's PR?

If the answer to any of these is "no," continue polishing.

## Mindset

Treat this pass as if you are the **last engineer who will ever deeply look at this code before it ships**.

The goal is not to make the reviewer happy.

The goal is to make the code *good*.

Reviewers (AI or human) will move on. Future you, or future teammates, will live with the result.

## Common Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| "I'll add docs later" | Later never comes. Docs are part of the change, not a follow-up. |
| "The reviewer didn't complain about X" | Reviewers are not a substitute for your own standards. |
| "This is just a small change" | Small changes compound. Polish is cheap insurance. |
| Mass-resolving threads to make the dashboard green | You are optimizing for the wrong metric. |
| Skipping the quality gate run because "it passed earlier" | Your changes may have broken something. Run it again. |

## Time Expectations

| PR Size | Expected Polish Time |
|---------|---------------------|
| < 100 lines, low complexity | 5-10 minutes |
| 100-400 lines, moderate complexity | 15-25 minutes |
| 400+ lines or high complexity | 30-60 minutes |

If it takes longer than this, the PR may be too large. Consider splitting.

## After the Polish Pass

The PR should now be:
- Well-documented (exports + non-obvious decisions)
- Free of obvious smells and dead code
- Passing all quality gates locally
- Ready for final human review or merge

This is the standard. Hold yourself to it.
