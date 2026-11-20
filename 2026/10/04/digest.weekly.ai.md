## Long threads

### stash: in-core index merge for `stash apply --index`
*Long thread: 53 replies so far from 5 participants over 13 days.*

D. Ben Knoble's series replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing a non-re-entrant `save_autostash()` lock when autostashing with staged entries and `stash.index=true`. After v4 exposed a NULL-dereference risk on hard errors, v5 distinguished `result.clean < 0` from `!result.clean`; Phillip Wood confirmed the range-diff and Junio C Hamano marked the series for `next` on October 1.

### sequencer: leave auto maintenance to the end of a sequence
*Long thread: 33 replies so far from 5 participants over 25 days.*

Thomas Bachem's v5 series makes merge-backend rebase sequences run auto maintenance once at completion, matching the apply backend, by calling `run_auto_maintenance()` from `builtin/rebase.c` and `builtin/revert.c` while passing `maintenance.auto=false` to spawned commands. Patrick Steinhardt noted the series is already cooking in `next` and agreed it looks ready.

### uploadpack.lazyFetchTrusted server-side lazy fetch control
*Long thread: 34 replies so far from 3 participants over 85 days.*

Christian Couder's v5 rebases the series adding the protected `uploadpack.lazyFetchTrusted` config variable, letting server operators mark repositories trusted for lazy fetching during clones when large blobs live on a promisor remote. The series touches `builtin/upload-pack.c`, `promisor-remote.c`, and `setup.c`, and adds a `GIT_INTERNAL_LAZY_FETCH_DEPTH` recursion limit; v5 contains only comment, documentation, and test updates.

### refs: report old values to transaction hooks
*Long thread: 29 replies so far from 5 participants over 13 days.*

Maciej Ciemborowicz's v6 single-patch series resolves old object IDs in the common reference-transaction layer before the "preparing" hook and refreshes them after the backend locks refs, so batched branch, tag, and remote-ref deletions report real old OIDs instead of all-zero values. Patrick Steinhardt had argued the fix belongs at the transaction layer rather than in individual callers.

### rerere: wait for MERGE_RR.lock and skip in auto gc
*Long thread: 23 replies so far from 4 participants over 31 days.*

Thomas Bachem's v6 makes every rerere writer wait `rerere.lockTimeout` milliseconds for `MERGE_RR.lock`, adds a hidden `--skip-locked` option to `git rerere gc` used by `git maintenance run --auto` and `git gc --auto`, and lets conflict-time callers warn and continue via the renamed `RERERE_WARN_LOCKED` flag while `git commit` and `git am --continue` still fail.

### Documentation: new gitmergeconflicts guide
*Long thread: 32 replies so far from 6 participants over 10 days.*

Julia Evans's seven-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-pull`, `git-rebase`, `git-revert`, and `git-cherry-pick`. v2 added a "WHAT IS A MERGE CONFLICT?" section; reviewers including Patrick Steinhardt and Junio C Hamano suggested making `merge.conflictstyle=diff3` the default and enriching conflict markers with commit IDs and dates.

### xdiff: use size_t for mmfile_t
*Long thread: 21 replies so far from 4 participants over 3 days.*

Jeff King's v2 seven-patch series consolidates `mmbuffer_t` into `mmfile_t`, switches buffer sizes to `size_t`, NUL-terminates buffers read by `read_mmfile()`, and makes `merge-ll` report errors when reading external merge results fails. Patrick Steinhardt reviewed the consolidation and Junio C Hamano accepted keeping the `mmfile_t` name.

### format-patch range-diff notes options
*Long thread: 25 replies so far from 4 participants over 42 days.*

Kristoffer Haugsbakk's v5 series teaches `git format-patch` the `--[no-]range-diff-notes` options so the range diff can use a different notes ref than the patches themselves. v5 adopts Junio C Hamano's condensed documentation wording; the implementation uses a parse-options callback where `--no-range-diff-notes` means no range-diff notes at all.

