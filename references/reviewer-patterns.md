# Reviewer-Specific Patterns

## Purpose

Different reviewers have different strengths, weaknesses, and output styles. This reference documents how to extract signal from each common source.

The goal is **not** to optimize for reviewer scores. The goal is to **improve the code** by intelligently consuming what each reviewer offers.

## Devin

### Strengths

- Excellent at spotting missing error handling and edge cases
- Good at walking through "what if" scenarios
- Often catches integration issues across files
- Strong narrative context — explains *why* something matters

### Weaknesses

- Produces very long, meandering sessions
- Sometimes suggests large refactors that are out of scope
- Can over-index on "clean code" abstractions that add complexity without clear benefit
- Narrative framing can obscure concrete suggestions

### How to Read Devin Output

1. **Skip the intro and conclusion.** Devin often starts with "I've reviewed the code..." and ends with "Let me know if you have questions..." — neither is actionable.
2. **Look for concrete suggestions.** Search for phrases like:
   - "I recommend..."
   - "You should..."
   - "This will cause..."
   - "Suggested change:"
3. **Extract code snippets.** Devin often shows "before/after" diffs inline. These are the highest-signal items.
4. **Ignore speculative "what if" discussions** unless they reveal a real gap in the current code.
5. **Watch for scope creep.** Devin sometimes suggests changes far beyond the PR. Note these as "future work" and ignore for this PR.

### Typical Output Structure

```
[Long narrative intro]

## Finding 1: Missing validation on webhook signature
[Explanation of why this matters]
[Suggested code change]

## Finding 2: ...
```

**Action:** Extract the "Finding N" sections. Ignore everything else.

### Mapping to Severity

| Devin Language | Map To |
|----------------|--------|
| "This is a bug" / "This will break" / "Security issue" | `blocking` |
| "I recommend" / "You should" / "This could cause issues" | `important` |
| "Nit" / "consider" / "style preference" | `nit` |
| "Question" / "I'm not sure why..." | `question` |

## Codex

### Strengths

- Excellent at spotting edge cases and alternative approaches
- Strong on security and input validation patterns
- Often provides concrete, minimal code suggestions
- Good at identifying "this looks like a copy-paste bug"

### Weaknesses

- Can over-index on abstractions ("you should extract this into a helper")
- Sometimes suggests changes that are technically correct but increase complexity
- Confidence scores are often well-calibrated but should still be treated as one signal

### How to Read Codex Output

1. **Prioritize concrete suggestions over abstractions.** A concrete bug fix is more valuable than a "you could make this cleaner" suggestion.
2. **Evaluate complexity tradeoffs.** If Codex suggests a 10-line helper to save 2 lines of duplication, consider whether the helper is worth the cognitive overhead.
3. **Use the confidence score.** Codex usually provides `confidence: 0-100`. Treat < 60 as "interesting but verify."
4. **Look for security and validation findings first.** Codex is particularly strong here.

### Typical Output Structure

Codex often produces semi-structured output:

```json
{
  "findings": [
    {
      "severity": "high",
      "confidence": 85,
      "file": "src/foo.ts",
      "line": 42,
      "description": "...",
      "suggestion": "..."
    }
  ]
}
```

Or in markdown with clear sections.

**Action:** Parse the structured findings. Act on `high` severity first.

### Mapping to Severity

Codex often uses `high` / `medium` / `low`. Map directly:

| Codex | Map To |
|-------|--------|
| `high` | `blocking` or `important` (use judgment) |
| `medium` | `important` or `nit` |
| `low` | `nit` |

## Greptile

### Strengths

- Excellent static analysis (unused code, potential bugs, type safety)
- Well-calibrated confidence scores for its domain
- Fast and consistent
- Good at catching "I didn't realize this was a problem" issues

### Weaknesses

- Noisy on generated code, intentional patterns, or domain-specific idioms
- Weak on architectural concerns (coupling, abstraction boundaries)
- Can flag "problems" that are intentional by design

### How to Read Greptile Output

