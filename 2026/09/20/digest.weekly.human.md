## Long threads

### receive-report hook in `builtin/receive-pack.c`

*Long thread: 58 replies so far from 7 participants over 29 days.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client, receiving the pkt-line encoded report on stdin and allowing its stdout to replace the report. Junio C Hamano queued a coccinelle fix on top of the v10 series removing a redundant nullness check before `FREE_AND_NULL`, and Nayak confirmed the fix looks good. The series remains in flight with that cleanup applied.

### sequencer auto maintenance timing

*Long thread: 30 replies so far from 5 participants over 14 days.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, passing `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS`. Bachem posted v5 with wording updates only, adopting Patrick Steinhardt's documentation and Phillip Wood's commit message suggestions. The remaining question is placement: Steinhardt prefers calling maintenance at the end of `pick_commits()` plus single-pick return paths, while Bachem keeps the call in the two builtins per Wood's preference, offering to reroll if Steinhardt insists.

### unifying ref storage format terminology

*Long thread: 29 replies so far from 4 participants over 12 days.*

Patrick Steinhardt's 13-patch v3 series renames inconsistent ref storage format knobs to `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases. Reviewers debated whether `OPT_ALIAS()` entries should be hidden; Kaartic Sivaraam noted existing deprecated aliases remain visible, and Junio C Hamano explained `OPT_HIDDEN` belongs to a later deprecation stage, calling the inability to combine them a bug to mark with `#leftoverbit`. Hamano otherwise found the series in very good shape.

### advice squelch hints with `--global`

*Long thread: 25 replies so far from 4 participants over 9 days.*

Vsevolod Myalitsin's series addresses the `defaultBranchName` advice, whose suggested local-scope `git config set` only affects the current repository. Jeff King suggested simply recommending `--global` in all advice squelch messages, and Junio C Hamano posted v5 adding `--global` to the `git config set` command across `advice.c` and 11 test files. The discussion converged on a boolean global hint rather than a full `enum config_scope`, with Myalitsin confirming the topic reached its intended conclusion.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 22 replies so far from 4 participants over 37 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so Git can check stapled OCSP responses, defaulting to false. Gordon posted v7 as the combined version with tests split between `t5551` for cases needing no OCSP setup and the new `t5585` for responder-dependent cases, with `LIB_HTTPD_OCSP` support moved into lib-httpd. Junio C Hamano said the updated tests look good and that he will replace the existing tests with them.

## The period in brief

The week of 2026/09/14 through 2026/09/20 saw seven active days of steady traffic, with several long-running series reaching decision points and new bug fixes arriving for production-hit races. Readers should not miss the v2 of the repack `.keep` race fix, Tyler Cipriani's v6 resolving the `--force-if-includes` fast-forward conflict, and Maciej Ciemborowicz's v2 restoring old OIDs to batched ref deletions. Git v2.56.0-rc1 was also announced for testing.

## Key developments

### repack race with mid-run `.keep` files

qeesung (Qin ShiCheng) posted v2 of a five-patch series fixing a race where `git repack -d` deletes a pack whose objects were never copied because a concurrent push installs a `.keep` between repack's two scans. The v2 drops the receive-pack side, which Justin Tobler will fix separately by having the ODB transaction create the `.keep` at commit time. The core change has repack write its startup snapshot of kept packs to a tempfile and pass it to pack-objects via a new `--keep-pack-from-file=<file>` option instead of `--honor-pack-keep`, with supporting fixes for the cruft walk's kept-pack cache and binary lookup of kept packs.

### `--force-if-includes` fast-forward and non-branch handling

Tyler Cipriani advanced his series from v4 through v6, making `--force-if-includes` reject non-branch pushes while fixing a regression where valid fast-forward pushes were rejected when the local reflog was expired. The v6 patch 3/3 defers the reflog verification until after determining whether a force is needed, so fast-forwards proceed; patch 1/3 resolves the pushed ref via `ref->peer_ref->name` and rejects non-branch refs lacking a per-branch reflog; patch 2/3 adds a `pushRefUnverifiable` advice message suggesting `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`. D. Ben Knoble flagged that rejecting tag pushes breaks his `<TAG>:hotfix` workflow, which Cipriani confirmed.

