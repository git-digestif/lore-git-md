# Git mailing list daily digest for 2026/10/18

## Long threads

### branch: let --delete-merged find squash merged branches

*Long thread (root 2026/09/29/07-30-30): 32 replies so far from 5 participants over 20 days; 1 message that day.*

This series changes `git branch --delete-merged` so that branches merged via GitHub's "Squash and merge" or "Rebase and merge" -- whose tips are not reachable from upstream -- are treated as merged when some upstream commit contains all of their changes, naming that commit in the output as "landed as". Earlier revisions added a `--no-squashed` flag to skip the slower squash search. Today Harald Nordgren reported on a local experiment with a range-diff-based approach: range-diff pairs commits on the diff alone (log message and author appear only in its output), so it did not help with matching. On his clone with 65 branches, the v2 implementation takes about 0.5s and finds 14 branches to delete, while the range-diff version takes about 10s and finds the same 14 branches.

### push: check pushed ref for --force-if-includes

*Long thread (root 2026/09/04/21-01-20): 20 replies so far from 5 participants over 45 days; 1 message that day.*

This three-patch series fixes `git push --force-if-includes`, which consulted the reflog of the local branch named after the destination instead of the ref actually being pushed -- causing both data loss (a push clobbering the remote main could pass) and false rejections. Today Tyler Cipriani sent v7, rebased on maint at v2.56.0. Patch 1/3 makes `check_if_includes_upstream()` consult the reflog of the branch the pushed ref resolves to via `ref->peer_ref`, explicitly rejects non-branch sources (detached HEAD, tags, oids) as unverifiable, and allows deletions by detecting null peer_ref OIDs. Patch 2/3 introduces `REF_STATUS_REJECT_UNVERIFIABLE` and the `pushRefUnverifiable` advice so such rejections suggest `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes` instead of the misleading "remote ref updated since checkout". Patch 3/3 defers the reflog reachability check until after fast-forward rules are evaluated in `set_ref_status_for_push()`, so valid fast-forward pushes -- including from tags or with expired reflogs -- are no longer rejected when no force is needed.

### doc: add more AsciiDoc cross-references

*Long thread (root 2026/09/22/19-29-02): 20 replies so far from 5 participants over 27 days; 1 message that day.*

This documentation change replaces plain references like "see EXAMPLES below" with the two-part AsciiDoc form `<<EXAMPLES,EXAMPLES>>`, which renders as plain `EXAMPLES` rather than `the section called "EXAMPLES"` or `[EXAMPLES]`; it also changes some heading HTML IDs from `_examples` to `EXAMPLES`, which can break existing links. Today Jean-Noël AVILA argued for the double-reference notation because the manual pages are now translated, and translated terms can require changes in target languages that are not compatible with simply inserting the title.

## The day in brief

Tyler Cipriani sent v7 of the `--force-if-includes` fix series, rebased on maint, deferring the reflog check until after fast-forward rules so valid fast-forward pushes are no longer rejected. Harald Nordgren posted v2 of the reflog `--operations` and `--historical-branch` series and reported that a range-diff-based alternative to squash-merge detection was much slower on his 65-branch clone. Olof Fairbrook submitted two small fixes: one for `git bundle create` freeing uninitialized memory on bad arguments, and one making `git config --blob` read bytes as unsigned characters.

## Notable threads

### odb: ignore refs of shallow and grafted alternates

This two-patch series fixes the connectivity check in `connected.c` for deepening fetches with alternates -- previously it walked the alternate's whole history with `--alternate-refs` as a positive revision, taking 227s with a 7M-object alternate -- and adds `odb_for_each_alternate_ref()` in `odb.c` so shallow and grafted alternates' refs are ignored. Today Junio C Hamano reported that the test in patch 2/2 seems to break CI only on Windows, citing specific failing GitHub Actions jobs, and said he will eject the topic from `seen` for now.

### read-cache: do not trust a size change when conversion is active

