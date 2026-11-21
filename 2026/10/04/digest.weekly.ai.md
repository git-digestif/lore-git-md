## Long threads

### stash: in-core index merge for `stash apply --index`

*Long thread: 53 replies so far from 5 participants over 13 days.*

D. Ben Knoble's series fixes the interaction between autostashing, staged index entries, and `stash.index=true` by replacing the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. The thread traced a t5520.69 failure to reflog entries lost when `stash apply --index` stopped spawning `git reset`, bisected to the auto-maintenance commit 2d1fa0323f. After v4 introduced a NULL-dereference risk on hard errors, v5 fixed the tri-state `merge_result.clean` handling, distinguishing `result.clean < 0` from `!result.clean`. Phillip Wood confirmed the range-diff and Junio C Hamano marked the series for `next` on October 1.

### uploadpack.lazyFetchTrusted server-side lazy fetch control

*Long thread: 34 replies so far from 3 participants over 85 days.*

Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config variable, letting server operators mark repositories whose git dir is trusted for lazy fetching during `upload-pack`, motivated by large-object promisor setups where servers cannot serve filtered clones for blobs moved to promisor remotes. V5, posted October 2, rebased on a018953688 with only comment, documentation, test, and commit message updates, including a NEEDSWORK comment about silently ignoring misspelled allowlist paths and new tests for `--separate-git-dir`. Junio C Hamano had previously called the control logic trivially clear and floated an optional `:(optional)` prefix for `safe.directory` paths.

### New gitmergeconflicts documentation guide

*Long thread: 32 replies so far from 6 participants over 10 days.*

Julia Evans's seven-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-pull`, `git-rebase`, `git-revert`, and `git-cherry-pick`. Reviewers shaped a new "WHAT IS A MERGE CONFLICT?" section for v2, and Patrick Steinhardt proposed extending conflict-marker labels to include commit ID, message, and date, agreeing that `merge.conflictstyle=diff3` should become the default. D. Ben Knoble endorsed the new section, and Junio C Hamano agreed the conservative approach is "more honest" than aggressive merging.

### refs: report old values to transaction hooks

*Long thread: 29 replies so far from 5 participants over 13 days.*

Maciej Ciemborowicz's single-patch v6 resolves old object IDs in the common reference-transaction layer, before the "preparing" hook and again after the backend locks refs, so batched branch, tag, and remote-ref deletions report real old values without constraining updates. Patrick Steinhardt had argued this transaction-layer approach was the proper fix, avoiding interface changes at callers. Ciemborowicz later lowered the priority of the underlying bug after resolving his branch and tag deletion issues using data returned by the transaction in the prepared state, naming branch renaming as the remaining issue.

### Repack corner cases for cruft-less MIDXs

*Long thread: 27 replies so far from 5 participants over 4 days.*

Taylor Blau's v2 eight-patch series fixes corner cases where `repack.midxMustContainCruft=false` can produce a multi-pack-index whose objects are not closed under reachability, which can make bitmap generation fail. The series uses an oidset for extra roots and traverses kept packs during geometric repacks. Elijah Newren objected that the oidset conversion can regress pack quality since iteration order can visit a subtree first and cause blobs to be written in full instead of delta-compressed. Jeff King accepted the double-`prepare_revision_walk()` evidence and the sorting explanation for the preferred-pack marking.

### format-patch range-diff notes options

*Long thread: 25 replies so far from 4 participants over 42 days.*

Kristoffer Haugsbakk's series teaches `git format-patch` the `--[no-]range-diff-notes` options so the range diff can show a different notes ref, or none, than the patches themselves. V5, posted October 4, adopts Junio C Hamano's condensed wording for the documentation paragraph while keeping patch 1/2 unchanged. Hamano had reviewed v4, proposing a shorter replacement for the "why not error out like `--creation-factor`" paragraph and calling the patch otherwise good.

### rerere MERGE_RR.lock waiting

*Long thread: 23 replies so far from 4 participants over 31 days.*

Thomas Bachem's series makes every rerere writer wait `rerere.lockTimeout` for `MERGE_RR.lock` before failing, adds a hidden `--skip-locked` option to `git rerere gc` used by `git maintenance run --auto` and `git gc --auto`, and lets conflict-time callers warn and go on via the renamed `RERERE_WARN_LOCKED` flag. V6, posted October 2, incorporated Patrick Steinhardt's agreement to keep the hidden option after Bachem found that calling `rerere_gc()` directly from `builtin/gc.c` would die with exit code 128 on a held lock.

### shallow-repo fetch of new remotes via refmap

*Long thread: 23 replies so far from 4 participants over 16 days.*

