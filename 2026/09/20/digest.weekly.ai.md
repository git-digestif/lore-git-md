## Long threads

### receive-report hook in `builtin/receive-pack.c`

*Long thread: 58 replies so far from 7 participants over 29 days.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client, receiving the pkt-line encoded report on stdin and allowing its stdout to replace the report while a non-zero exit marks all references rejected. The thread saw steady progress across the week, with Junio C Hamano queueing a coccinelle fix on top of the v10 series to drop a redundant nullness check before `FREE_AND_NULL`, which Nayak confirmed looked good. The series remains in flight with the hook mechanics settled and cleanup patches applied.

### sequencer auto maintenance timing

*Long thread: 30 replies so far from 5 participants over 14 days.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, passing `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS` to prevent background maintenance from racing the sequence for locks. The week saw v5 arrive with wording updates only, adopting Patrick Steinhardt's documentation suggestions and Phillip Wood's commit message rewording, while the placement of the maintenance call remained under discussion between calling it at the end of `pick_commits()` versus in the two builtins.

### unifying ref storage format terminology

*Long thread: 29 replies so far from 4 participants over 12 days.*

Patrick Steinhardt's 13-patch v3 series renames inconsistent ref storage format interfaces to consistent `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases and accepting `format://payload` URIs. Reviewers debated whether `OPT_ALIAS()` entries should be hidden, with Junio C Hamano explaining that `OPT_HIDDEN` belongs to a later deprecation stage and that if `OPT_ALIAS()` cannot take it, that is an infrastructure bug. The series was found to be in very good shape by the end of the week.

### advice squelch hints with `--global`

*Long thread: 25 replies so far from 4 participants over 9 days.*

Vsevolod Myalitsin's series addresses the `defaultBranchName` advice by suggesting the global config scope, since the local-scope command only affects the repository where the advice was already shown. The discussion converged on a boolean global hint rather than a full `enum config_scope`, with Jeff King agreeing that passing a pointer to the `advice_setting` to `vadvise()` should go as a separate patch, and Junio C Hamano posting a simpler v5 that just adds `--global` to the squelch message for all advice.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 22 replies so far from 4 participants over 37 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so Git can check stapled OCSP responses, defaulting to false to avoid blocking connections to servers that send no stapled response. The week saw v7 posted as the combined version with tests split between `t5551` for cases needing no OCSP setup and the new `t5585` for responder-dependent tests, with Junio C Hamano approving the test updates and saying he would replace the existing tests with them.

## The period in brief

The week of 2026/09/14 through 2026/09/20 saw heavy traffic across seven active days, with five long-running threads continuing alongside several new bug fixes and feature series. Readers should not miss the repack race fix that prevents data loss when `.keep` files appear mid-run, the resolution of the `--force-if-includes` fast-forward rejection bug, and Maciej Ciemborowicz's work restoring old OIDs to batched ref deletions and routing copy/rename through transactions.

## Key developments

### repack race with mid-run `.keep` files

A production-hit race where `git repack -d` could delete a pack whose objects were never copied -- because a concurrent push linked a `.keep` file between repack's initial scan and pack-objects' rescan -- drew sustained attention across four days. Qin ShiCheng posted a six-patch series making repack hand pack-objects its startup snapshot of kept packs via a new `--keep-pack-from-file=<file>` option, while qeesung posted v2 dropping the receive-pack side in favor of Justin Tobler's plan to have the ODB transaction create `.keep` files at commit time. The core fix ensures both processes work from one snapshot, with supporting patches handling the cruft walk's kept-pack cache and binary lookup.

### `--force-if-includes` push semantics

Tyler Cipriani's series making `--force-if-includes` reject non-branch sources and fix fast-forward rejections progressed through v4, v5 and v6 during the week. The final v6 defers reflog verification until after fast-forward rules are evaluated, so valid fast-forward pushes with expired reflogs proceed instead of being rejected, while still rejecting tag pushes and adding a `pushRefUnverifiable` advice message suggesting `--force-with-lease=<ref>:<expect>`. D. Ben Knoble flagged that the earlier behavior broke his workflow of pushing `<TAG>:hotfix`, which Cipriani confirmed and addressed in the later iterations.