1. **Trust the confidence score more than other reviewers.** Greptile's confidence is often well-calibrated for static analysis.
2. **Apply a "generated code" filter.** If the flagged file is in a `generated/` directory or has a clear "do not edit" header, ignore.
3. **Apply a "known pattern" filter.** If your project has a `.review-lessons.json` or similar, check if this is a known false positive.
4. **Escalate only on high confidence + high severity.** Greptile with `confidence: 90` + `bug` category is worth taking seriously.

### Typical Output Structure

Greptile produces structured JSON:

```json
{
  "file": "src/foo.ts",
  "line": 42,
  "category": "bug",
  "confidence": 85,
  "message": "Potential null dereference"
}
```

**Action:** Filter by confidence first (drop < 50), then by category.

### Mapping to Severity

| Greptile Category | Map To |
|-------------------|--------|
| `bug`, `security`, `vulnerability` | `blocking` (if confidence > 70) |
| `style`, `readability`, `naming` | `nit` |
| `performance` | `important` (verify) |

## Human Reviewers

### Strengths

- Best at understanding *intent* and *context*
- Can spot architectural issues that AI misses
- Understand the broader project and team dynamics
- Can give nuanced feedback ("this is fine for now, but consider X later")

### Weaknesses

- Inconsistent (depends on the human's experience, energy, and familiarity with the codebase)
- Can be biased toward their own preferences
- May miss things that AI catches (edge cases, static analysis findings)

### How to Read Human Feedback

1. **Read the actual words.** Do not over-interpret. Humans are better at context than AI.
2. **If a human is confused, the code is unclear.** Don't just "fix the symptom" — improve the clarity (docs, naming, structure).
3. **Push back respectfully with evidence.** Humans appreciate tradeoffs and data more than AI does.
4. **Ask clarifying questions.** If you don't understand the concern, say so. Don't guess.

### Typical Output Structure

Human feedback varies widely:

- Inline comments on specific lines
- High-level PR comments
- "Request changes" with a list of required fixes
- Casual "LGTM with one nit"

**Action:** Triage by severity implied by language, not by the review event type (`APPROVE` / `REQUEST_CHANGES` / `COMMENT`).

### Mapping to Severity

| Human Language | Map To |
|----------------|--------|
| "This will break" / "Security issue" / "Must fix" | `blocking` |
| "Should fix" / "Please address" / "Concerned about..." | `important` |
| "Nit" / "optional" / "consider" | `nit` |
| "Question" / "I'm not sure I follow..." | `question` |

## Multiple Reviewers (Mixed Signals)

When two reviewers contradict each other:

1. **Understand both positions.** Don't average them.
2. **Make a deliberate choice.** Document it.
3. **Resolve both threads** with a reference to your decision.

Example resolution comment:
```
Devin suggested approach A (session 2, finding 3). Codex suggested approach B (comment on line 45). I chose A because [reason with evidence]. See PR description for details.
```

## Source-Specific Anti-Patterns

| Source | Anti-Pattern | Consequence |
|--------|--------------|-------------|
| Devin | Treating the entire narrative as equally important | Wastes time on low-signal content |
| Codex | Implementing every abstraction suggestion | Code becomes over-abstracted and hard to follow |
| Greptile | Trusting every finding without a "generated code" filter | Wasted time on false positives |
| Human | Dismissing concerns because "the AI didn't flag it" | Misses context that AI lacks |
| Any | Mass-resolving threads to make the dashboard green | Ships mediocre code |

## General Heuristics

- **High confidence + concrete suggestion + clear harm** → Act immediately (`blocking` or `important`)
- **Medium confidence + vague suggestion** → Investigate before acting
- **Low confidence or speculative** → Note and ignore unless you independently verify
- **Out of scope** → Document as future work, resolve the thread
- **Contradictory across sources** → Make a deliberate choice, document it

## Remember the Goal

The goal is not to make all reviewers happy.

The goal is to **improve the code**.

Every reviewer suggestion should be evaluated against that standard.
