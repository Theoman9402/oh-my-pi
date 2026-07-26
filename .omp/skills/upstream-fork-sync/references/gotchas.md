# Gotchas

- A disposable worktree is not a routine sync requirement; it was needed here because `main` was dirty.
- A separate worktree does not contain untracked files or ignored `node_modules` from the original worktree.
- A candidate created from dirty `main` can omit downstream work and is unsafe to promote without explicit preservation.
- Plain `git diff --check` can pass on a clean candidate while the integrated commit range contains whitespace errors.
- A conflict-marker search that treats bare `=======` as a marker flags legitimate Markdown setext headings.
- Missing Biome or Cargo prevents meaningful checks before any sync defect is established.
- Existing Rustup settings can select the repository-pinned channel; the warning is not itself a sync failure.
- Native-dependent tests can fail because the addon is absent even when JavaScript dependencies are installed.
- Building the Windows native addon can require CMake and libclang after Cargo is available.
- The Windows native build wrapper can fail while invoking `napi.exe` through Bun before Rust compilation begins.