### stash autostash duplicate entry fix

D. Ben Knoble posted a two-patch series fixing a bug reported by Eli Barzilay where `stash.index=true` causes a successfully applied merge autostash to remain as a redundant stash entry and deleting `MERGE_AUTOSTASH` to fail. The second patch replaces the subprocess-based index merge in `git stash apply --index` with an in-core `merge_incore_nonrecursive()` call, fixing the non-reentrant file-system round-trip that caused the lock failure. Phillip Wood reviewed the approach as looking good and suggested minor refinements including empty conflict labels and `verbosity=0`.

### old OIDs for batched ref deletions and transactional copy/rename

Maciej Ciemborowicz posted v2 of a series teaching `refs_delete_refs()` to accept an optional `oid_array` of old OIDs, making branch, tag, fetch and remote prune deletions conditional compare-and-delete operations that fail if the ref changed after resolution, restoring race protection lost in commit `8198907795`. A separate patch routes reference copy and rename through the transaction API so the `reference-transaction` hook observes them, representing a rename as deletion plus creation in one transaction with backend-specific reflog handling and rollback on hook rejection. Karthik Nayak noted the old-OID verification introduces a possible race for callers that checked the OID beforehand.

### incremental connectivity check via rev-list

Kristofer Karlsson posted a two-patch series adding an opt-in incremental connectivity check gated behind `transfer.connectivityCheck=incremental`, running inside the rev-list subprocess via a new internal `--verify-trees-incremental` flag. Benchmarks report up to ~190x speedup and much lower RSS for small pushes against a 3.4 GB boundary closure. Junio C Hamano called the benchmarks exciting but raised concerns about a possibly too-late `is_promisor_object()` call and a possible NULL from `get_commit_tree_oid()`; Karlsson confirmed both, explaining the promisor path is safe because `--exclude-promisor-objects` sets `fetch_if_missing = 0`, and said he will add a blob test and a NULL check with `die()`.

### commit-reach parses commits in caller's repository

Orestis Floros posted a patch passing a `struct repository *` into `can_all_from_reach()` and `can_all_from_reach_with_flag()` so commits are parsed in the caller's repository rather than `the_repository`, fixing a "fatal: invalid commit position" when merging a superproject whose submodule pointer advanced on both sides with commit-graphs present in both repositories. Kristofer Karlsson confirmed the regression test reproduces the bug and noted it reduces `the_repository` references in `commit-reach.c` from 17 to 11. Junio C Hamano called the change reasonable and said he will queue it.

### dir pathspec prefix with exclusions

Yannik Tausch's v4 series fixes the pathspec prefix optimization in `dir.c`: exclude items now match against the full pathname, and the common prefix is derived from the first non-exclude item to preserve the directory-walk optimization. Junio C Hamano said this round looks ready for `next`.

### upload-pack shallow-info ordering

