# Phase 11 — Reduce advisor transcript bandwidth

## Finding

The advisor system prompt is 5,529 bytes, about 1,383 estimated tokens. When enabled, `AdvisorRuntime.onTurnEnd()` queues a model update after each primary turn (`packages/coding-agent/src/advisor/runtime.ts:110-120`). The delta renderer includes hidden thinking, tool intent, expanded primary contexts, and expanded edit diffs (`:203-226`). The advisor keeps its own append-only context and re-primes from the current primary transcript near its context limit.

Existing code already collapses byte-identical plan/goal context messages because the primary re-injects them (`:229-247`). Emission guards enforce one advice per update because prose alone did not prevent repeated advice.

**Token cost:** about 1.4k static plus the full rich delta on every primary turn; high for edit-heavy or reasoning-heavy sessions.

**Quality value:** the advisor can catch skipped consumers, weak verification, and wrong approaches. Thinking and diffs sometimes expose risks absent from final text.

**Tradeoff:** removing reasoning/diff detail may lower defect detection. Fewer advisor calls increase latency before intervention.

## Change

1. Instrument advisor input by content class: user text, assistant visible text, thinking, tool arguments, tool results, edit diffs, recurring contexts, and system prompt.
2. Compress the advisor system prompt around its unique job:
   - inspect a new delta;
   - stay silent unless a concrete new risk exists;
   - verify before advising;
   - emit one severity-tagged note.
   Remove repeated user-intent/process/scope explanations once enforced by the emission gate and evaluation.
3. Default transcript policy:
   - keep user requests/corrections;
   - keep assistant decisions and visible claims;
   - keep tool errors and concise success summaries;
   - replace successful edit diffs with files plus changed ranges/statistics;
   - omit raw hidden thinking unless a risk trigger requests it;
   - keep exact failing diff/error excerpts needed to diagnose.
4. Batch low-risk adjacent turns into one advisor update. Flush immediately on tool error, destructive operation, user correction, completion claim, or verification result.
5. Keep a bounded rolling advisor state or structured risk ledger instead of append-only raw deltas. Re-prime from compact primary state, not full history.
6. Retain a debug setting for full-rich transcript to compare missed findings.

## Verification

- Replay sessions containing a missed callsite, unsafe edit, failing test, user correction, premature completion, and clean progress.
- Compare full-rich and compact policies on true findings, false positives, intervention latency, duplicate advice, input tokens, and advisor cost.
- Confirm secrets and hidden tool arguments remain redacted/unknown.
- Confirm repeated unchanged mode context contributes only a marker or disappears after Phase 01.
- Advisor disabled: no extra prompt work or model calls.

## Acceptance

At least 50% lower advisor input tokens on edit-heavy replay sessions, with no loss of high-severity findings and no increase in duplicate or speculative advice.
