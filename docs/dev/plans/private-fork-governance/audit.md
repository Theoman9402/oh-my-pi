# Private Fork Governance Audit

- **Workspace:** downstream OMP development repository
- **Public upstream:** `https://github.com/can1357/oh-my-pi`
- **Canonical path:** `docs/dev/plans/private-fork-governance/audit.md`
- **Status:** Option A hosted fork and remote protection configured; governance context applied in root `AGENTS.md`; documentation normalization and controlled upstream integration pending

## 1. Operating model

Hard invariants:

1. Public OMP is an integration source only.
2. Never push, publish, release, contribute, open issues or PRs, create contribution branches, or comment on public OMP from this workspace.
3. Private source changes may be made anywhere required.
4. Downstream-specific instructions and documentation may exist only in:
  - root `AGENTS.md`
  - `docs/dev/**`
5. All other `docs/**` content is upstream-owned.
6. No private or confidential content may be pushed to a public repository.
7. Governance must remain proportionate to demonstrated risks.

Terminology:

- **GitHub fork:** a repository inside GitHub’s fork network.
- **Private downstream repository:** a separate private repository that merges public OMP but is not recognised by GitHub as a fork.

## 2. Verified current state

Local inspection found:

- `origin` uses `https://github.com/Theoman9402/oh-my-pi.git` for fetch and push.
- `upstream` fetches `https://github.com/can1357/oh-my-pi.git` and has push URL `no_push://public-omp`.
- Local `main` tracks `upstream/main`; default pushes target `origin`.
- No private remote or active pre-push hook exists.
- Root `AGENTS.md` contains downstream no-upstream-write and visibility guidance; `.omp/RULES.md` is intentionally absent.
- `docs/dev/` is untracked and contains the token-optimisation plan.
- `docs/prompts/` is untracked and violates the private-documentation boundary.
- Any branch-behind count is relative to cached `upstream/main` until a permitted fetch occurs.

Current public OMP confirms:

- Root `AGENTS.md` is loaded through the `agents-md` provider.
- OMP supports a nearest `.omp/RULES.md` always-apply sticky rule, but this downstream intentionally does not use it for governance.
- OMP includes normal project context in the provider-facing system prompt on every model request; root `AGENTS.md` is sufficient for this infrequent workflow.
- A root `.omp/AGENTS.md` can shadow root `AGENTS.md` because the native provider wins at the same depth.
- `@path` references expand inline before context injection; they are not progressive disclosure.
- `.omp/` is tracked.
- `scripts/release.ts` commits, tags, and pushes branch and tag refs to `origin`.

GitHub account inspection found:

- `Theoman9402/pi` is a GitHub-recognised public fork.
- No hosted OMP fork is currently visible on the connected account.

## 3. GitHub fork feasibility

GitHub requires every fork of a public repository to remain public. An individual fork’s visibility cannot be changed. Detaching a fork allows a private standalone repository, but it then ceases to be a GitHub fork.

Therefore public OMP cannot have a private GitHub-recognised fork on GitHub.com.

Choose one model before any hosted push:

| Model | Hosted repositories | Use when |
|---|---|---|
| **A — Public fork** | Public GitHub fork only | All pushed source, plans, history, Actions logs, and metadata may be public. |
| **B — Private downstream** | Private standalone repo only | Any downstream material must remain private. GitHub will not label it a fork. |
| **C — Dual repository** | Clean public fork + private standalone repo | The fork relationship and private storage both have independent value. |

Recommendation:

- Use **A** only when public visibility is acceptable.
- Use **B** when confidentiality matters.
- Use **C** only when the public fork has a real purpose; it adds operational complexity and must never receive private commits.

Official references:

- `https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-permissions-and-visibility-of-forks`
- `https://docs.github.com/en/pull-requests/how-tos/work-with-forks/fork-a-repo`
- `https://docs.github.com/en/pull-requests/how-tos/work-with-forks/detaching-a-fork`
- `https://cli.github.com/manual/gh_repo_fork`

## 4. Material risks

### R1 — Public-upstream push exposure: resolved

`upstream` pushes fail locally through `no_push://public-omp`. Normal and release pushes still target the public fork at `origin`, so publication remains destination-sensitive.

### R2 — Fork visibility may contradict privacy: critical

A genuine fork of public OMP is public. Private plans, source, credentials, workflows, or history must never enter its reachable commits.

### R3 — Private work is not durable: high

`docs/dev/` has no Git history or off-machine backup. Tracking private documents is safe only in private history or storage.

### R4 — Existing documentation-boundary violation: medium