### shallow-repo fetch of new remotes via refmap
*Long thread: 23 replies so far from 4 participants over 16 days.*

Harald Nordgren's v6 four-patch series avoids fetching every branch of a new remote in shallow repositories by adding `remote.<name>.refmap`, inferring branches from local upstreams, and adding `git remote add --limited-fetch` defaulting in shallow repos. v6 removes a leftover reference to deleted default-branch logic; Nordgren answered Junio C Hamano's design question by proposing that `remote.<name>.fetch` should win over `refmap` when both are set.

### push: fix --force-if-includes when the remote-tracking ref has no reflog
*Long thread: 23 replies so far from 6 participants over 27 days.*

Aleksei Sviridkin's v4 initializes the timestamp cutoff to 0 when the remote-tracking ref has no reflog, fixing false "remote ref updated since checkout" rejections of valid pushes on machines where the uninitialized value was non-zero. Junio C Hamano marked the patch for `next` and noted trivial t5533 conflicts with another topic.

### branch: let --delete-merged find squash-merged branches
*Long thread: 10 replies so far from 3 participants over 1 day.*

Harald Nordgren's patch makes `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all its changes, covering squash- and rebase-merged branches. D. Ben Knoble noted the check can be expensive and floated a `--delete-merged=squashed,rebased` option value; Nordgren said performance would be the only reason to hide the behavior.

### repack: corner cases for cruft-less MIDXs
*Long thread: 27 replies so far from 5 participants over 4 days.*

Taylor Blau's v2 eight-patch series fixes reachability-closure gaps when `repack.midxMustContainCruft=false` can produce a multi-pack-index whose objects are not closed under reachability, using an oidset for extra roots and traversing kept packs during geometric repacks. Elijah Newren objected that oidset iteration order can regress pack quality by visiting subtrees first and preventing delta compression.

### git var -z, multiple variables, broken-out idents
*Long thread: 22 replies so far from 6 participants over 39 days.*

Andrew Pleeter's v9 four-patch series adds a `-z` NUL-terminated output mode, support for multiple variable arguments, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables to `git var`. Phillip Wood reviewed the series as good with only minor test-style nits about `test_env()` usage.

### CI annotations linked to test scripts
*Long thread: 20 replies so far from 4 participants over 10 days.*

Harald Nordgren's two-patch series links GitHub Actions failure and leak annotations to the test script's file and line, and stops leak-sanitizer jobs at the first failure. He asked whether the inability to link annotations to test files unchanged in a PR constitutes a regression worth blocking the change.

### RFC for git-brebase
*Long thread: 12 replies so far from 2 participants over 1 day.*

Alejandro Colomar posted an RFC for `git-brebase`, a 164-line shell script that rebases onto the first conflicting commit of a target branch using `git bisect` internally. Nico Williams preferred a `--first-conflict` option on `git rebase` itself and suggested script improvements including a `trap` and avoiding a temp callback file.

## The period in brief

The week of 2026/09/28 through 2026/10/04 saw steady, high-volume traffic across seven active days, with several long-running series reaching decisive milestones. D. Ben Knoble's stash index-merge fix and Patrick Steinhardt's reftable reflog-timezone correction were both marked for `next`, while Christian Couder's `uploadpack.lazyFetchTrusted` and Thomas Bachem's rerere lock handling advanced to v5 and v6 respectively. Readers should not miss the submodule merge corruption investigation pointing to a stale delta-base cache, the new `gitmergeconflicts` documentation guide, and the `repack --dry-run` bug report showing pack files written despite the documented no-write promise.

## Key developments

### Reftable reflog timezone encoding fixed
Patrick Steinhardt posted a three-patch series correcting `refs/reftable-backend.c` to store reflog timezones as signed minute offsets per the reftable specification rather than `[+-]HHMM` integers, adding `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c` and a t0610 test. Junio C Hamano confirmed the fix looks ready and marked it for `next`; fallout is limited since timezones are display-only and JGit reads the value as minutes.

