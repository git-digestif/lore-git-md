## Long threads

### stash: in-core index merge for `stash apply --index`
*Long thread: 53 replies so far from 5 participants over 13 days.*
D. Ben Knoble's series replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing a non-re-entrant `save_autostash()` lock hit when autostashing with staged entries and `stash.index=true`. After several iterations addressing a NULL-dereference risk on hard errors and a t5520 test failure traced to reflog entries lost when `stash apply --index` stopped spawning `git reset`, Junio C Hamano marked the v5 series for `next` on October 1.

### uploadpack.lazyFetchTrusted server-side lazy fetch control
*Long thread: 34 replies so far from 3 participants over 85 days.*
Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config variable, letting server operators mark repositories whose git dir is trusted for lazy fetching during clones, motivated by promisor setups where the server cannot serve filtered clones for blobs moved to a promisor remote. V5, posted October 2, rebased on a018953688 with only comment and test updates, including new tests for `--separate-git-dir` and a NEEDSWORK comment about silently ignoring misspelled allowlist paths.

### New gitmergeconflicts documentation guide
*Long thread: 32 replies so far from 6 participants over 10 days.*
Julia Evans's seven-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from the `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull` man pages. V2 added a "WHAT IS A MERGE CONFLICT?" section explaining three-way merge; reviewers including Patrick Steinhardt and D. Ben Knoble shaped the wording, and Junio C Hamano agreed that `merge.conflictstyle=diff3` should become the default.

### refs: report old values to transaction hooks
*Long thread: 29 replies so far from 5 participants over 13 days.*
Maciej Ciemborowicz's v6 makes a single change in the common transaction layer in `refs.c` so the reference-transaction hook always receives real old object IDs, resolved before the "preparing" hook and refreshed after the backend locks refs. This fixes batched branch, tag, and remote-ref deletions reporting all-zero old OIDs. Ciemborowicz lowered the priority of the underlying bug after resolving his branch and tag deletion issues using prepared-state data, naming branch renaming as the remaining case needing this fix.

### Repack corner cases for cruft-less MIDXs
*Long thread: 27 replies so far from 5 participants over 4 days.*
Taylor Blau's v2 eight-patch series fixes cases where `repack.midxMustContainCruft=false` produces a multi-pack-index not closed under reachability, which can fail bitmap generation. The series uses an oidset for extra roots, traverses kept packs during geometric repacks, and addresses the `--write-midx=incremental` gap. Elijah Newren objected that oidset iteration order can regress pack quality by visiting subtrees first, causing blobs to be written in full instead of delta-compressed.

### format-patch range-diff notes options
*Long thread: 25 replies so far from 4 participants over 42 days.*
Kristoffer Haugsbakk's series teaches `git format-patch` the `--[no-]range-diff-notes` options so the range diff can use a different notes ref than the patches. V5, posted October 4, adopts Junio C Hamano's condensed documentation wording; earlier revisions settled on `--no-range-diff-notes` meaning no range-diff notes at all, implemented via a parse-options callback.

### rerere: wait for MERGE_RR.lock and skip in auto gc
*Long thread: 23 replies so far from 4 participants over 31 days.*
Thomas Bachem's v6 makes every rerere writer wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds a hidden `--skip-locked` option to `git rerere gc` used by `git maintenance run --auto` and `git gc --auto`, and lets conflict-time callers warn and continue via the renamed `RERERE_WARN_LOCKED` flag. Patrick Steinhardt agreed to keep the hidden option after Bachem found that calling `rerere_gc()` directly from `builtin/gc.c` would die with exit code 128 on a held lock.

### git var -z, multiple variables, broken-out idents
*Long thread: 22 replies so far from 6 participants over 39 days.*
Andrew Pleeter's v9 splits the work into four patches: string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, support for multiple variable arguments, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables. Phillip Wood reviewed the series as good with only minor test-style nits, which Pleeter acknowledged.

