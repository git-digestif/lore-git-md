# Git mailing list daily digest for 2026/10/11

## The day in brief
Junio C Hamano approved the `git blame` default ignore-revs file series for `next`, while René Scharfe posted a fix for ZIP timestamp wrapping. The `git push` refspec-matching performance series saw its first substantive review, and a `fast-import` crash fix gained test confirmation. Documentation updates to `SubmittingPatches` sparked discussion about test practices.

## Notable threads

### rerere: fix race condition between rebase and background maintenance
**What changed**: The first patch in Thomas Bachem’s v7 series, introducing `rerere.lockTimeout` (default 1000 ms) for the `MERGE_RR.lock` file, received maintainer feedback and was updated.

**Problem**: A long-standing race condition in Git’s `rerere` subsystem, where concurrent writers contend for the shared `rr-cache` directory lock, was exacerbated by the geometric maintenance strategy in Git 2.54.

**Subsystem**: `rerere` (reuse recorded resolution), maintenance.

**Impact**: Bugfix; prevents crashes or silent failures during concurrent `rerere` operations (e.g., `git rebase` and `git maintenance run`).

**Today’s developments**:
Junio C Hamano reviewed patch 1/2, requesting documentation and test improvements. He asked to remove redundant lock-behavior explanation (since Git’s timeout-based locks consistently fail after timeout), tighten error-message matching in tests, and clarify test logic for background process cleanup. Thomas Bachem incorporated all feedback: dropped the redundant documentation line, adopted `test_grep ! "Unable to create.*MERGE_RR\.lock"` for error-message matching, and moved the `wait` call to `test_when_finished` to ensure cleanup runs regardless of test outcome. The patch remains technically ready for integration.

---

### blame: default to HEAD:.git-blame-ignore-revs if no ignore file configured
**What changed**: Ravi Mistry posted v3 of the series, replacing `the_repository` with `sb->repo` in `parse_default_ignore_revs_blob()` to align with the ongoing `the_repository` removal effort.

**Problem**: `git blame` and `git annotate` do not automatically use `HEAD:.git-blame-ignore-revs` as the default ignore-revs file, unlike hosting platforms (GitHub, GitLab).

**Subsystem**: `blame`, ignore-revs file parsing.

**Impact**: Feature; aligns local behavior with hosting platforms, reducing friction for users who rely on ignore-revs files for large-scale code changes.

**Today’s developments**:
The v3 series addressed Junio C Hamano’s feedback by replacing the global `the_repository` variable with `sb->repo` in `parse_default_ignore_revs_blob()`. Junio confirmed the series is technically sound and marked it ready for the `next` integration branch unless objections arise. The series implements two key changes: (1) security hardening of the ignore-revs parser and tag-peeling logic (rejecting malformed input and preventing promisor fetches in partial clones), and (2) automatic use of `HEAD:.git-blame-ignore-revs` as the default ignore file when none is configured. The hardening includes optional NUL-byte rejection, which Junio has previously questioned but not demanded be removed. The author has signaled willingness to drop this check if Junio prefers.

---

### worktree repair: fix incorrect repair of unrelated worktrees
**What changed**: Yoichi NAKAYAMA agreed to split patch 1/2 into smaller commits to separate behavioral changes from pure refactoring, improving clarity.

**Problem**: `git worktree repair` can incorrectly modify `.git` files or gitdirs belonging to unrelated worktrees when cross-references are not validated before repair.

**Subsystem**: `worktree`.

**Impact**: Bugfix; prevents accidental corruption of unrelated worktrees in edge cases (e.g., manually swapped worktree directories).

**Today’s developments**:
Yoichi NAKAYAMA responded to Junio C Hamano’s review of patch 1/2, clarifying the intended behavior changes and proposing a revised commit structure. The discussion centered on how `infer_backlink()` handles malformed `.git` files, particularly around whitespace trimming and validation of the `gitdir:` prefix. Yoichi explained that `worktree repair` prioritizes fixing common issues (e.g., relocated worktrees) over repairing manually corrupted `.git` files, so stricter validation is acceptable. The response commits to splitting the patch into smaller, more focused commits to separate behavioral changes from pure refactoring, making it easier to verify correctness. The thread is now at a stage where the author is preparing a v2, but no new code has been posted yet.

