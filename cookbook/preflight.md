# Pre-flight Coverage Gate (Hard Opinionated)

## Context

This is the most important gate in the entire skill. Weak test coverage is the fastest way for AI-driven fixes to introduce regressions.

## Philosophy

We do **not** chase perfect reviewer scores at the expense of test coverage.

## Process

### 1. Launch Coverage Subagent

Use a fast, structured model (e.g. Haiku or equivalent) with this prompt:

```
You are a senior test coverage auditor.

Analyze the PR diff for the current branch.

Answer these questions precisely:

1. Unit test coverage:
   - Which new or modified functions/logic paths have direct unit test coverage?
   - Which important paths are missing unit tests?

2. E2E / integration coverage:
   - Which user-facing flows touched by this PR have E2E or integration test coverage?
   - Which flows are uncovered?

3. Risk assessment:
   - What is the highest-risk area in this change that lacks test coverage?

Be specific. Reference file paths and function names.
```

### 2. Evaluate the Result

**Green light (proceed):**
- All new public APIs / important logic have reasonable unit test coverage.
- Critical user flows have at least one E2E or integration test path.

**Yellow / Red light (stop or force confirmation):**
- New complex logic with zero unit tests.
- User-facing flows with no test coverage.
- High-risk areas (auth, payments, data migration, security boundaries) that are untested.

### 3. Decision

If coverage is insufficient:

- Clearly state the gaps to the user.
- Ask: "Do you want to add tests first, or proceed anyway with explicit risk acceptance?"

Default behavior: **Do not proceed with heavy iteration** until the user explicitly accepts the risk.

## Output

The subagent should produce a short, scannable report. The main agent then decides whether to continue.

This gate exists to protect long-term code quality.