### Submodule merge corruption traced to delta-base cache
Philippe Blain bisected a superproject merge failure reading a commit from the wrong submodule to a commit-graph lookup change, and Patrick Steinhardt identified the root cause as stale delta-base cache entries keyed by `struct packed_git` address that are never evicted on pack close, reproducible via recursive submodule merges. Steinhardt posted a two-patch fix; Jeff King characterized the core bug as use-after-free and suggested ASan would find it reliably.

### http: curl credential stripping handled
Jeff King posted a patch because upcoming curl 8.23.0 strips credentials from `CURLINFO_EFFECTIVE_URL`, which would make Git's direct string comparison falsely detect a redirect and flush `http_auth` credentials. Patrick Steinhardt suggested `CURLINFO_REDIRECT_COUNT` but King reported it counts automatic auth retries; Junio C Hamano called the `curl_url()`-based `match_effective_url()` approach "very nice".

### "Limbo" object-format state proposed for empty repositories
Jeff King proposed letting an empty server-side repository advertise a "limbo" object format so first pushes from either SHA-1 or SHA-256 clients succeed. brian m. carlsson argued `object-format=limbo` would break existing clients, and King proposed a new ignorable capability ("magic-limbo") paired with a valid `object-format=` value; Junio C Hamano lamented that multiple `object-format` capability lines would have been the right design.

### repack --dry-run writes packs bug reported
Coy Geek reported that on Git 2.56.0, `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` writes four new files (`.pack`, `.idx`, `.promisor`, `.rev`) to the pack directory and, with explicit `-d`, deletes 12 existing files, contradicting the manual's promise that `--dry-run` lists candidates without rebuilding or deleting.

### merge-ort: --no-overwrite-ignore bypassed at rename destination
Coy Geek reported that `git merge --no-overwrite-ignore` fails to protect an ignored, untracked file when directory rename detection moves a committed file onto that path, overwriting the ignored content in both default and `conflict` modes; a direct collision is correctly refused, so only rename-detection destinations bypass the guard, reproduced on versions 2.54.0 through 2.56.0.

### post-worktree hook with prune notification
Domen Kožar posted v3 of a series adding a single `post-worktree` lifecycle hook for add, move, and remove events, extended to pruning so each pruned entry emits a remove event (skipped during dry runs), with hook failures reflected in the exit status while remaining entries are still processed.

### filter-branch state-branch map fix reaches v3
Grant Moyer posted v3 fixing `git filter-branch --state-branch` map initialization, which since f6d855091e in 2.50.0 interpreted object pairs in reverse, causing the state branch to be effectively ignored; the patch restores `$from_commit:$to_commit` order and adds a test that incremental rewrites use rewritten parents. Michele Locati had confirmed the bug and supplied a Tested-by tag.

## In brief

**connected: incremental connectivity check** -- Kristofer Karlsson posted v2 of the opt-in `transfer.connectivityCheck=incremental` mode running the verifier inside the rev-list subprocess, reporting up to 190x speedup versus the full check for single-commit pushes to large boundary-closure repos.

**Optional sha1dc Rust backend** -- Johannes Schindelin conceded his "compile-time" option is misleading since `sha1dc_choose()` selects the backend at runtime, and reported the Rust and C implementations cover the same collision-detection space; brian m. carlsson objected that the crate requires Rust 1.87 while Git supports older versions.

**CI install failures** -- Patrick Steinhardt suggested the CI script should fail outright when perforce, git-lfs, or JGit fail to install rather than warn; Harald Nordgren agreed.

**git -C alias bug** -- Maya Barriault reported that `git -C other_repo` run via an alias from a worktree sets `GIT_DIR` and `GIT_COMMON_DIR` to the worktree's values instead of the target repository's.

**What's cooking** -- Junio C Hamano noted Git 2.56 has been tagged and the next version will be Git 2.98, followed by 2.99 in preparation for Git 3.0.

