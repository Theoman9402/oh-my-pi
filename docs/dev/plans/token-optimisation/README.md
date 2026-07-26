# Token optimisation audit and implementation plan

## Purpose

Reduce model-visible input without reducing correctness, tool-use reliability, or recovery behavior. This is a private fork: use local history and local evaluation evidence; do not create upstream work or push anywhere.

Each phase is a self-contained vertical slice. A session may implement one phase without reading the others. `phase-00` supplies common measurement infrastructure but is not a prerequisite: every phase names a local measurement fallback.

## Measurement basis

OMP's default estimator is UTF-8 bytes divided by four (`packages/agent/src/tokenizer.ts:3-16`). Estimates below use that same rule. Exact provider tokenization varies. Static prefixes may receive provider cache discounts, but cached tokens still occupy the context window and can incur cache-read charges.

Observed source inventory:

| Surface | Current size | Estimated tokens | Runtime shape |
|---|---:|---:|---|
| Core system template | 18,036 B | 4,509 | Every normal model request |
| Project wrapper template | 1,761 B | 441 before inserted content | Every normal model request |
| Repository `AGENTS.md` | 18,360 B | 4,590 | Inserted in full for this checkout |
| User-level `AGENTS.md` observed during audit | 13,154 B | 3,289 | External to repository; inserted in full on this workstation |
| Tool prompt corpus (`src/prompts/tools`) | 72,623 B | 18,176 | Active subset sent with tool schemas |
| Default essential tool descriptions, including hashline edit | about 28.9 KB | about 7,233 | Before JSON schemas; normal default session |
| Conditional system/notices corpus | 85,108 B | 21,298 | Subsets injected by modes, retries, and side channels |
| Built-in agent prompt corpus | 20,621 B | 5,156 | Role-dependent, multiplied by subagent fan-out |
| Compaction prompt corpus | 6,630 B | 1,658 | Separate summarization calls |
| In-band dialect prompt corpus | 15,204 B | 3,801 | One dialect for non-native tool calling |

A normal session in this checkout therefore starts around **12.8k estimated tokens before skills, rules, tool descriptions, and tool JSON schemas**. Default essential tool descriptions add about **7.2k** before schemas. This is a context estimate, not a provider bill estimate.

## Finding summary and recommended order

| Phase | Finding | Cost | Quality value now | Removal tradeoff | Priority |
|---|---|---|---|---|---|
| [00](phase-00-measurement.md) | Existing `/context` categories hide per-source regressions | Indirect | Diagnostics only | Instrumentation maintenance | P0 enabler |
| [01](phase-01-transient-mode-context.md) | Plan/goal/vibe context is appended as a new persisted message every primary prompt | Plan mode: about 2.6k per turn, cumulative | Keeps mode rules current | Incorrect replacement could drop active constraints | P0 |
| [02](phase-02-subagent-bootstrap.md) | Every subagent inherits the full primary prompt/context plus role and often full plan | Roughly 15–25k static input per spawn, multiplied by fan-out | Strong consistency, much content irrelevant to child | Lean children may miss project or coordination rules | P0 |
| [03](phase-03-context-progressive-disclosure.md) | Full context files are unconditional; this checkout contributes 7.9k before wrappers | High, every request | Project rules are load-bearing | Progressive loading can miss a rule | P0 |
| [04](phase-04-tool-prompts-and-schemas.md) | Tool descriptions and schemas contain tool-specific overlap candidates; default descriptions need per-tool ablation | Default descriptions about 7.2k plus schemas | DSL examples and incident scar tissue are valuable | Over-pruning increases malformed calls/retries | P1 |
| [05](phase-05-core-system-prompt.md) | Core prompt repeats completion, evidence, workflow, and tool-routing rules | 4.5k every request | Several rules prevent real failures; many restate each other | Broad cuts can weaken persistence or safety | P1 |
| [06](phase-06-tool-result-scaffolding.md) | Todo mutations replay the full list; repeated state is quadratic across updates | Medium to very high on long plans | Full state prevents identifier drift | Delta-only results require a reliable `view` recovery path | P1 |
| [07](phase-07-dynamic-injections.md) | MCP, memory, skills, rules, and auto-learn append independently with weak aggregate budgets | Up to 1k/server MCP, about 4k memory, unbounded totals | Some content is task-critical | Relevance filtering can hide mandatory instructions | P1 |
| [08](phase-08-notices-and-reminders.md) | Workflow/orchestration/mode notices restate base and tool contracts | 0.2–2.6k each; some repeat | Triggered guidance corrects model-specific failures | Removing scar tissue can restore loops | P2 |
| [09](phase-09-structured-output-prompts.md) | Agent and commit prompts restate machine-supplied output schemas | Hundreds to low thousands per auxiliary call | Examples anchor formatting | Weak models may need one canonical example | P2 |
| [10](phase-10-compaction-and-handoffs.md) | Summary prompts demand preservation and allow deletion simultaneously; summaries retain recoverable state | Grows after every compaction | Resume fidelity | Aggressive pruning can lose pending work | P2 |
| [11](phase-11-advisor.md) | Advisor has a 1.4k system prompt and receives thinking plus expanded diffs every update | High when advisor enabled | Independent review can catch defects | Reduced transcript may weaken diagnosis | P2 |
| [12](phase-12-tool-activation-lifetime.md) | Discovered tools remain active for the whole session | Each activation adds description + schema to every later turn | Avoids rediscovery | Short leases can cause repeated discovery and cache churn | P2 |
| [13](phase-13-auxiliary-inference.md) | Commit, autoresearch, title, speech, and label paths have independent prompt/input bloat | Conditional; corpus is tens of KB | Feature-specific quality | Changes need feature-specific evaluation | P3 |
| [14](phase-14-non-native-dialects.md) | Non-native tool calling may duplicate catalog information and carries protocol prose | About 1.2–1.5k per selected dialect plus catalog | Protocol instructions are load-bearing | A small syntax loss can break all tool calls | P3, verify before changing |

## Global acceptance rules

Apply these to every phase:

1. Record before/after estimated tokens for the exact rendered request, not Markdown source alone.
2. Separate context-window reduction, uncached input reduction, and provider-billed reduction.
3. Run the changed behavior end to end. A prompt snapshot or typecheck alone is insufficient.
4. Compare malformed tool calls, retries, task completion, and correctness—not token count alone.
5. Before deleting prompt text, inspect local history for incident-driven scar tissue. Private-fork status changes upstream workflow, not the value of local history.
6. Prefer deletion or progressive disclosure over moving the same text to another always-loaded prompt.
7. Keep a compact recovery route (`read`, `view`, internal URI, or targeted reminder) for information removed from the hot path.
8. Reject a change that saves tokens only by causing extra model turns or tool retries.

## Explicit non-findings

- Hashline file tags, line numbers, and edit grammar cost tokens but protect precise edits. Keep unless an edit-success benchmark proves a replacement safer and smaller.
- `read` structural summaries, output truncation, artifacts, deferred tool discovery, and advisor duplicate-context collapsing are existing token-saving mechanisms, not deletion targets by default.
- Research-only Markdown, changelogs, and user documentation do not consume inference tokens unless imported into a prompt; source size alone is not a finding.
- User task content and necessary tool payload data are not waste. Optimize repeated framing and recoverable duplication around them.
