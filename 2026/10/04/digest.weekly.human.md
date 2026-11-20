## Long threads

### stash: in-core index merge for `stash apply --index`

*Long thread: 53 replies so far from 5 participants over 13 days.*

D. Ben Knoble's series evolved from v4 to v5 during the week, fixing the tri-state handling of `merge_result.clean` that Junio C Hamano had diagnosed. The final patch replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, resolving a non-re-entrant lock bug when autostashing with `stash.index=true`. Phillip Wood confirmed the range-diff and welcomed the removal of subprocesses, and Hamano marked the series for `next` on 10/01.

### uploadpack.lazyFetchTrusted server-side lazy fetch control

*Long thread: 34 replies so far from 3 participants over 85 days.*

Christian Couder posted v5 of the series introducing the protected `uploadpack.lazyFetchTrusted` config variable, which lets server operators mark repositories trusted for lazy fetching during clones. V5 was rebased with no functional changes from v4, only documentation and test updates, including a NEEDSWORK comment about silently ignoring misspelled allowlist paths. Junio C Hamano reviewed the control logic as trivially clear and suggested doc wording fixes.

### sequencer: leave auto maintenance to the end of a sequence

*Long thread: 33 replies so far from 5 participants over 25 days.*

Thomas Bachem's v5 series makes merge-backend rebase sequences run auto maintenance once at completion, matching the apply backend, by calling `run_auto_maintenance()` from `builtin/rebase.c` and `builtin/revert.c`. Patrick Steinhardt noted on 9/28 that the series is already cooking in `next` and agreed it looks ready, effectively concluding the thread's active development phase for this period.

### Documentation: new `gitmergeconflicts(7)` guide

*Long thread: 32 replies so far from 6 participants over 10 days.*

Julia Evans's seven-patch series adds a `gitmergeconflicts` manual page linked from merge, rebase, revert, cherry-pick, and pull. The week saw v2 add a "WHAT IS A MERGE CONFLICT?" section explaining 3-way merge, with Junio C Hamano offering review reactions and an illustration of why `diff3` style is essential. D. Ben Knoble endorsed the section on 10/03, and Patrick Steinhardt proposed extending conflict-marker labels to include commit ID, message, and date.

### refs: report old values to reference-transaction hooks

*Long thread: 29 replies so far from 5 participants over 13 days.*

Maciej Ciemborowicz's v6 single-patch series resolves old object IDs in the common transaction layer before the "preparing" hook and after backend locks, fixing batched deletions reporting all-zero old OIDs. Patrick Steinhardt had earlier argued for this transaction-layer approach over caller-specific fixes. On 10/01 Ciemborowicz lowered the priority of the underlying bug after resolving his branch and tag deletion issues, naming branch renaming as the remaining case needing this fix.

### repack: corner cases for cruft-less MIDXs

*Long thread: 27 replies so far from 5 participants over 4 days.*

Taylor Blau's v2 eight-patch series fixes cases where `repack.midxMustContainCruft=false` produces a multi-pack-index not closed under reachability, which can fail bitmap generation. The week saw detailed review of the oidset conversion for extra roots, kept-pack traversal, and the `--write-midx=incremental` gap. Elijah Newren objected that oidset iteration order can regress pack quality by visiting subtrees first, causing blobs to be written in full instead of delta-compressed.

### format-patch `--range-diff-notes` options

*Long thread: 25 replies so far from 4 participants over 42 days.*

Kristoffer Haugsbakk's series teaches `git format-patch` `--[no-]range-diff-notes=<ref>` so the range diff can use a different notes ref than the patches. The week saw v4 trim expository text and refactor the callback, then v5 adopt Junio C Hamano's condensed documentation wording. Hamano reviewed v4 2/2 as otherwise good, and Haugsbakk posted v5 with the shorter paragraph.

### push: fix `--force-if-includes` when the remote-tracking ref has no reflog

*Long thread: 23 replies so far from 6 participants over 27 days.*

Aleksei Sviridkin's v4 initializes the timestamp cutoff to 0 when the remote-tracking ref has no reflog, fixing spurious "remote ref updated since checkout" rejections. Junio C Hamano said on 9/29 that the v4 changes look sensible, will replace the patch and mark it for `next`, noting trivial t5533 conflicts with another topic.

### shallow-repo fetch of new remotes via refmap

*Long thread: 23 replies so far from 4 participants over 16 days.*

