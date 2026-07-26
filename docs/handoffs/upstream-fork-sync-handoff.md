# upstream-fork-sync-handoff

## Goal
Test the new model-invoked skill `.omp/skills/upstream-fork-sync/SKILL.md` in the OMP downstream fork. The next session should exercise the review-only and integration gates, diagnose unsafe or ambiguous behavior, and edit the skill or its references when evidence shows a defect. Do not publish upstream changes.

Harness session/agent lookup id: `Main`.

## Sources
Read before proceeding.

- [1] https://docs.github.com/en/pull-requests/how-tos/work-with-forks/syncing-a-fork - upstream fork synchronization: fetch upstream, merge locally, and push separately; `gh repo sync --force` can overwrite a destination branch.
- [2] https://git-scm.com/docs/git-fetch.html - `git fetch` downloads refs/objects and updates remote-tracking branches without integrating them into the checked-out branch.
- [3] https://git-scm.com/docs/git-merge - merge incorporates another history, may stop for conflicts, and warns that non-trivial uncommitted changes are difficult to reconstruct after `git merge --abort`.
- [4] https://git-scm.com/docs/git-rebase - rebase reapplies commits onto a new base and rewrites the downstream commit identities; conflicts stop the operation for resolution.

## Suggested skills

- `upstream-fork-sync` - the skill under test; governs remote safety, review, candidate integration, conflict handling, verification, and promotion.
- `git-workflow` - supports remote-role checks, fetch/merge sequencing, branch protection, and publication safety.
- `diagnosing-bugs` - use if a skill test takes an unsafe action, skips a gate, mishandles conflicts, or reports an incorrect result.
- `system-prompts` and `writing-great-skills` - use only if the fix changes model-facing instructions, metadata, progressive disclosure, or completion criteria.

## State

- New untracked skill directory: `.omp/skills/upstream-fork-sync/`.
  - Canonical entrypoint: `.omp/skills/upstream-fork-sync/SKILL.md`.
  - Reference files: `.omp/skills/upstream-fork-sync/references/strategy-selection.md` and `.omp/skills/upstream-fork-sync/references/conflict-resolution.md`.
  - The entrypoint metadata is `name: upstream-fork-sync`; its description triggers on inspecting upstream updates, updating a fork, merging/rebasing upstream changes, resolving fork divergence, or maintaining downstream patches.
  - It explicitly separates review-only, sync, and named merge/rebase requests; requires `git fetch upstream` before review; creates a rollback branch/candidate; chooses fast-forward, merge, or rebase from observed topology; requires semantic conflict resolution; runs `git status`, `git diff --check`, conflict-marker checks, baseline checks, and affected tests; and permits downstream publication only to `origin` after explicit approval.
- No executable code or full script was provided. The canonical implementation is Markdown in the files above plus the two references.
- Current branch and cached topology: branch `main`; `HEAD` `3047c27c332c5629c8e063283d349384c10c9a56`; cached `upstream/main` `667111575ebba136dadfd6989379e7f67e0d40d9`; `git rev-list --left-right --count HEAD...upstream/main` returned `0 2306`. The count is cached and must not be treated as current until a new fetch.
- Remote safety currently observed:
  - `origin` fetch/push: `https://github.com/Theoman9402/oh-my-pi.git`.
  - `upstream` fetch: `https://github.com/can1357/oh-my-pi.git`.
  - `upstream` push: `no_push://public-omp`.
  - `remote.pushDefault`: `origin`.
- Current working tree is not clean: `AGENTS.md` is modified; `docs/dev/` is untracked; `.omp/skills/upstream-fork-sync/` is untracked; `docs/handoffs/` is untracked. Existing `docs/handoffs/private-fork-governance-handoff.md` records the earlier public-fork governance work and should be read rather than duplicated.
- No upstream fetch, merge, rebase, push, release, commit, or project check was performed while authoring this skill. The new files are not committed.

## Decisions

- Treat the public OMP repository as upstream integration-only. Verify `git remote get-url --push upstream` before any publication step; it currently resolves to the local blocking URL `no_push://public-omp`.
- Treat `origin` as the only allowed downstream publication destination, and require explicit user approval before pushing. If rewritten history makes a push necessary, use `--force-with-lease` only after explicit confirmation.
- Fetch and review are separate from integration. Do not hide review inside `git pull`; GitHub's documented command-line flow fetches upstream, then merges locally, with a separate push needed to update the hosted fork [1][2][3].
- Protect the original branch before integration. Uncommitted work may be preserved only through an explicit user-approved WIP commit, stash, external backup, or disposable worktree; never silently discard or overwrite it.
- Prefer fast-forward when no downstream commits are ahead; use a merge when published commit identities should remain; use rebase only for a controlled downstream patch stack when history rewriting is authorized [3][4].
- Resolve conflicts semantically. Never use whole-file `ours`/`theirs`, `git merge -s ours`, blanket `-X ours`/`-X theirs`, or `git reset --hard` to bypass review.
- The earlier governance decision is Model A: a public GitHub fork with downstream rules in root `AGENTS.md`; use `docs/handoffs/private-fork-governance-handoff.md` and `docs/dev/plans/private-fork-governance/audit.md` as the existing record.

