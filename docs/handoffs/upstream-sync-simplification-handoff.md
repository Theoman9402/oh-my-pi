# upstream-sync-simplification-handoff

## Goal
The next session should test and adopt the simpler routine upstream-sync path the user requested: fetch the upstream remote, review the divergence, and avoid disposable worktrees unless the change is genuinely large, risky, or conflict-heavy. Then simplify the model-invoked skill around that routine path while retaining the heavy-sync safeguards as disclosed reference.

Prior detailed sync history is in `docs/handoffs/upstream-fork-sync-handoff.md`.

## Sources
Read before proceeding.

- None.

## Suggested skills
- `upstream-sync` - routine model-invoked skill for direct fetch/review/fast-forward or ordinary merge.
- `git-workflow` - routine fetch, branch, merge, rollback, and publication sequencing.
- `writing-great-skills` - tighten the description, default path, branching, and progressive disclosure.

## State
- Routine skill: `.omp/skills/upstream-sync/SKILL.md`; its description starts `Routine upstream sync` and routes divergent, conflict-heavy, or semantic downstream-preservation work to the heavy skill.
- Heavy skill: `.omp/skills/upstream-fork-sync/SKILL.md`; its description starts `Heavy upstream sync` and retains candidate, conflict, and downstream-patch safeguards.
- Heavy references remain under `.omp/skills/upstream-fork-sync/references/`; `SKILL.md` is 194 lines.
- Remote roles: `origin` is the public downstream fork; `upstream` is public OMP and fetch-only; upstream push is `no_push://public-omp`; `remote.pushDefault=origin`.
- Main: branch `main`, `HEAD=403931b9687012ec83754e0826b7d97f551e5a17`, `upstream/main=403931b9687012ec83754e0826b7d97f551e5a17`, divergence `0 0`, fork point is `HEAD`.
- Main contains restored downstream work: `M AGENTS.md`; untracked `.omp/skills/upstream-fork-sync/`, `docs/dev/`, and `docs/handoffs/`. The upstream merge was a fast-forward; no commit or push was performed.
- Test candidate retained at `C:/AI_Tasks/Forks/oh-my-pi-sync-test-20260726`, branch `sync-main-upstream-main-20260726-review`, `HEAD=6d2a8de1bb60d01b376f7f9e7cf082042149fbf0`. Rollback branch: `backup-main-pre-upstream-20260726-review` at the old `HEAD`.
- Candidate fast-forwarded with `git merge --ff-only upstream/main`; its worktree is clean. It is a test artifact, not the required permanent sync layout.
- Biome is installed as project dependencies in main and candidate. Rustup/Cargo is installed under `C:\Users\theod\.cargo\bin`; `rust-toolchain.toml` pins `nightly-2026-04-29`.
- Candidate validation: Biome and TypeScript checks passed; Rust check fails on `crates/pi-uutils-ctx/src/lib.rs:32` with `error: field \`stdin_fd\` is never read` under `-D warnings`.
- Candidate coding-agent tests ended `16 pass / 72 fail` because the Windows native addon was absent. The native build then required CMake and libclang; the wrapper also failed when invoking `napi.exe` through Bun with `Expected ";" but found ""` before direct NAPI invocation reached those prerequisites.

## Review-only fetch result
- Timestamp: `2026-07-26T22:25:22+01:00`.
- Scope: inspected the dirty `main` worktree and remote safety, then ran `git fetch upstream`; no merge, rebase, branch switch, commit, promotion, or push was performed.
- Pre-fetch branch/state: branch `main`; `M AGENTS.md`; untracked `.omp/skills/upstream-fork-sync/`, `docs/dev/`, and `docs/handoffs/`.
- Remotes: `origin` fetch/push `https://github.com/Theoman9402/oh-my-pi.git`; `upstream` fetch `https://github.com/can1357/oh-my-pi.git`; upstream push `no_push://public-omp`; `remote.pushDefault=origin`.
- Pre-fetch topology: `HEAD=3047c27c332c5629c8e063283d349384c10c9a56`; cached `upstream/main=6d2a8de1bb60d01b376f7f9e7cf082042149fbf0`; merge base `3047c27c332c5629c8e063283d349384c10c9a56`; divergence `0 2554`.
- Fetch result: `upstream/main` advanced to `403931b9687012ec83754e0826b7d97f551e5a17`.
- Post-fetch topology: merge base `3047c27c332c5629c8e063283d349384c10c9a56`; divergence `0 2556`; `HEAD` remained `3047c27c332c5629c8e063283d349384c10c9a56`; branch remained `main`.
- Change surface from `git diff main...upstream/main`: `1745 files`, `+210449`, `-47100`; statuses `1324 M`, `375 A`, `44 D`, `1 R051`, `1 R085`. The complete changed-path set is the output of `git diff --name-status main...upstream/main`.
- Post-fetch state exactly matched the pre-fetch dirty-file state; only fetched remote-tracking refs changed.

