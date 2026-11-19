# Git mailing list daily digest for 2026/10/15

## Long threads

### CI housekeeping and modernization series reaches v4

*Long thread: 38 replies so far from 6 participants over 8 days; 8 messages that day.*

Patrick Steinhardt posted [PATCH v4 0/8] "ci: some housekeeping and modernizations", an eight-patch series fixing smaller CI issues and moving jobs off EOL Docker images. The only change from v3 is in patch 1, which reworks the t5004 archive test to work under both SHA-1 and SHA-256 by forcing the test repo to SHA-1. Jeff King said the updated patch looks good and that v4 looks good to him, and Junio C Hamano marked the series for `next`. Earlier, the thread had settled the "linux-feature-testbed" job name and the reworked reftable-plus-SHA-256 coverage job.

### credentialStore.lockTimeout review feedback

*Long thread: 37 replies so far from 5 participants over 44 days; 1 message that day.*

Junio C Hamano reviewed Okhuomon Ajayi's Outreachy patch adding `credentialStore.lockTimeout` as the preferred setting for credential lock retries, with `lockTimeoutMS` as a deprecated alias. He noted the modified section already has overly long lines, suggested a for loop over both variable names to keep duplicated tests in sync, and questioned whether the "lockTimeout allows a store" test would fail with a 0 ms timeout.

### Incremental connectivity check review

*Long thread: 26 replies so far from 3 participants over 32 days; 9 messages that day.*

Patrick Steinhardt reviewed Kristofer Karlsson's [PATCH v3 0/4] incremental connectivity check. He asked for Makefile and Meson build entries for the new connectivity-check.adoc, suggested restructuring the config patch to accept only "full" first, and raised implementation concerns: die() calls in library code, int counters that should be size_t, and recursion up to 2048 layers deep. Karlsson agreed to most fixes, argued the larger-tree speedup is unbounded while the commit-count regression is bounded near 1.5x, and floated a configurable threshold such as transfer.connectivityCheckIncrementalMaxCommits; he also noted no single indicator decides when to fall back to the full check.

### Blame reads .git-blame-ignore-revs by default

*Long thread: 24 replies so far from 4 participants over 35 days; 1 message that day.*

Ravi Mistry's [PATCH v6 0/2] teaches `git blame` and `git annotate` to use `.git-blame-ignore-revs` at the working-tree root (or `HEAD:.git-blame-ignore-revs` in bare repositories) by default. Phillip Wood said he was not sure the `lstat()` before `open_nofollow()` was needed, but everything else looked good and he would be happy to see that version included. Earlier revisions had added a 100MB blob size cap and switched non-bare repositories to the working-tree file.

## The day in brief
Patrick Steinhardt's CI housekeeping series reached v4 and was marked for `next`, while Ravi Mistry's blame default-ignore-revs series earned a positive review. New patches also addressed `git log --follow` performance on non-linear history, tag backfill during shallow fetches, and a rebase failure when the index is locked.

## Notable threads

### Speeding up git log --follow on non-linear history
Miklos Vajna submitted a patch adding two shortcuts in `propagate_follow_pathspec_to_parents()`: skip expensive rename detection unless the path was added relative to a parent, and follow a parent's history directly when the path is unchanged. The linux.git `rsc_table.h` case drops from 6m43s to 29s. Junio C Hamano noted the followed-path clue is lost after `log_tree_diff()` flushes its output and suggested inspecting the diff queue in `log_tree_diff_flush()` instead, or rethinking the approach from commit 304812ed33.

### Commit fetched refs before backfilling tags
Karthik Nayak posted [PATCH v3] "fetch: commit references fetched before backfilling tags", fixing shallow fetches where newly fetched refs were unavailable as 'haves' during tag backfill because batched updates were not yet committed. The patch commits the fetched refs first, uses a separate `commit_ref_ret` error variable, and adds tests for 'haves' advertisement and upstream configuration when the main ref update is rejected.

### cache-tree no longer dies when the index is locked
Szymon Wilczek submitted a patch dropping `LOCK_DIE_ON_ERROR` from `write_index_as_tree()` in cache-tree.c and skipping the index write-back when `index.lock` is held, so `git write-tree` and `git am` proceed while a background `git status` holds the lock. This fixes a rebase failure where the sequencer died before writing `rebase-merge/message`.

## In brief
- Jeff King reviewed Igor Wiedler's fix negating `--alternate-refs` in deepening-fetch connectivity checks and the companion change making `odb_for_each_alternate_ref()` ignore refs of shallow and grafted alternates, calling the approach exactly what he would have expected.
- Julia Evans posted v3 of a rewritten detached HEAD guide (`git help detached-head`); Junio C Hamano suggested stronger wording about why detached HEAD is useful and other phrasing changes.
- Phillip Wood reviewed Johannes Schindelin's Coverity overflow fixes, proposing `intmax_t` multiplication for the `approxidate` fix so it works on all platforms, not just GCC and Clang.
- Junio C Hamano questioned whether a fetch `--depth` fix for multi-remote fetches affects submodule fetches and proposed separate conditionals for the shallow options.
- Patrick Steinhardt posted v3 of the odb/files alternates cleanups; Justin Tobler reviewed it and said it looks ready.
- Harald Nordgren posted a series adding `--operations` and `--historical-branch` to `git reflog`; Kristoffer Haugsbakk critiqued a commit message and suggested `tig reflog` as a workflow alternative.
- Venkat Ragavan fixed loose-ref seeking across directory boundaries in `cache_ref_iterator_seek()`; Junio C Hamano explained the test loop covers loose, packed, and mixed ref storage.
