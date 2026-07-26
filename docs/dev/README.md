# Downstream development

This directory contains downstream-specific documentation for the public Model A fork of OMP.

## Operating model

- Hosted repository: `Theoman9402/oh-my-pi`, a public fork of `can1357/oh-my-pi`.
- `origin`: public fork; the permitted push destination when a push is explicitly requested.
- `upstream`: public OMP integration source; fetch only. The local push URL is `no_push://public-omp`.
- All committed source, plans, documentation, workflows, history, and Actions logs are public.

Downstream governance is defined in the root `AGENTS.md`. Downstream documentation belongs under `docs/dev/**`; upstream-owned documentation elsewhere under `docs/**` is not modified for downstream policy. Do not duplicate this governance in `.omp/RULES.md` or create `.omp/AGENTS.md`.

## Documents

- [Upstream integration](upstream-integration.md) — fetch, review, merge, verification, and destination checks.
- [Divergence](divergence.md) — durable merge-sensitive downstream differences.
- [Plans](plans/) — implementation plans for downstream work.
- [Prompts](prompts/) — retained, public-safe prompt research.
