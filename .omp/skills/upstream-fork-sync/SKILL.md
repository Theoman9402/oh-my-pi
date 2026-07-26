---
name: upstream-fork-sync
description: Heavy upstream sync. Use for large, risky, or conflict-heavy upstream changes that require fetch/review, merge/rebase, or semantic preservation of downstream patches.
---

# Upstream Fork Sync

Bring upstream changes into a downstream fork while preserving downstream intent.

<critical>
- Upstream is fetch-only. Verify its push URL before any publication step.
- `origin` is the only permitted downstream push destination.
- Review fetched changes before merge or rebase.
- Preserve custom behavior; resolve conflicts semantically.
- Never use whole-file `ours`/`theirs` or hard reset to bypass conflict review.
- Force-push only with explicit approval and `--force-with-lease`.
</critical>

## Branches

**Review-only request?** Fetch and report; leave the worktree and refs unchanged except fetched remote-tracking refs.

**Sync request?** Continue through candidate creation, application, verification, and promotion.

**Merge or rebase request?** The named local history operation is authorized; pushing remains separately gated.

## 1. Checkpoint

1. Identify the source remote, target branch or tag, and local branch.
2. Inspect remotes:

   ```bash
   git remote -v
   git remote get-url upstream
   git remote get-url --push upstream
   git config --get remote.pushDefault
   ```

3. Confirm `upstream` points to the intended public source and its push URL is blocked or otherwise non-routable.
4. Inspect the worktree:

   ```bash
   git status --short
   ```

5. Preserve uncommitted work with an explicit user-approved WIP commit, stash, or external backup. Never silently discard, overwrite, or hide it.

**Done when:** source, target, remote roles, push protection, and worktree state are recorded.

## 2. Baseline

1. Run the checks that define the current fork as healthy.
2. Use the repository's required checks; at minimum use `bun check` and affected-package tests when this repository is the target.
3. Record failures before touching the candidate.

**Done when:** the pre-sync state has a known-good result or an explicitly accepted baseline failure.

## 3. Fetch

Fetch explicitly; keep network acquisition separate from integration:

```bash
git fetch upstream
```

Do not replace this with an uninspected `git pull`. Resolve the target ref:

- explicit user target → use it;
- latest upstream branch → use the fetched default branch;
- latest release → inspect stable tags/releases and ask before choosing among candidates.

Record the target ref and object ID.

**Done when:** the target ref exists locally and its object ID is recorded.

## 4. Review

Compute topology and the complete change surface before mutation:

```bash
git log --graph --oneline --decorate --left-right HEAD...upstream/<target>
git diff --stat HEAD...upstream/<target>
git diff --name-status HEAD...upstream/<target>
git rev-list --left-right --count HEAD...upstream/<target>
```

1. Find the fork point:

   ```bash
   git merge-base HEAD upstream/<target>
   ```

2. Classify every changed path as upstream-only, downstream-only, or overlapping.
3. For overlapping paths, inspect upstream's change range and the local custom intent.
4. Summarize commit counts, changed areas, risky files, and the proposed strategy.
5. If the range is large, review aggregate changes first, then inspect individual commits for overlapping or behavior-critical paths.

**Done when:** every changed area has a review disposition and a strategy is justified by the observed topology.

## 5. Candidate

Create a rollback point from the clean committed state, then work on a candidate branch or disposable worktree:

```bash
git branch backup-<branch>-pre-upstream-<timestamp>
git switch -c sync-<branch>-<target>
```

Use a separate worktree for large, risky, or conflict-heavy updates. A backup branch does not preserve uncommitted files; checkpoint those before this step.

Read [`references/strategy-selection.md`](references/strategy-selection.md) when choosing among fast-forward, merge, and rebase.

**Done when:** the original branch has a named rollback point and all integration work is isolated from it.

## 6. Apply

Choose one strategy from the reviewed topology:

- no downstream commits ahead → `git merge --ff-only upstream/<target>`;
- preserve published commit identities → `git merge --no-edit upstream/<target>`;
- controlled downstream patch stack and rewrite approved → rebase the candidate onto the target;
- discard downstream commits → stop and obtain explicit confirmation before any destructive reset.

Never use `git merge -s ours`, blanket `-X ours`, blanket `-X theirs`, or `git reset --hard` to avoid understanding a conflict.

Read [`references/conflict-resolution.md`](references/conflict-resolution.md) when the operation stops on conflicts.

**Done when:** the candidate includes the target upstream history and every conflict has an explicit resolution.

## 7. Verify

1. Check for unresolved state:

   ```bash
   git status
   git ls-files -u
   git diff --check
   git diff --check backup-<branch>-pre-upstream-<timestamp>...HEAD
   git grep -n -E '^(<<<<<<<|>>>>>>>)' -- . ':!*.lock'
   ```

2. Re-run the baseline checks and affected-package checks.
3. Compare candidate behavior and diff against the rollback point.
4. Confirm the candidate's base and downstream commits:

   ```bash
   git log --graph --oneline --decorate -20
   git diff --stat backup-<branch>-pre-upstream-<timestamp>...HEAD
   ```

5. Record every conflict decision, skipped upstream change, test result, and follow-up risk.

**Done when:** checks pass, no conflict markers remain, the candidate's history is understood, and the report is complete.

## Gotchas

Read [`references/gotchas.md`](references/gotchas.md) when a dirty worktree, separate candidate, or failed check changes the safe sync path.

## 8. Promote

After verification, promote the candidate to the requested local branch. Keep the rollback branch until the user accepts the result.

Before any push:

```bash
git remote get-url --push upstream
git remote get-url --push origin
git config --get remote.pushDefault
```

Push only to `origin`, only after explicit user approval, and never with a blanket force option. If rewritten history requires it, explain the reason and use `--force-with-lease` only after confirmation.

**Done when:** the requested local branch is updated, the rollback point is retained or intentionally removed, and push status is explicitly reported.

## Report

Return a compact sync report:

- source remote and target ref/object ID;
- fork point and ahead/behind counts;
- selected strategy and why;
- backup branch or worktree;
- applied, adapted, and skipped areas;
- conflicts and semantic resolutions;
- baseline and post-sync checks;
- remaining risks;
- whether anything was pushed, and to which verified destination.

<critical>
- Fetch, review, then integrate; never hide the review inside `pull`.
- Protect the original branch before candidate work.
- Preserve downstream intent; understand both sides of every conflict.
- Push only `origin` after explicit approval.
</critical>