Harald Nordgren's four-patch series avoids fetching every branch of a new remote in a shallow repository by adding `remote.<name>.refmap` as a per-remote default for `--refmap`, inferring branches to fetch from local upstreams, and adding `git remote add --limited-fetch`. V6, posted October 4, removed a leftover reference to deleted default-branch logic. Junio C Hamano flagged that with only patch 1/4 applied, a remote with `remote.origin.refmap` and no tracked branches makes `git fetch origin` die, and Nordgren proposed that `remote.<name>.fetch` should win when both are configured.

## The period in brief

The week of 2026/09/28 through 2026/10/04 saw seven active days of heavy, eventful traffic across the Git mailing list. Readers should not miss that D. Ben Knoble's stash index-merge series was marked for `next`, Patrick Steinhardt's reftable reflog-timezone fix was declared ready, and Junio C Hamano's "What's cooking" reported Git 2.56 tagged with the next version being Git 2.98.

## Key developments

### Reftable reflog timezone encoding fixed

Patrick Steinhardt posted a three-patch series fixing `refs/reftable-backend.c`, which stored reflog timezones in "[+-]HHMM" format instead of the specification's signed minute offset. The series adds `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c`, fixes a `dump-reftable -t` segfault, and adds a t0610 test. Junio C Hamano said the v2 looks really ready and would be marked for `next`.

### Sequencer auto maintenance cooking in `next`

Thomas Bachem's v5 series makes merge-backend rebase sequences run auto maintenance once at completion, like the apply backend, by calling `run_auto_maintenance()` from `builtin/rebase.c` and `builtin/revert.c`. Patrick Steinhardt noted the series is already cooking in `next` and agreed it looks ready.

### push `--force-if-includes` fix marked for `next`

Aleksei Sviridkin's v4 fixes `is_reachable_in_reflog()` initializing the timestamp cutoff to 0 when the remote-tracking ref has no reflog, preventing valid pushes from being rejected with "remote ref updated since checkout". Junio C Hamano said the changes look sensible, will replace the patch, and mark it for `next`.

### Submodule merge delta-base cache bug

Philippe Blain bisected a superproject merge failure reading a commit from the wrong submodule to bb5da75d61 ("commit: use commit graph in lookup_commit_reference_gently()"). Patrick Steinhardt agreed a delta-base cache issue "seems to be the issue indeed", noting packfiles should be cleared from the delta-base cache when closing but currently are not, and posted a two-patch fix.

### http: curl credential stripping fix

Jeff King posted a patch handling upcoming curl 8.23.0 stripping credentials from `CURLINFO_EFFECTIVE_URL`, which would falsely detect redirects and flush `http_auth` credentials. Patrick Steinhardt suggested `CURLINFO_REDIRECT_COUNT`, but King reported it does not work because curl counts automatic auth retries as redirects; Hamano called the `curl_url()`-based `match_effective_url()` approach "very nice".

### "Limbo" object-format state for empty repositories

Jeff King proposed letting an empty server-side repository advertise a "limbo" object format so first pushes from either sha1 or sha256 clients succeed. brian m. carlsson argued `object-format=limbo` would break existing clients, and King proposed a new ignorable capability ("magic-limbo") paired with a valid `object-format=` value. Hamano lamented that multiple `object-format` capability lines would have been the right design.

### repack `--dry-run` writes packs bug

Coy Geek reported that on Git 2.56.0, `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` writes four new files to the pack directory and, with explicit `-d`, deletes 12 existing files, contradicting the manual's promise that `--dry-run` lists candidates without rebuilding or deleting.

## In brief