Inspect `docs/prompts/`; move retained material to `docs/dev/prompts/` or delete confirmed obsolete material.

### R5 — Fork identity and governance context: resolved

Option A remote roles are configured, and root `AGENTS.md` carries the downstream no-upstream-write and visibility constraints. A separate sticky rules file is not used.

### R6 — Governance overlaps token Phase 03: low

Phase 03 owns root `AGENTS.md` compression. Governance should define what downstream content survives, not perform the entire compression independently.

## 5. Recommended instruction design

### 5.1 Root `AGENTS.md`

Keep existing upstream content until token Phase 03 performs its planned compression. Add or maintain:

```markdown
## Downstream Repository

This repository is a public downstream fork of public OMP. Public OMP is an integration source, not a contribution destination.

- Never push, publish, release, create issues or pull requests, create contribution branches, or comment on `https://github.com/can1357/oh-my-pi` or any remote resolving to public OMP.
- Before any push, verify the destination. `origin` is the public fork push destination; `upstream` is fetch-only.
- Never push private or confidential content to this public fork.
- Do not run release or publication commands unless the user explicitly requests one and every destination has been verified.
- Downstream governance instructions and documentation may exist only in root `AGENTS.md` and `docs/dev/**`.
- Private source changes may be made wherever required. Minimise unnecessary divergence, preserve compatible upstream improvements, and record durable merge-sensitive divergence in `docs/dev/divergence.md`.
```

Because `AGENTS.md` is included in the provider-facing system prompt on every model request, do not add a pointer to `.omp/RULES.md` solely to keep these infrequent git safeguards visible. `.omp/RULES.md` remains valid for independent hard invariants; do not duplicate the downstream governance text there.

After choosing A, B, or C, add one sentence defining the role and visibility of each hosted remote.

Do not create `.omp/AGENTS.md` or a custom merge driver.

### 5.2 Durable downstream documentation

```text
docs/dev/
├── README.md
├── upstream-integration.md
├── divergence.md
├── prompts/                         # only if retained
└── plans/
    ├── token-optimisation/
    └── private-fork-governance/
        └── audit.md
