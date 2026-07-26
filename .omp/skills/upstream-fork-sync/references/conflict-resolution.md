# Conflict resolution

A conflict is a semantic review task, not a side-selection task.

## Inspect both histories

For each conflicted path:

```bash
git status --short
git diff --cc -- <path>
git diff upstream/<old>..upstream/<target> -- <path>
git show backup-<branch>-pre-upstream-<timestamp>:<path>
```

Read the full local file and the relevant upstream change before editing. Identify the downstream invariant, the upstream intent, and whether the change is additive, replacing, or deleting behavior.

## Resolve

1. Preserve downstream behavior that remains intentional.
2. Incorporate compatible upstream fixes and API changes.
3. Adapt the implementation when both sides changed the same concept.
4. Keep a deletion only when the downstream behavior no longer needs the path.
5. Stage the resolved file and continue the chosen merge or rebase.

```bash
git add <path>
# merge: git merge --continue
# rebase: git rebase --continue
```

Never choose `ours` or `theirs` for a whole file without reading both versions. Never mark a conflict resolved while markers or unresolved index entries remain.

## Uncertainty

If the intended behavior cannot be established from code, tests, history, or project rules, stop the candidate and report the exact path, competing intents, and smallest user decision needed.

## Completion

A conflict pass is complete only when every conflicted path has a recorded resolution, the index is clean, `git diff --check` passes, and the affected checks pass.
