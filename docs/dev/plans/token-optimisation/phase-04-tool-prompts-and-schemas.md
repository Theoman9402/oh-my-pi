# Phase 04 — Remove schema overlap from tool descriptions

## Finding

Active tools send both rendered description prose and JSON wire schemas. The tool prompt corpus is about 18,176 estimated tokens; default essential descriptions plus hashline edit are about 7,233 before schemas.

Large descriptions mix load-bearing operational knowledge with possible schema overlap. The overlap must be established per tool:

- `eval.md` repeats `language`, `code`, `title`, `timeout`, and `reset` already described in `evalSchema` (`packages/coding-agent/src/tools/eval.ts:86-119`).
- `search-tool-bm25.md` repeats inputs and result keys exposed by schema.
- `todo.md` restates operation/field requirements represented by its input schema.
- `task.md` repeats some field mechanics, while assignment format and blank-context warnings are unique.
- `bash.md` is mostly complementary: its short schema describes fields while prose owns routing, shell limits, defaults, and recovery. Prune only overlap proven by ablation.
- `browser.md` and `read.md` expose helper/selector DSLs not representable in JSON Schema; they are weaker pruning candidates.

**Token cost:** high on every request while a tool is active.

**Quality value:** exact DSL syntax, cross-tool routing, defaults absent from schema, and incident-derived anti-patterns measurably prevent malformed calls.

**Tradeoff:** prompt reduction that increases one retry can cost more than the saved static text.

## Change

1. Make the tool-prompt optimization probe runnable against this checkout's live factories and exact wire schemas. During this audit, `probe-builtin.ts --tool bash --show` failed because the skill script could not resolve `@oh-my-pi/pi-coding-agent/config/settings`; fix the workspace entry or supply the live schema manually before accepting probe results.
2. For every default essential tool, then every discoverable tool:
   - capture exact rendered prompt and wire schema;
   - run multi-model/schema-only inference and no-summary ablation;
   - classify lines as schema overlap, unique decision guidance, or incident scar tissue;
   - inspect local history before deleting candidates.
3. First-pass targets:
   - remove field lists, required/optional prose, ranges, enum lists, and output-key lists already present in schema;
   - keep cross-tool routing and failure recovery;
   - keep one canonical example per non-obvious syntax shape;
   - collapse repeated `<critical>` recaps to rules not already stated above;
   - remove implementation machinery invisible to model decisions.
4. Keep hashline/read DSL grammar until dedicated malformed-call benchmarks show a safe shorter form. Prefer generated compact grammar/reference from shared constants over prose duplication.
5. Add per-tool description/schema token snapshots with explicit budgets. Fail only on growth; reductions still require behavioral evaluation.

## Verification

For each changed tool, run at least 30 representative calls across two model classes:

- correct tool selection versus shell/manual alternatives;
- schema-valid arguments;
- first-attempt success;
- retry count;
- forbidden-route rate;
- output correctness.

Required boundary cases: bash versus eval/grep/glob/launch, read selectors and internal URIs, hashline insert/swap/delete/block ops, task batch format, todo transitions, browser observe/action/navigation, and search-tool activation.

Compare total tokens through successful completion, not prompt tokens alone.

## Acceptance

Default active tool description tokens fall by at least 20% with statistically unchanged first-attempt success and no increase in average turns-to-completion. Any tool failing that bar reverts independently.
