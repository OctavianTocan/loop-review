# Unified Feedback Model

## Purpose

This skill normalizes feedback from *any* reviewer into a simple common shape internally.

The exact source format (Devin session transcript, Codex structured output, Greptile JSON, human GitHub comments, Cursor annotations, Claude conversation) is irrelevant.

We care about the **intent** of the feedback and the **action** it implies.

## Normalized Shape

Every piece of feedback is reduced to:

```ts
interface FeedbackItem {
  source: string;           // "devin", "codex", "greptile", "human", "cursor", "claude", ...
  file?: string;            // Path, if file-specific
  line?: number;            // Line number, if applicable
  lineRange?: [number, number]; // For multi-line comments
  severity: 'blocking' | 'important' | 'nit' | 'question' | 'praise';
  confidence: number;       // 0-100, if provided by source; otherwise estimated
  summary: string;          // One-sentence description of the concern
  suggestedAction?: string; // Concrete suggestion, if provided
  threadId?: string;        // For resolution tracking (GitHub comment ID, etc.)
  context?: string;         // Surrounding context or rationale from the reviewer
}
```

## Mapping from Common Sources

### Devin

Devin produces long narrative sessions. Extract:

- Concrete code suggestions (often in "Suggested changes" or "I recommend..." sections)
- Explicit bug findings ("This will cause X because...")
- Missing error handling ("You should add a check for...")
- Ignore: Narrative framing, self-congratulation, speculative "what if" discussions

Map severity based on language:
- "This is a bug" / "This will break" → `blocking`
- "You should" / "I recommend" → `important`
- "Nit" / "style" / "consider" → `nit`

### Codex

Codex often produces structured or semi-structured output with confidence scores.

- Use Codex's severity/confidence directly when available.
- Codex is strong on edge cases and alternative approaches.
- Map "security concern" or "data loss" language → `blocking`

### Greptile

Greptile produces structured output with `confidence` (0-100) and category labels.

- Use Greptile's confidence directly.
- Map categories:
  - `bug`, `security`, `vulnerability` → `blocking` or `important`
  - `style`, `readability`, `naming` → `nit`
- Greptile can be noisy on generated code or intentional patterns. Apply judgment.

### Human GitHub Comments

Humans are better at intent and context than AI.

- Read the actual words. Don't over-interpret.
- If a human is confused, the code is likely unclear — treat as `important` for clarity.
- Push back respectfully with evidence.

### Cursor / Claude

These are conversational. Extract:

- Explicit suggestions ("Change X to Y because...")
- Questions that reveal gaps ("What happens if Z?")
- Praise ("This pattern is excellent")

Conversational sources often mix questions with suggestions. Clarify before acting.

## Deduplication Strategy

When multiple sources flag the same area:

1. **Same file + overlapping line range (±3 lines) + similar topic** → Merge into one item. Keep the most detailed description. Credit all sources.
2. **Cross-source agreement on severity** → Escalate if 2+ sources flag as `blocking` or `important`.
3. **Conflicting severity** → Take the highest severity. Document the conflict in the resolution note.

## Resolution Tracking

For GitHub-native feedback (reviews, inline comments):

- Use the `threadId` (comment ID) to resolve threads after addressing.
- Only resolve threads you've genuinely addressed or deliberately rejected with a comment.

For non-GitHub sources (Devin sessions, Codex transcripts):

- Track resolution in the PR description under "Reviewer feedback decisions."
- No automated thread resolution is possible.

## Filtering

Apply these filters before acting:

1. **Confidence gate:** Drop findings with confidence < 50 (or estimate < 50 if not provided).
2. **Scope gate:** If the feedback is about code outside this PR's diff, treat as informational only.
3. **Staleness gate:** If the feedback is on a commit older than the current HEAD, re-validate before acting.

## Output for Downstream Steps

The normalized feedback feeds into:

- `run-loop.md` — Triage and action
- `polish-pass.md` — Context for the self-review (you may discover reviewer-blind spots here)
- PR description updates — Document significant decisions

## Example Normalization

**Raw Devin output:**
> "I notice that in `src/payments/webhook.ts:42`, you're not validating the signature before processing the payload. This is a security issue — an attacker could send fake events. You should add a check using the Stripe SDK's `constructEvent` function."

**Normalized:**
```ts
{
  source: "devin",
  file: "src/payments/webhook.ts",
  line: 42,
  severity: "blocking",
  confidence: 90,
  summary: "Webhook signature not validated before processing",
  suggestedAction: "Use Stripe SDK constructEvent to validate signature",
  context: "Attacker could send fake events without validation"
}
```

**Raw Greptile output:**
```json
{
  "file": "src/payments/webhook.ts",
  "line": 42,
  "category": "security",
  "confidence": 85,
  "message": "Potential webhook spoofing vulnerability"
}
```

**Normalized:**
```ts
{
  source: "greptile",
  file: "src/payments/webhook.ts",
  line: 42,
  severity: "blocking",
  confidence: 85,
  summary: "Potential webhook spoofing vulnerability",
  context: "No signature validation detected"
}
```

Both normalize to the same shape. The source is preserved for attribution, but the action is the same: validate the signature.

## Why This Model Exists

Without normalization, agents get distracted by source-specific formatting and miss the underlying intent.

With normalization, the skill can:

- Deduplicate across sources
- Triage consistently
- Feed a single, clean view into the run-loop and polish pass

The model is intentionally simple. Complexity belongs in the *sources*, not in the consumption layer.
