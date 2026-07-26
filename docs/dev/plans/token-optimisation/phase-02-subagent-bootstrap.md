# Phase 02 — Build a lean subagent bootstrap

## Finding

`runSubprocess()` creates a normal coding-agent session with context files, skills, rules, tools, and workspace data, then inserts `subagent-system-prompt.md` into the normal default prompt (`packages/coding-agent/src/task/executor.ts:2429-2466`). It may also inline the complete approved plan (`:2452-2456`). Each role adds its own agent prompt and output schema.

A child therefore receives primary-only material—user communication style, top-level delegation policy, todo workflow, workstation detail, Mermaid/LaTeX notes, delivery prose—despite having a narrow assignment and a mandatory `yield` protocol.

**Token cost:** roughly 15–25k static input per child depending on role/tools/context, multiplied by fan-out. Four children can spend a primary context window before reading task data.

**Quality value:** shared project rules, tool routing, output schema, and completion behavior are load-bearing.

**Tradeoff:** lean children can violate repository rules or return malformed results if the split removes required contracts. Savings also matter less when providers fully cache identical child prefixes, though context occupancy remains.

## Change

1. Introduce a dedicated subagent prompt builder instead of calling the full primary `buildSystemPrompt()` and splicing one block.
2. Keep only:
   - role prompt body;
   - assignment and shared context;
   - applicable repository/context rules;
   - active tool names plus tool-specific descriptions/schemas;
   - read-only/worktree boundary;
   - IRC peer data when enabled;
   - caller output schema and terminal `yield` contract;
   - exact blocker/completion rule.
3. Remove primary-only sections:
   - workstation CPU/GPU/terminal/model;
   - user-facing tone and final-response formatting;
   - top-level todo/delegation rules;
   - primary cleanup/changelog workflow unless the assignment requests it;
   - diagram/display capabilities;
   - instructions for tools absent from the child.
4. Stop inlining the full approved plan by default. The task assignment must carry the selected plan step and shared contract. Include the plan URI plus a targeted-read instruction only when the assignment says broader plan context is required.
5. Filter context files/rules by the assignment's explicit target paths when targets are available. If targets are absent or parsing is ambiguous, retain repository-level rules rather than guessing.
6. Deduplicate output instructions: machine-rendered caller schema is authoritative; role prose must not restate field names and yield placement.
7. Preserve byte-stable common prefixes across children of the same role so provider caching still works.

## Verification

- Capture rendered bootstrap tokens for every built-in agent before/after.
- Run one representative task per role: scout read-only mapping, reviewer structured findings, librarian sourced API answer, designer UI recommendation, and general task edit.
- Assert read-only roles cannot mutate and isolated workers cannot escape their worktree.
- Assert caller output schema overrides role-native schema exactly once.
- Run a repository-rule fixture where a child target falls under a deeper rule and confirm the child receives it; run an unrelated target and confirm it does not.
- Execute a four-child fan-out and compare total input, malformed yields, retries, and parent-side correction rate.
- Resume/revive a child session and confirm the same lean prompt rebuilds.

## Acceptance

At least 40% less non-task bootstrap input per built-in child, with no increase in malformed yields, repository-rule violations, or parent corrective turns in the representative suite.
