# private-fork-governance-handoff

## Goal
Continue the private-fork governance audit and implementation for the OMP downstream. The canonical plan is `docs/dev/plans/private-fork-governance/audit.md`; read it before proceeding.

## Sources
Read before proceeding.

- None. The authoritative local source is `docs/dev/plans/private-fork-governance/audit.md`.

## Suggested skills
- `git-workflow` - safe remote roles, fetch/merge sequencing, and public-destination protection.
- `system-prompts` - context-file and `AGENTS.md`/`RULES.md` semantics if instruction design changes continue.

## State
- Model A is selected: public GitHub fork only. `Theoman9402/oh-my-pi` was created as a public fork of `can1357/oh-my-pi`. The sibling `../pi` is the existing public fork `Theoman9402/pi` of `earendil-works/pi`.
- OMP local remotes in `.git/config`: `origin` fetch/pushes `https://github.com/Theoman9402/oh-my-pi.git`; `upstream` fetches `https://github.com/can1357/oh-my-pi.git` and pushes to `no_push://public-omp`; `remote.pushDefault=origin`; `main` tracks `upstream/main`.
- Pi local remotes in `../pi/.git/config`: `origin` is `https://github.com/Theoman9402/pi.git`; `upstream` fetches `https://github.com/earendil-works/pi.git` and pushes to `no_push://public-omp`; `remote.pushDefault=origin`; `main` tracks `upstream/main`.
- Smoke verification in both repos: `git push --dry-run upstream main` failed locally with the exact error `fatal: protocol 'no_push' is not supported`. No real push or fetch was performed.
- Root `AGENTS.md` now contains `## Downstream Repository` with no-upstream-write, destination, visibility, release, documentation-boundary, and divergence guidance. Its `## Gotchas` section says not to add a pointer to `.omp/RULES.md` solely for downstream governance visibility; independent hard invariants may still use `.omp/RULES.md`.
- `docs/dev/plans/private-fork-governance/audit.md` now records Model A, the configured remotes, governance in root `AGENTS.md`, and the decision not to duplicate governance in `.omp/RULES.md`. It does not forbid independent `.omp/RULES.md` use.
- No project `.omp/RULES.md` or `.omp/AGENTS.md` was created. `.omp/` currently contains `skills/` and `commands/`.
- Known OMP working-tree changes: root `AGENTS.md`, the governance audit, and the handoff file. `docs/dev/` remains untracked; `docs/prompts/` remains untracked and is still a documented boundary violation. No commit was made.
- Before the remote setup, OMP was reported behind cached upstream by 2306 commits; no fetch has refreshed that count.

## Decisions
- Use public Model A, not Model B or C. Everything committed to the OMP fork, including plans, history, workflows, and Actions logs, is public.
- `origin` is the permitted public-fork push destination. `upstream` is integration-only and locally push-blocked with `no_push://public-omp`.
- Keep infrequent downstream git governance in root `AGENTS.md`; do not duplicate it in `.omp/RULES.md` or add a redundant pointer. Reserve `.omp/RULES.md` for independent hard invariants if a later demonstrated need arises.
- Do not create `.omp/AGENTS.md`, custom merge drivers, mandatory hooks, linters, ADR frameworks, or hidden instruction locations without evidence.

## Constraints
- Never push, publish, release, create issues or pull requests, create contribution branches, or comment on public OMP or any remote resolving to it.
- Never put private or confidential content in the public fork. Treat all hosted OMP fork content and Actions logs as public.
- Verify push destinations before any push. Do not run `bun run release` during ordinary maintenance.
- Keep downstream-specific instructions and documentation in root `AGENTS.md` and `docs/dev/**`; other `docs/**` content is upstream-owned.
- Do not merge yet: the audit requires a clean worktree and durable backup first. Use `git fetch upstream` before merging `upstream/main`; do not rely on `git fetch upstream main` followed by a merge.
- Do not commit unless explicitly requested. No release or publication command without an explicit destination-specific plan.

## Blocker
No hard blocker. The next decision point is which content in untracked `docs/prompts/` is retained; under Model A, any retained and committed content is public.
Assumption: `docs/prompts/` may contain material worth retaining, but its contents were not classified in this session; inspect before moving or deleting anything.

## Dead ends
- A public GitHub fork cannot be made private. A private standalone repository would be Model B, and a public fork plus private repository would be Model C; neither is part of this setup.
- Using `.omp/RULES.md` merely to keep the infrequent downstream git safeguards visible was rejected as redundant because project context is included in the provider-facing system prompt on every model request.
- Do not interpret that decision as banning `.omp/RULES.md`; independent hard invariants remain a valid use.
- No upstream fetch, merge, push, release, pre-push hook, or hosted publication was performed.

## Next steps
1. Read `docs/dev/plans/private-fork-governance/audit.md` and treat its current Model A / root-`AGENTS.md` wording as canonical.
2. Inspect `docs/prompts/`; move retained material to `docs/dev/prompts/` or delete confirmed-obsolete material.
3. Create `docs/dev/README.md`, `docs/dev/upstream-integration.md`, and `docs/dev/divergence.md`; record Model A and the remote roles without adding governance duplication to `.omp/RULES.md`.
4. Apply the exact revised wording to `docs/dev/plans/token-optimisation/phase-03-context-progressive-disclosure.md`.
5. Start a fresh OMP session and verify root `AGENTS.md` is loaded with the downstream governance section. Do not require a sticky-rule check for this governance design.
6. Confirm all retained documents are public-safe, establish durable backup/history, and obtain a clean worktree before upstream integration.
7. Run the controlled sequence from the audit: `git fetch upstream`, inspect status and `HEAD...upstream/main`, merge `upstream/main`, run required checks, and push only to `origin` if explicitly requested.