---

### branch --delete-merged: detect squash-merged and rebase-merged branches
**What changed**: Harald Nordgren posted v2 of the series, adding a `--no-squashed` flag to skip slower squash detection and fixing a merged-branch attribution bug.

**Problem**: `git branch --delete-merged` only deletes branches whose tips are reachable from upstream, failing for squash merges, rebases, or patch-based workflows.

**Subsystem**: `branch`, refspec matching.

**Impact**: Feature; reduces manual cleanup for users of hosting services (e.g., GitHub) or local `git merge --squash`.

**Today’s developments**:
The v2 series addressed performance and correctness concerns raised in v1. Key improvements include:
- **Performance**: The algorithm now batches diff calculations and reuses results across branches, addressing Phillip Wood’s concern about recalculating upstream diffs for each branch.
- **Correctness**: Fixed a bug where merged branches were incorrectly attributed to a merge commit if another branch was searched first. The output now shows the topic commit itself for work integrated via merges.
- **Usability**: Added `--no-squashed` to let users opt out of the slower squash detection (e.g., in large repos or blobless clones), though the feature defaults to "on" for all branches.
- **Documentation**: Expanded examples to cover local `git merge --squash` and GitHub’s "Squash and merge" buttons.

The implementation is now more robust, handling edge cases like upstream modifying the same files as the squashed branch. Tests verify behavior with reverted changes, partial landings, and mixed workflows. The `--no-squashed` flag reflects a compromise between Phillip Wood’s performance concerns and Harald Nordgren’s goal of a unified default behavior.

---

### ZIP archive: fix extended timestamp wrap
**What changed**: René Scharfe posted a patch omitting the Extended Timestamp (UT) extra field when the commit time exceeds the 32-bit signed integer range (`> 0x7fffffff`).

**Problem**: `git archive --format=zip` wraps post-2106-02-07 commit timestamps to 1970 in the UT extra field, causing tools like UnZip to misbehave in update mode.

**Subsystem**: `archive-zip.c`, ZIP export.

**Impact**: Bugfix; prevents silent data inconsistencies in UnZip’s update mode and cross-format inconsistencies with TAR.

**Today’s developments**:
René Scharfe posted a patch targeting the ZIP extended timestamp wrap issue. The patch omits the UT extra field for out-of-range commit times, preventing the silent timestamp wrap that causes tools like UnZip to misbehave in update mode. The implementation uses a `strbuf` to conditionally prepare the UT extra field, simplifying the output logic and avoiding repeated range checks. Tests verify that the UT field is included for 32-bit timestamps (e.g., `2147483647`) and omitted for out-of-range values (e.g., `2147483648`). The patch aligns with René’s updated position advocating **rejection** (omission) of out-of-range timestamps rather than clamping, prioritizing correctness over tool compatibility.

---

### test-mergesort: migrate to Clar, retire unused subcommands
**What changed**: Junio C Hamano requested commit message clarification for patch 1/4, questioning whether the first sentence describes a justification or precondition.

**Problem**: The mergesort tests use a shell-based test suite and include unused subcommands (`generate`, `test`), which are being migrated to the Clar unit-test framework.

**Subsystem**: Test infrastructure.

**Impact**: Cleanup; reduces code size (net 307 lines removed) and adopts the modern Clar framework.

**Today’s developments**:
Junio C Hamano reviewed patch 1/4, requesting clarification of the commit message’s first sentence ("The numeric sorting tests only use mergesort.h"). He found the phrasing ambiguous—it could be read as either a justification for the move or a precondition enabling it. The rest of the commit message (storing list items in an array, removing `t0071`, and retaining the `sort` command for the now-deleted benchmark) was not addressed, suggesting the feedback is purely stylistic. No technical concerns were raised about the Clar migration, test coverage, or build-system changes.