This patch in `read-cache.c` fixes `git status` reporting a file as modified while `git diff` and `git add` agree its contents are unchanged: `ie_modified()` treated a size change as proof of modification, but with conversion active (text/eol attributes, `core.autocrlf`, clean filters) different bytes can convert to the same blob. The fix falls through to `ce_modified_check_fs()` for such paths. Today Torsten Bögershausen suggested that, looking at `CE_MATCH_RACY_IS_DIRTY`, the call to `would_convert_to_git()` may be better done in `ie_match_stat()`, and reported testing the patch with a real repo using a clean filter that replaces a TAB with 4 spaces, where `git status` no longer claims the file to be modified.

### bundle: do not free uninitialized memory on bad arguments

Today Olof Fairbrook submitted a patch to `bundle.c` fixing `git bundle create` freeing uninitialized memory when passed an unrecognized argument: since 11ee9a75e7 the error path clears `revs_copy.pending` before `revs_copy` is filled from `revs`, so the cleanup reads uninitialized stack memory. The patch initializes `revs_copy` to `{ 0 }` at declaration and adds a test in `t/t5607-clone-bundle.sh` verifying that an unknown argument after the revisions errors out without leaving a bundle file.

### config: read the bytes of a blob as unsigned characters

Today Olof Fairbrook submitted a patch fixing `git config --blob` and other in-memory config readers: `config_buf_fgetc()` in `config.c` returned each byte as a plain `char`, so on platforms where `char` is signed a byte `0xff` came back as `-1` (EOF), silently truncating the blob, and a UTF-8 byte order mark was reported as a bad config line. The patch returns the byte as an `unsigned char` and updates `config_buf_ungetc()` to compare it as such, adding tests for `0xff` bytes and BOM-prefixed blobs in `t/t1307-config-blob.sh`.

### reflog: add --operations and --historical-branch

Today Harald Nordgren posted v2 of this three-patch series adding two options to `git reflog` and `git log -g`: `--operations` shows one entry per command, folding a rebase into the entry that finished or aborted it, and `--historical-branch` decorates each entry with the branch HEAD was on when it was recorded. Changes in v2 move the historical-branch decoration rightward in the display, after the `HEAD@{n}` selector, for cleaner alignment, and improve commit messages.

## In brief

- Colin Hinton submitted a patch deferring `remote.<name>.followRemoteHEAD` validation in `remote.c` until `do_fetch()`, so commands like `git push` or `git remote -v` no longer warn about the setting; `fetch.followRemoteHEAD` now also accepts `warn-if-not-<branch>`.
- Harald Nordgren proposed that with `branch.autoSetupMerge=inherit`, a new branch should first inherit the upstream of the branch being forked from, falling back to tracking the remote-tracking start point itself when the current branch has no upstream.
- Andy Koppe submitted a Makefile patch passing `--target-dir target` directly in the cargo build recipes for gitcore, so builds do not fail with a link error when a custom target directory is configured.
- Shashank S submitted a patch removing `USE_THE_REPOSITORY_VARIABLE` from `prune-packed.c`, making `prune_packed_objects()` take a `struct repository *` argument.
- Omri Sarig submitted a documentation patch removing stale "INCOMPATIBLE OPTIONS" cross-references from `--allow-empty-message`, `--ignore-date`/`--reset-author-date`, and `--signoff` in `Documentation/git-rebase.adoc`.
- Srujith Poojary posted v3 of a test modernization patch replacing old-style `test -f` checks with `test_path_is_file` and `test_path_is_missing` in `t/t2030-unresolve-info.sh`.
- Srujith Poojary submitted a patch to `t/t0001-init.sh` avoiding piping `git version --build-options` directly into `sed`.
- Phillip Wood noted that Keryle Fondji's `git var` parse-options refactor was already done in commit e19f0779c67, merged to master.
- Phillip Wood advised Keryle Fondji to scan the mailing list archives and the `seen` branch to avoid duplicating work, and Junio C Hamano thanked him for noting that the point of a microproject is familiarity with the patch process.
- Aniket Gupta asked whether the fsck `Q_()` pluralization microproject is still available.
- Usman Akinyemi replied to Keryle that the `archive-zip.c` `Q_()` microproject appears unclaimed and can proceed.
- A user proposed adding a help-message note to `git remote add` clarifying that "origin" is the default name used by `git clone` but any valid name can be specified.
