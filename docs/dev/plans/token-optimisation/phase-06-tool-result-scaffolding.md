# Phase 06 — Make stateful tool results delta-oriented

## Finding

`TodoTool.formatSummary()` emits every remaining task, aggregate counts, the active phase, then every phase and every task after every mutation (`packages/coding-agent/src/tools/todo.ts:499-558`). Each snapshot remains in conversation history. For `K` updates over a `K`-item plan, repeated list text approaches quadratic growth.

Task results also append spawn metadata and conditional specialization/coordination advisories (`packages/coding-agent/src/task/index.ts:393-450,651-675`). Job status can repeat roster and output previews. These are useful on transition but wasteful when unchanged.

**Token cost:** low for one call; hundreds to thousands across a long task. Todo is the clearest high-confidence source.

**Quality value:** full snapshots prevent the model from inventing task identifiers or losing state.

**Tradeoff:** delta-only output can make recovery harder after compaction or if the model forgets prior state. The tool must preserve an explicit full-state operation.

## Change

1. Change todo mutation results (`start`, `done`, `drop`, `append`, `block`, `unblock`, partial remove) to return:
   - the transition just applied;
   - newly active task and phase;
   - overall closed/open counts;
   - errors, if any.
2. Keep full state for `init`, `view`, malformed identifier recovery, and an explicit model request for expansion.
3. Keep canonical state in `details` for UI/renderers and goal-mode internals, but ensure provider-visible `content` is the compact delta. Do not serialize hidden details into model content through adapters.
4. When a task identifier mismatch occurs, return the active phase's candidate strings rather than the entire plan unless ambiguity remains.
5. Emit task specialization/coordination advisory at most once per session/config generation, or only when the triggering anti-pattern occurs. Do not append unchanged advice to later spawn results.
6. Make job/hub status delta-oriented for mutating/wait operations. Keep `list`/`jobs` as explicit full snapshots. Avoid repeating completed output previews already delivered through async result messages.
7. Preserve task-result artifact handles and truncation; those already prevent large child outputs from being re-inlined.

## Verification

- Simulate a 20-item, four-phase todo through init plus 20 completions. Compare cumulative provider-visible todo result tokens; target linear rather than quadratic growth.
- After compaction, call `view` and confirm the full exact task strings are recoverable.
- Trigger a wrong task string and confirm the response gives enough candidates for a successful next call.
- Verify goal-mode todo context still receives complete internal state even though mutation content is compact.
- Spawn generic and specialized batches; advisory appears only on its triggering transition and remains understandable.
- Complete async jobs, call status repeatedly, and confirm output bodies are not duplicated after delivery.
- Confirm TUI expanded views remain unchanged by using structured `details`, not model-facing prose.

## Acceptance

A 20-item completion sequence reduces cumulative todo result text by at least 70%, while every transition, recovery path, goal integration, and expanded UI view remains correct.
