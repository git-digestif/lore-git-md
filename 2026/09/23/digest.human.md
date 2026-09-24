# Git mailing list daily digest for 2026/09/23

## The day in brief
A regression fix for `git reflog expire` default expiry times was approved and is ready for `next`. The `git stash` autostash bugfix series (v2) addressed all prior feedback and is now well-structured. Junio C Hamano acked a critical patch in the repack data-loss fix series, clearing the way for integration. The `git imap-send` shell injection vulnerability patch must be revised to restore error-handling semantics. A new `parse-options` early-scanning API (v2) was posted, focusing on the `git fast-import` bugfix. The `git ls-files` untracked cache reuse series (v2) expanded to three patches, with significant performance improvements. The `git commit --amend` regression during interactive rebase will be reverted for Git 2.56, deferring architectural changes until after the release.

## Notable threads

### [PATCH v5] reflog: fix default expiry times regression
**What changed**: The v5 patch restores the correct default expiry periods for `git reflog expire` (reachable=90d, unreachable=30d), addressing a Git 2.50 regression where the values were swapped. The patch consolidates all test cases into a single repository to address Junio C Hamano’s test-structure concern.

**Why it matters**: The regression affects all platforms and breaks external tools that rely on the documented expiry behavior. The fix is narrowly targeted, with no on-disk format changes, and includes comprehensive test coverage.

**Current status**: Approved by Junio and ready for `next`.

---

### [PATCH 0/4] stash: fix autostash with staged index entries
**What changed**: The v2 series is now a four-patch refactoring that replaces subprocess-based index merging with in-core logic via `merge-ort.c`. The series addresses all prior feedback, including Phillip Wood’s concerns about merge algorithm consistency, assertion safety, and test coverage.

**Why it matters**: The bug affects users of `stash.index=true` during merge operations, causing incorrect index state preservation. The fix eliminates a race condition and simplifies the code, with thorough test coverage for conflicted index merges and graceful failure.

**Current status**: Under review; all technical concerns resolved. The series is well-structured and ready for integration.

---

### [PATCH 0/6] repack: fix concurrent-push race leading to data loss
**What changed**: Junio C Hamano acked patch 2/5 (v3) after the author moved the kept-pack cache invalidation logic to `packfile.c`, addressing the maintainer’s layering concern. The patch is now ready for integration.

**Why it matters**: The series fixes a race condition where concurrent pushes can cause `git repack -d` to delete packs whose objects were never copied elsewhere, resulting in data loss. The fix is critical for repository integrity and has been validated in GitLab CI.

**Current status**: Patch 2/5 acked; remaining patches under review. The series is likely to proceed to `next` soon.

---

### [PATCH] git-p4: prevent shell injection from user-supplied commit ID
**What changed**: Junio C Hamano identified a regression in the original patch: the new `diffTreeApply()` helper ignores the exit status of the diff-apply pipeline, whereas the original code raised an exception on failure via `p4_system()`.

**Why it matters**: The vulnerability allows arbitrary command execution via crafted commit IDs. The fix must restore the original error-handling semantics while preserving the shell injection mitigation.

**Current status**: Under review; the patch must be revised to address the error-handling regression.

---

### [PATCH 0/6] parse-options: new sub-API for early argument scanning
**What changed**: The v2 series is a major redesign that reuses `struct option` instead of introducing a separate `early_scan_option` structure. The series now focuses exclusively on the `git fast-import` bugfix, dropping the `git bisect` and `git rev-parse` conversions.

**Why it matters**: The series introduces a minimal, reusable `parse-options` sub-API to replace fragile hand-rolled early argument scans. The `git fast-import` bugfix is a real-world issue where `--allow-unsafe-features` was silently ignored when preceded by `--depth 5`.

**Current status**: Under review. Junio C Hamano raised a substantive usability concern about the API’s requirement for callers to handle argument skipping (`--option value`), which could lead to duplication or inconsistencies.

---

### [PATCH 0/3] imap-send: OpenSSL compatibility and correctness fixes
**What changed**: Patrick Steinhardt acknowledged the necessity of the OpenSSL 4.1 compatibility macro, suggesting the commit message explicitly mention `OPENSSL_API_COMPAT` for clarity.

