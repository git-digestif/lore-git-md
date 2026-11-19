# Git mailing list daily digest for 2026/10/11

## Long threads

### rerere: wait for MERGE_RR.lock before giving up

*Long thread: 29 replies so far from 4 participants over 40 days; 2 messages that day.*

Thomas Bachem's series changes `rerere` so that a rebase hitting a conflict no longer dies at once when a background `git rerere gc` holds `MERGE_RR.lock`: `setup_rerere()` now waits up to `rerere.lockTimeout` milliseconds (default 1000) before failing, and a hidden `--skip-locked` option on `git rerere gc` makes auto maintenance quietly do nothing while the lock is held. Earlier revisions dropped the patch that let conflict-stopping commands go on without rerere. Today Junio C Hamano reviewed patch 1/2, accepting the millisecond unit for `rerere.lockTimeout` and the call to `repo_hold_lock_file_for_update_timeout()`, while questioning a doc sentence about lock-timeout behavior and several test details. Thomas Bachem answered, agreeing to drop the sentence, moving the `wait` safeguard into `test_when_finished`, and using Junio's more specific `Unable to create.*MERGE_RR\.lock` error pattern in the tests.

### branch: let --delete-merged find squash merged branches

*Long thread: 22 replies so far from 4 participants over 13 days; 1 message that day.*

Harald Nordgren's series teaches `git branch --delete-merged` to treat a branch as merged when some upstream commit since the fork point contains all of its changes, printing `Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)` and keeping branches whose changes only partly landed. Today he reposted the series as v2, which diffs each upstream commit once and shares the result between all branches for speed, fixes a branch merged with a merge commit being reported as "landed as" the merge when another branch was searched first, uses `git merge --squash` as the example, and adds `--no-squashed` to skip the squash search, which is slower and may download blobs in a partial clone.

## The day in brief

Junio C Hamano marked the `repack --dry-run` fix for `next` and said the v3 blame ignore-revs series looks good and would be marked ready for `next` unless objections arrive. Harald Nordgren reposted his squash-merge detection for `git branch --delete-merged` as v2 with a `--no-squashed` opt-out, and Thomas Bachem answered Junio's review of the rerere lock-wait patch.

## Notable threads

### repack: do not rebuild packs on --dry-run

A patch fixes `git repack --drop-filtered --dry-run`, which is documented to list droppable objects without rebuilding any pack or deleting anything but actually writes a new promisor pack and deletes old packs because the `--dry-run` block falls through into the regular repack code. The fix returns right after the candidates are listed, with a test that the pack directory is unchanged with and without `-d`. Junio C Hamano marked the topic for `next`.

### blame: ignore revs in HEAD:.git-blame-ignore-revs by default

Ravi Mistry posted v3 of the series that makes `git-blame` and `git-annotate` automatically use the `HEAD:.git-blame-ignore-revs` blob when it exists, so local runs match hosting platforms without per-clone `blame.ignoreRevsFile` configuration. The only change since v2 is that `parse_default_ignore_revs_blob()` uses `sb->repo` instead of `the_repository`. Junio C Hamano said v3 looks good and that he would mark the topic ready for `next` unless objections are heard in a few days.

### status: agree with diff and add when conversion is active

Curtis Allen Smith's series fixes `git status` disagreeing with `git diff` and `git add` when a clean filter is active: under `text eol=lf`, a tool rewriting a file with CRLF endings makes `git status` report it modified while `git diff` shows nothing, because `ie_match_stat()` trusts a size difference as proof of change. Torsten Bögershausen corrected the description of `git add` (it stages nothing but still updates the index), confirmed the issue is not Windows-specific, and objected to the proposed `core.convertAwareStatus` opt-out as unnecessary.

## In brief

- Marc Becker's patch fixes a crash in `git fast-import` when looking up commits created in the same session; Aniket Gupta tested it, confirmed the fix, and requested a test in `t9300`.
- Xiangfei Ding's patch makes `imap-send.c` use `ASN1_STRING_length` and `strncasecmp` instead of assuming null-terminated `ASN1_STRING` data; Junio C Hamano reviewed it, asking about the in-body `From:` line and line wrapping.
- Yoichi NAKAYAMA explained adjusted error-case behavior in `worktree repair` and said he will split the commits to separate modified from unchanged behavior.
- Jon Simons documented in `Documentation/SubmittingPatches` the preference for submitting a bug fix's test as `test_expect_success` in the same commit as the code change; Junio C Hamano suggested shorter wording.
- René Scharfe's patch makes `archive-zip.c` omit the UT extra field when the commit time exceeds the signed 32-bit range.
- **Jon Simons on v2 commit squashing**: Jon Simons said he will look at squashing test-then-fix commits for v2 of his series speeding up client-side refspec matching in push.

- Junio C Hamano asked for clarification on the commit message of the patch moving mergesort tests to Clar.
- Aniket Gupta introduced himself as a GSoC applicant and proposed an fsck plural-message microproject using `Q_()`.
