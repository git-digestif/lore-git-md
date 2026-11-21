## Long threads

### history: add squash subcommand to fold a range

*Long thread: 91 replies so far from 9 participants over 87 days.*

Harald Nordgren's series adding `git history squash <revision-range>` reached v15 during the month, fixing `istarts_with()` to `starts_with()` so mixed-case hex in `fixup!` subjects is no longer accepted, correcting a `squshed` typo, and adding subcommand completion tests. Elijah Newren and Junio C Hamano reviewed the series extensively, with Hamano noting that the "complete 'git history'" topic now in `master` eliminated the need for an evil merge in the completion test. The series remained in active review at month's end.

### receive-report hook for git-receive-pack

*Long thread: 58 replies so far from 7 participants over 29 days.*

Karthik Nayak's series introducing a `receive-report` hook that runs after the reference transaction but before the report reaches the client progressed through v10, fixing a `BUG()` when no report was requested, a leak in `override_cmds_error()`, and the `UNKOWN` spelling. Patrick Steinhardt approved v7, and Junio C Hamano queued a coccinelle fix on top of v10 removing a redundant nullness check before `FREE_AND_NULL`. The series addresses the gap where no backend supports post-commit rollback, letting the hook's stdout replace the pkt-line report and a non-zero exit reject all references.

### stash: in-core index merge for `stash apply --index`

*Long thread: 51 replies so far from 5 participants over 12 days.*

D. Ben Knoble's series fixing the interaction between autostashing, staged index entries, and `stash.index=true` reached v5, replacing the shell-out `diff-tree`/`apply --cached`/`reset` sequence with an in-core `merge_incore_nonrecursive()` call. The fix resolves a non-re-entrant `save_autostash()` lock on a deleted `MERGE_AUTOSTASH` ref. Reviewers diagnosed transposed tree arguments and a tri-state `merge_result.clean` handling bug, which Knoble fixed in v4 and v5, while pinning `gc.reflogExpire` to `never` in t5520 to prevent auto-maintenance from expiring needed reflog entries.

### Convert build-time USE_NSEC to runtime core.useNanosec

*Long thread: 46 replies so far from 6 participants over 36 days.*

D. Ben Knoble's series converting the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` setting reached v7, which differed from v6 only in a documentation typofix. Junio C Hamano queued the three patches, noting they matched what he had already queued from v6. The series replaces `#ifdef` checks in `read-cache.c` and `statinfo.c` with runtime checks so distributed builds can use nanosecond timestamps to avoid racy-git false positives, with Jeff King clarifying that `NO_NSEC` builds fake nsec entries as "0".

### git-replay(1) `--linearize` option

*Long thread: 39 replies so far from 7 participants over 86 days.*

Toon Claes's series adding a `--linearize` option to `git-replay(1)` that drops merge commits and replays reachable commits into a single linear history received a Reviewed-by from Elijah Newren, who said "I think this version is good to go." Earlier revisions reworked error messages and documentation around the multiple-branches restriction. The series mimics `git rebase --no-rebase-merges` behavior and remained in review at month's end.

### `git repo info` path keys series

*Long thread: 37 replies so far from 5 participants over 74 days.*

K Jayatheerth's GSoC series adding seven `path.*` keys to `git repo info` reached v7, refactoring `get_superproject_working_tree()` to take a `struct repository *` instead of relying on the process working directory, fixing empty `path.superproject-root.absolute` values under `--git-dir`. The series also made `path.cdup` work outside the working tree and split the superproject fix into its own commit. Junio C Hamano noted dead null checks after `repo_get_index_file()` and suggested splitting the fix from the new keys.

### repack: add --drop-filtered to reclaim space in partial clones

*Long thread: 37 replies so far from 5 participants over 51 days.*

This GSoC series letting users reclaim space in partial clones by dropping promisor blobs above a size threshold saw Samuel Bronson validate it on a real llvm-project clone, proposing to skip index-referenced blobs with a warning instead of aborting. Siddharth Shrimali refined the design, proposing a separate skip-set so `--dry-run` still lists those blobs while the drop log excludes them. The series relies on existing lazy-fetch machinery to restore dropped objects.

### writing alternates at object database creation

*Long thread: 36 replies so far from 5 participants over 18 days.*

Patrick Steinhardt's nine-patch series refactoring `init_db()` into `create_repository()` and moving alternate writing into `odb_source_create_on_disk()` reached v5, renaming collection functions and fixing a "refernce" typo. Justin Tobler reviewed v4 as good, and the series removes the ad-hoc `odb_source_write_alternate()` callback, writing `info/alternates` at creation time via a lockfile.

### packfile recovery when a MIDX names a removed pack

*Long thread: 36 replies so far from 5 participants over 15 days.*

Elijah Newren's series fixing objects reported missing despite being present when a stale multi-pack-index names a pack removed by concurrent geometric repack received Derrick Stolee's "Good patch. LGTM." on patch 4/4. The v3 renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return and rescans the MIDX's packs for a surviving copy on second read, with Newren clarifying the fallback scan runs over packs in the MIDX rather than all packs.

