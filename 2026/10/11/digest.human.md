# Git mailing list daily digest for 2026/10/11

## The day in brief
The `git blame` default ignore-revs file feature reached integration-ready status after Junio C Hamano approved the v3 series. A ZIP archive timestamp bugfix was posted, omitting extended timestamps for out-of-range commits. The `git fast-import` crash fix gained test confirmation, and a new OpenSSL compatibility patch for `imap-send` was submitted. Documentation updates clarified patch submission guidelines for test-and-fix patterns.

## Notable threads

### `rerere` race condition fix nears integration
The v7 series fixing a long-standing race condition in Git’s `rerere` subsystem received final polish. Junio C Hamano reviewed patch 1/2, requesting documentation and test improvements to align with Git’s existing patterns for lock timeouts. Thomas Bachem incorporated all feedback, dropping redundant documentation and tightening test error-message matching. The series—introducing a configurable `rerere.lockTimeout` and `--skip-locked` mode for background maintenance—is now technically complete and ready for integration.

### `git blame` default ignore-revs file approved for `next`
The v3 series making `git blame` automatically use `HEAD:.git-blame-ignore-revs` as the default ignore file was approved by Junio C Hamano. Ravi Mistry addressed the last maintainer request by replacing the global `the_repository` variable with `sb->repo` in `parse_default_ignore_revs_blob()`. Junio confirmed the series looks good and marked it for the `next` branch. The hardening (rejecting malformed input and preventing promisor fetches) and default file lookup are now feature-complete, with no unresolved technical concerns.

### ZIP archive timestamp bugfix posted
René Scharfe posted a patch addressing one of three date-handling bugs in Git’s ZIP export. The fix omits the Extended Timestamp (UT) extra field when the commit time exceeds the 32-bit signed integer range, preventing silent timestamp wrap that causes tools like UnZip to misbehave in update mode. The patch aligns with René’s updated position advocating rejection (omission) of out-of-range timestamps rather than clamping, prioritizing correctness over tool compatibility. Tests verify the UT field is included for 32-bit timestamps and omitted for out-of-range values.

### `git fast-import` crash fix gains test confirmation
Aniket Gupta confirmed the `git fast-import` crash fix resolves the issue for commits, tags, and tags-of-tags created mid-session. The patch adds a cache lookup in `parse_from_existing` before falling back to the object database, preventing crashes when resolving in-session objects. Aniket requested test coverage in `t9300` to ensure the fix is covered by the test suite. The patch remains minimal and targeted, touching only `builtin/fast-import.c`.

### OpenSSL compatibility patch for `imap-send`
Xiangfei Ding posted a patch updating `imap-send.c` to stop assuming `ASN1_STRING` data is null-terminated. The change uses `ASN1_STRING_length` and `strncasecmp` to safely compare hostnames in TLS certificates, addressing an upcoming breaking change in OpenSSL derivatives like BoringSSL. Junio C Hamano reviewed the patch, requesting minor commit message and code hygiene improvements. The fix is forward-looking and low-risk, with no behavior changes for existing OpenSSL versions.

## In brief
- **`git worktree repair` validation**: Yoichi NAKAYAMA agreed to split patch 1/2 into smaller commits to separate behavioral changes from refactoring, improving clarity for the worktree ID validation logic.
- **`git branch --delete-merged` squash detection**: Harald Nordgren posted v2 of the series, adding a `--no-squashed` flag to skip slower squash detection and fixing a merged-branch attribution bug.
- **Test infrastructure cleanup**: Junio C Hamano requested commit message clarification for the first patch in the mergesort Clar migration series.
- **`git repack --dry-run` regression fix**: Junio marked the topic for the `next` branch after D. Ben Knoble provided his `Reviewed-by`.
- **`git push` refspec-matching performance**: Jon Simons acknowledged Kristoffer Haugsbakk’s feedback about combining regression tests and fixes into a single commit, planning to squash them in v2.
- **GSoC microproject proposal**: Aniket Gupta proposed pluralizing the "Checking connectivity (%d objects)" message in `builtin/fsck.c` using `Q_()`.
- **Documentation update**: Jon Simons and Junio C Hamano discussed wording for a `SubmittingPatches` update discouraging the "add failing test, then fix" anti-pattern.