Harald Nordgren's four-patch series avoids fetching every branch of a new remote in shallow repositories by adding `remote.<name>.refmap`, inferring branches from local upstreams, and adding `git remote add --limited-fetch`. On 10/04 he posted v6 removing a leftover reference to deleted default-branch logic. Junio C Hamano reviewed v6 as much better but flagged that with only patch 1/4 applied, a remote with `refmap` and no tracked branches makes `git fetch` die; Nordgren proposed that `remote.<name>.fetch` should win when both are configured.

### rerere: wait for `MERGE_RR.lock` and skip in auto gc

*Long thread: 23 replies so far from 4 participants over 31 days.*

Thomas Bachem's series makes rerere writers wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds a hidden `--skip-locked` option to `git rerere gc` for auto-maintenance, and lets conflict-time callers warn and continue. On 10/02 he posted v6 renaming the option to `--skip-locked` and the flag to `RERERE_WARN_LOCKED`, after finding that calling `rerere_gc()` directly from `builtin/gc.c` would die with exit code 128 on a held lock. Patrick Steinhardt agreed to keep the hidden option.

### git var `-z`, multiple variables, broken-out idents

*Long thread: 22 replies so far from 6 participants over 39 days.*

Andrew Pleeter's v9 series splits into four patches adding string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, support for multiple variable arguments, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables. Phillip Wood reviewed the series as good with only minor test-style nits about `test_env()` and collapsing per-component tests, which Pleeter acknowledged.

### xdiff: use `size_t` for `mmfile_t`

*Long thread: 21 replies so far from 4 participants over 3 days.*

Jeff King's seven-patch v2 consolidates `mmbuffer_t` into `mmfile_t`, switches buffer sizes to `size_t`, NUL-terminates buffers read by `read_mmfile()`, and makes `merge-ll` report errors when reading external merge results fails. Patrick Steinhardt flagged a commit-message nit in the NUL-termination patch and okayed the `merge-ll` change; Junio C Hamano accepted keeping the `mmfile_t` name and agreed the error-reporting fix is easier after the `read_mmfile()` switch.

### CI annotations linked to test scripts

*Long thread: 20 replies so far from 4 participants over 10 days.*

Harald Nordgren's two-patch series links GitHub Actions failure and leak annotations to the test script's file and line, and stops leak-sanitizer jobs at the first failure. On 10/04 he asked whether the inability to link annotations to unchanged test files is enough to call the change a regression, suggesting that part might not be worth doing, and proposed deciding on the regression question before progressing.

### RFC for `git-brebase`

*Long thread: 12 replies so far from 2 participants over 1 day.*

Alejandro Colomar posted an RFC for `git-brebase`, a 164-line shell script that rebases onto the first conflicting commit of a target branch using `git bisect` internally. Nico Williams preferred a `--first-conflict` option on `git rebase` itself and suggested script improvements including a `trap`; Colomar argued `--abort` should return only to the start of the current iteration and said he now defaults to brebase for every rebase.

### branch: let `--delete-merged` find squash-merged branches

*Long thread: 10 replies so far from 3 participants over 1 day.*

