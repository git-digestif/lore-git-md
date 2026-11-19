# Git mailing list daily digest for 2026/10/15

## Long threads

### CI housekeeping and modernization series reaches v4

*Long thread (root 2026/10/08/10-01-18): 38 replies so far from 6 participants over 8 days; 8 messages that day.*

Patrick Steinhardt's eight-patch series "ci: some housekeeping and modernizations" fixes smaller issues in Git's CI configuration and moves jobs off end-of-life Docker images to supported ones. Today he posted v4, whose only change reworks the t5004 archive test to work under both SHA-1 and SHA-256 by forcing the test repository to SHA-1; patches 2-8 are unchanged from v3. Jeff King said the updated patch looks good and that v4 looks good to him, and Junio C Hamano marked the series for `next`. Earlier, the thread had settled the rename of the `linux-TEST-vars` job to `linux-feature-testbed`, reworked the reftable job to exercise reftables with SHA-256, and agreed to defer CI script cleanup to a follow-up.

### credential-store lockTimeout config option

*Long thread (root 2026/09/02/08-31-37): 37 replies so far from 5 participants over 44 days; 1 message that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries in `builtin/credential-store.c`, retaining `lockTimeoutMS` as a deprecated alias, with tests in `t/t0302-credential-store.sh`. Today Junio C Hamano reviewed it: he noted the modified section of `builtin/credential-store.c` already has overly long lines, suggested replacing the duplicated lockTimeout/lockTimeoutMS tests with a for loop over both variable names, clarified that the "wins when both are set" test verifies the error message reports "in 0 ms", and questioned whether the "allows a store" test adds anything since any timeout succeeds without a competing lock.

### Incremental connectivity check for fetches

*Long thread (root 2026/09/14/09-47-54): 26 replies so far from 3 participants over 32 days; 9 messages that day.*

Kristofer Karlsson's four-patch series adds an incremental mode for the connectivity check in `connected.c`, gated behind `transfer.connectivityCheck=incremental`, so verification scales with the number of incoming changes rather than preexisting objects. Today Patrick Steinhardt reviewed the documentation, perf-test, and implementation patches, asking for Makefile/Meson build entries for the new `connectivity-check.adoc`, a PERL prerequisite and p5412-to-p5413 fixes in the benchmarks, `size_t`/`intmax_t` counters, `bool` for `need_subtree_bases`, comments on trust-boundary expansion, and a possible work queue instead of recursion. Kristofer Karlsson replied agreeing to most fixes, argued the larger-tree win is unbounded while the commit-count regression is bounded, and floated a `transfer.connectivityCheckIncrementalMaxCommits` config flag; Patrick Steinhardt suggested making the commit threshold configurable with roughly 1k commits as a cutoff.

### Blame ignores .git-blame-ignore-revs by default

*Long thread (root 2026/09/11/23-29-44): 24 replies so far from 4 participants over 35 days; 1 message that day.*

Ravi Mistry's series teaches `git blame` and `git annotate` to use `.git-blame-ignore-revs` at the working-tree root (or `HEAD:.git-blame-ignore-revs` in bare repositories) by default, with a 100MB size cap. Earlier revisions switched from the HEAD blob to the working-tree file in non-bare repositories and cleaned up the `build_ignorelist()` handling. Today Phillip Wood said he was not sure the `lstat()` call before `open_nofollow()` was needed but that everything else looked good, and he would be happy to see that version merged.

## The day in brief
Junio C Hamano marked Patrick Steinhardt's v4 CI housekeeping series for `next`, and Phillip Wood signaled approval of Ravi Mistry's blame default-ignore-revs series. Kristofer Karlsson's incremental connectivity check drew detailed review with a proposed configurable commit threshold, while Karthik Nayak posted v3 of the fetch fix that commits references before backfilling tags.

## Notable threads

### cache-tree no longer dies when the index is locked
Szymon Wilczek submitted a patch changing `write_index_as_tree()` in `cache-tree.c` to take the index lock without `LOCK_DIE_ON_ERROR` and skip `write_locked_index()` when the lock is held, so `git write-tree` and `git am` proceed while another process holds `index.lock`. This fixes a rebase failure where a background `git status` holding the lock caused the sequencer to die before writing `rebase-merge/message`; a test in `t/t0090-cache-tree.sh` verifies `git write-tree` succeeds while the index is locked.

