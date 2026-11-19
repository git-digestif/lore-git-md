# Git mailing list daily digest for 2026/10/18

## Long threads

### branch: let `--delete-merged` find squash-merged branches

*Long thread: 32 replies so far from 5 participants over 20 days; 1 message that day.*

This series makes `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all of its changes -- covering GitHub "Squash and merge" and "Rebase and merge", whose tips are not reachable from upstream -- naming that commit as "landed as", with a `--no-squashed` option to skip the slower search. Earlier revisions shared each upstream diff across all branches for speed. Today Harald Nordgren reported that a local range-diff approach pairs commits on the diff alone (log message and author appear only in its output), so it did not help matching; on his 65-branch clone the v2 code takes about 0.5s and finds 14 branches to delete, while the range-diff version takes about 10s and finds the same 14.

### push: check pushed ref for `--force-if-includes`

*Long thread: 20 replies so far from 5 participants over 45 days; 1 message that day.*

This series fixes `--force-if-includes` to consult the reflog of the ref actually being pushed rather than the local branch named after the destination, a mismatch that allowed clobbering pushes and false rejections. Tyler Cipriani sent v7 today, rebased on maint at v2.56.0. Patch 1/3 resolves the pushed ref via `ref->peer_ref`, rejects non-branch sources (detached HEAD, tags, oids) as unverifiable, and allows deletions. Patch 2/3 adds a `pushRefUnverifiable` advice suggesting `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes` instead of the misleading "remote ref updated since checkout". Patch 3/3 defers the reflog check until after fast-forward rules in `set_ref_status_for_push()`, so valid fast-forward pushes, including from tags or with expired reflogs, proceed. The new tests fail on maint and pass with the series.

### doc: add more AsciiDoc cross-references

*Long thread: 20 replies so far from 5 participants over 27 days; 1 message that day.*

Julia Evans's patch updates `Documentation/` to use the verbose `<<EXAMPLES,EXAMPLES>>` cross-reference form, since the shorthand is sometimes rendered as `the section called "EXAMPLES"` or `[EXAMPLES]` while the two-part form renders as plain `EXAMPLES`; some heading HTML IDs change from `_examples` to `EXAMPLES`, which can break links. Today Jean-Noël AVILA argued for the double-reference form because the manual pages are now translated and translated terms can require changes incompatible with simply inserting the title.

## The day in brief

Tyler Cipriani posted v7 of the `--force-if-includes` series, rebased on maint, checking the pushed ref's own reflog and letting fast-forward pushes through. Junio C Hamano will eject the alternates connectivity-check topic from `seen` after its test broke Windows CI. Harald Nordgren reported that a range-diff approach to detecting squash-merged branches was slower and no better at matching than the current implementation.

## Notable threads

### odb: ignore refs of shallow and grafted alternates

This two-patch series fixes the connectivity check in `connected.c` for deepening fetches with alternates, which previously walked the alternate's whole history via `rev-list --alternate-refs` (227s with a 7M-object alternate), and adds `odb_for_each_alternate_ref()` in `odb.c` so shallow and grafted alternates' refs are ignored by the connectivity check and by fetch-pack's and receive-pack's "have"s. Today Junio C Hamano reported that the test in patch 2/2 breaks CI only on Windows and said he will eject the topic from `seen` for now.

### read-cache: do not trust a size change when conversion is active

This patch fixes `ie_modified()` in the read-cache code so `git status` no longer reports a file as modified when a CRLF rewrite or clean filter converts different working-tree bytes to the same blob; for paths subject to conversion, a size change now falls through to `ce_modified_check_fs()`. Today Torsten Bögershausen suggested the `would_convert_to_git()` call may belong in `ie_match_stat()` given `CE_MATCH_RACY_IS_DIRTY`, and reported that with a clean filter replacing a TAB with four spaces, `git status` no longer claims the file is modified.

### bundle: do not free uninitialized memory on bad arguments

Olof Fairbrook fixed `git bundle create` in `bundle.c` freeing uninitialized stack memory on an unrecognized argument: since 11ee9a75e7 the error path clears `revs_copy.pending` before `revs_copy` is filled, so the patch initializes it to `{ 0 }` at declaration and adds a test in `t/t5607-clone-bundle.sh`.

## In brief

- Olof Fairbrook made `config_buf_fgetc()` return blob bytes as `unsigned char`, so `0xff` is no longer treated as EOF and BOM-prefixed blobs parse, with tests in `t/t1307-config-blob.sh`.
- Harald Nordgren posted v2 adding `--operations` and `--historical-branch` to `git reflog` and `git log -g`, moving the branch decoration rightward for alignment.
- On `autoSetupMerge=inherit` tracking a remote-tracking start point, Phillip Wood judged the change not clearly better, and Harald Nordgren proposed inheriting the forked branch's upstream with the start point as fallback.
- Colin Hinton deferred `remote.<name>.followRemoteHEAD` validation to `do_fetch()` so `git push` and `git remote -v` stop warning; `fetch.followRemoteHEAD` now also accepts `warn-if-not-<branch>`.
- Andy Koppe pinned `--target-dir target` in the Makefile's cargo recipes, fixing link errors when `CARGO_TARGET_DIR` or `~/.cargo/config.toml` sets a custom target directory.
- Shashank S removed `USE_THE_REPOSITORY_VARIABLE` from `prune-packed.c`, passing a `struct repository *` via a new `struct prune_data` to the loose-file callbacks.
- Srujith Poojary posted v3 replacing old-style `test -f` checks in `t/t2030-unresolve-info.sh` with `test_path_is_file` and `test_path_is_missing`.
