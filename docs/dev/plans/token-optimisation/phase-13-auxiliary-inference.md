# Phase 13 — Audit auxiliary inference paths independently

## Finding

Several features issue separate model requests with their own prompts and inputs. They do not bloat every coding turn, but can dominate the operation that invokes them:

- commit analysis/changelog/summary/map-reduce under `packages/coding-agent/src/commit/` can make several calls per commit and pass diffs plus project context;
- autoresearch replaces the system prompt with a rendered template containing the full base prompt plus experiment instructions/state (`packages/coding-agent/src/autoresearch/index.ts:357-415`); the main template is 5,209 bytes and is rebuilt per iteration;
- title, task-label, speech rewrite, memory extraction/consolidation, review requests, and auto-thinking prompts are smaller but may receive oversized source input;
- agent-creation prompts can generate permanently bloated agent definitions.

Research prompts under `packages/snapcompact/research/prompts/` are not runtime inference inputs and are excluded.

**Token cost:** conditional. Commit and autoresearch are high; title/label/speech are low per call but frequent.

**Quality value:** feature-specific. A single global rewrite is unsafe.

**Tradeoff:** auxiliary prompts often target smaller models that benefit from stronger examples and explicit output constraints.

## Change

Treat each feature as an independent sub-slice; implement and revert separately.

### Commit pipeline

- Count calls and tokens for conventional analysis, summary retries, changelog generation, and agentic/map-reduce modes.
- Pass context files once as a compact release-rule extract, not the entire coding-agent operating context to every map item.
- Avoid replaying the same diff to stages that can consume the structured result of an earlier stage.
- Apply Phase 09 schema deduplication.

### Autoresearch

- Split stable experiment protocol from changing run state.
- Keep the stable base/policy byte-identical for cache reuse.
- Send only latest/best/relevant run deltas; retain full ledger in the experiment store and expose a handle.
- Remove general coding workflow text from the autoresearch overlay when already present in the embedded base prompt.

### Tiny side calls

- Cap source input to decision-relevant excerpts for title, task label, speech rewrite, auto-thinking, and memory extraction.
- Do not send tool schemas or project context to calls that cannot use tools or repository facts.
- Keep prompts under explicit token budgets suitable for their local/small models.

### Agent creation

- Generate role-specific semantic criteria only; inherit common runtime/yield/tool contracts from the shared subagent bootstrap.
- Enforce a body budget and report duplicated global rules before saving.

## Verification

- Produce per-feature before/after call graphs and token totals.
- Commit fixtures: small change, multi-package change, changelog-required change, large map-reduce diff; compare classification/changelog quality.
- Autoresearch fixture over 20 iterations: state tokens plateau, best-result selection and discard/keep behavior remain correct, resume works.
- Title/label/speech fixtures preserve exact output contracts and quality on long source inputs.
- Memory extraction retains user corrections and durable facts while dropping transient tool chatter.
- Generated agent runs one representative task with schema-valid output.

## Acceptance

Each sub-slice reduces total tokens for its complete operation without extra model calls or lower feature-specific scoring. Do not merge sub-slices into one rollout gate.
