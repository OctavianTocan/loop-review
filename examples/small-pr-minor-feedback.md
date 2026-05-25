# Example: Small PR with Minor Feedback

## Scenario

You have a 47-line PR that adds a simple utility function. One reviewer (Codex) left 2 comments:
- One `important` finding about missing input validation
- One `nit` about a variable name

You want to address the feedback and ship.

## Step 1: Pre-flight (Quick)

```bash
/review preflight
```

Result: Green. The change is a pure function with no side effects. Existing test patterns cover the new code path. No high-risk areas touched.

Proceed.

## Step 2: Stack Check (Skipped)

Not in a stack. Skip.

## Step 3: Feedback Loop (Light)

Read the Codex output:

```
[important] src/utils/format.ts:18 — email input not validated before use
[nit] src/utils/format.ts:22 — variable name `str` is unclear
```

**Triage:**

- `important`: Actionable code change. Add validation.
- `nit`: Style preference. Rename variable.

**Changes:**

```ts
// Before
export function formatEmail(str: string): string {
  return str.trim().toLowerCase();
}

// After
export function formatEmail(email: string): string {
  if (!email || typeof email !== 'string') {
    throw new Error('Email must be a non-empty string');
  }
  return email.trim().toLowerCase();
}
```

Commit:
```
fix: validate email input before formatting (addresses Codex finding)
```

Push. Codex re-evaluates and marks both findings resolved.

## Step 4: Polish Pass (Always)

Even for a small PR, run the polish pass:

1. **Documentation:** Add TSDoc.
2. **"Why" comments:** Not needed — behavior is obvious.
3. **Smell sweep:** None found.
4. **Quality gates:** Run `npm run typecheck && npm run lint && npm test`. All pass.

Commit:
```
docs: add TSDoc to formatEmail (polish pass)
```

## Step 5: Update PR Description

```markdown
## What does this do?

Adds `formatEmail` utility for normalizing email addresses before comparison or storage.

## How did we test this?

- Unit tests in `src/utils/format.test.ts`
- Existing integration tests for login flow exercise this code path

## Reviewer feedback

- Addressed Codex finding on input validation
- Renamed `str` to `email` for clarity
```

## Result

- All reviewer feedback addressed
- Coverage strong (pre-flight green)
- Polish pass complete
- PR description updated
- Ready for final review or merge

**Time:** ~15 minutes total.

This is the standard, even for small PRs.
