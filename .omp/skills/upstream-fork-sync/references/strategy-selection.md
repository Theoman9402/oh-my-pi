# Strategy selection

Choose from the observed graph, not preference.

## Fast-forward

Use when the current branch is an ancestor of the target and no downstream commits need preservation:

```bash
git merge --ff-only upstream/<target>
```

This keeps every upstream commit unchanged and fails without mutation if the graph diverged.

## Merge

Use when downstream commits are published or their identities must remain stable:

```bash
git merge --no-edit upstream/<target>
```

This preserves both histories and records the integration point. Expect a merge commit when both sides contain unique commits.

## Rebase

Use when downstream commits form a controlled patch stack and rewriting their IDs is approved:

```bash
git rebase --onto upstream/<target> <fork-point>
```

This keeps downstream patches visibly on top of upstream, but may require `origin` force-with-lease after review. Create a backup first.

## Hosted fork sync

`gh repo sync OWNER/FORK -b <branch>` updates the hosted fork directly. Use only when the user explicitly requests a hosted sync and the branch has no local review requirement. It is not a substitute for inspecting a local candidate; conflict handling or `--force` can overwrite the destination.

## Discard

A hard reset discards downstream commits. It requires explicit confirmation naming the commits or changes to lose. Never infer discard intent from being behind upstream.

## Escalation

Use a disposable worktree when the range is large, the candidate touches agent instructions, or conflict resolution is uncertain. Use a direct candidate branch for routine fast-forwards and small merges.
