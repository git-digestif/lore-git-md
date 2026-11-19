# Git mailing list daily digest for 2026/10/11

## Long threads

### rerere: wait for MERGE_RR.lock before giving up

*Long thread (root 2026/09/02/08-31-37): 29 replies so far from 4 participants over 40 days; 2 messages that day.*

This thread concerns `rerere.c` and the `MERGE_RR.lock` file: a rebase that hits a conflict dies when a background `git rerere gc` holds the lock at that moment. Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for the lock before failing, and adds a hidden `--skip-locked` option to `git rerere gc` so the gc started by auto maintenance does nothing while the lock is held. Earlier revisions had carried a third patch letting conflict-stopping commands go on without rerere, but that was dropped in v7. Today Junio C Hamano reviewed patch 1/2, accepting the millisecond unit for `rerere.lockTimeout` (noting all existing timeout variables are in milliseconds) and the use of `repo_hold_lock_file_for_update_timeout()`, while questioning whether a sentence about failing when time is up is necessary. He also questioned several new tests: whether the background unlocker reliably unlocks within the 5000 ms timeout on loaded CI, whether `wait` guards against future breakage, and whether the `test_grep` pattern should be more specific. Thomas Bachem answered, agreeing to drop the sentence, confirming the unlocker can be up to four seconds late with the same numbers used in t0610 and t1300, moving the `wait` into `test_when_finished`, adopting the more specific `Unable to create.*MERGE_RR\.lock` pattern, and arguing no pid-file removal is needed.

### branch: let --delete-merged find squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 22 replies so far from 4 participants over 13 days; 1 message that day.*

This thread concerns `builtin/branch.c` and `git branch --delete-merged`: branches merged via GitHub's "Squash and merge" or "Rebase and merge" are never deleted because their tips are not reachable from upstream. Harald Nordgren reposted the series as v2 today. The first patch treats a branch as merged when some upstream commit since the fork point contains all of its changes, printing "Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)". The v2 changes diff each upstream commit once and share the result between all branches for speed, fix a branch merged with a merge commit being reported as "landed as" the merge when another branch was searched first, use `git merge --squash` as the example, and add `--no-squashed` to skip the squash search, which is slower and may download blobs in a partial clone. Earlier discussion had covered performance concerns in large and partial clones, with Phillip Wood endorsing the `git cherry` patch-id model and arguing a way to turn detection off is still wanted.

## The day in brief

Junio C Hamano marked the `repack --dry-run` fix for `next`, and said the v3 of the blame default-ignore-revs series looks good and would be marked ready for `next` unless objections are heard. Harald Nordgren reposted the squash-merge detection for `git branch --delete-merged` as v2 with a shared-diff speedup and a `--no-squashed` opt-out.

## Notable threads

### repack: do not rebuild packs on --dry-run

This is a bugfix in `builtin/repack.c`: `git repack --drop-filtered --dry-run` is documented to list objects without rebuilding any pack or deleting anything, but the `--dry-run` block falls through into the regular repack code, writing a new promisor pack and deleting old packs with `-d`. The fix returns right after the candidates are listed, with a test that the pack directory is unchanged. Today Junio C Hamano marked the topic for `next`.

### blame: ignore revs in HEAD:.git-blame-ignore-revs by default

This series teaches `git blame` and `git annotate` in `builtin/blame.c` to automatically use the `HEAD:.git-blame-ignore-revs` blob when it exists, so local runs match hosting platforms without manual `blame.ignoreRevsFile` configuration. Ravi Mistry posted v3 today; the only change since v2 is that `parse_default_ignore_revs_blob()` uses `sb->repo` instead of `the_repository`, following Junio C Hamano's suggestion that the scoreboard already holds a repository pointer. Junio C Hamano then said v3 looks good and that he would mark the topic ready for `next` unless objections are heard in a few days.

### status: agree with diff and add when conversion is active

This series addresses a contradiction in `read-cache.c`: under `text eol=lf`, a tool rewriting an unchanged file with CRLF endings makes `git status` report it modified while `git diff` shows nothing and `git add` stages nothing, because `ie_match_stat()` trusts a size difference as proof of content change. The series makes status re-check content when conversion is active and adds a `core.convertAwareStatus` opt-out. Today Torsten Bögershausen corrected the description of `git add` (it stages nothing but does update the index), objected to the opt-out option as unnecessary since users without filters are unaffected, and confirmed the issue is not Windows-specific.

### SubmittingPatches: avoid add-failing-test-then-fix anti-pattern

This is a documentation-only change to `Documentation/SubmittingPatches` by Jon Simons, documenting the preference that a bug fix submit its test as `test_expect_success` in the same commit as the code change rather than adding a failing test and flipping it later. Today Junio C Hamano reviewed it, saying it matches what contributors have been told, but suggested dropping the initial advice to write a `test_expect_failure` test and proposed a shorter replacement sentence about including tests in `test_expect_success()` blocks that demonstrate the breakage when the fix is omitted.

### fast-import: fix lookup for new commits

This is a bugfix in `builtin/fast-import.c`: queries for commits created during the same `git fast-import` session, such as `reset refs/heads/newbranch` with `from <new-commit-oid>`, crash because new objects may not yet be reachable via the ODB layer. Marc Becker's patch resolves requested tags/commits via the object cache first. Today Aniket Gupta reported testing on top of 6de20f609 with DEVELOPER=1: the original crash reproduces, and with the patch fsck is clean and lookups work for new commits, same-session tags, and tags of tags, while existing objects behave as before. He also noted a double space in the commit message, asked whether the lookup speed improvement was measured, pointed out an over-80-column declaration, and requested a test in t9300 using `get-mark`.

## In brief

- Xiangfei Ding posted a patch to `imap-send.c` making `host_matches` and `verify_hostname` use `ASN1_STRING_length` and `strncasecmp` instead of assuming `ASN1_STRING` data is null-terminated, since OpenSSL derivatives like BoringSSL may stop null-terminating; Junio C Hamano confirmed the changes are correct, asked about the in-body `From:` line, and suggested wrapping a long line and declaring `pat_len` as `size_t`.
- Yoichi NAKAYAMA explained that the preimage `infer_backlink()` accepted the `gitdir:` prefix more leniently than `read_gitfile_gently()`, and said he will split the worktree-repair commits to distinguish behavior changes from pure refactoring and to explain the adjusted error-case handling.
- René Scharfe posted a patch to `archive-zip.c` omitting the Extended Timestamp Extra Field (UT) for zip entries when the commit time exceeds a signed 32-bit integer, with tests in `t/t5004-archive-corner-cases.sh` for mtimes at and beyond the limit.
- Junio C Hamano asked for clarification on the commit message of the mergesort series that moves sorting tests from the shell helper to Clar unit tests and retires the p0071 benchmark, questioning the stated purpose of noting the tests only use `mergesort.h`.
- Jon Simons acknowledged feedback on the push refspec-matching speedup series and said he will squash commits that introduce `test_expect_failure` cases into the commits that make them pass for v2, after letting v1 soak.
- Aniket Gupta introduced himself as a GSoC applicant and proposed as a microproject changing the "Checking connectivity (%d objects)" message in `builtin/fsck.c` to use `Q_()` for plural handling, asking whether the task is still available.
