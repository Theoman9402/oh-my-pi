# Phase 05 — Compress the core system and project wrappers

## Finding

`packages/coding-agent/src/prompts/system/system-prompt.md` is 18,036 bytes, about 4,509 estimated tokens, before dynamic content. It repeats the same behavioral contracts across General, Exploration, Delegation, Workflow, Delivery Contract, Completeness, Evidence, Yielding, and final Critical sections.

Concrete overlaps include:

- tool routing in both core system and individual tool descriptions;
- persistence/completion in workflow, contract, completeness, yielding, and final critical;
- grounding/no-fabrication in General, Research, Contract, and Evidence;
- todo/delegation mechanics in core system and tool prompts;
- verification rules repeated by repository/user context;
- prose explaining why a prohibition exists after the decision is already clear.

`project-prompt.md` adds CPU, GPU, kernel, terminal, architecture, model, date, and cwd. OS/cwd sometimes change commands; GPU/CPU/terminal almost never change agent decisions.

**Token cost:** 4.5k plus roughly 441 static wrapper tokens on every request; dynamic inventory/context is additional.

**Quality value:** high for a compact set of invariants, low for repeated rationale and display capabilities.

**Tradeoff:** the prompt is incident-shaped. A purely stylistic rewrite can revive premature yielding, unsafe tool routing, or fabricated evidence.

## Change

1. Build a rule inventory mapping every core sentence to:
   - unique required decision;
   - duplicate of another source;
   - model/display capability;
   - rationale/example;
   - conditional rule better owned elsewhere.
2. Keep one compact top-level contract for:
   - follow instruction authority;
   - use tools for discoverable facts;
   - inspect before editing and preserve user work;
   - complete the literal request without scope shrink/creep;
   - verify changed behavior;
   - report grounded results and blockers.
3. Let tool descriptions own tool-specific mechanics. Let repository context own repository conventions. Let mode prompts own mode overrides.
4. Collapse workflow and delivery sections into one ordered procedure plus one pre-yield checklist. State each invariant once; repeat only a short critical recap at the end.
5. Remove default LaTeX/color/Mermaid coaching unless a rendering mode specifically requires it.
6. Replace the workstation block with decision-relevant facts by default:
   - OS/platform and shell semantics;
   - architecture only when binary/toolchain behavior depends on it;
   - cwd and date;
   - model only when model-specific prompting is active.
   Expose full hardware through a command/internal resource instead of the hot prompt.
7. Preserve Handlebars gating and byte-stable ordering for provider prompt caching.

## Verification

- Local-history review for every deleted rule; retain incident scar tissue until a regression test exists.
- Render matrix: native/non-native tools, task absent/present, skills/rules absent/present, secrets, MCP discovery, custom prompt, workspace tree, Windows/POSIX.
- Behavioral suite covering premature yield, missing lookups, destructive actions, exported-symbol references, verification selection, incomplete checklists, and tool routing.
- Compare task success, unsupported claims, retries, and total completion tokens across at least two strong model families and one weak subagent model.
- Confirm prompt-cache prefix remains stable across identical session state.

## Acceptance

At least 35% reduction in the static primary core plus project wrapper, with unchanged completion and verification behavior in the evaluation matrix. Keep rejected cuts documented only in test names or local history—not as new prompt prose.