Harald Nordgren posted a patch making `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all its changes, covering squash- and rebase-merged branches. D. Ben Knoble reviewed the approach, noting the check can be expensive, and floated a `--delete-merged=squashed,rebased` option value; Nordgren said performance would be the only reason to hide the behavior.

## The period in brief

The week of 2026/09/28 through 2026/10/04 saw heavy traffic across seven active days, with several long-running series reaching new revisions or landing in `next`. D. Ben Knoble's stash index-merge fix was marked for `next`, Patrick Steinhardt's reftable reflog timezone fix looked ready, and multiple documentation and infrastructure efforts advanced. The tone was eventful, with detailed review on cruft-less MIDX repacking, merge-conflict documentation, and server-side lazy fetch controls.

## Key developments

### Reftable reflog timezone storage fixed to match specification

Patrick Steinhardt posted a three-patch series fixing `refs/reftable-backend.c`, which stored reflog timezones as signed HHMM integers instead of the specification's signed minute offsets. He added `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c`, fixed a `dump-reftable -t` segfault, and added a t0610 test. Junio C Hamano agreed Git should be fixed to match the spec since JGit reads the value as minutes, and on 10/01 said the v2 looks really ready and will be marked for `next`.

### Packfile delta-base cache corruption in submodule merges

Philippe Blain bisected a superproject merge failure reading a commit from the wrong submodule to commit bb5da75d61 using the commit graph in `lookup_commit_reference_gently()`. Patrick Steinhardt identified the root cause as stale delta-base cache entries keyed by `struct packed_git` address that are never evicted on pack close, and posted a two-patch fix on 10/02. Jeff King characterized the core bug as use-after-free and suggested ASan would find it reliably.

### HTTP transport fix for curl 8.23.0 credential stripping

Jeff King posted a patch because upcoming curl 8.23.0 strips credentials from `CURLINFO_EFFECTIVE_URL`, which would make Git's string comparison falsely detect a redirect and flush `http_auth` credentials. Patrick Steinhardt suggested `CURLINFO_REDIRECT_COUNT` but King reported it counts automatic auth retries as redirects; Junio C Hamano called the `curl_url()`-based `match_effective_url()` approach "very nice", and Steinhardt agreed to proceed with the fix on 9/29.

### "Limbo" object-format state for empty repositories

Jeff King proposed letting an empty server-side repository advertise a "limbo" object format so first pushes from either SHA-1 or SHA-256 clients succeed. On 10/03 brian m. carlsson argued `object-format=limbo` would break existing clients, and King proposed a new ignorable capability ("magic-limbo") paired with a valid `object-format=` value. Junio C Hamano lamented that multiple `object-format` capability lines would have been the right design.

### filter-branch state-branch commit map fix

Grant Moyer posted v3 fixing `git filter-branch --state-branch` map initialization, which since f6d855091e in 2.50.0 interpreted object pairs in reverse, causing the state branch to be effectively ignored. Michele Locati confirmed the bug and tested the fix on 2.56.0, adding a Tested-by tag. Junio C Hamano asked for a commit-log explanation of a test change on 10/01.

### What's cooking: Git 2.56 tagged, next version 2.98

Junio C Hamano's "What's cooking" report on 9/28 noted that Git 2.56 has been tagged and the next version will be Git 2.98, followed by 2.99 in preparation for Git 3.0. The report also tracked the stash series, reftable timezone fix, and other topics moving through `next` and `seen`.

## In brief

**Optional sha1dc Rust backend for SHA-1** -- Johannes Schindelin proposed a compile-time option to use the Rust `sha1dc` crate for roughly three times faster `git index-pack` on Windows, with a `core.sha1dcBackend=c` escape hatch; on 10/03 he conceded "compile-time" is misleading since the backend is chosen at runtime, and brian m. carlsson objected that the crate requires Rust 1.87 while Git supports older versions.

**CI hard-fail on optional dependency install failures** -- Patrick Steinhardt suggested the CI script should fail outright when perforce, git-lfs or JGit fail to install rather than warn; Harald Nordgren agreed.

**`git -C` alias worktree environment bug** -- Maya Barriault reported that `git -C other_repo` run via an alias from a worktree sets `GIT_DIR` and `GIT_COMMON_DIR` to the worktree's values instead of the target repository's.

**merge-ll tempfile cleanup on interrupt** -- Jeff King's v3 "merge-ll: use tempfile API for external driver files" cleans up `.merge_file_*` temporaries on Ctrl+C; Junio C Hamano said he would mark it for `next`.

**Default reflog expiry periods swapped back** -- Pushkar Singh's v3 swaps back the default reflog expiry periods in `reflog.h` (reachable 90 days, unreachable 30); Patrick Steinhardt suggested testing at 88 days, which Junio C Hamano accepted.

**sparse-checkout clobbering untracked in-cone files** -- Webstrand reported that `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting, a regression from before 681c637b4a.

**merge-ort `--no-overwrite-ignore` bypass at rename destination** -- Coy Geek reported that `git merge --no-overwrite-ignore` fails to protect an ignored, untracked file when directory rename detection moves a committed file onto that path, reproduced on versions 2.54.0 through 2.56.0.

**Incremental connectivity check via rev-list** -- Kristofer Karlsson posted v2 of the opt-in `transfer.connectivityCheck=incremental` mode, which runs the verifier inside the rev-list subprocess, reporting up to 190x speedup for single-commit pushes to large boundary-closure repos.

**Alternates moved into files ODB backend** -- Patrick Steinhardt posted a 13-patch series moving alternates into the "files" object database backend, dropping the `read_alternates` callback from `struct odb_source`.

**Fetch commit-graph from updated refs only** -- Kristofer Karlsson posted a series making fetch write the commit-graph from updated refs only; Patrick Steinhardt reviewed, noting no benchmark was provided.

