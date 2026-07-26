# Phase 08 — Turn notices into compact deltas with cooldowns

## Finding

`packages/coding-agent/src/prompts/system/` contains about 21,298 estimated tokens across 64 conditional files. Not all are active together, but several large notices repeat the base prompt and tool descriptions:

- `plan-mode-active.md`: about 2,626 tokens;
- `workflow-notice.md`: about 1,484;
- `orchestrate-notice.md`: about 1,385;
- `subagent-yield-reminder.md`: about 485;
- `vibe-mode-active.md`: about 533;
- `eager-todo.md`: about 323;
- thinking/tool-loop and retry reminders: roughly 100–210 each.

Workflow and orchestration notices duplicate delegation gates, task field format, todo behavior, verification, completion, and anti-patterns already present in the core/task prompts. Retry reminders are smaller and often incident-driven; their value depends on trigger precision and repetition count.

**Token cost:** medium per trigger; very high if an unchanged notice is injected repeatedly or retained in history.

**Quality value:** mode overrides and loop breakers can be decisive.

**Tradeoff:** deleting corrective reminders based on source similarity alone can restore the exact failure that caused them to be added.

## Change

1. Inventory every conditional prompt with trigger, frequency, persistence, cooldown, and local-history origin.
2. Classify content:
   - **override delta**: keep in notice;
   - **existing base/tool rule**: reference or delete;
   - **reference mechanics**: move to canonical tool/skill;
   - **incident correction**: keep until benchmark proves redundant.
3. Replace workflow/orchestration notices with compact deltas:
   - trigger meaning;
   - what default rule changes;
   - required decomposition/closure behavior;
   - pointer to normal task/todo contracts.
   Do not repeat schemas, examples, or the whole lifecycle.
4. Inject stable mode notices once or through the replaceable context channel from Phase 01. If Phase 01 is not implemented, add per-`customType` replacement locally in this phase.
5. Add cooldown/state guards for repeated retry, loop, todo, and yield reminders. Re-inject only when behavior recurs after the model had a chance to act or when state materially changes.
6. Delete obsolete notices whose behavior is now enforced in code. Prefer boundary enforcement over repeated prose.

## Verification

- Build a trigger matrix mapping each notice to one deterministic test.
- For workflow/orchestration triggers, compare decomposition coverage, parallel dispatch, verification, and premature-yield rate before/after.
- For loop/retry reminders, replay the historical failing shape and count turns to recovery.
- Repeat the same failure several times; verify cooldown suppresses redundant text but re-arms after a genuine recurrence.
- Inspect serialized requests to ensure unchanged mode notices appear once, not once per historical turn.
- Confirm non-triggered sessions receive none of the conditional text.

## Acceptance

At least 50% reduction in workflow/orchestration notice tokens and no cumulative unchanged notices. Every incident-driven reminder retains or improves recovery rate in its targeted replay.