**Why it matters**: The series addresses three correctness and forward-compatibility issues in `git imap-send`’s TLS certificate verification logic. The OpenSSL 4.1 compatibility macro is a forward-compatibility fix to avoid build failures under `DEVELOPER=1`.

**Current status**: Under review. The series faces unresolved technical concerns, including a memory-safety bug in patch 2 and incomplete RFC 6125 compliance in patch 3.

---

### [PATCH 0/3] ls-files: reuse untracked cache populated by status
**What changed**: The v2 series expanded to three patches: (1) fix ignore-file hash computation, (2) enable cache sharing between `git status -unormal` and `git status -uall`, and (3) teach `git ls-files` to reuse the cache with optional index writes. Benchmarks show a 7.3× speedup (241 ms → 33 ms) for wildcard queries on a synthetic tree with 100 k files.

**Why it matters**: The series eliminates redundant directory traversals in common workflows (e.g., `git status` followed by `git ls-files`), with significant performance improvements. The changes are well-scoped and maintain backward compatibility.

**Current status**: Under review. The series is self-contained and ready for substantive review.

---

### [PATCH 0/2] ci: reduce resource exhaustion in GitHub Actions
**What changed**: The v1 series addresses two CI resource exhaustion issues: (1) replacing `diff` with `cmp` in `t4205-log-pretty-formats.sh` to avoid OOM kills, and (2) dynamically setting `-j` for `make` and `prove` to the runner’s CPU count to prevent ENOSPC errors.

**Why it matters**: The changes are minimal and pragmatic, addressing real CI bottlenecks without disabling long-running tests. The series is well-motivated and ready for review.

**Current status**: Under review.

---

### regression in `git commit --amend` during interactive rebase
**What changed**: Junio C Hamano, Elijah Newren, and Patrick Steinhardt agreed to revert the regressing commit (6257588252) entirely for the Git 2.56 release cycle, deferring architectural changes until after the release. The revert is now the immediate action item.

**Why it matters**: The regression breaks established workflows where users amend commits during interactive rebase to split commits or reword messages. The revert prioritizes release stability over partial fixes.

**Current status**: Revert planned for Git 2.56-rc2. The broader architectural question of rejecting plain `git commit` during conflict resolution is deferred until after the release.

---

## In brief
- **[PATCH v3] rev-parse: add advice for shallow history in `<rev>~N` and `<rev>^N`**: The v3 patch narrowed scope to `<rev>~N`/`<rev>^N` syntax, fixed a bug where chained carets printed the advice twice, and used `<rev>` consistently in documentation. Ready for `next`.
- **[PATCH] completion: add support for `git worktree repair`**: Approved by Patrick Steinhardt and ready for integration.
- **[PATCH] compat/winansi: fix die_lasterr() formatting bug**: Approved by Johannes Sixt and ready for `next`.
- **[PATCH 0/4] Fix GitLab CI breakage after Rust enabled for Windows**: The original patch was approved by Junio, but the newly reported GitLab Runner regression is now in the hands of GitLab’s infrastructure team.
- **[PATCH] reference-transaction hook omits new ref on branch rename**: The v2 patch confines copy/rename state to per-ref-update data, preserving transaction genericity and enabling multi-ref operations. Ready for review.
- **regression in reference-transaction hook: all-zero OIDs on branch/tag deletion**: The v5 series was merged to `next` after addressing all prior feedback, including a build-breakage fix.
- **[RFC PATCH 0/6] Introduce a "safe" strbuf API that cannot die()**: Jeff King proposed an alternative design using fixed-size, stack-allocated buffers to avoid `malloc()` entirely, arguing this would better serve the trace2 use case.
- **submodule merge: stale delta-base cache causes "repository corrupt" or misread commits**: Guillaume Chauvel reported a bug where merging a superproject with two submodules causes Git to attempt to read a commit from the wrong submodule, either failing with “repository corrupt” or misreading the commit due to stale delta-base cache data.
- **[PATCH] git-rm: remove unnecessary comma**: Junio C Hamano requested two adjustments: real-name Signed-off-by and removal of the comma in a second location. The patch is likely to be accepted after the requested changes.