**RFC: sign SHA-256 digest of tree contents** -- Scott Chacon posted an RFC signing a SHA-256 digest of tree contents in commits and tags; brian m. carlsson rejected it in favor of moving to SHA-256.

**Branch hook gaps for copy and reflog rewrite** -- Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook while `-c`/`-m` invoke it without the new value.

**refs copy/rename transaction hook impasse** -- Maciej Ciemborowicz's v2 patch representing branch copy and rename as reference transactions hit a review impasse when Patrick Steinhardt declared it not reviewable due to apparently AI-generated content; Ciemborowicz apologized and offered a v3 split into more commits.

**fetch.followRemoteHEAD deferred validation** -- Colin Hinton posted v5 deferring `fetch.followRemoteHEAD` validation to the point of use in `do_fetch()`, storing the raw string with a `follow_remote_head_seen` flag after Junio C Hamano endorsed dying rather than warning on bad values.

**history: sign rewritten commits** -- Souma posted v5 adding Git completion coverage for the existing `-S`/`--gpg-sign` and `--no-gpg-sign` options on history subcommands; Junio C Hamano reported that with the topic merged, `seen` breaks t9902.

**Documentation synopsis cleanup** -- Julia Evans published v2 removing required SYNOPSIS sections from section 7 man pages, with Tuomas Ahola's declarative lint-script rewrite.

**Zsh completion deduplication** -- Fionn Fitzmaurice patched the Zsh completion to exclude files already present in the argument list.

**interpret-trailers documentation examples** -- Kristoffer Haugsbakk posted v2 rephrasing the `trailer.<key-alias>.cmd` examples; Junio C Hamano marked the topic for `next` on 10/04.

**Precompiled header `__FILE__` expansion** -- SZEDER Gábor noted the precompiled `git-compat-util.h` changes `__FILE__` expansion in error messages, which he judged acceptable.

**repack `--dry-run` writes packs bug** -- Coy Geek reported that on Git 2.56.0, `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` writes new pack files and with `-d` deletes existing ones, contradicting the documented no-write promise.

**fsmonitor untracked cache invalidation** -- Ilia K posted a patch adding `untracked_cache_invalidate_stale_dirs()` so that after the fsmonitor daemon sends a trivial response, stale untracked-cache directories are invalidated before the index is saved.

**post-worktree hook with prune notification** -- Domen Kožar posted v3 of the series adding a single `post-worktree` lifecycle hook for add, move, and remove, extended to pruning with remove events for each pruned entry.

**git stash create `-u`/`-a` options** -- Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`; Junio C Hamano critiqued the commit message's user-visible framing.

**docs: remove gittutorial-2** -- Julia Evans replied that she is not replacing `gittutorial-2` since `gitdatamodel` and `gitcore-tutorial` cover its material; Junio C Hamano asked that `po/` translations be left intact.

**@{p} shorthand for @{push}** -- Harald Nordgren's patch makes `@{p}` short for `@{push}`; Junio C Hamano marked it queued.

**packed-refs fwrite() optimization** -- Karthik Nayak's packed-refs change writes unchanged refs with `fwrite()`, about 20% faster on deletes; Toon Claes listed sanitizations lost by fwrite() and proposed a separate preformatted writer function.

**git checkout -m conflict labels** -- Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`, fixing CRLF parsing with `strbuf_getline()`; Junio C Hamano agreed to document the common repository files separately.

**git repo structure ref-type filters** -- Patrick Steinhardt objected to Mark Chu-Carroll's ref-type filter flags for `git repo structure`, preferring revision-based filtering.

**backfill `--dry-run` RFC** -- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee questioned the size metric and proposed `--info=(count|size)`.

**early_scan_options review** -- Kaartic Sivaraam reviewed Christian Couder's `early_scan_options()` series, flagging abbreviation handling and test questions.

## Looking ahead

Several series posted late in the week will dominate review in the coming days. Harald Nordgren's shallow-repo fetch series v6 needs resolution of the precedence question between `remote.<name>.fetch` and `remote.<name>.refmap`, while Thomas Bachem's rerere v6 with the hidden `--skip-locked` option and Christian Couder's `uploadpack.lazyFetchTrusted` v5 await further maintainer feedback. Taylor Blau's cruft-less MIDX v2 faces the oidset pack-quality objection from Elijah Newren, and Julia Evans's `gitmergeconflicts` guide will see whether the proposed conflict-marker extensions are adopted.