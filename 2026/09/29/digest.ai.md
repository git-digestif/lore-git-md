# Git mailing list daily digest for 2026/09/29

## The day in brief
The Git mailing list saw significant progress on several long-running patch series, with maintainer approvals for key bugfixes and feature enhancements. Notable developments include the finalization of the `uploadpack.lazyFetchTrusted` security feature, resolution of the `--force-if-includes` reflog walk bug, and integration of the stash autostash fix. New RFCs for submodule URL handling and stash sharing emerged, while performance optimizations for SHA-1 collision detection were proposed.

## Notable threads

### `uploadpack.lazyFetchTrusted` security feature finalized
**What changed**: Christian Couder's v4 patch series implementing `uploadpack.lazyFetchTrusted` received maintainer approval after addressing all prior feedback. The series replaces the client-side `GIT_NO_LAZY_FETCH=fromAccepted` proposal with a server-side protected configuration variable.

**Problem/goal**: Provide a server-side mechanism for operators to explicitly declare which repositories are safe to lazy-fetch from, addressing security concerns about untrusted repositories triggering arbitrary code execution via lazy-fetch hooks.

**Technical details**:
- Files touched: `promisor-remote.c`, `setup.c`, `upload-pack.c`, documentation files
- New configuration: `uploadpack.lazyFetchTrusted` (protected configuration variable)
- New behavior: Server operators can mark repositories as trusted for lazy fetching with recursion safeguards
- Security model: Shifts trust decisions to server operators, modeled after Git's `safe.directory` configuration

**Status**: Junio's final review of 5/5 was entirely positive with only surface-level documentation wording tweaks suggested. The series remains under review but is effectively ready for integration.

**Today's development**: [2026/09/29/17-26-12] Junio identified a latent design issue in the existing `safe.directory` logic that this patch inherits: silent ignoring of non-existent paths. Christian acknowledged the concern and proposed addressing it in a follow-up patch.

### `--force-if-includes` reflog walk bugfix merged
**What changed**: Aleksei Sviridkin's v4 patch fixing an uninitialized variable bug in the `--force-if-includes` safety mechanism was accepted by Junio and marked for 'next'.

**Problem/goal**: Fix a bug where the reflog walk would terminate prematurely when the remote-tracking ref had no reflog, causing false positives in the "remote ref updated since checkout" check.

**Technical details**:
- Files touched: `remote.c`, `t/t5533-push-cas.sh`
- Fix: Initialize `timestamp_t date = 0` in `remote.c`
- Test coverage: Updated test reproduces the bug by expiring only the remote-tracking reflog
- Backend scope: Files backend only; reftable backend unaffected

**Status**: Merged to 'next' with Junio's approval. The fallback-value discussion is closed with `0` as the accepted value.

**Today's developments**:
- [2026/09/29/01-10-08] Tyler Cipriani provided concrete timing data showing the performance penalty for `date = 0` is negligible (15ms for 2,000 entries)
- [2026/09/29/09-13-18] Aleksei posted v4 with improved commit message and test
- [2026/09/29/16-54-46] Junio accepted v4 and marked it for 'next'

### Stash autostash fix with staged index entries
**What changed**: D. Ben Knoble's five-patch series fixing autostash handling of staged index entries when `stash.index=true` is set was fully integrated after addressing all prior blockers.

**Problem/goal**: Fix a bug where autostashing fails to correctly handle staged index entries during merge operations, particularly when interrupted by SIGINT/SIGQUIT.

**Technical details**:
- Files touched: `builtin/stash.c`, test scripts
- Key change: Replaced subprocess-based logic with in-core merging via `merge-ort.c`
- Safety improvements: Added NULL-dereference checks, preserved verbosity settings
- Test coverage: Comprehensive tests for conflict scenarios and edge cases

**Status**: Fully integrated into `master` after Junio's final review.

**Today's developments**:
- [2026/09/29/20-07-02] Junio identified a latent error-handling bug where hard errors are incorrectly treated as conflicts
- [2026/09/29/15-48-14] Phillip Wood confirmed the series is ready for integration after addressing all prior feedback