## The period in brief
The week of September 28 through October 4, 2026 saw seven active days of steady traffic on the Git mailing list. D. Ben Knoble's stash index-merge series reached `next`, Patrick Steinhardt's reftable reflog timezone fix was marked ready, and Junio C Hamano's "What's cooking" noted Git 2.56 has been tagged with the next version numbered 2.98. Readers should not miss the submodule merge corruption bug traced to a stale delta-base cache, the "limbo" object-format proposal for empty repositories, and Coy Geek's report that `repack --dry-run` writes pack files on Git 2.56.0.

## Key developments

### Reftable reflog timezone encoding fixed
Patrick Steinhardt posted a three-patch series fixing `refs/reftable-backend.c`, which stored reflog timezones as signed HHMM integers instead of the specification's signed minute offsets. The series adds `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c`, fixes a `dump-reftable -t` segfault, and adds a t0610 test. Junio C Hamano said the v2 looks really ready and would be marked for `next`.

### Push --force-if-includes fix marked for next
Aleksei Sviridkin's v4 fixes `is_reachable_in_reflog()` leaving an uninitialized timestamp when the remote-tracking ref has no reflog, which on some machines rejected valid pushes with "remote ref updated since checkout". Junio C Hamano said the changes look sensible, would replace the patch and mark it for `next`, and noted trivial t5533 conflicts with another topic.

### Submodule merge corruption traced to delta-base cache
Philippe Blain bisected a superproject merge failure reading a commit from the wrong submodule to bb5da75d61 ("commit: use commit graph in lookup_commit_reference_gently()"). Patrick Steinhardt identified a stale delta-base cache issue -- packfiles are not cleared from the cache when closing -- and posted a two-patch fix; Jeff King characterized the core bug as use-after-free and suggested ASan would find it reliably.

### "Limbo" object-format state for empty repositories
Jeff King proposed letting an empty server-side repository advertise a "limbo" object format so first pushes from either SHA-1 or SHA-256 clients succeed. brian m. carlsson argued `object-format=limbo` would break existing clients; Jeff King proposed a new ignorable capability ("magic-limbo") paired with a valid `object-format=` value, while Junio C Hamano lamented that multiple `object-format` capability lines would have been the right design.

### Repack --dry-run writes packs bug
Coy Geek reported that on Git 2.56.0, `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` writes four new files to the pack directory and, with explicit `-d`, deletes 12 existing files, contradicting the manual's promise that `--dry-run` lists candidates without rebuilding or deleting.

### HTTP credential handling for curl 8.23.0
Jeff King posted a patch because upcoming curl 8.23.0 strips credentials from `CURLINFO_EFFECTIVE_URL`, which would make Git's string comparison falsely detect a redirect and flush credentials. Patrick Steinhardt suggested `CURLINFO_REDIRECT_COUNT`, but Jeff King reported it does not work because curl counts the automatic auth retry as a redirect; Junio C Hamano called the `curl_url()`-based approach "very nice".

## In brief
- **Shallow-repo fetch via refmap**: Harald Nordgren posted v6 of a four-patch series adding `remote.<name>.refmap` and `git remote add --limited-fetch` to avoid fetching every branch of a new remote in shallow repositories; Junio C Hamano flagged an unresolved question about `remote.<name>.fetch` versus `remote.<name>.refmap` precedence, which Nordgren answered by proposing `fetch` should win.

- **Branch --delete-merged squash detection**: Harald Nordgren posted a patch making `git branch --delete-merged` treat squash- and rebase-merged branches as merged; Phillip Wood endorsed the `git cherry` patch-id algorithm as the model and cautioned that rebased branches may evade detection.

- **Sequencer auto maintenance**: Thomas Bachem's v5 series making merge-backend rebase sequences run auto maintenance once at completion was confirmed cooking in `next` by Patrick Steinhardt.

- **Incremental connectivity check**: Kristofer Karlsson posted v2 of `transfer.connectivityCheck=incremental`, running the verifier inside the rev-list subprocess with up to 190x speedup reported for single-commit pushes to large boundary-closure repos.

