# Phase 09 — Stop restating machine-supplied output schemas

## Finding

Built-in agent frontmatter defines structured output fields, descriptions, enums, and required/optional status. The subagent bootstrap renders the normalized caller schema again (`packages/coding-agent/src/prompts/system/subagent-system-prompt.md:60-68`). Several role bodies then restate the same contract.

Examples:

- `reviewer.md` frontmatter defines finding/verdict fields, then lines 116–130 repeat every field and yield section.
- reviewer output examples teach XML/Markdown shapes even though the actual return is structured `result.data`.
- `commit/prompts/analysis-system.md` includes a full output JSON object and multiple complete JSON examples while the call uses a structured analysis tool/schema.
- agent-creation prompts encourage long role prompts that can duplicate global operating rules in every generated agent.

**Token cost:** hundreds to low thousands per subagent or auxiliary inference call.

**Quality value:** one canonical example can help weak models; repeated field inventories add little when schema is present.

**Tradeoff:** weak models may satisfy a prose example more reliably than JSON Schema alone. Remove in measured steps.

## Change

1. Establish one authority order:
   - caller output schema;
   - agent-native frontmatter schema when no caller override;
   - prose only for semantic decision criteria not representable in schema.
2. Render the effective schema exactly once. Do not include overridden native schema fields in the model prompt.
3. Remove role-body field lists, JSON skeletons, and repeated required/optional statements already present in the effective schema.
4. Keep at most one short canonical example only when ablation shows a weak-model formatting or classification benefit.
5. Put semantic rules in prose:
   - what counts as a review finding;
   - evidence thresholds;
   - priority meaning;
   - source-verification requirements.
6. Apply the same rule to commit analysis/changelog, memory extraction, agent creation, and any `completion(..., schema=...)` prompt.
7. Add an authoring check that reports schema field names repeated in adjacent prompt prose. Treat it as an audit warning, not an automatic deletion.

## Verification

- Reviewer: valid incremental findings plus final verdict, empty-findings case, caller schema override, and patch-anchoring quality.
- Librarian: exact source list/signatures and optional caveats.
- Designer/scout/task: role-specific valid terminal yield.
- Commit classification: type/scope/details correctness across feature, fix, refactor, and no-changelog fixtures.
- Run weak and strong model panels; record schema-valid rate, semantic accuracy, retries, and total tokens.
- Confirm no prompt contains both an effective rendered schema and a prose field-by-field copy.

## Acceptance

At least 25% reduction in role/structured-output prompt tokens, unchanged schema-valid rate, and unchanged semantic scoring. Retain one example only where ablation proves it earns its tokens.
