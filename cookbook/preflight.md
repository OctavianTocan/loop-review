# Pre-flight Coverage Gate (Hard Opinionated)

## Context

This is the most important gate in the entire skill.

Weak test coverage is the fastest way for AI-driven fixes to introduce regressions that no reviewer will catch until production.

## Philosophy

We do **not** chase perfect reviewer scores at the expense of test coverage.

A PR with:
- 98% reviewer approval
- 0 new unit tests on 400 lines of new logic
- 1 happy "LGTM" from a human

...is a **liability**, not a win.

This gate exists to protect long-term code health and the humans who will maintain this code after you.

## When to Run

- Automatically as step 2 of `/review` (start)
- Manually via `/review preflight` on any branch you care about
- Before opening a PR (pre-PR mode)

## Process

### 1. Gather Context

Identify the PR (or branch if pre-PR):

```bash
gh pr view --json number,headRefName,baseRefName,additions,deletions,changedFiles
```

If no PR yet, note the branch and changed file count.

### 2. Launch Coverage Subagent

Use a fast, structured model (Haiku, Sonnet, or equivalent) with this prompt:

```
You are a senior test coverage auditor with 15+ years of experience shipping reliable production systems.

Analyze the PR diff for the current branch (or the branch provided).

Answer these questions **precisely and specifically**:

1. Unit test coverage:
   - List every new or significantly modified function, class, or logic path.
   - For each, state whether it has direct unit test coverage (yes/no/partial).
   - If partial or missing, note the specific gap.

2. Integration / E2E coverage:
   - List every user-facing flow or critical path touched by this change.
   - For each, state whether it has at least one integration or E2E test path (yes/no).
   - If missing, note the specific flow that is uncovered.

3. Risk assessment:
   - What is the single highest-risk area in this change that lacks test coverage?
   - Why is it high-risk (blast radius, data loss, security boundary, etc.)?
   - If this change ships with the current coverage, what is the most likely class of regression?

Be specific. Reference file paths and function names. Do not use vague language like "some tests" or "reasonable coverage."

If the diff is large (>15 files or >800 lines), prioritize the highest-risk areas first.
```

### 3. Evaluate the Result

**Green light (proceed without discussion):**
- All new public APIs and important internal logic have direct unit test coverage.
- Critical user-facing flows have at least one integration/E2E test path exercising the change.
- No high-risk areas (auth, payments, data migrations, security boundaries, financial calculations) are untested.

**Yellow light (proceed only with explicit risk acceptance):**
- New complex logic with partial or zero unit tests.
- User-facing flows with no test coverage on the changed behavior.
- Moderate-risk areas untested.

**Red light (strongly recommend adding tests before proceeding):**
- High-risk areas (auth, payments, data boundaries, security, financial logic) with zero test coverage.
- Large surface area changes with minimal or no test changes.
- New public contracts (APIs, hooks, components) with no test coverage.

### 4. Decision

If coverage is insufficient (Yellow or Red):

1. Clearly state the gaps to the user with specific file/function references.
2. Ask one of:
   - "Do you want to add tests first?"
   - "Do you want to accept the risk and continue anyway?"
   - "Do you want to reduce scope to only the covered parts?"

**Default behavior:** Do not proceed with heavy iteration (loop, polish, or merge) until the user explicitly accepts the risk in writing.

Document the risk acceptance in the PR description if proceeding.

### 5. Output Format

The subagent should produce a short, scannable report:

```
COVERAGE PRE-FLIGHT — PR #1234 (feat/new-payment-flow)

Unit test coverage:
  ✓ src/payments/charge.ts:processPayment (covered by payments.test.ts:45)
  ✗ src/payments/charge.ts:validateWebhookSignature (NO UNIT TESTS)
  ✓ src/payments/refund.ts:issueRefund (covered by refunds.test.ts:88)

Integration/E2E coverage:
  ✓ POST /api/payments/charge (e2e/payments.spec.ts:120)
  ✗ Webhook retry flow on signature failure (NO COVERAGE)

Risk assessment:
  HIGHEST RISK: validateWebhookSignature has zero test coverage.
  This is a security boundary. An attacker who can forge a signature bypasses all payment logic.
  If this ships untested, the most likely regression is: unauthorized refunds or duplicate charges.

Recommendation: RED — Do not proceed without tests on the webhook path.
```

The main agent then presents this to the user and enforces the decision gate.

## Special Cases

**Pre-PR (branch not yet opened as PR):**
- Same process, but use `git diff main...HEAD` (or equivalent) instead of `gh pr diff`.
- The report should still be actionable before the PR is created.

**Trivial changes (<50 lines, no logic):**
- Coverage analysis may be light or skipped with a note.
- Still run the gate so the user sees the reasoning.

**Generated code / config changes:**
- Note that generated code often doesn't need direct unit tests.
- Focus coverage questions on the *generator* or the *integration points*.

**Test-only changes:**
- If the PR *is* the tests, coverage is inherently strong. Note this and green-light quickly.

## Why This Gate Exists

AI reviewers optimize for what they can see in the diff and what patterns they've been trained on.

They cannot see:
- The production incident that will happen at 2am because an edge case had no test
- The future maintainer who will curse your name because the critical path has zero coverage
- The data loss scenario that only appears under load

This gate is your last line of defense before those realities hit.

Use it.