---

### repack --drop-filtered --dry-run: fix regression
**What changed**: Junio C Hamano marked the topic ready for the `next` branch after D. Ben Knoble provided his `Reviewed-by`.

**Problem**: `git repack --drop-filtered --dry-run` incorrectly rebuilds packs and deletes old ones despite the `--dry-run` flag.

**Subsystem**: `repack`, promisor-object handling.

**Impact**: Bugfix; restores documented `--dry-run` behavior.

**Today’s developments**:
Junio C Hamano marked the topic for inclusion in the `next` branch after D. Ben Knoble confirmed he is satisfied with the v2 patch and provided his `Reviewed-by`. The patch adds an early `return` in `cmd_repack()` after listing candidate objects for dropping, preventing the code from proceeding to the actual repack and deletion steps when `--dry-run` is used. The regression, which has existed since `--dry-run` was introduced in commit `1e746b00aa` (2025), is now fixed. The topic is cooking in `next` and will likely graduate to `master` in the next integration cycle unless issues surface.

---

### fast-import: fix crash when resolving unreachable in-session commits
**What changed**: Aniket Gupta confirmed the crash is reproducible and fixed by the patch, requesting test coverage in `t9300`.

**Problem**: `git fast-import` crashes with `not a valid commit: <oid>` when attempting to resolve a commit created earlier in the same session but not yet reachable via the object database.

**Subsystem**: `fast-import`.

**Impact**: Bugfix; prevents crashes during mid-session commit lookups.

**Today’s developments**:
Aniket Gupta reviewed the patch, confirming the bug is reproducible and that the fix resolves it for four scenarios: `reset` with a new commit OID, `commit` with `from <new-oid>`, a tag created in the same session, and a tag-of-a-tag. Existing commits, tags, and blob references continue to work as before. The review is substantive: it includes reproduction steps, test results, and specific feedback on the commit message and code formatting. Aniket also requested a test case in `t9300` to ensure the fix is covered by the test suite. The patch itself remains unchanged in this reply.

---

### imap-send: fix OpenSSL compatibility
**What changed**: Xiangfei Ding posted a patch to `imap-send.c` using `ASN1_STRING_length` and `strncasecmp` to avoid assuming null-termination in OpenSSL derivatives, and Junio C Hamano reviewed it.

**Problem**: Upcoming changes in OpenSSL derivatives (e.g., BoringSSL) will no longer guarantee null-termination for `ASN1_STRING` data, risking security issues or crashes.

**Subsystem**: `imap-send.c`, TLS certificate validation.

**Impact**: Bugfix; anticipates a breaking change in OpenSSL derivatives.

**Today’s developments**:
Xiangfei Ding posted a patch updating `host_matches` and `verify_hostname` to use `ASN1_STRING_length` and `strncasecmp` for safe comparison, avoiding assumptions about null-termination. Junio C Hamano reviewed the patch, requesting commit message clarification (whether the change affects only derivatives or upstream OpenSSL as well), removal of a redundant in-body `From:` line, and a line-wrap fix for `host_matches`. He also suggested declaring `pat_len` as `size_t` to eliminate a cast, though he did not demand the change. The patch remains uncontroversial and well-motivated, targeting a latent correctness issue.

---

## In brief
- **[blame default ignore-revs]**: Ravi Mistry confirmed the `the_repository` cleanup in v3, and Junio C Hamano marked the series ready for `next`.
- **[SubmittingPatches]**: Jon Simons posted a patch discouraging the "add failing test, then fix" anti-pattern, and Junio C Hamano suggested revised wording.
- **[GSoC microproject]**: Aniket Gupta proposed pluralizing the "Checking connectivity (%d objects)" message in `builtin/fsck.c` using `Q_()`.
- **[push refspec matching]**: Jon Simons acknowledged Kristoffer Haugsbakk’s feedback about combining regression tests and fixes into a single commit, planning to squash them in v2.