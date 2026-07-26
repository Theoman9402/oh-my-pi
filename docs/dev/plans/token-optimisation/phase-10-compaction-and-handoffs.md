# Phase 10 — Bound compaction and handoff state

## Finding

The compaction corpus is about 1,658 estimated prompt tokens, but output growth matters more than static instructions. `compaction-summary.md`, `compaction-update-summary.md`, `branch-summary.md`, and `handoff-document.md` repeat Goal/Constraints/Progress/Decisions/Next Steps scaffolding. The update prompt says both "preserve all information" and "remove anything no longer relevant" (`packages/agent/src/compaction/prompts/compaction-update-summary.md:1-10`), biasing conservative retention.

Recoverable state—completed todo items, plan bodies, long tool outputs, repository rules, and file contents—can be copied into summaries even when a stable URI or current filesystem is the better source. Compaction may perform multiple model calls against serialized conversation context; static prompt savings are multiplied by those calls.

**Token cost:** about 1.7k static prompt corpus plus an unbounded carried summary and repeated transcript serialization on each compaction/handoff call.

**Quality value:** handoff fidelity is critical; losing a pending user request or uncommitted change is worse than moderate bloat.

**Tradeoff:** hard caps without priority rules produce silent task loss.

## Change

1. Define one canonical handoff state schema shared by initial, update, branch, and explicit handoff paths. Keep mode-specific deltas rather than four prose copies.
2. Resolve the preservation rule:
   - preserve active goals, user constraints, pending questions, blockers, changed artifacts, decisions needed for continuation;
   - replace recoverable bulk with exact path/URI and a reason to read it;
   - remove completed mechanics once their resulting state is recorded;
   - remove superseded errors/attempts unless they constrain the next action.
3. Add section budgets and a total summary budget with deterministic priority. Never truncate pending user requests, blockers, or uncommitted-change locations.
4. Store large exact outputs in artifacts and summarize semantic result plus handle.
5. Reference active plan/todo state from their canonical stores instead of copying full bodies. Include enough active-step text to resume if the store is temporarily unavailable.
6. Avoid multiple compaction calls over the same full transcript where one structured pass can supply short summary/turn prefix derivatives. If multiple calls are required for quality, reuse a compact intermediate rather than replaying raw history.
7. Keep the summarization system prompt minimal and common.

## Verification

- Long-session fixture with completed and pending work, failed attempts, plan, todos, large tool output, user question, and uncommitted files.
- Compact repeatedly; summary token count reaches a bounded plateau rather than monotonically carrying all prior details.
- Resume with a fresh model and complete the pending task using only summary plus referenced artifacts.
- Branch summary and explicit handoff preserve different active branches without mixing completed state.
- A pending exact user question survives byte-for-byte.
- Compare one-pass versus current multi-call quality, latency, input tokens, and failure recovery.

## Acceptance

Bounded summaries retain all continuation-critical facts in the fixture, fresh-session resume succeeds, and repeated compaction does not grow summary tokens absent new active state.