```

Do not pre-create ADR, architecture, operations, or policy structures without real documents.

### 5.3 Source divergence

- Private changes may modify inherited source, scripts, configuration, or workflows.
- Prefer the smallest coherent design, not artificial isolation.
- Record only durable, merge-sensitive divergence.
- Keep upstream-sync commits distinct from private feature commits where practical.
- Resolve conflicts semantically: understand upstream, preserve the downstream requirement, adopt compatible improvements, adapt the implementation, then verify both behaviours.
- Never apply blanket “ours wins” or “upstream wins” rules.

### 5.4 Releases and CI

Changing `origin` does not prove package registries, Actions, tags, binaries, or release services are safe.

- Do not run `bun run release` during ordinary maintenance.
- Before the first hosted push, inspect inherited workflows, Actions settings, secrets, publication targets, and visibility.
- Under A, pushed content and Actions logs are public.
- Under B or C, verify that only the private repository receives private commits.

## 6. Remote topology

### Model A — public GitHub fork

```text
origin    -> public user fork
upstream  -> public OMP, fetch-only
```

Create with GitHub UI or:

```bash
gh repo fork can1357/oh-my-pi --clone=false
```

Inside the existing clone, `gh repo fork --remote` can create the fork and configure remotes. Inspect the result; do not assume names.

Expected GitHub metadata:

- `isFork=true`
- `isPrivate=false`
- parent is `can1357/oh-my-pi`

### Model B — private standalone downstream

```text
origin    -> private standalone repo
upstream  -> public OMP, fetch-only
```

Expected GitHub metadata:

- `isFork=false`
- `isPrivate=true`

### Model C — dual repository

```text
origin       -> private standalone repo
upstream     -> public OMP, fetch-only
public-fork  -> clean public GitHub fork
```

The public fork must remain free of private commits. Prefer GitHub Sync Fork or `gh repo sync` when it needs only upstream updates.

### Public-upstream push protection

After `upstream` exists:

```bash
git remote set-url --push upstream no_push://public-omp
```

Do not use an empty `pushurl`; Git can fall back to the fetch URL.

Safe check:

```bash
test "$(git remote get-url --push upstream)" = "no_push://public-omp"
git push --dry-run upstream main
```

Expected: local failure because protocol `no_push` is unsupported.

Under B or C, set normal pushes to private `origin`:

```bash
git config remote.pushDefault origin
```

Defer a URL-aware pre-push hook until repeated mistakes, multiple clones, or public-write credentials justify it.

## 7. Rejected approaches

| Approach | Reason |
|---|---|
| Make a public OMP fork private | GitHub does not permit an individual public fork to change visibility. |
| Fork publicly, then detach and make private | The result is standalone, not a GitHub fork. |
| Call a private standalone repo a GitHub fork | Technically false. |
| `.omp/AGENTS.md` | Outside the approved boundary and can shadow root `AGENTS.md`. |
| `@docs/...` as progressive disclosure | OMP expands imports inline. |
| Empty `remote.<name>.pushurl` | Git can fall back to the fetch URL. |
| `git fetch upstream main` then merge `upstream/main` | The explicit fetch may update only `FETCH_HEAD`; use `git fetch upstream`. |
| `push.default=current` or `nothing` as the main safeguard | `current` pushes; `nothing` does not block explicit refspecs. |
| Untracked or gitignored `docs/dev/` | No durable history or recovery. |
| Local branch as the only private copy | No off-machine durability. |
| Custom merge driver, linter, ADR framework, or mandatory hook now | No evidence yet justifies the maintenance burden. |
| Duplicate downstream governance in `.omp/RULES.md` | Root `AGENTS.md` is sufficient for infrequent governance; `.omp/RULES.md` remains available for independent hard invariants. |

## 8. Token-plan reconciliation

Phase 03 owns root `AGENTS.md` compression. It must preserve:

- downstream identity and selected hosted model;
- public OMP as an integration source, never a contribution destination;
- no-upstream-write and visibility constraints directly in root `AGENTS.md`; no `.omp/RULES.md` pointer;
- pointer to `docs/dev/**` for downstream material;
- deliberate source divergence.

### Required Phase 03 edit

**File:** `docs/dev/plans/token-optimisation/phase-03-context-progressive-disclosure.md`
**Section:** `## Change`, item 1

**Before:**

```markdown
1. Compress root `AGENTS.md` to the always-applicable core:
   - package focus and terminology;
   - private-fork/no-push boundary;
   - critical code conventions;
   - Bun-over-Node default at summary level;
   - no commit unless asked;
   - required verification and changelog routing pointers.
```

**After:**

```markdown
1. Compress root `AGENTS.md` to the always-applicable core:
   - package focus and terminology;
   - downstream identity, governance constraints in root `AGENTS.md`, and the `docs/dev/**` documentation boundary;
   - critical code conventions;
   - Bun-over-Node default at summary level;
   - no commit unless asked;
   - required verification and changelog routing pointers.
```

No other token-plan edit is required.

Before Phase 02 changes subagent bootstrap, verify that mutating child agents retain root `AGENTS.md` context. Change Phase 02 only if inspection proves otherwise.

## 9. Implementation plan

### Phase 0 — Choose and create hosted model

Before any private push:

1. Determine whether downstream content may be public.
2. Select A, B, or C.
3. Confirm repository name, owner, and authenticated GitHub account.
4. Confirm no name conflict.
5. Record the decision in `docs/dev/README.md`.

For A:

```bash
gh repo fork can1357/oh-my-pi --clone=false
gh repo view Theoman9402/<FORK_NAME> --json isFork,isPrivate,parent,nameWithOwner,url
```

Do not continue if private visibility was expected.

For B:

```bash
gh repo create Theoman9402/<PRIVATE_REPO_NAME> --private
```

For C, create both separately and never share private commits with the public fork.

### Phase 1 — Configure safe remotes

Inspect existing remotes first.

For A, expected final state:

```text
origin    https://github.com/Theoman9402/<FORK_NAME>.git
upstream  https://github.com/can1357/oh-my-pi.git
```

Then:

```bash
git remote set-url --push upstream no_push://public-omp
git config remote.pushDefault origin
git config branch.main.remote upstream
git config branch.main.merge refs/heads/main
```

For B:

```bash
git remote rename origin upstream
git remote set-url --push upstream no_push://public-omp
git remote add origin <PRIVATE_REMOTE_URL>
git config remote.pushDefault origin
git config branch.main.remote upstream
git config branch.main.merge refs/heads/main
```

For C, use the B commands and add:

```bash
git remote add public-fork <PUBLIC_FORK_URL>
```

Verify:

```bash
git remote -v
git remote get-url --push upstream
git config --get remote.pushDefault
git config --get branch.main.remote
git config --get branch.main.merge
```

### Phase 2 — Add governance context

- Maintain the root `AGENTS.md` downstream section from section 5.1.
- Record the chosen model and remote roles.
- Do not create `.omp/AGENTS.md`. Do not duplicate downstream governance in `.omp/RULES.md`; it remains available for independent hard invariants.
- Do not perform full Phase 03 compression here.

Verify in a fresh OMP session that root `AGENTS.md` is loaded and contains the downstream governance context. No sticky-rule verification is required for this design.

### Phase 3 — Normalise and version private docs

1. Inspect `docs/prompts/`.
2. Move retained content to `docs/dev/prompts/`.
3. Create `docs/dev/README.md`, `upstream-integration.md`, and `divergence.md`.
4. Apply the exact Phase 03 edit.
5. Track approved private documents only in storage permitted by the selected model.

If the documents are confidential, A is invalid; use B or C.

### Phase 4 — First controlled upstream integration

Preconditions: Phases 0–3 complete, clean worktree, durable backup established.

```bash
git fetch upstream
git status --short
git log --oneline --left-right --boundary HEAD...upstream/main
git merge --no-edit upstream/main
```

Conflict handling:

- preserve upstream improvements and the concise downstream section in `AGENTS.md`;
- preserve root `AGENTS.md` downstream invariants;
- preserve `docs/dev/**` and review for semantic staleness;
- resolve source/configuration conflicts semantically.

Run canonical checks from root `AGENTS.md`; at minimum run `bun check` and tests relevant to affected packages.

Push only to a destination allowed by A, B, or C.

### Phase 5 — Deferred hardening

Add only when triggered by evidence:

- URL-aware pre-push hook;
- idempotent Windows/Git-Bash clone bootstrap script;
- separate public worktree for C;
- merge automation after repeated real conflicts;
- destination-specific release flow.

## 10. Verification matrix

| Requirement | Check |
|---|---|
| Hosted model explicit | `docs/dev/README.md` records A, B, or C. |
| Genuine fork | A/C public repo reports `isFork=true`, `isPrivate=false`, correct parent. |
| Private storage | B/C private repo reports `isFork=false`, `isPrivate=true`. |
| Public OMP fetch-only | `upstream` fetch URL is public; push URL is `no_push://public-omp`. |
| Push default correct | `remote.pushDefault` matches the intended destination. |
| Upstream tracking correct | `branch.main.remote=upstream`; merge ref is `refs/heads/main`. |
| Governance context active | Fresh OMP session includes the root `AGENTS.md` downstream section; no `.omp/RULES.md` is required. |
| Root guidance preserved | Fresh session includes root `AGENTS.md`; no `.omp/AGENTS.md`; downstream governance is not duplicated in any `.omp/RULES.md`. |
| Boundary clean | No private instructions/docs outside approved paths. |
| Confidential material safe | No private content reachable from a public fork branch. |
| Token plan aligned | Phase 03 contains the exact revised wording. |
| Sync correct | `git fetch upstream` precedes merge of `upstream/main`. |
| Release controlled | No publication command without a destination-specific plan. |

## 11. Open decisions

1. **Resolved:** public visibility is accepted under Model A.
2. **Resolved:** hosted repository is `Theoman9402/oh-my-pi`; no private repository is part of this setup.
3. **Resolved:** Model A.
4. Whether inherited Actions should run on the hosted repository.
5. Whether a release process will be needed.
6. Whether direct-URL push risk justifies a hook.
7. Whether mutating subagents retain root `AGENTS.md` context.

## 12. Final recommendation

Visibility is resolved under Model A:

- A genuine GitHub fork of public OMP is public.
- A confidential downstream would require a private standalone repository.
- This setup uses only the public fork, with no private repository.

Next maintain safe remotes, root `AGENTS.md` governance context, tracked `docs/dev/**`, and standard merges from `upstream/main`. Do not duplicate downstream governance in `.omp/RULES.md`; reserve it for independent hard invariants. Do not add a private `.omp/AGENTS.md`, custom merge driver, linter, ADR framework, or hidden instruction locations without later evidence of need.

## 13. References

- GitHub fork visibility: `https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-permissions-and-visibility-of-forks`
- Create a fork: `https://docs.github.com/en/pull-requests/how-tos/work-with-forks/fork-a-repo`
- GitHub CLI fork: `https://cli.github.com/manual/gh_repo_fork`
- Detach a fork: `https://docs.github.com/en/pull-requests/how-tos/work-with-forks/detaching-a-fork`
- OMP context files and sticky-rule behavior: `docs/context-files.md`
- OMP root guidance: `AGENTS.md`
- OMP release behaviour: `scripts/release.ts`
- OMP `.omp/` tracking: `.gitignore`
- Git configuration: `https://git-scm.com/docs/git-config`
- Git fetch: `https://git-scm.com/docs/git-fetch`
- Git merge: `https://git-scm.com/docs/git-merge`
- Context engineering: `https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models`