## Direct upstream merge result
- Temporarily preserved the dirty and untracked downstream work with stash `pre-upstream-merge-20260726`, ran `git merge --ff-only upstream/main`, restored the work, verified it, and dropped the temporary stash.
- Final topology: `main` and `upstream/main` both point to `403931b9687012ec83754e0826b7d97f551e5a17`; `git rev-list --left-right --count main...upstream/main` is `0 0`.
- No unresolved conflict markers were found.
- `git diff --check 3047c27c332c5629c8e063283d349384c10c9a56..HEAD` reports an upstream-range whitespace issue at `python/robomp/tests/test_github_events.py:978` (`new blank line at EOF`); it was not modified.

## Decisions
- [Assumption] The user's “fetch remote” simpler approach means routine `git fetch upstream` plus local topology review, not `git pull` or hosted `gh repo sync`.
- Routine syncs use the current branch after dirty work is explicitly preserved; candidate branches and disposable worktrees remain escalation tools for heavy or uncertain integration.
- Review-only fetch must remain safe on a dirty worktree; no local branch or file mutation is needed beyond fetched remote-tracking refs.
- The verification block now checks unresolved index entries with `git ls-files -u`, checks both the clean worktree and rollback-to-candidate range with `git diff --check`, and searches conflict start/end markers without treating bare `=======` as a marker.
- Do not expand ordinary sync scope into native-addon setup unless the user explicitly authorizes environment setup.
- Publication remains separate: never push upstream; push only to verified `origin` after explicit approval. Do not commit unless explicitly requested.

## Constraints
- Do not silently stash, commit, reset, overwrite, or promote over the dirty main worktree.
- Do not repeat the disposable-worktree plus dependency-install process for routine upstream updates; the routine skill uses direct current-branch integration after preservation.
- Do not treat missing Biome, Cargo, CMake, libclang, native addons, or other check prerequisites as merge defects.
- Use `bun check` rather than `tsc`; retain the repository's Rust toolchain pin.
- Keep durable downstream governance in root `AGENTS.md` and `docs/dev/**`; do not create duplicate rules files.
- Do not push, release, create upstream issues/PRs, or publish during skill testing.

## Blocker
Main still contains unprotected uncommitted and untracked downstream work, including the skill under test. A routine integration test cannot promote into `main` until that work is explicitly preserved or the user authorizes another safe preservation method. Review-only `git fetch upstream` remains available without resolving that blocker.

## Dead ends
- Treating a disposable candidate worktree as the default upstream-sync workflow created unnecessary repetition; use direct current-branch integration for routine fast-forwards and ordinary merges.
- Installing dependencies and attempting native-addon builds was outside the original skill test; CMake/libclang remain unresolved environment prerequisites.
- Plain `git diff --check` on a clean candidate missed a committed-range whitespace error; the skill now checks the rollback-to-candidate range.
- `git grep -n -E '^(<<<<<<<|=======|>>>>>>>)'` falsely matched Markdown setext headings; the skill now searches conflict start/end markers and checks the index separately.
- Do not use uninspected `git pull`, whole-file `ours`/`theirs`, blanket merge strategies, `git reset --hard`, or `gh repo sync` as substitutes for local review.

## Next steps
1. Run a fresh routine review-only smoke test using `.omp/skills/upstream-sync/SKILL.md` and record the result in this handoff.
2. Exercise direct current-branch fast-forward and dirty-worktree preservation only for authorized routine integrations; route conflicts or semantic downstream decisions to `upstream-fork-sync`.
3. Re-read both skills and run a scoped contradiction search after future edits. Do not push or commit.
