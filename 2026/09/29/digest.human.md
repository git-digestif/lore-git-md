# Git mailing list daily digest for 2026/09/29

## The day in brief
The Git mailing list saw active development across several key areas today. A critical bugfix for `git stash` autostash handling was finalized and merged, addressing a long-standing issue with staged index entries. The `uploadpack.lazyFetchTrusted` series received maintainer feedback on a latent design issue, while a new RFC proposed optional per-repository consent for running local hooks. Performance improvements took center stage with a new `sha1dc-accel` subsystem promising 2.85× faster SHA-1 collision detection. Several documentation improvements were also proposed and reviewed.

## Notable threads

### `git stash` autostash with staged index entries
**What changed**: D. Ben Knoble's five-patch series fixing autostash handling of staged index entries when `stash.index=true` is set was finalized and merged to `master`. The series replaces subprocess-based logic with in-core merging via `merge-ort.c`.

**Problem/goal**: The bug caused incorrect index state preservation during merge operations that triggered autostash, potentially leading to SIGSEGV when the merge process was interrupted.

### Technical details

- Files touched: `builtin/stash.c`, `t/t3903-stash.sh`, `t/t5520-pull.sh`, `t/t7600-merge.sh`
- Key functions: `do_apply_stash()` refactored to use `merge_incore_nonrecursive()`
- New test: `stash apply --index merges the correct trees` validates staged changes preservation

**Impact**: The fix resolves a long-standing bug affecting users who rely on `stash.index=true` during merge operations. The series includes comprehensive test coverage and addresses all prior review feedback, including a latent error-handling bug identified by Junio C Hamano.

**Status**: Merged to `master` with all patches integrated.

---

### `uploadpack.lazyFetchTrusted` server-side configuration
**What changed**: Junio C Hamano reviewed the second patch in Christian Couder's five-part v4 series introducing `uploadpack.lazyFetchTrusted`, identifying a latent design issue in the existing `safe.directory` logic that this patch inherits.

**Problem/goal**: The series provides a server-side mechanism for operators to explicitly declare which repositories are safe to lazy-fetch from, addressing security concerns about untrusted repositories triggering arbitrary code execution.

### Technical details

- Files touched: `promisor-remote.c`, `setup.c`, `upload-pack.c`, documentation
- New symbols: `lazy_fetch_objects()`, `try_promisor_remotes()`, `path_allowlist_apply()`
- Design issue: Silent ignoring of non-existent paths in `safe.directory` logic

**Impact**: The review highlights a usability concern where misspelled or misconfigured paths go unnoticed, potentially leading to unintended trust decisions. Christian acknowledged the concern and proposed addressing it in a follow-up patch.

**Status**: Under review; Junio's final review of 5/5 was entirely positive with only surface-level documentation wording tweaks suggested.

---

### Optional per-repository consent for local hooks
**What changed**: Dmytro Lymarenko proposed an RFC for an opt-in safety mechanism requiring explicit user consent before running local hooks in untrusted repositories.

**Problem/goal**: Protect users from malicious or unexpected hooks installed by tools or project setups.

### Technical details

- Proposed prompt: Shows hook path with options to run once, trust all hooks in repository, or decline
- Trust state: Would be stored per-repository
- Re-prompt: For modified in-worktree hooks

**Impact**: Junio C Hamano and Brian M. Carlson raised fundamental security concerns about the trust mechanism being vulnerable to manipulation by the same process that installs hooks. Carlson also noted the interactive prompt would break non-interactive environments like CI systems.

**Status**: Early RFC; no implementation posted. The discussion highlights significant design challenges that would need to be addressed before proceeding.

---

### `sha1dc-accel`: Faster SHA-1 collision detection
**What changed**: Scott Chacon posted a four-patch series introducing `sha1dc-accel`, a new subsystem that uses vectorized (SSE2, AVX2, NEON) and hardware-accelerated (x86-64 SHA-NI, ARMv8 SHA-1) versions of the UBC check and SHA-1 compression steps.

**Problem/goal**: Reduce the collision-detection overhead from 1.5-2.5× down to 0.2× relative to OpenSSL SHA-1, yielding a 2.7-2.85× speedup in SHA-1 throughput.

### Technical details

- Files touched: `Makefile`, `meson.build`, `CMakeLists.txt`, new `sha1dc-accel/` directory
- Performance gains: 2.85× speedup on Apple M5 Max, 50% reduction in `index-pack` runtime
- Test coverage: Unit tests, collision test vectors, performance benchmarks

**Impact**: Significant performance improvement for operations like `index-pack`, particularly beneficial for server environments. The series is well-tested and includes platform-specific optimizations with fallback paths.

**Status**: Under review; no maintainer decision yet.

---

### `git branch --delete-merged` squash-merge detection
**What changed**: Harald Nordgren posted a patch extending `git branch --delete-merged` to recognize branches whose changes have been squash-merged or rebase-merged into an upstream branch.

**Problem/goal**: Address a common pain point for users who prefer GitHub-style squash merges, where branches were previously not deleted by `--delete-merged` because their tips don't appear in the upstream history.

### Technical details

- Files touched: `builtin/branch.c`, documentation, tests
- New behavior: Detects when an upstream commit contains all changes from a branch
- Output format: `Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)`

**Impact**: Improves workflow for users who rely on squash merges, particularly in GitHub-style workflows. The patch includes comprehensive test coverage for edge cases.

**Status**: Under review; Phillip Wood identified two backward-compatibility issues (exit status change and option parsing regression) that would need to be addressed.

## In brief

- **`git filter-branch` bugfix**: Michele Locati provided a Tested-by trailer confirming the fix for commit mapping inversion with `--state-branch`.
- **`git reflog expire` regression**: Patrick Steinhardt suggested tightening test timing thresholds for better precision in verifying the 90-day expiry policy.
- **`git remote` documentation**: Junio C Hamano approved v2 of Matthias Goergens' patch clarifying that `git remote` commands affect only the local view.
- **`git stash create` options**: Phillip Wood identified backward-compatibility regressions in Kazumasa Shigeta's patch extending `git stash create` to support `--include-untracked` and `--all`.
- **`git config --list --show-origin`**: Christophe Lohr reported that the command displays `.git/config` as a relative path, creating confusion about the `.git` directory's location.
- **`git submodule` root-relative URLs**: Stanislav Aleksandrov posted an RFC patch introducing `^/` syntax for submodule URLs, allowing them to be resolved relative to the superproject's remote server root.
- **Reftable timezone encoding**: Patrick Steinhardt posted a three-patch series fixing the reflog timezone encoding discrepancy in the reftable backend.
- **`includeIf "hostname:..."`**: Junio C Hamano expressed mild regret that Ignacio Encinas' 2024 hostname-normalization series stalled, reinforcing that the design work was on the right track.
- **`git(1)` man page rewrite**: Julia Evans proposed a tiered approach to documenting help system access, addressing feedback about balancing simplicity and functionality.

The day's activity reflects a strong focus on performance improvements, bug fixes, and usability enhancements across Git's core functionality.