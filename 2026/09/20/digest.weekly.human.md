## Long threads

### receive-report hook in `builtin/receive-pack.c`

*Long thread: 58 replies so far from 7 participants over 29 days.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client, receiving the pkt-line encoded report on stdin and allowing its stdout to replace the report, with a non-zero exit marking all references rejected. The thread progressed through v10 with fixes for a BUG() when no report was requested and a leak in `override_cmds_error()`, and Junio C Hamano queued a coccinelle fix on top removing a redundant nullness check before `FREE_AND_NULL`. By the end of the week Nayak confirmed the coccinelle fix looked good, leaving the series in a stable state awaiting further review or integration.

### sequencer auto maintenance timing

*Long thread: 30 replies so far from 5 participants over 14 days.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, passing `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS` to prevent background maintenance from racing the sequence for `MERGE_RR.lock`. The thread saw v5 arrive with wording updates only, adopting Patrick Steinhardt's suggestions for the commit message and documentation while retaining a sentence on NULL values in `git_config_append_parameter()`. Bachem and Steinhardt debated whether to call auto maintenance at the end of `pick_commits()` plus single-pick return paths or in the two builtins, with Bachem offering to reroll according to Steinhardt's preference.

### unifying ref storage format terminology

*Long thread: 29 replies so far from 4 participants over 12 days.*

Patrick Steinhardt's 13-patch v3 series renames inconsistent ref storage format interfaces -- `--ref-format=`, `GIT_REFERENCE_BACKEND`, `init.defaultRefFormat` -- to consistent `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases and accepting `format://payload` URIs. Reviewers debated whether `OPT_ALIAS()` entries should be hidden, with Kaartic Sivaraam noting existing deprecated aliases remain visible and Junio C Hamano explaining that `OPT_HIDDEN` belongs to a later deprecation stage, suggesting that if `OPT_ALIAS()` cannot take `OPT_HIDDEN` it is an infrastructure bug. Hamano found the series otherwise in very good shape.

### advice squelch hints with `--global`

*Long thread: 25 replies so far from 4 participants over 9 days.*

Vsevolod Myalitsin's series addresses the `defaultBranchName` advice, whose suggested `git config set advice.defaultBranchName false` writes to local scope and thus only affects the repository where the advice was shown. Jeff King suggested simply recommending `--global` in all advice squelch messages, which Junio C Hamano agreed with, and Hamano posted a v5 patch adding `--global` to the `git config set` command in the advice message across `advice.c` and 11 test files. The thread converged on this simpler approach rather than a full `enum config_scope` mechanism, with Myalitsin confirming the topic reached its intended conclusion.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 22 replies so far from 4 participants over 37 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so Git can check stapled OCSP responses, defaulting to false because enabling it blocks connections to servers that send no stapled response. The thread reached v7 as the combined version discussed with Junio C Hamano and Patrick Steinhardt, splitting tests between `t5551` for cases needing no OCSP setup and a new `t5585` for responder-dependent tests, with `LIB_HTTPD_OCSP` support moved into lib-httpd. Hamano approved the updated tests and said he would replace the existing tests with them.

## The period in brief

The week of 2026/09/14 through 2026/09/20 saw heavy, eventful traffic across all seven days, anchored by the announcement of Git v2.56.0-rc1 and Git for Windows 2.56.0-rc0. Readers should not miss the evolving fix for the `git repack -d` race that can delete packs whose objects were never copied, Tyler Cipriani's three iterations of `--force-if-includes` push safety improvements, and Maciej Ciemborowicz's work restoring old OIDs to batched ref deletions.

## Key developments

### repack race with mid-run `.keep` files

Qin ShiCheng (posting as qeesung) drove a series fixing a production-hit race where `git repack -d` deletes a pack whose objects were never copied because a concurrent push links a `.keep` file onto an existing pack between repack's initial scan and pack-objects' rescan. The series evolved from six patches to five as Justin Tobler agreed to fix the receive-pack side by having the ODB transaction create the `.keep` itself at commit time. The core change has repack write its startup snapshot of kept packs to a tempfile and pass it to pack-objects via a new `--keep-pack-from-file=<file>` option instead of `--honor-pack-keep`, ensuring both processes work from one snapshot, with supporting patches handling the cruft walk's kept-pack cache and binary lookup.