### stash autostash with `stash.index=true`

The bug where `stash.index=true` left a redundant stash entry after a successfully applied merge autostash -- because `git stash apply --index` reached a reset mode that discarded branch state -- was diagnosed and fixed during the week. D. Ben Knoble posted a two-patch series replacing the subprocess-based index merge with an in-core `merge_incore_nonrecursive()` call, fixing the lock failure on deleted `MERGE_AUTOSTASH` refs, with Phillip Wood reviewing the WIP approach as looking good and suggesting minor refinements.

### refs: old OIDs for batched deletions and transactional copy/rename

Maciej Ciemborowicz posted a v2 series teaching `refs_delete_refs()` to accept an optional `oid_array` of old OIDs, restoring compare-and-delete race protection for branch, tag, fetch and remote prune operations that was lost when commit `8198907795` converted them to batched deletion. A companion patch routes reference copy and rename through the transaction API so the `reference-transaction` hook observes them, representing a rename as a deletion plus creation in one transaction with backend-specific reflog handling and rollback on hook rejection.

### incremental connectivity check via rev-list

Kristofer Karlsson posted a two-patch series adding an opt-in incremental connectivity check gated behind `transfer.connectivityCheck=incremental`, running inside the rev-list subprocess with benchmarks reporting up to ~190x speedup for small pushes. Junio C Hamano called the benchmarks exciting but raised concerns about a possibly too-late `is_promisor_object()` call and a possible NULL from `get_commit_tree_oid()`, which Karlsson confirmed and promised to address with a blob test and a NULL check.

### commit-reach: parse commits in the given repository

Orestis Floros fixed `can_all_from_reach()` and `can_all_from_reach_with_flag()` in `commit-reach.c` to parse commits in the caller's repository rather than `the_repository`, fixing a "fatal: invalid commit position" crash when merging superprojects whose submodule pointers advanced on both sides with commit-graphs present. Junio C Hamano called the change reasonable and a welcome step for libification, saying he would queue it, while Kristofer Karlsson noted it reduces `the_repository` references in the file from 17 to 11.

### dir: pathspec prefixes with exclusions

Yannik Tausch's v4 series fixing the pathspec prefix optimization in `dir.c` was marked ready for `next` by Junio C Hamano. The series derives the common prefix from non-exclude items and matches exclude items against full pathnames, fixing an incorrect match and an out-of-bounds read when an exclude item appeared first in the pathspec.

### upload-pack: shallow-info ordering