## Constraints

- Never push to `upstream`, create upstream issues/PRs, comment on public OMP, or publish confidential material. All committed fork content and hosted CI logs are public.
- Do not run `bun run release` during maintenance. Do not commit unless the user explicitly asks.
- Do not merge or rebase the current `main` while its uncommitted changes are unprotected. A backup branch preserves committed state only; it does not preserve uncommitted files.
- Refresh the stale topology with an explicit `git fetch upstream` before making claims about current upstream divergence. Do not assume the cached `2306` count is live.
- For this repo, use `bun check` rather than `tsc`; run affected-package tests after integration when the candidate is safe to test.
- Keep fixes scoped to the skill and its references. Use the canonical prompt-authoring guidance; avoid adding code, hooks, merge drivers, aliases, or duplicate governance files without evidence.
- Do not create a bundle unless the next session explicitly requests bundle output; this handoff is the requested `docs/handoffs/upstream-fork-sync-handoff.md` document.

## Blocker

No hard blocker prevents a review-only test. The current worktree is dirty and the cached upstream ref is stale, so a real merge/rebase test requires either explicit protection of the uncommitted work or an isolated disposable worktree. The next session must not silently stash, commit, reset, or overwrite the current changes.

Diagnostic targets for the test:

- The skill must inspect remote push URLs and worktree state before fetching or integrating.
- A review-only request must leave the worktree and local branches unchanged except for fetched remote-tracking refs.
- A sync request must not mutate the original branch before creating a rollback point and candidate.
- The skill must not choose among ambiguous release targets without asking; it must record the target object ID and review changed paths before integration.
- Conflict handling must inspect both sides, preserve downstream invariants, incorporate upstream fixes, and stop/report when intent is uncertain; it must not side-select whole files.
- Verification must include no unresolved index entries or conflict markers, `git diff --check`, baseline/affected checks, and an explicit report of resolutions, skipped changes, risks, and push status.

## Dead ends

- `gh repo sync` is not the default for this local workflow: GitHub documents it as a hosted synchronization shortcut, and conflicts can require a pull request or `--force`; it does not replace a local reviewed candidate [1].
- An uninspected `git pull` is rejected because it combines network acquisition and integration before the changed surface is reviewed.
- Whole-file `ours`/`theirs`, `git merge -s ours`, blanket merge strategy options, and `git reset --hard` are rejected because they can silently discard downstream behavior or upstream fixes.
- The cached `HEAD...upstream/main` result must not be reused as proof of current divergence; no fetch was run during this skill-authoring session.
- The prior governance session rejected duplicating infrequent downstream rules in `.omp/RULES.md`; do not reopen that decision while testing this skill unless new evidence shows the existing root `AGENTS.md` boundary fails.

## Next steps

1. In the fresh session, read this handoff, `docs/handoffs/private-fork-governance-handoff.md`, `docs/dev/plans/private-fork-governance/audit.md`, root `AGENTS.md`, `.omp/skills/upstream-fork-sync/SKILL.md`, and both reference files.
2. Start with a review-only test and record exact behavior: `git status --short`, `git remote -v`, `git remote get-url --push upstream`, `git remote get-url --push origin`, `git config --get remote.pushDefault`, `git branch --show-current`, and the pre-fetch topology.
3. If network refresh is appropriate, run only `git fetch upstream`; record the fetched target OID, `git merge-base main upstream/main`, `git rev-list --left-right --count main...upstream/main`, `git diff --stat`, and `git diff --name-status` before any integration.
4. For a real sync test, first obtain explicit approval for the preservation method or use a disposable worktree. Create the rollback branch and candidate required by the skill; never operate directly on the dirty original branch.
5. Exercise the selected fast-forward/merge/rebase path only when the request authorizes it. For conflicts, use the procedures in `references/conflict-resolution.md` and record each semantic decision.
6. Verify the candidate with `git status`, `git diff --check`, the conflict-marker check from the skill, `bun check`, and affected tests. Compare the candidate against the rollback point and report any skipped or adapted upstream areas.
7. If behavior violates the skill contract, edit the smallest canonical Markdown section, re-read the changed range, and run a scoped contradiction search. Do not add a workaround outside the skill.
8. Do not push during testing. If the user later explicitly authorizes publication, recheck both push URLs and `remote.pushDefault`, then push only to `origin`; retain the rollback branch until acceptance.
