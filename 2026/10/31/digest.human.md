# Git mailing list daily digest for 2026/10/31

## Long threads

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread: 53 replies so far from 6 participants over 33 days; 2 messages that day.*

Harald Nordgren's series adds a `--delete-integrated <pattern>` option to `git branch` that deletes branches whose changes landed upstream via squash or rebase merges, even when their tips are not reachable from the upstream, naming the landing commit in output like "Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)". Earlier revisions restructured the feature so `--delete-merged` again only deletes branches whose tips are reachable, and Jasper Mayone's testing of v4 against master confirmed both options behave correctly across squash merges, renames, and partial landings. Today Harald Nordgren suggested renaming a local `opt` variable to `diffopt` in `find_landed_commits()` to match the `diff_options` type, and he raised the same question Junio C Hamano had asked about `merge_keeps_upstream()`: whether the index is guaranteed to be read before `ll_merge()` is called with `the_repository->index`, since an unpopulated istate would prevent `ll_merge()` from consulting custom low-level merge attributes; he asked whether ignoring attributes is intentional and said he would take a look.

## The day in brief

Nakul V S posted v2 of a patch to `builtin/revert.c` and the sequencer refusing to pick or revert shallow boundary commits by default across revert, cherry-pick, and rebase. Linus Torvalds submitted a fix for `git describe` miscounting commit distance when commit dates are skewed. Elia Pinto posted two series switching object and pack size handling from `unsigned long` to `size_t` to avoid truncation of objects 4GiB and larger on platforms where `long` is 32 bits.

## Notable threads

### Sequencer refuses shallow boundary commits by default

Nakul V S posted v2 of a patch to `builtin/revert.c` and the sequencer that refuses to revert, cherry-pick, or rebase commits at the shallow boundary of a shallow clone, where such a commit looks like a root that added every file at once and reverting it would empty the working tree. The check now runs when each command builds its list of commits to replay, so a refused command leaves no sequencer or rebase state behind; `--allow-shallow-boundary` overrides the check for revert and cherry-pick only, while rebase always refuses and suggests `git fetch --unshallow`. In replies, Nakul V S explained to Phillip that the escape hatch is kept out of caution and offered to drop it, and told Junio C Hamano that `git commit --amend` is left for a separate patch.

### `git describe` distance counting with skewed commit dates

Linus Torvalds submitted a patch to `builtin/describe.c` fixing `git describe` miscomputing the commit distance when commit dates are unreliable, as seen in the Linux v7.4 merge window where the reported count included commits already in the tagged release. The fix changes the priority queue ordering from `compare_commits_by_commit_date` to `compare_commits_by_gen_then_commit_date` and adds a test in `t/t6120-describe.sh` for a tag dated before its parent. Tuomas Ahola shared an alternative rewrite of the test using `test_tick` arithmetic instead of explicit dates.

### `size_t` for streamed large blobs in pack-objects

Elia Pinto posted v2 of a patch to `builtin/pack-objects.c` fixing corruption of blobs 4GiB and larger on platforms where `long` is 32 bits, where the streamed path in `write_no_reuse_object()` truncated the size in the pack header and `write_large_blob_data()` could overflow the compressed-length accumulation, misaligning all subsequent objects. The patch uses `size_t` for the object size and `off_t` for the accumulated compressed length and return value, following Simon Richter's comment, with a `Helped-by` tag added.

## In brief

- Ravi Mistry posted a 4-patch series for HTTP retry handling: fixing an off-by-one retry count and `Retry-After` Trace2 timing in `http.c`, adding exponential backoff when no `Retry-After` header is present, a new `http.retryStatusCodes` config for transient 5xx codes, and retry support for POST requests in `post_rpc()` in `remote-curl.c`.
- Thomas Bachem submitted a patch making merge-ort accept `merge.renames`, `diff.renames`, and `merge.directoryRenames` given without a value (meaning true per git-config docs) by reading them with `repo_config_get_value()` instead of `repo_config_get_string()`, with tests in `t/t6423` and `t/t6434`.
- Cynthia Uche reported that all 118 `t0040` and 261 `t9300` tests pass on Christian Couder's early-scan-options series, suggested a comment clarifying that `EARLY_SCAN_STOP_AT_NON_OPTION` stops the scan at the first non-option, and found no other issues.
- Harald Nordgren posted v2 making `@{push}` resolve to the push remote's same-named branch under `push.default=simple` when `remote.pushDefault` points at a fork, moving the "simple" resolution into a new `branch_get_push_simple()` helper.
- Jasmeet (Jazz) Bhatia submitted a patch passing the already prepared untracked file list from `do_push_stash()` to `do_create_stash()` to skip redundant index work, reporting 16.7% faster `stash -p` without fsmonitor in p3904 tests.
- Jeff King posted a patch to `t/t5702` replacing `env`-redirect-and-grep checks of `GIT_PROTOCOL` with direct `echo` output compared via `test_cmp`, because GNU coreutils 9.12 quotes values containing `=`; Junio C Hamano confirmed the patch is correct and Collin Funk explained the coreutils quoting change.
- Elia Pinto posted a 4-patch series using `size_t` when parsing and checking objects in `object-file.c`, `fsck.c`, `commit.c`, and `tag.c` to fix truncated sizes above 4GiB on LLP64 platforms; Simon Richter noted `size_t` is 32-bit on 32-bit architectures and suggested `off_t` for piecewise reads.
- Elia Pinto posted a 4-patch series using `size_t` when receiving and verifying packs in `unpack-objects`, `index-pack`, and `pack-check`, fixing dies, false "SHA1 COLLISION FOUND" errors, and prefix-only fsck checks for objects 4GiB and larger on 32-bit `long` platforms.