Royce Remer posted v2 swapping the order of `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow fetches with `uploadpack.allowRefInWant=true` work, fixing a "fatal: expected 'packfile', received 'shallow-info'" error. Junio C Hamano agreed the explanation was good commit-log material and noted the bug had existed since the original ref-in-want implementation.

### RFC: a "safe" strbuf API

Derrick Stolee posted an RFC proposing a `strbuf-safe` API whose methods cannot transitively call `die()`, motivated by recursive die loops in trace2 under allocation failure. Phillip Wood objected to the name "safe" and identified that `sstrbuf_grow()` calls `st_add3()`, which dies on overflow, suggesting a sticky error bit instead.

### reflog expire default expiry times swapped

r.norouzi reported that `git reflog expire` prunes reachable reflog entries older than 30 days instead of the documented 90, because the default expiry values were swapped in commit `85658275702b` first released in Git 2.50.0, reversing the assignments in `builtin/reflog.c`.

## In brief

- **Git v2.56.0-rc1** -- Junio C Hamano announced the release candidate comprising 721 non-merge commits since v2.55.0 from 90 people, 37 of them new.
- **Git for Windows 2.56.0-rc0** -- Johannes Schindelin announced the Windows release dropping Windows 8.1 support and bundling Git v2.56.0-rc0, Git LFS v3.8.0, cURL v8.22.0 and OpenSSL v3.5.8.
- **`git var` v8** -- Andrew Pleeter posted a version printing `VARIABLE=value` pairs for multi-variable queries, omitting unset variables while still exiting 1.
- **rerere lock timeout v4** -- Thomas Bachem posted a series making `setup_rerere()` wait up to `rerere.lockTimeout` for `MERGE_RR.lock`.
- **packed-refs transaction skip** -- Junio C Hamano said he would queue Ariel Keselman's v3 patch skipping the packed-refs transaction for root-ref deletions in the files backend.
- **`git status --ignored` bug** -- René Scharfe bisected a partial-pathspec match bug to commit 95c11ecc73 and proposed removing an optimization check in `treat_directory()`.
- **commit-buffer leak** -- Kaartic Sivaraam posted v4 fixing a leak in `builtin/history.c` by pairing `repo_logmsg_reencode()` with `repo_unuse_commit_buffer()`.
- **packfile flush during ODB transaction** -- Justin Tobler's two-patch series fixes flushing with `core.fsyncMethod=batch`, reviewed positively by Karthik Nayak.
- **`git pull` NULL guard** -- Jiri Kuncar posted a patch keeping `git pull` from crashing on an unresolvable merge head, which Junio C Hamano called a good improvement.
- **precompiled headers** -- SZEDER Gábor posted v2 of a series precompiling `git-compat-util.h`, reporting about 35% faster builds.
- **Coverity fixes** -- Johannes Schindelin posted a seven-patch series fixing Coverity issues from Git for Windows, including `writev_in_full()` overflow and rerere failure handling.
- **`git rerere remaining` fix** -- Junio C Hamano confirmed Mikko Rantalainen's diagnosis that consecutive conflicted paths with only stage-1 entries were skipped, and posted a patch adding a same-path check.
- **`diff --no-index -R`** -- Haokai Ding fixed file/directory conflicts where the early queue entry bypassed reverse handling, collecting reviews from René Scharfe and Jacob Keller.
- **Rust `gitcore` crate move** -- Mike Hommey posted v5 moving the crate into a `rust/` subdirectory to allow use as a submodule, answering review questions about `--manifest-path` and meson `subdir()`.
- **RUNTIME_PREFIX on OpenBSD** -- Brad Smith submitted a patch enabling it via `getexecpath()`, prompting questions from Junio C Hamano about version applicability.
- **Outreachy project** -- Christian Couder and Kaartic Sivaraam proposed "Implement promisor remote fetch ordering" for the December 2026 cohort.
- **GitLab CI cargo fixes** -- Johannes Schindelin posted a four-patch series fixing the cargo invocation in Windows CI jobs, with the `build:mingw64` job succeeding.
- **`fetch.shallow` config** -- Harald Nordgren submitted a patch adding an option making refspec-less fetches in shallow repositories fetch only the current branch's upstream.
- **`shallowHistory` advice** -- Harald Nordgren posted a patch adding advice when `<ref>~N` resolution stops at a shallow boundary, suggesting `git fetch --deepen=<n>`.
- **worktree lifecycle hooks RFC** -- Maciej Ciemborowicz proposed native hooks for worktree add, remove, move, lock, unlock and prune events.
- **`LESS=FRX` documentation** -- brian m. carlsson explained Git sets `LESS=FRX` only when `LESS` is unset, with Todd Zullinger suggesting documenting `LESS=""` overrides.
- **test modernization** -- Mark C. Chu-Carroll posted v4 of a three-patch series modernizing `t40*` tests with setup blocks and consistent capitalization.

## Looking ahead

Maciej Ciemborowicz's patch routing reference copy and rename through the transaction API, posted on 2026/09/20, will likely see review next week alongside his v2 series on old OIDs for batched deletions. Harald Nordgren's `shallowHistory` advice and his RFC for worktree lifecycle hooks, both posted late in the week, are expected to draw discussion. Derrick Stolee's RFC for a "safe" strbuf API, posted on 2026/09/18, awaits broader feedback on whether the direction is worthwhile given Phillip Wood's objection to the naming and the `st_add3()` overflow issue. Kristofer Karlsson's incremental connectivity check series needs follow-up patches addressing Junio C Hamano's concerns about the promisor object check and NULL handling before it can advance.