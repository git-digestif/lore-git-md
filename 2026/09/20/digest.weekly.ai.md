## Long threads

### receive-report hook in `builtin/receive-pack.c`

*Long thread: 58 replies so far from 7 participants over 29 days.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client, receiving the pkt-line encoded report on stdin and allowing its stdout to replace the report, with a non-zero exit marking all references rejected. Junio C Hamano queued a coccinelle fix on top of the v10 series removing a redundant nullness check before `FREE_AND_NULL`, and Nayak confirmed the fix looks good. The series has stabilized with earlier revisions addressing a `BUG()` when no report was requested and a leak when reassigning command strings.

### sequencer auto maintenance timing

*Long thread: 30 replies so far from 5 participants over 14 days.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, passing `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS` to prevent background maintenance from racing the sequence for `MERGE_RR.lock`. V5 arrived with wording updates only, adopting Patrick Steinhardt's documentation and Phillip Wood's commit message rewording, while Bachem asked Steinhardt to choose between calling auto maintenance at the end of `pick_commits()` plus single-pick return paths or in the two builtins, saying he will reroll accordingly.

### unifying ref storage format terminology

*Long thread: 29 replies so far from 4 participants over 12 days.*

Patrick Steinhardt's 13-patch v3 series renames inconsistent ref storage format interfaces -- `--ref-format=`, `GIT_REFERENCE_BACKEND`, `init.defaultRefFormat` and friends -- to consistent `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases and letting `--ref-storage-format=` accept `format://payload` URIs. Karthik Nayak questioned whether hidden `OPT_ALIAS()` entries make sense, Kaartic Sivaraam wondered whether tests should cover deprecated variants, and Junio C Hamano explained that `OPT_HIDDEN` belongs to a later deprecation stage, saying that if `OPT_ALIAS()` cannot take `OPT_HIDDEN` that is an infrastructure bug to be marked with a `#leftoverbit`.

### advice squelch hints with `--global`

*Long thread: 25 replies so far from 4 participants over 9 days.*

Vsevolod Myalitsin's series addresses the `defaultBranchName` advice, whose suggested `git config set advice.defaultBranchName false` writes to local scope and so only affects the repository where the advice was already shown. Jeff King suggested simply recommending `--global` in all advice squelch messages, Junio C Hamano agreed with the simplicity and posted a v5 adding `--global` to the `git config set` command in the advice message across `advice.c` and 11 test files, while Myalitsin confirmed the topic reached its intended conclusion and proposed submitting a separate patch passing a pointer to the `advice_setting` to `vadvise()`.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 22 replies so far from 4 participants over 37 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so Git can check stapled OCSP responses, defaulting to false because enabling it blocks connections to servers that send no stapled response. V7 arrived as the combined version discussed with Junio C Hamano and Patrick Steinhardt, splitting tests between `t5551` for cases needing no OCSP setup and the new `t5585` for everything requiring a responder, with `LIB_HTTPD_OCSP` support moved into lib-httpd. Hamano said the updated tests look good and that he will replace the existing tests with them.

## The period in brief

The week of 2026/09/14 through 2026/09/20 saw steady traffic across seven active days, with five long-running threads continuing and several new bug fixes and performance improvements arriving. Readers should not miss the v2 repack race fix addressing a production-hit data-loss bug, Tyler Cipriani's v6 `--force-if-includes` series that finally allows fast-forward pushes, and Maciej Ciemborowicz's work restoring old OIDs to batched ref deletions and routing copy/rename through the transaction API.

## Key developments

### repack race with mid-run `.keep` files

Qin ShiCheng (posting as qeesung) posted v2 of a five-patch series fixing a production-hit race where `git repack -d` deletes a pack whose objects were never copied: a concurrent push can link a `.keep` onto an existing pack between repack's initial scan and pack-objects' rescan, so pack-objects leaves the objects out and repack deletes the pack anyway. The core change has repack write its startup snapshot of kept packs to a tempfile and pass it to pack-objects via a new `--keep-pack-from-file=<file>` option instead of `--honor-pack-keep`, so both processes work from one snapshot. Justin Tobler reviewed the approach, agreeing that a process should only delete a `.keep` it created itself, and offered to fix the receive-pack side by having the ODB transaction create the `.keep` itself at commit time.

### `--force-if-includes` allows fast-forward pushes

Tyler Cipriani posted v6 of the three-patch series checking the pushed ref for `--force-if-includes`. Patch 3/3 defers the reflog rejection until after fast-forward rules are evaluated, so valid fast-forward pushes, including those with expired reflogs, proceed instead of being rejected; patch 1/3 resolves the pushed ref via `ref->peer_ref->name` and rejects non-branch refs that lack a per-branch reflog; and patch 2/3 adds a `pushRefUnverifiable` advice message suggesting `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`. D. Ben Knoble had flagged that earlier versions broke his workflow of pushing `<TAG>:hotfix`, which Cipriani confirmed, noting the check previously consulted the same-named local branch's reflog rather than the pushed tag.