### Reftable reflog timezone encoding fix
**What changed**: Patrick Steinhardt posted a three-patch series fixing a timezone encoding discrepancy in the reftable backend.

**Problem/goal**: Align Git's reflog timezone storage with the reftable specification, which requires signed offsets in minutes rather than the "[+-]HHMM" format used elsewhere in Git.

**Technical details**:
- Files touched: `date.c`, `refs/reftable-backend.c`, test files
- New symbols: `parse_timezone_minutes()`, `format_timezone_minutes()`
- On-disk change: Timezone offsets now stored as signed minutes
- Test coverage: New tests verify round-trip behavior and edge cases

**Status**: Under review; v1 posted.

**Today's development**: [2026/09/29/09-56-28] Patrick posted the complete three-patch series addressing the timezone encoding issue.

### SHA-1 collision detection performance optimization
**What changed**: Scott Chacon posted a four-patch series introducing `sha1dc-accel`, a faster SHA-1 collision detection implementation using vectorized and hardware-accelerated backends.

**Problem/goal**: Reduce the collision-detection overhead from 1.5-2.5× down to 0.2× relative to OpenSSL SHA-1, yielding a 2.7-2.85× speedup in SHA-1 throughput.

**Technical details**:
- New subsystem: `sha1dc-accel` with platform-specific backends
- Optimizations: SSE2, AVX2, NEON vectorization; x86-64 SHA-NI; ARMv8 SHA-1 instructions
- Performance gains: 2.85× speedup on Apple M5 Max, ~50% reduction in `index-pack` runtime
- Test coverage: Comprehensive unit tests and performance benchmarks

**Status**: Under review; v1 posted.

**Today's development**: [2026/09/29/11-25-40] Scott posted the complete four-patch series introducing the performance-optimized collision detection implementation.

## In brief
- **[filter-branch bugfix]** Michele Locati provided Tested-by for the `--state-branch` commit mapping inversion fix [2026/09/29/14-31-50]
- **[Merge driver cleanup]** Jeff King's bugfix series for resource cleanup in merge driver infrastructure was approved by Junio [2026/09/29/16-51-53]
- **[git-contacts stdin feature]** Brigham Campbell's documentation follow-up for the merged stdin feature was approved [2026/09/29/16-30-17]
- **[Reflog expiry regression]** Patrick Steinhardt and Junio agreed on test timing thresholds for the merged regression fix [2026/09/29/18-31-06]
- **[Documentation fixes]** Several documentation patches were approved, including clarifications for `git remote` [2026/09/29/16-28-25] and `git-interpret-trailers` [2026/09/29/19-10-48]
- **[HTTP redirect handling]** Patrick Steinhardt endorsed Jeff King's URL-based approach for handling libcurl 8.23.0's credential-stripping behavior [2026/09/29/05-42-59]
- **[Branch --delete-merged enhancement]** Harald Nordgren's patch extending `--delete-merged` to detect squash-merged branches received substantive review [2026/09/29/16-08-08]
- **[Stash create enhancement]** Phillip Wood identified backward-compatibility issues in Kazumasa Shigeta's patch adding `--include-untracked` support to `git stash create` [2026/09/29/16-08-08]
- **[Config --show-origin bug]** Christophe Lohr reported that `git config --list --show-origin` displays `.git/config` as a relative path [2026/09/29/12-36-20]
- **[Submodule URL RFC]** Stanislav Aleksandrov proposed a `^/` syntax for root-relative submodule URLs [2026/09/29/18-51-22]
- **[Hook consent RFC]** Brian m. carlson raised operational concerns about Dmytro Lymarenko's proposal for optional per-repository consent before running local hooks [2026/09/29/21-32-28]
- **[includeIf.hostname RFC]** Junio expressed mild regret that Ignacio Encinas' 2024 hostname-normalization series stalled [2026/09/29/19-55-19]