**docs: remove gittutorial-2** -- Julia Evans replied that she is not replacing `gittutorial-2` since `gitdatamodel` and `gitcore-tutorial` cover its material; Junio C Hamano asked that `po/` translations be left intact and later reported doc-lint fails with "no link: gittutorial-2" once merged.

**merge-ll tempfile cleanup** -- Jeff King's v3 "merge-ll: use tempfile API for external driver files" cleans up `.merge_file_*` temporaries on Ctrl+C; Junio C Hamano said he would mark it for `next`.

**reflog expiry periods** -- Pushkar Singh's v3 swaps back the default reflog expiry periods in `reflog.h` (reachable 90 days, unreachable 30); Patrick Steinhardt suggested testing at 88 days, which Junio C Hamano accepted.

**stash create flags** -- Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`; Phillip Wood objected to the changed no-changes exit status and recommended `PARSE_OPT_STOP_AT_NON_OPTION`.

**sparse-checkout clobbering** -- Webstrand reported that `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting, whereas before 681c637b4a Git refused the checkout and preserved it.

**backfill --dry-run RFC** -- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee questioned the size metric and proposed `--info=(count|size)`.

**checkout -m conflict labels** -- Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`, fixing CRLF parsing with `strbuf_getline()`; Junio C Hamano agreed to document the common repository files separately.

**@{p} shorthand** -- Harald Nordgren's patch makes `@{p}` short for `@{push}`; Junio C Hamano marked it queued.

**packed-refs fwrite** -- Karthik Nayak's change writes unchanged refs with `fwrite()`, about 20% faster on deletes; Toon Claes listed sanitizations lost by `fwrite()` and proposed a separate preformatted writer function.

**alternates into files ODB** -- Patrick Steinhardt posted a 13-patch series moving alternates into the "files" object database backend, dropping the `read_alternates` callback from `struct odb_source`.

**commit-graph from updated refs** -- Kristofer Karlsson posted a series making fetch write the commit-graph from updated refs only; Patrick Steinhardt noted no benchmark was provided.

**SHA-256 tree digest RFC** -- Scott Chacon posted an RFC signing a SHA-256 digest of tree contents in commits and tags; brian m. carlsson rejected it in favor of moving to SHA-256.

**branch hook gaps** -- Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook while `-c`/`-m` invoke it without the new value.

**fetch.followRemoteHEAD** -- Colin Hinton posted v5 deferring validation to the point of use in `do_fetch()`, storing the raw string with a `follow_remote_head_seen` flag after Junio C Hamano endorsed dying rather than warning on bad values.

**Zsh completion** -- Fionn Fitzmaurice patched the Zsh completion to exclude files already present in the argument list.

**interpret-trailers examples** -- Kristoffer Haugsbakk posted v2 rephrasing the `trailer.<key-alias>.cmd` examples; Junio C Hamano marked the topic for `next`.

**precompiled header** -- SZEDER Gábor noted the precompiled `git-compat-util.h` changes `__FILE__` expansion in error messages, which he judged acceptable.

**fsmonitor untracked cache** -- Ilia K posted a patch adding `untracked_cache_invalidate_stale_dirs()` so that after the fsmonitor daemon sends a trivial response, stale untracked-cache directories are invalidated before the index is saved.

**history sign v5** -- Souma posted v5 adding Git completion coverage for the existing `-S`/`--gpg-sign` and `--no-gpg-sign` options on history subcommands.

## Looking ahead

Several series reached v5 or v6 this week and are likely to see final review or graduation in the coming days: Christian Couder's `uploadpack.lazyFetchTrusted`, Thomas Bachem's rerere lock handling, Maciej Ciemborowicz's transaction-hook old-OID fix, and Harald Nordgren's shallow-repo refmap fetch all await further maintainer feedback. Julia Evans's `gitmergeconflicts` guide and Taylor Blau's cruft-less MIDX repack fixes absorbed substantial review and may need another iteration. The submodule delta-base cache fix and the `repack --dry-run` bug report are fresh and will likely generate follow-up discussion.