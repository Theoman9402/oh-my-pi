---
name: upstream-sync
description: Routine upstream sync. Use when the user wants to fetch upstream, review a normal branch update, and directly fast-forward or merge it into the current downstream branch; route divergent, conflict-heavy, or semantic downstream-preservation work to upstream-fork-sync.
---

# Routine Upstream Sync

Fetch, review, and integrate an ordinary upstream update on the current branch. Keep the routine path direct: preserve dirty work, fetch once, inspect topology, integrate, restore, and verify.

<critical>
- Treat `upstream` as fetch-only. Verify its push URL before any publication step.
- Treat `origin` as the only downstream publication destination; pushing requires explicit user approval.
- Keep fetch/review separate from integration and record the review before changing the branch.
- Routine integration runs on the current branch after dirty work is preserved. A disposable worktree belongs to the heavy-sync escalation, not this path.
- Preserve every dirty and untracked downstream file through the selected preservation method and verify restoration before cleanup.
</critical>

## 1. Establish the target

1. Identify the current branch and requested upstream target.
2. Inspect remote roles and push safety:

   ```bash
   git remote -v
   git remote get-url upstream
   git remote get-url --push upstream
   git remote get-url --push origin
   git config --get remote.pushDefault
   ```

3. Inspect the worktree:

   ```bash
   git status --short
   ```

4. Use an explicit user target. Otherwise resolve the fetched default branch; do not guess a release tag.

**Done when:** current branch, target ref, remote roles, push safety, and dirty state are recorded.

## 2. Fetch and review

Fetch only the upstream remote:

```bash
git fetch upstream
```

Resolve the target object and review its complete change surface before integration:

```bash
git rev-parse upstream/<target>
git merge-base HEAD upstream/<target>
git rev-list --left-right --count HEAD...upstream/<target>
git diff --stat HEAD...upstream/<target>
git diff --name-status HEAD...upstream/<target>
```

For this repository, record run-specific results in `docs/handoffs/upstream-sync-simplification-handoff.md` under the appropriate result heading. Keep run history out of this skill.

**Done when:** target OID, merge base, ahead/behind counts, diff stat, and changed paths are recorded before branch mutation.

## 3. Preserve and integrate

- For a review-only request, stop after Step 2 and report the review.
- For an authorized integration with a dirty worktree, state the preservation action, then use a temporary include-untracked stash unless the user selected an explicit WIP commit or external backup:

  ```bash
  git stash push --include-untracked --message pre-upstream-sync-<timestamp>
  ```

  Keep the stash reference until restoration is verified.
- Read the topology as `local-only upstream-only` commit counts from `git rev-list --left-right --count`:
  - `0 N`: the local branch is an ancestor; use `git merge --ff-only upstream/<target>`.
  - `N 0`: upstream adds nothing beyond the local branch; report that no integration is needed.
  - `N M`: both histories contain unique commits; use `git merge --no-edit upstream/<target>` for a routine merge.
- Run the selected operation on the current branch. If it conflicts, requires semantic downstream decisions, or the target cannot be safely classified as routine, abort the operation, restore preserved work, and hand off to `upstream-fork-sync`.

**Done when:** the branch is up to date with the target or the routine merge completed, with no in-progress operation and preserved work still recoverable.

## 4. Restore and verify

After a successful integration, restore a temporary stash with `git stash apply <stash-ref>`. Verify the restored dirty and untracked paths, then drop the temporary stash. Keep it when restoration is incomplete.

Verify the resulting state:

```bash
git status --short
git ls-files -u
git rev-parse HEAD
git rev-parse upstream/<target>
git rev-list --left-right --count HEAD...upstream/<target>
git diff --check
```

Check the integrated range with `git diff --check <pre-sync-head>...HEAD` when the operation changed `HEAD`, and search for conflict start/end markers. Recheck both push URLs and `remote.pushDefault` before reporting publication status.

**Done when:** the target is present in the current branch, no unresolved index entries or conflict markers remain, preservation is restored or explicitly retained, checks and known failures are recorded, and push status is explicit.

## Report

Return:

- source remote, target ref, and target OID;
- pre-sync and final `HEAD`, merge base, and ahead/behind counts;
- selected operation and topology reason;
- preservation method and restoration status;
- changed-path summary and verification results;
- remaining known failures or risks;
- whether anything was pushed, and to which verified destination.
