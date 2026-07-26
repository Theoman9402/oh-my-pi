# Phase 14 — Verify non-native tool catalog duplication before pruning

## Finding

`packages/ai/src/dialect/` contains 12 model-specific protocol prompts totaling about 3,801 estimated tokens; one dialect is selected for a non-native tool-calling model. `renderToolInventory()` produces per-tool descriptions, TypeScript-like parameters, and examples (`packages/ai/src/dialect/inventory.ts:16-28`). `renderToolCatalog()` can produce compact JSON function catalogs (`packages/ai/src/dialect/catalog.ts:9-28`).

Coding-agent prompt assembly intentionally uses compact tool-name list mode only for provider-native tools. Non-native sessions inline full `# Tool:` sections (`packages/coding-agent/src/system-prompt.ts:738-742`). Inline descriptor mode also strips provider-schema descriptions to avoid one known duplicate path.

The remaining risk is request-shape dependent: a dialect/provider adapter may receive tool name/description/schema in more than one representation, or the selected protocol prompt may restate generic tool-call rules shared across dialects. Source similarity alone does not prove wire duplication.

**Token cost:** low-to-medium protocol overhead plus the full active catalog; high when many tools are active.

**Quality value:** extremely high. Exact delimiters, escaping, call IDs, parallel-call rules, and result framing are required for any tool use.

**Tradeoff:** one missing syntax rule can turn a modest saving into complete tool-call failure.

## Change

1. Capture the final serialized provider request for every owned dialect with two tools and with the default active tool set.
2. Annotate each occurrence of tool name, description, schema, example, and protocol instruction by source:
   - coding-agent system inventory;
   - provider-native tool schema;
   - in-band catalog;
   - dialect prompt.
3. Remove only proven duplicate representations. Keep exactly one complete catalog and one protocol grammar.
4. Extract genuinely shared dialect rules into a generated compact base only if final rendered bytes shrink; source-code deduplication without wire reduction is not a token optimization.
5. Keep model-specific syntax deltas adjacent and testable. Do not normalize dialects that require different escaping, thinking delimiters, or call/result ordering.
6. Prefer compact schema serialization and omit empty descriptions/examples. Preserve property descriptions that change argument semantics.
7. Add per-dialect rendered token snapshots and tool-call conformance fixtures.

## Verification

For Anthropic/XML, DeepSeek, Gemini, Gemma, GLM, Harmony, Hermes, Kimi, MiniMax, Qwen, and any other registered dialect:

- no-tool answer;
- one call;
- parallel calls;
- nested JSON/string escaping;
- tool error/result;
- thinking plus tool call;
- malformed call recovery.

Compare first-attempt parse rate, exact arguments, retries, and total tokens. Test both two-tool and large-catalog requests.

## Acceptance

No tool metadata appears twice in a serialized non-native request unless the conformance evaluation proves both copies necessary. Every dialect retains baseline first-attempt tool-call success; otherwise leave that dialect unchanged.
