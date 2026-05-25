# Example: Large PR with Heavy Devin Feedback + Graphite Stack

## Scenario

You have a 600-line PR (`feat/new-payment-flow`) that is the middle of a 3-PR Graphite stack:

```
main
  └── feat/payment-models (PR #123, merged)
        └── feat/new-payment-flow (PR #124, current)
              └── feat/payment-ui (PR #125, in progress)
```

Devin left a long session with 12 findings:
- 3 `blocking` (missing auth check, webhook signature not validated, potential double-charge)
- 6 `important` (missing error handling, N+1 query pattern, type not propagated)
- 3 `nit` (naming, style)

You also notice your diff is "dirty" — it includes changes from `main` that aren't yours.

## Step 1: Stack Hygiene (Critical)

```bash
/review stack
```

**Diagnosis:** Your branch is behind `main` by 47 commits. The diff noise is from a recent refactor to the auth middleware that landed on `main`.

**Action:** Restack the entire stack from bottom up:

```bash
gt checkout feat/payment-models  # already merged, skip
gt checkout feat/new-payment-flow
gt stack fix  # restacks current + children
```

Resolve 2 small conflicts in `src/payments/webhook.ts` (yourself, not via agent).

```bash
gt stack submit --force
```

Add a comment on PR #124:
```
Restacked on latest main (commit abc1234). Resolved conflicts in webhook.ts. No functional changes from the rebase.
```

Verify the diff is now clean:
```bash
gh pr diff | head -100
```
Only your changes remain. Good.

## Step 2: Pre-flight Coverage (Mandatory)

```bash
/review preflight
```

**Result:** RED.

- New `processPayment` logic (src/payments/flow.ts:45-120) has zero unit tests
- Webhook retry flow has no integration coverage
- `validateWebhookSignature` is a security boundary with no tests

**Decision:** You explicitly accept risk for the webhook path ("we'll add tests in a follow-up PR") but agree to add unit tests for `processPayment` before proceeding.

Add tests. Re-run pre-flight. Now YELLOW (acceptable with documented risk).

## Step 3: Feedback Loop (Heavy Lifting)

Read the Devin session transcript. Extract 12 concrete findings. Normalize using the unified model.

**Triage batch 1 (blocking items):**

1. Missing auth check on `/api/payments/charge` → Add middleware. Commit.
2. Webhook signature not validated → Implement `validateWebhookSignature`. Commit.
3. Potential double-charge on retry → Add idempotency key. Commit.

Push after each batch so Devin can re-evaluate.

**Triage batch 2 (important items):**

4-9. Address 4, note 2 as "out of scope for this PR, documented in description."

**Triage batch 3 (nits):**

10-12. Apply 2, reject 1 with a polite note ("prefer current naming for consistency with Stripe SDK").

Resolve threads as you go. Do not mass-resolve.

## Step 4: Polish Pass (Highest Value)

This is the step that separates "reviewer-approved" from "actually good."

1. **Documentation:**
   - Add TSDoc to all 8 new exported functions
   - Document the idempotency key tradeoff in a "Why" comment

2. **"Why" comments:**
   - Explain why you chose idempotency keys over distributed locks
   - Explain why the webhook retry uses exponential backoff with max 3 attempts

3. **Smell sweep:**
   - Remove 2 `console.log` statements left from debugging
   - Extract a `calculateChargeAmount` helper from a 12-line expression
   - Rename `data` to `paymentIntent` in 3 places

4. **Quality gates:**
   - `npm run typecheck` → 1 error, fix it
   - `npm run lint` → clean
   - `npm test` → 2 new tests fail, fix them
   - Full CI suite (via `npm run ci`) → passes

5. **Commit:**
   ```
   chore: self-review polish pass — docs, clarity, quality gates

   - Added TSDoc to all new payment helpers
   - Extracted calculateChargeAmount helper
   - Removed debug logging
   - Fixed type error and 2 test failures
   ```

## Step 5: Update PR Description

```markdown
## What does this do?

Implements the core payment flow for charging customers via Stripe.

## Why are we doing this?

[Context from original description, updated with final scope]

## How did we test this?

- Unit tests: `src/payments/flow.test.ts` (processPayment, calculateChargeAmount)
- Integration: `e2e/payments.spec.ts` (happy path + retry)
- Manual: Tested in Stripe test mode with real webhook events

## Coverage note

Pre-flight identified 2 gaps:
- `validateWebhookSignature` has no unit tests (security boundary)
- Webhook retry flow has no integration coverage

Risk accepted for this PR. Follow-up issue #456 filed to add coverage.

## Reviewer feedback decisions

- Devin #3 (double-charge): Chose idempotency keys over distributed locks because [reason].
- Devin #7 (N+1 query): Out of scope for this PR. Documented in issue #457.
- All blocking and 4/6 important findings addressed.

## Stack

- #123 feat/payment-models (merged)
  - #124 feat/new-payment-flow (this PR)
    - #125 feat/payment-ui (in progress)
```

## Result

- Stack hygiene restored
- Coverage gaps identified and (partially) addressed; risk accepted and documented
- All 3 blocking + 4/6 important Devin findings addressed
- Polish pass complete (docs, clarity, smells, gates)
- PR description updated with decisions and coverage notes
- Ready for final human review

**Time:** ~3 hours (spread over 2 days, with pushes between iterations)

This is what a disciplined, high-quality review optimization session looks like on a non-trivial stacked PR.
