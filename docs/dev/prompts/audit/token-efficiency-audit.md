# Token Efficiency Audit — GPT-5.6 Sol Prompt

## Source Prompt (Before Refinement)

> You are in the OMP fork dir. I want you to audit it and suggest modifications to make it less token heavy and improve its token efficiency without sacrificing inference quality.
>
> Create a detailed implementation plan under docs/dev/plans/token-optimisation/ for a new session to evaluate, improve and eventually implement. Your plan should be split into phases, with distinct, vertical slices where applicable, so that new sessions can iterate, expand and eventually implement. Consider that this is a private only fork (which will be reflected in this project's AGENTS.md for future work), no relation to upstream, we never want to push upstream, only pull when we feel like it.

---

## Refined Prompt

```text
You are auditing the OMP fork of oh-my-pi — a private fork with no upstream
relationship. We pull from upstream when we choose to; we never push. This
context will be reflected in AGENTS.md for future sessions.

Goal: Audit the codebase for token inefficiency. Identify every source of
unnecessary token consumption — bloated system prompts, redundant instructions,
excessive tool descriptions, verbose scaffolding, repeated rules, and any
pattern that costs tokens without improving output quality. Use your full
judgement on scope and priority; this is an open audit, not a checklist exercise.

For each finding, assess:
- token cost (estimated or relative)
- whether it measurably improves output quality or is dead weight
- the tradeoff of removing it

Deliver the audit as a phased implementation plan under
docs/dev/plans/token-optimisation/. Structure it so that a new session can pick
up any phase independently, evaluate the suggestion, and implement it without
needing the full audit context. Use vertical slices where a change is
self-contained. Each phase should state what to change, why it costs tokens
without proportional value, and what to verify after the change.
```

---

## Why This Prompt Works Better

### 1. Outcome-first, not process-first

The original told the model *how* to structure the plan in detail. The refined version defines the destination (a phased plan at a specific path) and success criteria (new sessions can pick up any phase independently). GPT-5.6 Sol's intent understanding handles the rest — it doesn't need you to enumerate every structural requirement.

> Evidence: OpenAI's GPT-5.6 Sol prompting guide states: "You often do not need to prescribe every step. Continue to provide domain context, hard constraints, approval boundaries, and success criteria." Leaner prompts improved internal coding-agent eval scores by 10–15% while cutting tokens 41–66%. — https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6

### 2. "Full judgement" instead of tight constraints

The phrase "Use your full judgement on scope and priority; this is an open audit, not a checklist exercise" deliberately avoids constraining the model to a specific audit methodology. GPT-5.6 Sol follows prompt contracts so literally that tight constraints burn reasoning tokens trying to reconcile edge cases. Giving it a clear goal with room to navigate produces better results than a cage of rules.

> Evidence: OpenAI warns that "conflicting rules can create more instability than missing detail" and advises against absolutes like "always" or "never." The Instruction Complexity Cliff research shows models reliably satisfy only ~3 stacked constraints per instruction — beyond that, compliance degrades non-linearly. — https://tianpan.co/blog/2026-04-17-instruction-complexity-cliff-llm-compliance

### 3. No "be concise" — direct outcome framing

The original didn't use "be concise" explicitly, but its language ("less token heavy") was vague. The refined version replaces it with concrete outcomes: "identify every source of unnecessary token consumption" and "costs tokens without proportional value." This gives the model a clear target without the compression bias that "be concise" triggers on GPT-5.6 (which is already concise by default).

> Evidence: "GPT-5.6 tends to be more concise by default than GPT-5.5. Broad brevity instructions such as 'Be concise' may be unnecessary and can sometimes make responses too brief." — OpenAI guide

### 4. Structured with clear authority boundaries

The refined prompt defines exactly what the model is authorized to do (full audit, use judgement) and what the deliverable looks like (phased plan at a specific path, vertical slices, independent phases). This is the compact autonomy policy pattern — one clear statement rather than repeated "do not" rules scattered throughout.

> Evidence: Anthropic's context engineering guidance recommends "the right altitude" — specific enough to guide behavior, flexible enough for the model to use strong heuristics. "Be thoughtful and keep your context informative, yet tight." — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

### 5. Explicit fork context without redundancy

The private fork relationship is stated once, clearly, with actionable implications ("we pull when we choose to; we never push"). The original stated it twice in slightly different ways. Repeated rules are the #1 source of token waste in system prompts.

> Evidence: OpenAI recommends "State each instruction once" and that removing repeated rules is the first optimization to try. — GPT-5.6 prompting guide

### 6. No XML scaffolding or process narration

The original had implicit process narration ("audit it and suggest modifications...create a detailed implementation plan...split into phases"). The refined version uses the compact outcome description pattern: goal → scope → deliverable format → success criteria. No `<persistence>` blocks, no enumerated step lists.

> Evidence: "The era of the 'mega-prompt' is ending. Remove the repeated rules. Delete the style instructions that do not change the output." — Multiple sources including OpenAI's July 2026 guide and Thoughtworks Technology Radar (April 2026) identifying "agent instruction bloat" as a caution technique.

---

## Research Sources

| # | Source | Date | Key Finding |
|---|--------|------|-------------|
| 1 | [OpenAI GPT-5.6 Sol Prompting Guide](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6) | Jul 2026 | Lean prompts: 10-15% score gain, 41-66% token reduction, 33-67% cost reduction |
| 2 | [Anthropic: Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Sep 2025 | Context is finite; find smallest high-signal token set; "informative, yet tight" |
| 3 | [Instruction Complexity Cliff](https://tianpan.co/blog/2026-04-17-instruction-complexity-cliff-llm-compliance) | Apr 2026 | Models reliably satisfy ~3 constraints; U-shaped attention; middle-of-prompt instructions ignored |
| 4 | [Thoughtworks: Agent Instruction Bloat](https://www.thoughtworks.com/radar/techniques/agent-instruction-bloat) | Apr 2026 | Context files accumulate; hand-written > LLM-generated; progressive disclosure |
| 5 | [Decrypt: Stop Over-Prompting](https://decrypt.co/373439/openai-new-gpt-5-6-prompt-guide-chatgpt) | Jul 2026 | "Define the destination, set stopping conditions, get out of the way" |
| 6 | [GPT-5.6 + Claude Fable 5 Guide](https://www.thepromptindex.com/gpt-5-6-and-claude-fable-5-prompting-guide.html) | Jul 2026 | Both labs: "prompt less, not more"; subtractive workflow |
| 7 | [DevGENT: Lean System Prompts](https://devgent.org/en/gpt-5-6-prompting-lean-system-prompts-autonomy-bounds-and-ptc-routing-en/) | Jul 2026 | What to cut vs keep; autonomy boundaries; migration checklist |
| 8 | [Beeble: Complex Prompts Making GPT-5.6 Dumber](https://beeble.com/en/blog/your-complex-ai-prompts-are-actually-making-gpt-5-6-dumber) | Jul 2026 | Every word is a constraint; five pages of instructions → reasoning capacity wasted on parsing |
