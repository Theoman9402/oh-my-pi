# Phase 03 — Progressive disclosure for context files, skills, and rules

## Finding

`buildSystemPrompt()` inserts every loaded context file into `project-prompt.md` (`packages/coding-agent/src/system-prompt.ts:750-815`; `packages/coding-agent/src/prompts/system/project-prompt.md:9-17`). Deduplication is exact-content only. This checkout's repository `AGENTS.md` is about 4,590 estimated tokens; the observed user-level file adds about 3,289. Both are paid on questions, documentation work, and unrelated package tasks.

The repository file mixes truly global invariants with large topic-specific references: Bun APIs, worker architecture, generated catalog rules, TUI sanitization, testing, changelog, and release instructions. Most improve quality only when their topic is active.

**Token cost:** about 7.9k on this workstation before XML wrappers; unbounded across nested/provider context files.

**Quality value:** high. These rules prevent concrete repository-specific mistakes.

**Tradeoff:** progressive disclosure fails badly if matching is incomplete. Mandatory constraints must remain available before the first mutation, not discovered after a violation.

## Change

1. Compress root `AGENTS.md` to the always-applicable core:
   - package focus and terminology;
   - private-fork/no-push boundary;
   - critical code conventions;
   - Bun-over-Node default at summary level;
   - no commit unless asked;
   - required verification and changelog routing pointers.
2. Move detailed topic rules to canonical capability rules or skills with explicit names, descriptions, and globs:
   - catalog generation;
   - worker entry architecture;
   - TUI sanitization;
   - Bun file/process/stream APIs;
   - test authoring;
   - changelog/release.
3. Load path-matched rules automatically before relevant edits. For semantic topics not derivable from path—new worker, release, prompt authoring—advertise a compact inventory and require targeted `rule://`/`skill://` reads.
4. Add context-source budgets and diagnostics. Never silently truncate mandatory files. On overflow, keep a compact generated index with path, authority, scope, and read URI; mark the session as requiring targeted loading before mutation.
5. Keep deeper-directory precedence and exact dedupe. Add normalized block dedupe only for byte-equivalent rendered rules; do not attempt fuzzy deletion of differently worded requirements at runtime.
6. Treat user-level context as externally owned. Report its cost and support progressive loading, but do not rewrite it from repository code.

## Verification

Use clean sessions for these scenarios:

- Catalog model change loads generated-file rules before edit and never writes `models.json` directly.
- New worker change loads worker-host rules and preserves selector/fallback behavior.
- TUI renderer change loads sanitization rules.
- TypeScript implementation loads Bun/code-quality rules.
- Test-only change loads test rules.
- Documentation-only audit does not load worker, generated-model, or release bodies.
- Nested context files still override parent guidance.
- A question requiring no mutation starts with materially fewer context tokens.

For each scenario, record loaded sources, total context tokens, first-attempt rule compliance, and extra tool turns used to fetch rules.

## Acceptance

Always-on repository context falls below 2,000 estimated tokens. Every scenario loads the relevant detailed rule before its first mutation, and no scenario spends more tokens on discovery/retries than it saves.