### sequencer: run auto maintenance once at the end of a sequence

*Long thread: 33 replies so far from 5 participants over 25 days.*

Thomas Bachem's series making rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes reached v5, moving the call from the sequencer into `builtin/rebase.c` and `builtin/revert.c` while passing `maintenance.auto=false` to spawned commands via `GIT_CONFIG_PARAMETERS`. Patrick Steinhardt confirmed the series is cooking in `next`, and Phillip Wood called it "ready for next". The change avoids lock contention on `MERGE_RR.lock` and prevents repacks from deleting open packs during sequences.

## The period in brief

September 2026 was a heavy, eventful month with traffic on all 30 days, marked by the Git v2.56.0-rc0 and rc1 release candidates and extensive discussion of the post-2.56 version roadmap toward 2.98, 2.99, and eventually 3.0. Readers should not miss the receive-report hook reaching v10 with maintainer approval, the stash in-core merge fix resolving a long-standing autostash bug, and the sequencer auto-maintenance series landing in `next`.

## Key developments

### Git v2.56.0 release candidates and version roadmap

Junio C Hamano announced Git v2.56.0-rc0 on September 10 with 635 non-merge commits since v2.55.0, followed by rc1 on September 16 with 721 commits. The month saw intense discussion of what follows 2.56, with brian m. carlsson, Patrick Steinhardt, and Emily Shaffer weighing GitHub's missing SHA-256 support as the main blocker for a 3.0 bump. Junio proposed a timeline of 2.98 at year's end, 2.99 in March 2027, and 3.0 differing only in enabled breaking changes, cautioning that anything beyond this year is "play it by ear".

### push: --force-if-includes fixes

Tyler Cipriani and Aleksei Sviridkin posted multiple revisions fixing `--force-if-includes` to consult the reflog of the ref actually being pushed rather than a same-named local branch, fixing false rejections and potential data loss. Cipriani's v6 defers reflog verification until after fast-forward rules are evaluated so valid fast-forward pushes with expired reflogs proceed, while Sviridkin's v4 initializes an uninitialized timestamp to 0 when the remote-tracking ref has no reflog. Junio C Hamano marked Sviridkin's fix for `next`.

### repack race with mid-run `.keep` files

Qin ShiCheng (qeesung) posted a series fixing a production-hit race where `git repack -d` deletes a pack whose objects were never copied when a concurrent push installs a `.keep` file between repack's two scans. The fix has repack write its startup snapshot of kept packs to a tempfile and pass it to pack-objects via a new `--keep-pack-from-file=<file>` option instead of `--honor-pack-keep`. Justin Tobler reviewed the receive-pack side, suggesting the ODB transaction create `.keep` files itself at commit time.

### reference-transaction hooks reporting old OIDs

Maciej Ciemborowicz posted v6 of a series fixing the reference-transaction hook receiving all-zero old object IDs for batched branch, tag, and remote-ref deletions. The final approach is a single transaction-layer change in `refs.c` that resolves old values before the "preparing" hook and refreshes them after backend locks, only when a hook exists. Patrick Steinhardt endorsed this common-layer approach over caller-specific changes, leaving `refs_delete_refs()` and callers unchanged.

### uploadpack.lazyFetchTrusted

Christian Couder posted v4 of a series adding a protected `uploadpack.lazyFetchTrusted` config variable letting server operators mark repos trusted for lazy fetching during `upload-pack`, motivated by large-object promisor setups where servers cannot serve filtered clones for blobs moved to promisor remotes. The series adds `GIT_INTERNAL_LAZY_FETCH_DEPTH` (max 5) to stop infinite recursion when a promisor remote resolves back to the served repo, and checks `GIT_NO_LAZY_FETCH` only after entering the repo.

### unifying ref storage format terminology

Patrick Steinhardt's 13-patch v3 series renames inconsistent interfaces -- `--ref-format=`, `GIT_REFERENCE_BACKEND`, `init.defaultRefFormat` -- to `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases and accepting `format://payload` URIs. Junio C Hamano found the series in very good shape, while Kaartic Sivaraam noted existing deprecated aliases are not hidden, prompting discussion of `OPT_HIDDEN` in the deprecation lifecycle.

### rerere lock timeout handling

Thomas Bachem's v5 series makes rerere writers wait `rerere.lockTimeout` (default 1000 ms) for `MERGE_RR.lock` before failing, adds `git rerere gc --auto` that quietly skips a held lock, and lets conflict-time callers warn and continue. The series addresses background `git rerere gc` from auto-maintenance killing rebases, with Patrick Steinhardt questioning the `--auto` name and suggesting `--skip-locked`.

### shallow clone push optimization

Elijah Newren posted v3 of a six-patch series keeping `send-pack` from transferring the whole tree when pushing from a shallow clone, introducing a tri-state `push.shallowExcludeBoundary` option and new push advice. Derrick Stolee reported a multi-ref flaw where shallow grafts are treated as reachable without checking advertised refs, which Newren argued is not corruption since receive-pack checks connectivity regardless.

## In brief