### refs: old OIDs for batched deletions and transactional copy/rename

Maciej Ciemborowicz posted v2 of a series teaching `refs_delete_refs()` to accept an optional `oid_array` of old OIDs, making branch, tag, fetch and remote prune deletions conditional and restoring the compare-and-delete race protection that existed before commit `8198907795` converted them to batched deletion. A separate patch routes reference copy and rename through the transaction API so the `reference-transaction` hook observes them, representing a rename as a deletion of the old reference plus creation of the new reference in the same transaction, with tests covering concurrent updates and prepared-hook rollback. Karthik Nayak reviewed patch 1/3, noting that verifying old OIDs introduces a possible race for callers that checked a ref's OID beforehand.

### stash autostash fix with in-core index merge

D. Ben Knoble posted a two-patch series fixing a bug reported by Eli Barzilay where, with `stash.index=true`, a successfully applied merge autostash is still stored as a stash entry and deleting `MERGE_AUTOSTASH` fails. The second patch replaces the subprocess-based index merge in `git stash apply --index` with an in-core `merge_incore_nonrecursive()` call, fixing the autostash bug where the file-system round-trip was not re-entrant and caused a lock failure. Phillip Wood confirmed Knoble's reading of the flow and reviewed the patch as looking good, suggesting empty conflict labels, `verbosity=0`, and `oidcpy`.

### incremental connectivity check via rev-list

Kristofer Karlsson posted a two-patch series adding an opt-in incremental connectivity check gated behind `transfer.connectivityCheck=incremental`, running inside the rev-list subprocess via a new internal `--verify-trees-incremental` flag, with benchmarks reporting up to ~190x speedup and much lower RSS for small pushes against a 3.4 GB boundary closure. Junio C Hamano called the benchmarks "exciting" but raised two concerns on patch 2/2 -- a possibly too-late `is_promisor_object()` call in `verify_blob()` and a possible NULL from `get_commit_tree_oid()` passed to `tree_map_add()` -- and Karlsson confirmed both, saying he will add a blob test and a NULL check with `die()`.

### commit-reach: parse commits in the given repository

Orestis Floros posted a patch passing a `struct repository *` into `can_all_from_reach()` and `can_all_from_reach_with_flag()` in `commit-reach.c` so commits are parsed in the caller's repository rather than `the_repository`, fixing a crash when merging a superproject whose submodule pointer advanced on both sides with commit-graphs existing in both repositories. Kristofer Karlsson confirmed the new tests reproduce the bug and noted it reduces `the_repository` references in `commit-reach.c` from 17 to 11. Junio C Hamano called the change reasonable and a welcome step for libification, and said he will queue it.

### RFC: a "safe" strbuf API that cannot reach die()

Derrick Stolee posted an RFC series proposing a `strbuf-safe` API in new `strbuf-safe.c` and `strbuf-safe.h` files whose methods cannot transitively call `die()` or `exit()`, motivated by recursive die loops in trace2 under allocation failure. The series moves the strbuf struct definition into `strbuf-safe.h`, adds `sstrbuf_grow()`, `sstrbuf_init()` and `sstrbuf_release()` under an `s` prefix, and adapts `json-writer.c` to use the new API. Phillip Wood endorsed APIs that return errors rather than dying but objected to the name "safe" and identified that `sstrbuf_grow()` calls `st_add3()`, which dies on overflow, undermining the RFC's verification goals.

## In brief