Royce Remer posted v2 swapping the order of `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow-info goes first, fixing protocol v2 shallow fetches against servers with `uploadpack.allowRefInWant=true` that previously exited with "fatal: expected 'packfile', received 'shallow-info'". Junio C Hamano agreed the explanation was good commit-log material and noted the bug has existed since the original ref-in-want implementation.

## In brief

- **Git v2.56.0-rc1** -- Junio C Hamano announced the release candidate comprising 721 non-merge commits since v2.55.0 from 90 people, 37 of them new.
- **Git for Windows 2.56.0-rc0** -- Johannes Schindelin announced the Windows release candidate, which drops Windows 8.1 support and bundles Git v2.56.0-rc0, Git LFS v3.8.0, cURL v8.22.0 and OpenSSL v3.5.8.
- **git var v8** -- Andrew Pleeter posted v8 printing `VARIABLE=value` pairs for multi-variable queries, omitting unset variables while still exiting 1.
- **rerere lock timeout v4** -- Thomas Bachem posted v4 making `setup_rerere()` wait up to `rerere.lockTimeout` for `MERGE_RR.lock`.
- **packed-refs transaction skip** -- Junio C Hamano said he will queue Ariel Keselman's v3 patch skipping the packed-refs transaction for root-ref deletions and mark it for `next`.
- **git status --ignored bug** -- René Scharfe bisected the partial-pathspec match bug to commit 95c11ecc73 and proposed removing the `!excluded` optimization check in `treat_directory()`.
- **commit-buffer leak in history.c** -- Kaartic Sivaraam posted v4 fixing a leak in `builtin/history.c` by pairing `repo_logmsg_reencode()` with `repo_unuse_commit_buffer()`.
- **packfile flush during ODB transaction** -- Justin Tobler's two-patch series fixes packfile flush with `core.fsyncMethod=batch`; Karthik Nayak reviewed both positively.
- **git pull NULL guard** -- Junio C Hamano reviewed Jiri Kuncar's patch keeping `git pull` from crashing on an unresolvable merge head.
- **precompiled headers** -- SZEDER Gábor posted v2 of the series precompiling `git-compat-util.h`, reporting about 35% faster builds.
- **Rust gitcore crate move** -- Mike Hommey posted v5 moving the crate into a dedicated `rust/` subdirectory so the Git source can be used as a submodule in Rust projects.
- **RUNTIME_PREFIX on OpenBSD** -- Brad Smith submitted a patch enabling it via `getexecpath()`; Junio C Hamano asked why it should replace the existing ports method.
- **Outreachy project** -- Christian Couder endorsed and Kaartic Sivaraam proposed the "Implement promisor remote fetch ordering" project for the December 2026 cohort.
- **die_lasterr in winansi.c** -- Yongqiang Tian posted a fix for passing `va_list` as an ordinary argument; Johannes Sixt and René Scharfe debated simpler designs.
- **git-contacts stdin** -- Brigham Campbell posted v2 reading patch contents from stdin when argv is empty.
- **GIT_WORK_TREE for hooks** -- Junio C Hamano proposed exporting `GIT_WORK_TREE` for hooks in worktrees where `GIT_DIR` is also set.
- **Coverity fixes** -- Johannes Schindelin posted a seven-patch series fixing issues in `writev_in_full()`, gpg-interface, MIDX, rerere and reftable.
- **safe strbuf API RFC** -- Derrick Stolee posted an RFC proposing a `strbuf-safe` API that cannot reach `die()` to prevent recursive die loops in trace2; Phillip Wood objected to the name and identified that `sstrbuf_grow()` still dies on overflow.
- **rerere remaining fix** -- Junio C Hamano confirmed Mikko Rantalainen's diagnosis that `check_one_conflict()` skips consecutive conflicted paths with only stage-1 entries and posted a fix adding a same-path check.
- **diff --no-index -R** -- Haokai Ding's patch swapping filespecs for file/directory conflicts collected reviews from René Scharfe and Jacob Keller.
- **reflog expire defaults swapped** -- r.norouzi reported that since Git 2.50 the default expiry values are swapped, pruning reachable entries older than 30 days instead of 90.
- **fetch.shallow config** -- Harald Nordgren submitted a patch adding an off-by-default option making refspec-less fetches in shallow repositories fetch only the tracked upstream.
- **shallowHistory advice** -- Harald Nordgren posted a patch adding advice when `<ref>~N` resolution stops at a shallow boundary.
- **worktree lifecycle hooks RFC** -- Maciej Ciemborowicz proposed native hooks for worktree add, remove, move, lock, unlock and prune events.
- **LESS=FRX documentation** -- brian m. carlsson explained Git sets `LESS=FRX` only when `LESS` is unset; Todd Zullinger suggested documenting `LESS=""` overrides.
- **t40* modernization** -- Mark C. Chu-Carroll posted v4 of the three-patch series modernizing `t/t4001-diff-rename.sh` and related tests.
- **GitLab CI cargo fix** -- Johannes Schindelin's four-patch series fixing the cargo invocation in Windows CI jobs saw the `build:mingw64` job succeed.
- **dir.c nested repo** -- René Scharfe posted a fix so an excluded directory containing a nested repository is pathspec-matched on prefix match.
- **RFC commit date warning** -- Yashwanth Sai posted then withdrew an RFC warning when a new commit is dated before its parent after brian m. carlsson objected to its AI co-authorship.