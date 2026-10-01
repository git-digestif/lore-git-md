# Git mailing list daily digest for 2026/09/30

## The day in brief
The Git mailing list saw active discussion on several fronts today. A new test case for `git filter-branch` confirmed a long-standing bug, while a `git backfill --dry-run` feature faced conceptual objections. The `gitmergeconflicts(7)` manpage received substantive feedback on terminology and user experience, and a sparse-checkout regression was reported. Several performance optimizations and bugfixes advanced, including improvements to the reftable backend and xdiff subsystem.

## Notable threads

### `git filter-branch`: Fix commit mapping inversion with `--state-branch`
**What changed?**
Michele Locati provided a new test case that directly exercises incremental runs with `--prune-empty` and `--subdirectory-filter`, ensuring the state branch file contains correctly ordered "original:rewritten" entries. The test fails on current `master` (a018953) without the fix, passes with the fix, and also passes on Git v2.49.1 (the last version before the regression).

**Why it matters**
The bug, introduced in Git 2.50.0 during the Perl-removal refactor, causes `git filter-branch` to incorrectly map commits when using `--state-branch`. This can lead to commits that map to nothing, and on subsequent runs, the script attempts to create files with empty names, emitting "Is a directory" errors. The new test case provides comprehensive coverage for this class of bug, making the fix more robust.

**Key technical details**
- Files touched: `git-filter-branch.sh`, `t/t7003-filter-branch.sh`
- The fix involves a two-line change to split a line on colon and write the to-commit into a file named by the from-commit
- The test case verifies the mapping inversion and the `--prune-empty` scenario

### `git backfill --dry-run`: Conceptual objections raised
**What changed?**
Derrick Stolee objected to the `--dry-run` feature for `git backfill` on conceptual grounds, arguing that the uncompressed size estimate may not meaningfully inform a user's decision to run the command. Junio C Hamano endorsed this view, making the feature unlikely to advance without evidence of user demand.

**Why it matters**
The discussion highlights a broader principle in Git development: features should be driven by concrete user needs rather than speculation. Stolee suggested that if the feature proceeds, it should use a more explicit argument like `--info=(count|size)` to clarify the trade-off between information depth and network effort.

**Key technical details**
- The feature would estimate the number and total uncompressed size of blobs that would be fetched
- The implementation was technically sound but lacked a clear use case
- The discussion may influence future feature proposals in Git

### `gitmergeconflicts(7)`: User experience and terminology feedback
**What changed?**
Patrick Steinhardt provided substantive feedback on the new `gitmergeconflicts(7)` manpage, focusing on terminology, technical completeness, and user experience. Julia Evans responded, accepting some suggestions while pushing back on others based on user feedback.

**Why it matters**
The discussion reveals tensions between Git's internal terminology and user-friendly documentation. Evans' responses highlight her empirical approach to documentation, prioritizing what users find helpful over theoretical completeness.

**Key technical details**
- Patrick suggested expanding the introduction to explain how Git performs a 3-way merge
- Julia declined this, citing user feedback that found such explanations confusing
- Junio C Hamano proposed making commit information more accessible during conflict resolution
- The discussion also touched on promoting `merge.conflictstyle=diff3` as a best practice

### Sparse-checkout regression reported
**What changed?**
Webstrand reported a regression in Git v2.27.0 (commit `681c637b4a`) where `git checkout` in a sparse-checkout working tree clobbers untracked in-cone files instead of aborting with an error.

**Why it matters**
This is a serious correctness problem for users relying on sparse-checkout, as it can silently destroy untracked work. The regression breaks the invariant that sparse checkouts should behave identically to non-sparse checkouts in equivalent scenarios.

**Key technical details**
- The bug affects all versions from v2.27.0 through current master
- The report includes a clear reproduction script
- The misleading warning message "paths were already present and thus not updated" incorrectly describes the conflict

### CI/build system improvements
**What changed?**
Tamir Duberstein posted v3 of a CI/build system series that now uses twice the CPU count for both GitHub and GitLab CI, replacing GitHub's fixed `-j10` and GitLab's one-job-per-CPU policy. Benchmark data shows a 7% speedup on Linux and avoids a 26% slowdown on macOS.

**Why it matters**
The changes improve CI performance and reliability across platforms. The series also includes a fix for OOM kills in GitHub Actions by replacing `test_cmp` with `test_cmp_bin` in a specific test.

**Key technical details**
- Files touched: `t/t4205-log-pretty-formats.sh`, `ci/lib.sh`
- The OOM fix reduces memory usage from 4 GiB to 1.2 MiB for large file comparisons
- The parallelism change uses native CPU-count queries for each platform

## In brief
- **[PATCH] filter-branch: fix commit mapping inversion with --state-branch**: Michele Locati provided a new test case confirming the bug and fix
- **[PATCH 0/3] rerere: fix race condition between rebase and background maintenance**: Patrick Steinhardt raised process-oriented concerns about test coverage
- **[PATCH 0/6] parse-options: new sub-API for early argument scanning**: Kaartic Sivaraam identified a behavioral inconsistency with `parse_options()`
- **[PATCH] fetch: defer validation of fetch.followRemoteHEAD until needed**: Patrick Steinhardt raised concerns about direct test coverage
- **[PATCH 0/2] ci: use cmp and align job-count selection**: Series merged to `next` with benchmark data showing performance improvements
- **[PATCH] submodule merge: stale delta-base cache causes "repository corrupt"**: Philippe Blain bisected the regression to two specific commits
- **[RFC PATCH 0/4] Improve error reporting for invalid --git-dir**: Junio questioned the series' direction, Patrick identified technical concerns
- **[PATCH 0/7] Meson build improvements**: Series received surface-level reviews
- **[PATCH 0/4] Fix MIDX reachability closure corner cases**: Junio confirmed refactoring correctness, Jeff King identified edge cases
- **[PATCH 0/2] checkout -m: remember original conflict labels**: Junio raised documentation and portability concerns
- **[PATCH] refs/packed-backend: optimize packed-refs file rewrites**: Karthik Nayak posted a performance optimization patch
- **[PATCH] revision: add `@{p}` as a shorthand for `@{push}`**: Junio queued the patch for integration
- **[PATCH] stash: extend `pop` with custom conflict-label options**: Junio questioned the fundamental motivation for the feature
- **[PATCH 0/7] xdiff: switch mmfile_t buffers from unsigned long to size_t**: Junio and Patrick provided substantive reviews
- **[PATCH 0/3] reftable: fix timezone handling in reflog entries**: Junio endorsed explicit sign-flipping logic for readability