- **Sparse-checkout clobbering untracked files**: Webstrand reported that `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting, a regression from before 681c637b4a.

- **Merge-ort --no-overwrite-ignore bypass**: Coy Geek reported that `git merge --no-overwrite-ignore` fails to protect an ignored file when directory rename detection moves a committed file onto that path, reproduced on versions 2.54.0 through 2.56.0.

- **History sign rewritten commits**: Souma posted v5 of the series adding `-S`/`--gpg-sign` to `git history` rewrite commands, adding Git completion coverage; Junio C Hamano reported that with the topic merged, `seen` breaks t9902.

- **Refs copy/rename transaction impasse**: Maciej Ciemborowicz's v2 patch representing branch copy/rename as reference transactions hit a review impasse when Patrick Steinhardt declared it not reviewable due to apparently AI-generated content; Ciemborowicz apologized and offered a v3 split into more commits.

- **Filter-branch state-branch fix**: Grant Moyer posted v3 fixing `git filter-branch --state-branch` map initialization, which since f6d855091e interpreted object pairs in reverse; Michele Locati had earlier supplied a Tested-by tag confirming the fix on Git 2.56.0.

- **Optional sha1dc Rust backend**: Johannes Schindelin conceded his "compile-time" option description was misleading since `sha1dc_choose()` selects the backend at runtime; brian m. carlsson objected that the crate requires Rust 1.87 while Git supports older versions.

- **Fetch commit-graph from updated refs**: Kristofer Karlsson posted a series making fetch write the commit-graph from updated refs only; Patrick Steinhardt noted no benchmark was provided.

- **Fetch followRemoteHEAD validation**: Colin Hinton posted v5 deferring `fetch.followRemoteHEAD` validation to `do_fetch()`, storing the raw string with a `follow_remote_head_seen` flag.

- **Post-worktree hook**: Domen Kožar posted v3 adding a `post-worktree` lifecycle hook for add, move, and remove, extended to pruning with remove events.

- **Fsmonitor untracked cache**: Ilia K posted a patch adding `untracked_cache_invalidate_stale_dirs()` to invalidate stale directories after a trivial fsmonitor response.

- **CI annotations linked to test scripts**: Harald Nordgren's two-patch series links GitHub Actions failure annotations to test script file and line; he asked whether the inability to link unchanged files is a regression worth blocking on.

- **Git backfill --dry-run RFC**: Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee proposed `--info=(count|size)` instead.

- **Checkout -m conflict labels**: Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`.

- **Branch hook gaps**: Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook.

- **Documentation updates**: Julia Evans published v2 removing required SYNOPSIS sections from section 7 man pages; Kristoffer Haugsbakk posted v2 rephrasing `trailer.<key-alias>.cmd` examples and was marked for `next`; Junio C Hamano asked that `po/` translations be left intact when removing `gittutorial-2`.

- **Test and CI improvements**: Patrick Steinhardt suggested CI should fail outright when perforce, git-lfs or JGit fail to install; Harald Nordgren posted v4 linking CI failure annotations to test scripts.

- **Miscellaneous**: Maya Barriault reported `git -C other_repo` via alias from a worktree sets `GIT_DIR` incorrectly; Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`; Karthik Nayak's packed-refs change writes unchanged refs with `fwrite()`; Toon Claes reviewed it, listing sanitizations lost; Patrick Steinhardt added subcommand groups to `git refs`; Harald Nordgren's `@{p}` shorthand for `@{push}` was marked queued; Pushkar Singh's v3 swaps default reflog expiry periods; Scott Chacon posted an RFC signing SHA-256 digests of tree contents; SZEDER Gábor noted precompiled header changes to `__FILE__` expansion; Fionn Fitzmaurice patched Zsh completion to exclude duplicate files; Kaartic Sivaraam reviewed Christian Couder's `early_scan_options()` series.