### `--force-if-includes` push safety

Tyler Cipriani posted three iterations (v4, v5, v6) of a series making `--force-if-includes` reject pushes whose source ref is not a branch, since the check previously consulted the same-named local branch's reflog rather than the pushed tag. The series progressed from unconditionally resolving the peer ref to deferring reflog verification until after fast-forward rules are evaluated, fixing a bug where valid fast-forward pushes were rejected when the local reflog was expired. Patch 2/3 adds a `pushRefUnverifiable` advice message suggesting `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`, and Cipriani noted D. Ben Knoble's workflow of pushing `<TAG>:hotfix` would break, which Cipriani confirmed was intended behavior.

### refs: old OIDs for batched deletions and transaction routing

Maciej Ciemborowicz posted a v2 series teaching `refs_delete_refs()` to accept an optional `oid_array` of old OIDs, restoring compare-and-delete race protection lost when commit `8198907795` converted single-ref deletions to batched calls before Git 2.31. The series makes branch, tag, fetch and remote prune operations conditional on the ref not having changed since the caller resolved it, with tests verifying concurrent updates are preserved. A separate patch routes reference copy and rename through the transaction API so the `reference-transaction` hook observes them, representing a rename as deletion plus creation in one transaction and handling D/F conflicts and hook rollback.

### stash autostash fix with in-core index merge

D. Ben Knoble posted a two-patch series fixing a bug reported by Eli Barzilay where `stash.index=true` causes a successfully applied merge autostash to remain as a redundant stash entry and deleting `MERGE_AUTOSTASH` to fail. The fix replaces the subprocess-based index merge in `git stash apply --index` with an in-core `merge_incore_nonrecursive()` call, implementing Phillip Wood's suggested approach and passing tests including a new fast-forward merge case with `--autostash` and `stash.index`.

### Git v2.56.0-rc1 and Git for Windows 2.56.0-rc0

Junio C Hamano announced Git v2.56.0-rc1, comprising 721 non-merge commits since v2.55.0 from 90 people, 37 of them new. Johannes Schindelin announced Git for Windows 2.56.0-rc0, which drops Windows 8.1 support by moving `git.exe` from `/mingw64/bin` to `/ucrt64/bin` and bundles Git v2.56.0-rc0, Git LFS v3.8.0, cURL v8.22.0 and OpenSSL v3.5.8.

### RFC: a "safe" strbuf API

Derrick Stolee posted an RFC proposing a `strbuf-safe` API whose methods cannot transitively call `die()` or `exit()`, motivated by recursive die loops in trace2 under allocation failure when json-writer uses strbuf. The series adds `sstrbuf_grow()`, `sstrbuf_init()` and `sstrbuf_release()` under an `s` prefix, plus `safe_memory_limit_check()` returning error codes without dying, and adapts `json-writer.c` to use the safe variants. Phillip Wood objected to the name "safe" and identified that `sstrbuf_grow()` calls `st_add3()` which dies on overflow, undermining the verification goal.

### incremental connectivity check via rev-list

Kristofer Karlsson posted a two-patch series adding an opt-in incremental connectivity check gated behind `transfer.connectivityCheck=incremental`, running inside the rev-list subprocess via a new internal `--verify-trees-incremental` flag with benchmarks reporting up to ~190x speedup and lower RSS for small pushes. Junio C Hamano called the benchmarks "exciting" but raised concerns about a possibly too-late `is_promisor_object()` call and a possible NULL from `get_commit_tree_oid()`, which Karlsson confirmed and promised to address with a blob test and NULL check.

### commit-reach: parse commits in the given repository

Orestis Floros posted a patch fixing `can_all_from_reach()` and `can_all_from_reach_with_flag()` in `commit-reach.c` to parse commits in the caller's repository rather than `the_repository`, fixing a fatal "invalid commit position" error when merging superprojects with submodules when commit-graphs exist in both repositories. Kristoffer Karlsson confirmed the regression test reproduces the bug and noted the change reduces `the_repository` references in the file from 17 to 11. Hamano called the change reasonable and said he would queue it.

## In brief

