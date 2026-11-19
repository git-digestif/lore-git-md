# Git mailing list daily digest for 2026/09/29

## Long threads

### stash: merge the index in-core for "stash apply --index"

*Long thread: 49 replies so far from 5 participants over 11 days; 8 messages that day.*

D. Ben Knoble posted v4 of "stash: clean up index-mode test merge", fixing the interaction between autostashing, staged index entries, and `stash.index=true` in `builtin/stash.c`. The last patch replaces the `diff-tree`/`apply --cached`/`reset` subprocesses with an in-core `merge_incore_nonrecursive()` call, fixing a non-re-entrant `save_autostash()` lock on a deleted MERGE_AUTOSTASH ref; v4 also copies the merge tree OID only on success and pins `gc.reflogExpire` to `never` in t5520 so auto-maintenance cannot expire the reflog entry the "--rebase -f" test needs. Phillip Wood called the range-diff ready apart from patch 4's commit message. Junio C Hamano flagged that on a hard error `result.clean` is -1, so the `if (!result.clean)` branch would dereference a NULL `result.tree`.

### upload-pack: server-side lazy-fetch control via uploadpack.lazyFetchTrusted

*Long thread: 31 replies so far from 3 participants over 82 days; 2 messages that day.*

Christian Couder's series adds the protected config variable `uploadpack.lazyFetchTrusted`, letting server operators mark repos whose git dir is trusted for lazy fetching when serving clones, motivated by large-object promisor setups where a server cannot serve filtered clones for blobs it moved to a promisor remote. It touches `builtin/upload-pack.c`, `upload-pack.c`, `promisor-remote.c`, and `setup.c`, and adds a `GIT_INTERNAL_LAZY_FETCH_DEPTH` recursion limit. Today Junio C Hamano suggested doc wording fixes, called the control logic in `builtin/upload-pack.c` trivially clear, and floated an optional `:(optional)` prefix for `safe.directory` paths.

### push: fix --force-if-includes when the remote-tracking ref has no reflog

*Long thread: 23 replies so far from 6 participants over 27 days; 4 messages that day.*

Aleksei Sviridkin posted v4 of a fix in `is_reachable_in_reflog()`: when the remote-tracking ref has no reflog (the files-backend default after clone), the timestamp cutting off the local reflog walk stays uninitialized, which on some machines rejects valid pushes with "remote ref updated since checkout". The patch initializes it to 0 and adds a t5533 test that fails without the fix only reliably under `-ftrivial-auto-var-init=pattern`. Junio C Hamano said the v4 changes look sensible, will replace the patch and mark it for `next`, and noted trivial t5533 conflicts with another topic.

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 20 replies so far from 4 participants over 11 days; 4 messages that day.*

Harald Nordgren posted v4 of a four-patch series for shallow repositories, where a wildcard refspec makes fetching a new remote unusably slow. It adds `remote.<name>.refmap` (reusing `--refmap`), infers branches to fetch from local upstreams, and adds `git remote add --limited-fetch`, defaulting to it when shallow; v4 drops the automatic default-branch fetch and fixes the "new ref HEAD" report. Junio C Hamano said he will replace the series and asked that t/t5585-fetch-refmap.sh be renumbered, as that number is taken by another topic.

### branch: let --delete-merged find squash-merged branches

*Long thread: 10 replies so far from 3 participants over 1 day; 11 messages that day.*

Harald Nordgren posted a patch making `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all its changes, covering squash- and rebase-merged branches on forges; the output names where the work landed, and partly-landed branches are kept. D. Ben Knoble reviewed the approach, noting the check can be expensive, and floated a `--delete-merged=squashed,rebased` option value; Nordgren said performance would be the only reason to hide the behavior.

## The day in brief
D. Ben Knoble's v4 stash series moved the "stash apply --index" index merge in-core, with one NULL-dereference bug still flagged. Aleksei Sviridkin's v4 `--force-if-includes` fix was marked for `next`, and Harald Nordgren's v4 shallow-repo fetch series was accepted for replacement. Patrick Steinhardt fixed the reftable reflog timezone format, and Grant Moyer's filter-branch commit-map fix gained a second confirmation.

## Notable threads

### refs/reftable: store reflog timezones as signed minute offsets
Patrick Steinhardt posted a three-patch series fixing `refs/reftable-backend.c`, which stored reflog timezones in "[+-]HHMM" format instead of the spec's signed minute offset. It adds `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c`, fixes a `dump-reftable -t` segfault, and adds a t0610 test; fallout is limited since timezones are display-only.

### filter-branch: commit map init fix tested

Michele Locati confirmed the state-branch commit map bug, traced it to f6d855091e in 2.50.0, and tested Grant Moyer's August fix on git 2.56.0 with --state-branch, --prune-empty and --subdirectory-filter: without it, new commits attach to original parents; with it, parents and state save correctly. He added a Tested-by tag.

### docs: remove gittutorial-2

Julia Evans replied that she is not replacing `gittutorial-2` since `gitdatamodel` and `gitcore-tutorial` cover its material. Junio C Hamano asked that `po/` translations be left intact, and later reported doc-lint fails with "no link: gittutorial-2" once merged.

## In brief
- Jeff King's "use size_t for xdiff mmfile_t" series consolidates `mmbuffer_t` into `mmfile_t`; follow-ups make `ll_ext_merge()` return `LL_MERGE_ERROR` when reading an external merge result fails, fixing a rerere path that could record a bogus empty resolution. *Long thread: 8 replies so far from 3 participants over 1 day; 9 messages that day.*
- Jeff King's v3 "merge-ll: use tempfile API for external driver files" cleans up `.merge_file_*` temporaries on Ctrl+C; Junio C Hamano said he would mark it for `next`.
- Patrick Steinhardt agreed to proceed with Jeff King's http fix stripping credentials from the requested URL before comparing with `CURLINFO_EFFECTIVE_URL`, needed for curl 8.23.0.
- Pushkar Singh's v3 swaps back the default reflog expiry periods in `reflog.h` (reachable 90 days, unreachable 30); Patrick Steinhardt suggested testing at 88 days, which Junio C Hamano accepted.
- Scott Chacon posted a four-patch series porting the sha1dc Rust crate's approach to C, claiming ~2.7x faster SHA-1 collision detection on x86 and ARMv8.
- Kaartic Sivaraam posted RFC v2 making the "not a git repository" error state its reason for explicit `--git-dir`, via `is_git_directory_verbose()` in `setup.c`.
- Kazumasa Shigeta's patch exposes `-u`/`-a` in `git stash create`; Phillip Wood objected to the changed no-changes exit status and recommended `PARSE_OPT_STOP_AT_NON_OPTION`.
- Thomas Bachem's v2 t5520 patch pins `gc.reflogExpire` to `never`; Junio C Hamano combined it with the stash series on a synthetic base in `seen` for D. Ben Knoble to reuse.
