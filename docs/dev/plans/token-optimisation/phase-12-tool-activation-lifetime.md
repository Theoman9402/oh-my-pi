# Phase 12 — Give discovered tools an explicit lifetime

## Finding

Default essentials are `read`, `bash`, `launch`, `edit`, `write`, `glob`, and `eval` (`packages/coding-agent/src/tools/index.ts:391-413`). Discovery hides other built-ins, but `search-tool-bm25.md` states that searches add tools to the active set for the rest of the session. Every activation adds name, rendered description, and JSON schema to later requests, even when the capability was needed once.

This creates a ratchet: long sessions progressively lose context to inactive tool contracts. Conversely, removing tools changes the provider tool list/system prefix and can reduce cache hits or make a later forced `tool_choice` invalid.

**Token cost:** tool-dependent; about 70–1,500 description tokens plus schema per activated built-in, every later turn.

**Quality value:** persistence avoids repeated discovery and keeps multi-turn workflows stable.

**Tradeoff:** short lifetimes can cause rediscovery turns, cache churn, and unavailable tools at the moment the model expects them.

## Change

1. Add activation provenance and lifetime to active-tool state:
   - essential/explicit/restored/forced: session lifetime;
   - discovery activation: current user turn by default;
   - optional `lifetime: "session"` on `search_tool_bm25` for known multi-turn work.
2. Define "turn" as the full agent loop for one user prompt, including all tool calls, continuations, async deliveries consumed before the next user prompt.
3. At the next user prompt, expire turn-scoped tools unless:
   - currently forced by tool choice/mode;
   - an active job depends on them;
   - explicitly promoted to session lifetime.
4. Return compact activation/expiry information once; avoid a repeated catalog dump.
5. Evaluate the essential set separately. `launch` is a strong discovery candidate because most tasks do not run long-lived processes. Keep `eval` essential unless routing benchmarks show that hiding it does not increase complex bash scripts.
6. Preserve restored selections only when the original provenance required session lifetime.
7. Batch tool-list changes at prompt boundaries to preserve a stable prefix during a turn.

## Verification

- Discover and use a one-shot tool: available immediately and throughout the current tool loop, absent on the next user turn.
- Discover with session lifetime: survives later turns and resume.
- Forced todo/resolve/goal tool choices never reference an expired schema.
- Active async job continues to expose required control tools until settled.
- Compare sessions using browser once, LSP repeatedly, and GitHub intermittently: discovery calls, tool tokens, task turns, and cache behavior.
- Evaluate moving `launch` out of essentials; ensure dev-server tasks discover it before attempting background bash.

## Acceptance

One-shot discovery no longer permanently grows the tool catalog. Across representative long sessions, total successful-completion tokens fall without increasing unavailable-tool errors or average discovery turns.