- **dir pathspec prefixes**: Yannik Tausch's v4 series fixes pathspec prefix handling with leading exclude items by matching exclude items against full paths and deriving the common prefix from the first non-exclude item; Hamano marked it ready for `next`.
- **upload-pack shallow fetch ordering**: Royce Remer's v2 swaps `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow fetches with `uploadpack.allowRefInWant=true` work, fixing a bug present since the original ref-in-want implementation.
- **rerere remaining skips paths**: Junio C Hamano confirmed Mikko Rantalainen's diagnosis that `check_one_conflict()` skips consecutive conflicted paths with only stage-1 entries and posted a fix adding a same-path check to the while-loop condition.
- **reflog expire defaults swapped**: r.norouzi reported that `git reflog expire` prunes reachable entries older than 30 days instead of 90 because the defaults were swapped in commit `85658275702b` first released in 2.50.0.
- **diff --no-index -R**: Haokai Ding fixed `diff --no-index -R` for file/directory conflicts by swapping filespecs of the early queue entry when `reverse_diff` is set, collecting reviews from René Scharfe and Jacob Keller.
- **Coverity fixes**: Johannes Schindelin posted a seven-patch series fixing Coverity issues from v2.56.0-rc0 including `writev_in_full()` overflow and rerere failure handling; Hamano approved the rerere patch.
- **git var v8**: Andrew Pleeter posted v8 printing `VARIABLE=value` pairs for multi-variable queries and omitting unset variables while exiting 1.
- **Rust gitcore crate**: Mike Hommey posted v5 moving the crate into a `rust/` subdirectory to allow use as a submodule, with Hamano questioning `--manifest-path` usage.
- **GitLab CI cargo fix**: Johannes Schindelin posted a four-patch series fixing the cargo invocation in Windows CI jobs broken since Rust was enabled there.
- **Outreachy project**: Kaartic Sivaraam and Christian Couder proposed "Implement promisor remote fetch ordering" for the December 2026 cohort.
- **Test modernization**: Mark C. Chu-Carroll posted v4 of the "t40*: modernize" series and accepted review suggestions.
- **fetch.shallow config**: Harald Nordgren submitted a patch adding a `fetch.shallow` option for shallow repositories.
- **shallowHistory advice**: Harald Nordgren posted a patch adding advice when `<ref>~N` resolution stops at a shallow boundary.
- **worktree lifecycle hooks**: Maciej Ciemborowicz posted an RFC asking whether native hooks for worktree add/remove/move events should be a single hook or separate post-operation hooks.
- **OpenBSD RUNTIME_PREFIX**: Brad Smith submitted a patch enabling `RUNTIME_PREFIX` via `getexecpath()`, with Hamano asking about version applicability.
- **winansi die_lasterr**: Yongqiang Tian fixed `die_lasterr()` in `compat/winansi.c` passing its `va_list` incorrectly to `die_errno()`.
- **precompiled headers**: SZEDER Gábor posted v2 reporting about 35% faster builds from precompiling `git-compat-util.h`.
- **odb transaction fsync**: Justin Tobler's two-patch series fixes packfile flush during ODB transaction commit with `core.fsyncMethod=batch`.
- **rerere lock timeout**: Thomas Bachem posted v4 making `setup_rerere()` wait up to `rerere.lockTimeout` for `MERGE_RR.lock`.
- **status --ignored bisect**: René Scharfe bisected a partial-pathspec match bug to commit 95c11ecc73 and proposed removing an optimization check in `treat_directory()`.
- **LESS=FRX documentation**: brian m. carlsson explained Git sets `LESS=FRX` only when `LESS` is unset, with Todd Zullinger suggesting documentation of `LESS=""` overrides.

## Looking ahead

Tyler Cipriani's v6 `--force-if-includes` series awaits further feedback before a potential reroll, particularly on variable naming. Derrick Stolee's "safe" strbuf API RFC needs a decision on whether the direction is worthwhile given the `st_add3()` overflow issue Phillip Wood identified. Maciej Ciemborowicz's worktree lifecycle hooks RFC will likely generate discussion on hook design in the coming week. The Outreachy "promisor remote fetch ordering" project proposal should see finalization for the December 2026 cohort.