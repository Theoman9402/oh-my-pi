# Phase 07 — Budget and scope dynamic prompt injections

## Finding

`rebuildSystemPrompt()` concatenates independent dynamic sources before the project append prompt (`packages/coding-agent/src/sdk.ts:2387-2449`): memory instructions, auto-learn guidance, MCP server instructions, user append text, skills, rule inventory, and context files.

Current limits are local rather than aggregate:

- MCP instructions are capped at 4,000 characters **per server** with no total cap (`packages/coding-agent/src/sdk.ts:882-883,2411-2424`). Instructions are injected even when a server's tools are discovery-hidden.
- Hindsight mental models default to a 16,000-character rendered budget (`packages/coding-agent/src/hindsight/mental-models.ts:198-220`).
- Auto-learn guidance adds about 794 bytes plus optional learn guidance whenever those built-ins exist.
- Skill names/descriptions and rule descriptions scale with installed capabilities.
- Append-source changes rebuild the system prefix, reducing prompt-cache reuse.

**Token cost:** about 1k per maximum-size MCP server, about 4k for maximum memory, plus unbounded aggregate inventories.

**Quality value:** memory and server instructions may contain task-critical constraints; skill/rule descriptions enable discovery.

**Tradeoff:** relevance filtering can hide mandatory safety, authentication, or repository instructions. Source authority and recovery must remain explicit.

## Change

1. Represent every appended block as a typed source segment with origin, authority, scope, stable hash, estimated tokens, and recovery URI where available.
2. Apply source-specific and aggregate budgets before rendering. On overflow, preserve:
   - authority/safety header;
   - compact source index;
   - active/relevant entries;
   - recovery instruction.
3. MCP policy:
   - add a total server-instruction budget;
   - prioritize servers with active tools, then recently queried discovery matches;
   - include hidden-server name plus compact summary rather than full instructions;
   - reveal full instructions when that server activates;
   - clearly mark server-controlled text as untrusted.
4. Memory policy:
   - render task-relevant models first;
   - keep stable compact summaries in the system prompt;
   - expose full entries through memory tools/internal resources;
   - avoid rebuilding when the rendered stable segment is byte-identical.
5. Skills/rules policy:
   - bound inventories;
   - retain names and discriminating descriptions;
   - fetch bodies only when matched;
   - never silently omit an always-apply rule.
6. Auto-learn policy: reduce the always-on append to the decision boundary and let tool descriptions teach mechanics.
7. Extend `/context` diagnostics with each dynamic source and truncation decision.

## Verification

- Three MCP servers each return 4,000 characters: total stays within budget; active server instructions are complete; hidden servers remain discoverable.
- Activate a hidden MCP tool and confirm its server instructions appear before the next model call.
- Memory at, below, and above budget: relevant entries survive, recovery URI works, and stable content does not invalidate the prefix.
- Install many skills/rules: inventory remains bounded and a relevant item can still be found and loaded before action.
- Always-apply rules are never dropped by aggregate budgeting.
- Compare task correctness with full versus scoped injections for MCP, memory, and rule fixtures.

## Acceptance

Dynamic injections have explicit per-source and total budgets, deterministic priority, visible diagnostics, and a recovery path. No fixture loses an active mandatory instruction.
