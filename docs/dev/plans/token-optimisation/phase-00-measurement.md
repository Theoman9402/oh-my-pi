# Phase 00 — Rendered-context measurement and budgets

## Finding

`computeNonMessageBreakdown()` reports four broad categories: system prompt, system context, tools, and skills (`packages/coding-agent/src/modes/utils/context-usage.ts:144-165`). It cannot attribute growth to a context file, tool description versus schema, dynamic append source, or recurring custom message. Source-file byte counts also miss Handlebars branches and active settings.

**Token cost:** indirect but high. Regressions of thousands of tokens can hide inside a stable category.

**Quality value of current behavior:** the existing `/context` total is useful and should remain cheap for normal UI updates.

**Tradeoff:** finer accounting adds code and deterministic fixtures. Keep it on-demand; do not put per-source tokenization on the streaming hot path.

## Change

1. Add a source-aware prompt manifest returned alongside `BuildSystemPromptResult` in `packages/coding-agent/src/system-prompt.ts`:
   - core system template;
   - skill inventory;
   - always-apply rules;
   - project wrapper;
   - each context file by path;
   - workspace tree;
   - active-repo context;
   - custom/append prompt.
2. Extend tool accounting in `packages/coding-agent/src/modes/utils/context-usage.ts` to split, per tool:
   - name/framing;
   - rendered description;
   - wire schema.
3. Add a model-request diagnostic that counts transient/custom messages by `customType`, including repeated occurrences. Do not alter persisted session format.
4. Add a developer-only JSON report reachable from the existing `/context` command or a package-local script. Required scenarios:
   - native tools, default essential set;
   - discovery mode with no activated extras;
   - one and ten plan-mode turns;
   - goal and vibe mode;
   - one subagent of each built-in role;
   - non-native/in-band tool dialect;
   - three MCP servers with maximum-size instructions;
   - maximum memory payload.
5. Add configurable budgets for tests only, not runtime rejection:
   - static primary prefix;
   - default tool descriptions and schemas;
   - one recurring mode message;
   - subagent bootstrap;
   - compaction prompt.

## Verification

- Unit check: every manifest component sums to `computeNonMessageTokens()` within the estimator's framing tolerance.
- Render each scenario twice and require byte-identical JSON.
- For ten plan-mode turns, report both the latest mode message and total mode-message tokens present in the serialized request.
- Change one tool description, one context file, and one MCP instruction independently; only the matching source row should change.
- Confirm normal status-line updates still use the memoized aggregate path and do not recompute the manifest.

## Acceptance

A fresh session can name the five largest model-visible sources from one command and compare before/after values for any later phase without manually counting Markdown bytes.