### Fetch commits references before backfilling tags
Karthik Nayak posted v3 of "fetch: commit references fetched before backfilling tags", which commits the fetched refs before backfilling tags using a separate `commit_ref_ret` error variable, so a failed backfill transaction no longer skips FETCH_HEAD writing and upstream configuration. The patch addresses shallow fetches where newly fetched refs were unavailable as 'haves' during tag negotiation, and adds tests verifying backfill advertises fetched refs and that upstream is set even when the main ref update is rejected.

### Speeding up log --follow on non-linear history
Miklos Vajna submitted a patch adding two shortcuts in `propagate_follow_pathspec_to_parents()`: only run expensive rename detection when the path is added relative to a parent, and follow the parent's history directly when the path is unchanged. On linux.git the `rsc_table.h` case drops from 6m43s to 29s while `t4219-log-follow-merge.sh` still passes. Junio C Hamano observed that `log_tree_commit()` calls the propagation only after `log_tree_diff()` has flushed its output, suggested inspecting the diff queue in `log_tree_diff_flush()` instead, and said it might be cleaner to rethink the approach from commit 304812ed33, though he was not sure.

### Fixing --depth and --unshallow for multi-remote fetches
A patch fixes `git fetch --all --depth=...` and `--unshallow` not actually deepening the fetch with multiple remotes, adding tests to the shallow-fetch test script. Today Junio C Hamano questioned whether the same lack of `--depth` propagation affects submodule fetches under `--recurse-submodules`, asked whether `depth` is NULL when only `--deepen-since` is given, proposed rewriting the patch as three separate conditionals for `depth`, `deepen_since`, and `deepen_not`, and warned that propagating `--shallow-exclude` to submodules is likely inappropriate.

### Ignoring refs of shallow and grafted alternates
Igor Wiedler's two-patch series fixes the connectivity check with alternates in deepening fetches: it negates `--alternate-refs` alongside `--all`, and makes `odb_for_each_alternate_ref()` ignore refs of shallow and grafted alternates so neither the connectivity check nor fetch/receive-pack treat history the alternate lacks as present. Today Jeff King reviewed the series, calling the `--alternate-refs` placement exactly the expected fix, noting replace refs alongside grafts, and agreeing that punting on partial-clone corner cases by ignoring such alternates' refs is fine since alternate-ref use is an optimization.

## In brief
- Andy Koppe's patch keeps root refs like HEAD out of the shared loose ref cache in the files backend, fixing `git for-each-ref --include-root-refs --merged=main` omitting root refs; he defended not caching root refs as simpler since the flag is used only twice.
- Venkat Ragavan fixed loose-ref seeking across directory boundaries in `cache_ref_iterator_seek()` by including the trailing slash when comparing directory components, with regression tests over loose, packed, and mixed storage.
- Harald Nordgren posted a three-patch series adding `--operations` and `--historical-branch` to `git reflog` and `git log -g`, folding rebases into single entries and decorating entries with the branch HEAD was on.
- Mark C. Chu-Carroll posted v7 of "repo: add revision filtering options to 'repo structure'", fixing a duplicated word in the commit message after Kristoffer Haugsbakk pointed it out.
- Patrick Steinhardt posted v3 of "odb/files: cleanups and improvements for alternates" with a regression test from Tyrie Vella; Justin Tobler reviewed it and said it looks ready.
- Isabella Caselli posted v2 adding a `hostname:` condition to includeIf and `GIT_HOSTNAME` to `git var`; Junio C Hamano said the implementation looks sensible.
- Johannes Schindelin's Coverity series adds `signed_mult_overflows`/`mult_overflows` macros and fixes an approxidate overflow; Phillip Wood noted the macros are GCC-only and proposed `intmax_t` arithmetic instead.
- A patch fixes the sequencer clobbering the HEAD ref name in worktree lookup by using `refs_resolve_refdup()`; Phillip Wood praised the explanation and the new test.
