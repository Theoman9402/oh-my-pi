# Phase 01 — Replace recurring mode messages instead of appending them

## Finding

Every primary prompt builds fresh plan, goal, and vibe custom messages in `AgentSession.prompt()` (`packages/coding-agent/src/session/agent-session.ts:8223-8245`). Builders live at `:7737-7795`. `Agent.agentLoop()` appends every input prompt to current context and emits it into persistent state (`packages/agent/src/agent-loop.ts:329-357`; `packages/agent/src/agent.ts:1227-1237`). `convertToLlm()` forwards custom messages as model-visible messages (`packages/coding-agent/src/session/messages.ts:790-805`). No default transform removes older mode copies (`packages/coding-agent/src/sdk.ts:2661-2664`).

The plan template alone is 10,502 bytes, about 2,626 estimated tokens. Advisor code already needs byte-deduplication because primary contexts are re-injected every turn (`packages/coding-agent/src/advisor/runtime.ts:79-85,229-247`).

**Token cost:** about 2.6k × plan-mode turns, plus changing goal todo context and the 533-token vibe prompt. Ten plan turns add about 26k tokens before conversation content.

**Quality value:** active mode constraints must be visible on every provider request; goal todo state must stay current.

**Tradeoff:** an incorrect transient-message implementation could make rules disappear after a tool loop, session restore, compaction, steering message, or provider retry.

## Change

1. First prove request behavior with a provider-spy test: serialize several prompts in each mode and count `customType` occurrences in every request.
2. Add an explicit transient/replaceable context channel to the agent request pipeline. Required invariant: one latest message per context key is sent to the provider, but it is not appended repeatedly to persisted conversation history.
3. Use stable keys:
   - `plan-mode-context`;
   - `goal-mode-context`;
   - `vibe-mode-context`;
   - future active-mode contexts through the same registry.
4. Keep `plan-mode-reference` one-shot behavior unchanged; it points to recoverable plan content and is not a recurring state block.
5. Rebuild changing goal/todo content before each request, replacing the previous value under the same key.
6. Make retries, tool-call continuation, steering, follow-up, compaction, branch/resume, and provider failover consume the same latest transient set.
7. After replacement semantics work, compress `plan-mode-active.md` to a compact contract:
   - read-only boundary and local-plan exception;
   - canonical plan path;
   - discover facts, ask only preferences;
   - required plan contents;
   - `resolve` exit protocol.
   Move edit-grammar instructions back to the edit tool and delegation mechanics back to the task tool.

## Verification

- Provider-spy test over ten user turns: exactly one message for each active context key in every serialized request.
- Persisted session history: recurring contexts do not grow linearly; resume reconstructs the current context from mode state.
- Plan mode: attempts to edit working-tree files remain blocked; creating/updating the local plan and applying through `resolve` still works.
- Goal mode: todo progress changes appear on the next provider request, with no stale prior todo snapshot.
- Vibe mode: required session control behavior remains available after several tool loops.
- Compaction and retry tests: active rules survive history rewrite and failed requests.
- Compare plan completion and forbidden-mutation rates before/after prompt compression, not just schema snapshots.

## Acceptance

Recurring mode overhead is constant per request, not proportional to prior turns. The latest state is present exactly once across normal, retry, resumed, and compacted sessions.
