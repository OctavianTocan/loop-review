# Coverage Pre-flight Gate Reference

## Purpose

This reference documents the philosophy, edge cases, and decision framework for the hard coverage pre-flight gate — the non-negotiable gate in the `/review` workflow.

## Philosophy

We do **not** chase perfect reviewer scores at the expense of test coverage.

A PR with:
- 98% reviewer approval (Devin + Codex + Greptile + 2 humans)
- 0 new unit tests on 400 lines of new logic
- 1 happy "LGTM" from a human reviewer

...is a **liability**, not a win.

This gate exists to protect:
- Long-term code health
- The humans who will maintain this code after you
- The on-call engineer who will get paged at 2am because an edge case had no test

## What "Coverage" Means

### Unit Test Coverage

Direct, behavior-focused tests for new or modified logic:

- New public functions → should have unit tests
- New internal functions with complex logic → should have unit tests
- Modified behavior in existing functions → should have tests for the new behavior
- Edge cases in new logic → should be explicitly tested

**Not required:**
- Trivial getters/setters
- One-line pass-through functions
- Generated code (the generator should be tested, not the output)

### Integration / E2E Coverage

Tests that exercise user-facing flows through multiple layers:

- New API endpoints → at least one integration test
- New user-facing UI flows → at least one E2E test (or a clear note that manual testing was performed)
- Critical paths (auth, payments, data mutations) → integration coverage is mandatory

## Risk Classification

### High-Risk Areas (Red Light if Untested)

These areas require explicit test coverage. No exceptions.

- Authentication / authorization logic
- Payment processing, financial calculations, billing
- Data migrations, schema changes, data loss scenarios
- Security boundaries (input validation, signature verification, encryption)
- Public API contracts (new endpoints, new hook contracts)
- Concurrency / race condition handling

### Medium-Risk Areas (Yellow Light if Untested)

These areas should have coverage. Yellow light if missing.

- Complex business logic with many branches
- Error handling and retry logic
- Integration points with third-party services
- State machine transitions

### Low-Risk Areas (Green Light Acceptable)

These areas can ship with lighter coverage.

- Pure utility functions with obvious behavior
- UI components that are purely presentational
- Internal refactoring with no behavior change
- Documentation-only changes

## Decision Framework

### Green Light (Proceed)

- All new public APIs have direct unit test coverage
- Critical user-facing flows have at least one integration/E2E path
- No high-risk areas are untested
- New complex logic has reasonable branch coverage

**Action:** Proceed without discussion.

### Yellow Light (Proceed with Explicit Risk Acceptance)

- New complex logic with partial or zero unit tests
- User-facing flows with no test coverage on the changed behavior
- Medium-risk areas untested

**Action:** Present the gaps to the user. Ask:
- "Do you want to add tests first?"
- "Do you want to accept the risk and continue anyway?"
- "Do you want to reduce scope to only the covered parts?"

Default: Do not proceed with heavy iteration until the user explicitly accepts the risk in writing.

Document the risk acceptance in the PR description.

### Red Light (Strongly Recommend Adding Tests)

- High-risk areas with zero test coverage
- Large surface area changes with minimal or no test changes
- New public contracts with no test coverage

**Action:** Strongly recommend adding tests before proceeding. If the user insists on continuing, require explicit risk acceptance and document it prominently in the PR description.

## Special Cases

### Pre-PR (Branch Not Yet Opened)

Same process, but use `git diff main...HEAD` instead of `gh pr diff`.

The report should still be actionable before the PR is created.

### Trivial Changes (<50 Lines, No Logic)

Coverage analysis may be light or skipped with a note.

Still run the gate so the user sees the reasoning.

### Generated Code / Config Changes

Note that generated code often doesn't need direct unit tests.

Focus coverage questions on the *generator* or the *integration points*.

### Test-Only Changes

If the PR *is* the tests, coverage is inherently strong.

Note this and green-light quickly.

### Refactoring with No Behavior Change

If the PR is a pure refactor (same behavior, different structure), coverage requirements are lighter.

Focus on: "Did the existing tests still pass?" If yes, green light.

### Large PRs (>15 Files or >800 Lines)

Prioritize the highest-risk areas first.

Do not attempt exhaustive coverage analysis. Focus on the blast radius.

## Why This Gate Is Hard Opinionated

AI reviewers optimize for what they can see in the diff and what patterns they've been trained on.

They cannot see:
- The production incident that will happen at 2am because an edge case had no test
- The future maintainer who will curse your name because the critical path has zero coverage
- The data loss scenario that only appears under load
- The security vulnerability that only manifests in production

This gate is your last line of defense before those realities hit.

Use it.

## Common Pushback and Responses

**"The reviewer approved it, why do I need more tests?"**

Reviewers are not a substitute for your own standards. The coverage gate is *your* process.

**"I'll add tests later."**

Later never comes. Coverage is part of the change, not a follow-up.

**"This is just a small change."**

Small changes compound. The gate is cheap insurance.

**"The existing tests cover it indirectly."**

Indirect coverage is better than none, but direct tests for new logic are still required. Indirect coverage often misses edge cases.

**"I don't know how to test this."**

That's a signal that the code may be too coupled or complex. Consider refactoring to make it testable, or document the testing gap explicitly.

## After the Gate

If the gate passes (green) or risk is accepted (yellow/red with explicit acceptance):

- Proceed to the feedback loop or polish pass
- Document any accepted gaps in the PR description
- File follow-up issues for significant coverage debt

If the gate fails (red, no acceptance):

- Add tests
- Re-run the gate
- Only proceed when green or yellow with explicit acceptance