- **dir: pathspec prefixes with exclusions** -- Yannik Tausch's v4 series fixes the pathspec prefix optimization in `dir.c` by deriving the prefix from non-exclude items and matching exclude items against full pathnames; Junio C Hamano said this round looks ready for `next`.
- **upload-pack: swap wanted-ref/shallow-info** -- Royce Remer posted v2 swapping the send order in `upload_pack_v2()` so shallow-info goes first, fixing protocol v2 shallow fetches against servers with `uploadpack.allowRefInWant=true` that exited with "fatal: expected 'packfile', received 'shallow-info'".
- **`rerere remaining` skips consecutive conflicted paths** -- Junio C Hamano confirmed Mikko Rantalainen's diagnosis that `check_one_conflict()` skips consecutive conflicted paths with only stage-1 entries and posted a patch adding a same-path check, which also fixes `git mergetool` exiting successfully while conflicts remain.
- **reflog expire default expiry times swapped** -- r.norouzi reported that `git reflog expire` prunes reachable reflog entries older than 30 days instead of the documented 90, because the default expiry values were swapped in commit `85658275702b` first released in 2.50.0.
- **`diff --no-index -R` file/directory conflicts** -- Haokai Ding fixed `queue_diff()` queuing the file before recursing into the directory, which bypassed reverse handling; the patch swaps the filespecs of the early queue entry when `reverse_diff` is set, with reviews from René Scharfe and Jacob Keller.
- **Git v2.56.0-rc1** -- Junio C Hamano announced the release candidate comprising 721 non-merge commits since v2.55.0 from 90 people, 37 of them new.
- **Git for Windows 2.56.0-rc0** -- Johannes Schindelin announced the release candidate, which drops Windows 8.1 support and bundles Git v2.56.0-rc0, Git LFS v3.8.0, cURL v8.22.0 and OpenSSL v3.5.8.
- **Coverity fixes from Git for Windows** -- Johannes Schindelin posted a seven-patch series fixing Coverity issues including `writev_in_full()` overflow, gpg-interface signature matching, and rerere failure handling; Hamano approved patch 4/7.
- **`git var` v8** -- Andrew Pleeter posted v8 printing `VARIABLE=value` pairs for multi-variable queries, omitting unset variables while still exiting 1.
- **rerere lock timeout** -- Thomas Bachem posted v4 of a two-patch series making `setup_rerere()` wait up to `rerere.lockTimeout` for `MERGE_RR.lock`.
- **packed-refs transaction for root-ref deletions** -- Junio C Hamano said he will queue Ariel Keselman's v3 patch skipping the packed-refs transaction for root-ref deletions in the files backend and mark it for `next`.
- **commit-buffer leak in `builtin/history.c`** -- Kaartic Sivaraam posted v4 pairing `repo_logmsg_reencode()` with `repo_unuse_commit_buffer()`.
- **packfile flush with `core.fsyncMethod=batch`** -- Justin Tobler's two-patch series fixes packfile flush during ODB transaction commit; Karthik Nayak reviewed both positively.
- **NULL-guard for `git pull`** -- Junio C Hamano reviewed Jiri Kuncar's patch keeping `git pull` from crashing on an unresolvable merge head.
- **precompiling `git-compat-util.h`** -- SZEDER Gábor posted v2 reporting about 35% faster builds.
- **`die_lasterr()` in `compat/winansi.c`** -- Yongqiang Tian posted a fix for passing `va_list` as an ordinary argument; Johannes Sixt and René Scharfe debated simpler designs.
- **`git-contacts` v2** -- Brigham Campbell posted v2 reading patch contents from stdin when argv is empty; Hamano floated a "-" filename alternative.
- **`GIT_WORK_TREE` for hooks in worktrees** -- Junio C Hamano proposed a `setup.c` patch exporting `GIT_WORK_TREE` for hooks in worktrees where `GIT_DIR` is also set.
- **Rust `gitcore` crate relocation** -- Mike Hommey posted v5 moving the crate into a dedicated `rust/` subdirectory to allow use of the Git source as a submodule in Rust projects.
- **`RUNTIME_PREFIX` on OpenBSD** -- Brad Smith submitted a patch enabling it via `getexecpath()`; Hamano asked why the new method should replace the one OpenBSD ports already use.
- **Outreachy December 2026 proposals** -- Christian Couder endorsed proposing the "Implement promisor remote fetch ordering" project, and Kaartic Sivaraam proposed to mentor it.
- **`fetch.shallow` config option** -- Harald Nordgren submitted a patch adding an off-by-default option making refspec-less fetches in shallow repositories fetch only the current branch's tracked upstream.
- **`shallowHistory` advice** -- Harald Nordgren posted a patch adding advice when `<ref>~N` or `<ref>^N` resolution stops at a shallow boundary, suggesting `git fetch --deepen=<n>`.
- **worktree lifecycle hooks RFC** -- Maciej Ciemborowicz proposed an RFC adding native hooks for worktree add, remove, move, lock, unlock and prune events.
- **branch rename hook visibility** -- Maciej Ciemborowicz reported that `git branch -m old new` does not report the destination ref to the reference-transaction hook; Karthik Nayak called the behavior absurd and offered to look into a fix.
- **GitLab CI cargo fix** -- Johannes Schindelin posted a four-patch series fixing the cargo invocation in Windows CI jobs broken since Rust was enabled there.
- **test modernization** -- Mark C. Chu-Carroll posted v4 of the "t40*: modernize" series and accepted Hamano's review suggestions on `t/t4001-diff-rename.sh`.
- **`git status --ignored` partial-pathspec bug** -- René Scharfe bisected the bug to commit 95c11ecc73 and proposed removing the `!excluded` optimization check in `treat_directory()`.
- **excluded directory with nested repository** -- René Scharfe posted a `dir.c` fix so an excluded directory containing a nested repository is pathspec-matched on prefix match.
- **AI co-authorship RFC withdrawn** -- Yashwanth Sai posted an RFC warning when a new commit is dated before its parent, then withdrew it after brian m. carlsson objected to its AI co-authorship under the `ai` section of `Documentation/SubmittingPatches`.
- **`LESS=FRX` explanation** -- brian m. carlsson explained that Git sets `LESS=FRX` only when `LESS` is unset; Todd Zullinger suggested documenting `LESS=""` overrides in `Documentation/config/core.adoc`.