- **What's cooking**: Junio C Hamano noted Git 2.56 has been tagged and the next version will be Git 2.98, followed by 2.99 in preparation for Git 3.0.
- **merge-ort `--no-overwrite-ignore` bypass**: Coy Geek reported that directory rename detection can move a committed file onto an ignored untracked path, bypassing the guard on versions 2.54.0 through 2.56.0.
- **sparse-checkout clobbering**: Webstrand reported `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting, a regression from before 681c637b4a.
- **history: sign rewritten commits**: Souma posted v5 adding Git completion coverage for `-S`/`--gpg-sign` and `--no-gpg-sign` on history subcommands; Hamano reported `seen` breaks t9902 with the topic merged.
- **refs copy/rename transactions**: Maciej Ciemborowicz's v2 patch was declared not reviewable by Patrick Steinhardt due to apparently AI-generated content; Ciemborowicz apologized and offered a v3 split into more commits.
- **filter-branch state-branch fix**: Grant Moyer posted v3 fixing the commit-map initialization that since f6d855091e interpreted object pairs in reverse, with Michele Locati supplying a Tested-by tag.
- **connected: incremental connectivity check**: Kristofer Karlsson posted v2 of the opt-in `transfer.connectivityCheck=incremental` mode reporting up to 190x speedup for single-commit pushes to large boundary-closure repos.
- **branch `--delete-merged` squash detection**: Harald Nordgren posted a patch making `git branch --delete-merged` treat squash- and rebase-merged branches as merged; Phillip Wood endorsed the `git cherry` patch-id algorithm as the model.
- **Optional sha1dc Rust backend**: Johannes Schindelin conceded "compile-time" is misleading since `sha1dc_choose()` selects at runtime; brian m. carlsson objected that the crate requires Rust 1.87 while Git supports older versions.
- **git var -z and multiple variables**: Andrew Pleeter's v9 splits the work into string_list representation, `-z` output, multiple arguments, and new identity variables; Phillip Wood reviewed it as good with minor test nits.
- **CI annotations linked to test scripts**: Harald Nordgren's series links GitHub Actions failure annotations to test script file and line, but he questioned whether the inability to link unchanged files is a regression worth blocking progress.
- **RFC for git-brebase**: Alejandro Colomar posted a 164-line shell script rebasing onto the first conflicting commit using `git bisect`; Nico Williams preferred a `--first-conflict` option on `git rebase` itself.
- **post-worktree hook**: Domen Kožar posted v3 adding a single `post-worktree` lifecycle hook for add, move, remove, and prune events.
- **fsmonitor untracked cache**: Ilia K posted a patch invalidating stale untracked-cache directories after a trivial fsmonitor response.
- **xdiff size_t consolidation**: Jeff King's v2 consolidates `mmbuffer_t` into `mmfile_t` and makes `merge-ll` report errors when reading external merge results fails.
- **merge-ll tempfile cleanup**: Jeff King's v3 cleans up `.merge_file_*` temporaries on Ctrl+C; Hamano said he would mark it for `next`.
- **Pushkar Singh's reflog expiry swap**: v3 swaps back the default reflog expiry periods in `reflog.h` to reachable 90 days, unreachable 30; Steinhardt suggested testing at 88 days.
- **Kazumasa Shigeta's stash create**: v2 lets `git stash create` accept `-u`/`-a`; Hamano critiqued the commit message's user-visible framing.
- **Phillip Wood's checkout -m labels**: His series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`, with CRLF parsing fixed via `strbuf_getline()`.
- **Patrick Steinhardt's alternates series**: A 13-patch series moves alternates into the "files" object database backend, dropping the `read_alternates` callback.
- **Kristoffer Karlsson's commit-graph fetch**: A series making fetch write the commit-graph from updated refs only; Steinhardt noted no benchmark was provided.
- **Colin Hinton's followRemoteHEAD**: v5 defers `fetch.followRemoteHEAD` validation to `do_fetch()`, storing the raw string with a `follow_remote_head_seen` flag.
- **Julia Evans's doc v2**: Removes required SYNOPSIS sections from section 7 man pages, with Tuomas Ahola's declarative lint-script rewrite.
- **Harald Nordgren's `@{p}`**: Patch makes `@{p}` short for `@{push}`; Hamano marked it queued.
- **Karthik Nayak's packed-refs**: Writes unchanged refs with `fwrite()`, about 20% faster on deletes; Toon Claes listed sanitizations lost by fwrite() and proposed a separate preformatted writer.
- **Patrick Steinhardt's `git refs` subcommands**: Series adds subcommand groups to `git refs`; Hamano called patch 3 "Nice".
- **Pablo Sabater's backfill RFC**: Adds `--dry-run` to `git backfill`; Derrick Stolee questioned the size metric and proposed `--info=(count|size)`.
- **Branch hook gaps**: Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook.
- **Maya Barriault's `git -C` alias bug**: Running `git -C other_repo` via an alias from a worktree sets `GIT_DIR` and `GIT_COMMON_DIR` to the worktree's values.
- **CI install failures**: Steinhardt suggested the CI script should fail outright when perforce, git-lfs or JGit fail to install rather than warn; Harald Nordgren agreed.
- **Fionn Fitzmaurice's Zsh completion**: Excludes files already present in the argument list.
- **Kristoffer Haugsbakk's trailer docs**: v2 rephrases the `trailer.<key-alias>.cmd` examples; Hamano marked the topic for `next`.
- **SZEDER Gábor's precompiled header note**: Precompiled `git-compat-util.h` changes `__FILE__` expansion in error messages, judged acceptable.