**Git for Windows and Rust in CI** -- Johannes Schindelin posted v4 of a 13-patch series upstreaming Git for Windows patches for the UCRT64 switch, which Junio C Hamano marked for `next`, and a series enabling Rust in Windows CI jobs by selecting a GCC-compatible Cargo target, which Hamano fast-tracked to `master`. Schindelin also announced Git for Windows 2.56.0-rc0 dropping Windows 8.1 support.

**Outreachy December 2026 cohort** -- Christian Couder submitted Git's Community Application for Outreachy's December 2026 cohort, proposing projects on removing global state and improving argument scanning, with Usman, Kaartic and Pablo volunteering as co-mentors.

**reftable reflog timezone fix** -- Patrick Steinhardt posted a three-patch series fixing `refs/reftable-backend.c` storing reflog timezones as signed HHMM integers instead of the spec's signed minute offsets, adding `tz_to_minutes()`/`minutes_to_tz()` helpers and a t0610 test.

**incremental connectivity check** -- Kristofer Karlsson posted v2 of an opt-in `transfer.connectivityCheck=incremental` mode running the verifier inside the rev-list subprocess, reporting up to 190x speedup and 14 MB RSS versus 3.4 GB for single-commit pushes to a large boundary-closure repo.

**pull segfault fix** -- Jiri Kuncar posted v2 adding NULL guards for `lookup_commit_reference()` in `builtin/pull.c` so invalid merge-head references fall back to a normal merge that reports the broken object instead of crashing.

**worktree repair validation** -- Yoichi NAKAYAMA submitted a two-patch series fixing `git worktree repair` to validate cross-references between linked worktrees and administrative data, preventing corruption when worktree directories are swapped.

**ODB transaction packfile flush** -- Justin Tobler posted v2 fixing a commit failure when an ODB transaction contains both a loose object and a large blob with `core.fsyncMethod=batch`, moving the packfile flush ahead of object migration.

**reflog expiry defaults swapped** -- Pushkar Singh posted v3 fixing swapped default reflog expiry periods in `reflog.h` (reachable 90 days, unreachable 30), tracing the flip to commit 8565827570 in Git 2.50.0.

**http.sslVerifyStatus OCSP** -- Grayson Gordon's patch adding `http.sslVerifyStatus` for stapled OCSP responses was reverted from `next` after SZEDER Gabor reported t5585 failures traced to a GnuTLS curl backend without OCSP support.

**advice scoping** -- Vsevolod Myalitsin's series and Junio C Hamano's v5 patch add `--global` to advice squelch hints so `advice.defaultBranchName` suggests the correct scope.

**early_scan_options API** -- Christian Couder posted v2 adding an `early_scan_options()` sub-API to parse-options, fixing `--` detection in bisect and `--allow-unsafe-features` in fast-import.

**git var extensions** -- Andrew Pleeter posted v9 extending `git var` with broken-down ident components, multiple variables per call, and `-z` output, dropping `GIT_SIGNING_KEY` due to `gpg.format` dependency.

**git-p4 shell injection fix** -- Anupam Mediratta posted v2 replacing the shell-based `git diff-tree | git apply` pipeline in `P4Submit.applyCommit()` with argument-vector subprocess calls, preventing execution of shell metacharacters in `--commit` values.

**rebase --amend regression** -- Patrick Steinhardt posted a fix for a regression where `git commit --amend` refusal persisted after conflict resolution, with discussion converging on reverting commit 6257588252 for Git 2.56.

**submodule merge repository corruption** -- Philippe Blain bisected a superproject merge reading commits from the wrong submodule to commit bb5da75d61 using commit-graph in `lookup_commit_reference_gently()`, with the fix in `next` but not master.

**sparse-checkout clobbering** -- Webstrand reported `git checkout` with sparse-checkout clobbering untracked in-cone files instead of aborting, a regression from before 681c637b4a.

**cruft-less MIDX corner cases** -- Taylor Blau posted four fixes for cruft-less MIDX reachability corner cases that can fail bitmap generation.

**documentation updates** -- Julia Evans posted a seven-patch series adding `gitmergeconflicts(7)` and announced plans to rewrite `gittutorial.adoc`; Kristoffer Haugsbakk posted v2 of `format-patch --[no-]range-diff-notes`; Jean-Noël Avila and others continued man page conversions.

**CI and build improvements** -- SZEDER Gabor posted v2 precompiling `git-compat-util.h` for ~35% build speedup; Jeff King proposed bumping the debian-11 CI job to debian-12; Johannes Schindelin fixed GitLab CI Windows cargo invocations.

**miscellaneous fixes** -- Jeff King fixed a use-after-free in `setup_revisions()` garbling `git stash show` output and two `parse_revision_opt()` bugs reporting unknown options as `(null)`; Yannik Tausch fixed an out-of-bounds read with negative pathspecs; Royce Remer fixed upload-pack sending wanted-ref info before shallow-info; René Scharfe fixed excluded directories with nested repos; Harald Nordgren added `fetch.shallow` config and `shallowHistory` advice; Maciej Ciemborowicz routed ref copy/rename through the transaction API.