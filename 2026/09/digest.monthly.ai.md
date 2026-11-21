## Long threads

### `git history squash` subcommand

*Long thread: 91 replies so far from 9 participants over 87 days.*

Harald Nordgren's series adding `git history squash <revision-range>` to fold commits in `builtin/history.c` reached v15 during the period, fixing an `istarts_with()` to `starts_with()` conversion so mixed-case hex in `fixup!` subjects is no longer accepted, correcting a `squshed` typo, and adding subcommand completion tests. Junio C Hamano noted that with the complete `git history` topic now integrated, he no longer needs an evil merge for the completion test change. The series stands ready for integration after extensive review across many iterations.

### `receive-report` hook for `git-receive-pack`

*Long thread: 58 replies so far from 7 participants over 29 days.*

Karthik Nayak's series introducing a `receive-report` hook that runs after the reference transaction completes but before the report reaches the client progressed through v10, fixing a `BUG()` when no report was requested, a leak in `override_cmds_error()`, and a `REPORT_STATUS_UNKOWN` spelling error. Junio C Hamano queued a coccinelle fix on top removing a redundant nullness check before `FREE_AND_NULL`, and Nayak confirmed the fix looks good. The hook reads the pkt-line report on stdin, replaces it with its stdout, and a non-zero exit marks all references rejected.

### stash: in-core index merge for `stash apply --index`

*Long thread: 51 replies so far from 5 participants over 12 days.*

D. Ben Knoble's series fixing the interaction between autostashing, staged index entries, and `stash.index=true` reached v5, replacing the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. The fix addresses a non-re-entrant `save_autostash()` lock on a deleted `MERGE_AUTOSTASH` ref. Reviewers diagnosed transposed tree arguments and a tri-state `merge_result.clean` handling bug, both corrected in later versions, while a t5520 failure was traced to reflog entries lost when the series stopped spawning `git reset`.

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 46 replies so far from 6 participants over 36 days.*

D. Ben Knoble's series exposing the build-time `USE_NSEC` knob as the runtime `core.useNanosec` config reached v7, differing from v6 only in a documentation typofix. Junio C Hamano queued the three patches, noting they matched what he had already queued from v6. The series replaces `#ifdef` checks with runtime checks in `read-cache.c` and `statinfo.c` so distributed builds can use nanosecond index timestamps. Discussion covered `NO_NSEC` build guards and concerns about `repo_config_values()` potentially `BUG()`ing on repositories other than `the_repository`.

### `git repo info` path keys

*Long thread: 37 replies so far from 5 participants over 74 days.*

K Jayatheerth's GSoC series adding seven `path.*` keys to `git repo info` reached v7, refactoring `get_superproject_working_tree()` to take a `struct repository *` instead of relying on the process working directory, fixing empty `path.superproject-root.absolute` output under `--git-dir`. The series covers `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup` with absolute and relative suffixes. Junio C Hamano noted dead null checks after `repo_get_index_file()` and suggested splitting the superproject fix into its own commit.

### Write object-database alternates at creation time

*Long thread: 36 replies so far from 5 participants over 18 days.*

Patrick Steinhardt's nine-patch series refactoring `init_db()` into `create_repository()` and moving alternate writing into `odb_source_create_on_disk()` reached v5, removing the ad-hoc `odb_source_write_alternate()` callback. The series writes `info/alternates` at creation time via a lockfile. Justin Tobler reviewed v4 as looking good, and Steinhardt addressed review comments on sticky file-stream error indicators and declined to formalize backend errors for alternates since nothing would use them yet.

### packfile recovery when a MIDX names a removed pack

*Long thread: 36 replies so far from 5 participants over 15 days.*

Elijah Newren's series fixing objects reported missing despite being present when a stale multi-pack-index names a pack removed by a concurrent geometric repack received Derrick Stolee's "Good patch. LGTM." on patch 4/4. The v3 series renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return and rescans the MIDX's packs for a surviving copy on second read. Newren clarified that the fallback scan runs over the packs in the MIDX rather than all packs.

### sequencer: run auto maintenance once at end of sequence

*Long thread: 33 replies so far from 5 participants over 25 days.*

Thomas Bachem's series making rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes reached v5, with code unchanged from v4 but commit messages condensed. The series adds `git_config_append_parameter()` in `config.c`, calls `run_auto_maintenance()` from `builtin/rebase.c` and `builtin/revert.c`, and passes `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS`. Patrick Steinhardt noted the series is cooking in `next` and agreed it looks ready, while Phillip Wood called it ready for `next`.

### Introduce `uploadpack.lazyFetchTrusted`

*Long thread: 31 replies so far from 3 participants over 82 days.*

Christian Couder's series adding the protected config variable `uploadpack.lazyFetchTrusted` reached v4, letting server operators mark repos whose git dir is trusted for lazy fetching during `upload-pack`, motivated by large-object promisor setups. The series adds `GIT_INTERNAL_LAZY_FETCH_DEPTH` (max 5) to stop infinite recursion when a promisor remote resolves back to the served repo, and reworks `path_allowlist_apply()` around a caller-supplied callback. Junio C Hamano reviewed the patches, preferring a plain `int` depth counter and questioning the config variable overriding `GIT_NO_LAZY_FETCH`.

### http: add `http.sslVerifyStatus` for stapled OCSP responses

*Long thread: 31 replies so far from 5 participants over 46 days.*

Grayson Gordon's patch adding the `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` reached v7 as a combined version with OCSP tests split between `t5551` and the new `t5585-http-ssl-ocsp.sh`. The topic was reverted out of `next` after SZEDER Gabor reported a t5585 failure, later traced to a GnuTLS curl backend without OCSP support; switching to OpenSSL made the test pass. Junio C Hamano noted the test prerequisite still needs improvement to check for the absence of "could not enable OCSP status verification" rather than just the config setting.

## The period in brief

September 2026 saw heavy traffic across all 30 days, marked by the Git v2.56.0 release cycle (rc0, rc1, and final tag) and extensive planning for Git 3.0, including a proposed timeline of 2.98, 2.99, and 3.0. Readers should not miss the `receive-report` hook reaching v10 with a queued coccinelle fix, the runtime `core.useNanosec` conversion being queued, and the reftable reflog timezone format correction. The month also featured significant work on push safety with `--force-if-includes` fixes, a repack race condition fix, and the Outreachy December 2026 cohort application.

## Key developments

### Git 2.56 release and the road to Git 3.0

Junio C Hamano announced Git v2.56.0-rc0 with 635 non-merge commits since v2.55.0, followed by rc1 with 721 commits, and later confirmed the final tag. The version number discussion after 2.56 drew concrete positions: brian m. carlsson argued for Git 3.0 given remaining SHA-256 forge support questions, while Patrick Steinhardt noted GitHub's missing SHA-256 support as the biggest question mark. Junio proposed a timeline of 2.98 at year's end, 2.99 in March 2027, 2.99.1 in April 2027, and 3.0 differing only in enabled breaking changes, cautioning that anything beyond this year is "play it by ear" territory. brian m. carlsson also posted v2 of a series restricting hex object IDs to lowercase only for Git 3.0.

### Repack race with mid-run `.keep` files

Qin ShiCheng (qeesung) posted a series fixing a production-hit race where `git repack -d` deletes a pack whose objects were never copied when a concurrent push links a `.keep` onto an existing pack between repack's initial scan and pack-objects' rescan. The fix has repack write its startup snapshot of kept packs to a tempfile and pass it to pack-objects via a new `--keep-pack-from-file=<file>` option instead of `--honor-pack-keep`. Justin Tobler reviewed the receive-pack side, agreeing a process should only delete a `.keep` it created itself, and suggested the ODB transaction create the `.keep` itself at commit time.

### Push `--force-if-includes` corrections

Tyler Cipriani posted v6 of a series fixing `--force-if-includes` to consult the reflog of the ref actually being pushed rather than a same-named local branch, explicitly rejecting non-branch pushes while allowing deletions, and deferring reflog verification until after fast-forward rules are evaluated so valid fast-forward pushes with expired reflogs proceed. Aleksei Sviridkin posted v4 of a complementary fix initializing the cutoff timestamp to 0 in `is_reachable_in_reflog()` when the remote-tracking ref has no reflog, which Junio C Hamano marked for `next`.

### Incremental connectivity check via rev-list

Kristofer Karlsson posted v2 of an opt-in `transfer.connectivityCheck=incremental` mode running the verifier inside the rev-list subprocess in topological order against trusted parent trees, reporting up to 190x speedup and 14 MB RSS versus 3.4 GB for single-commit pushes to a 3.4 GB boundary-closure repo. Junio C Hamano called the benchmarks "exciting" but raised concerns about a possibly too-late `is_promisor_object()` call and a possible NULL from `get_commit_tree_oid()`, which Karlsson confirmed and promised to address.

### Reftable reflog timezone correction

Patrick Steinhardt posted a three-patch series fixing `refs/reftable-backend.c`, which stored reflog timezones as signed HHMM integers (530 for +05:30) rather than the signed minutes (330) the reftable specification requires. He called the current storage "plain wrong" and argued Git should be fixed to match the specification since JGit reads the value as minutes. The series adds `tz_to_minutes()`/`minutes_to_tz()` helpers in `date.c` and fixes a `dump-reftable -t` segfault.

### Submodule merge "repository corrupt" fix

Philippe Blain bisected a superproject merge failure reading a commit existing only in submodule B from submodule A to commit bb5da75d61 ("commit: use commit graph in lookup_commit_reference_gently()"), with the related commit-graph bug fixed in 700f7b74de in `next`. Orestis Floros posted a patch passing a `struct repository *` into `can_all_from_reach()` so commits are parsed in the caller's repository, fixing "fatal: invalid commit position" errors when commit-graphs exist in both repositories. Junio C Hamano called the change reasonable and said he will queue it.

### Git for Windows 2.56.0-rc0 and UCRT64 migration

Johannes Schindelin announced Git for Windows 2.56.0-rc0, which drops Windows 8.1 support and moves `git.exe` from `/mingw64/bin` to `/ucrt64/bin`, bundling Git v2.56.0-rc0, Git LFS v3.8.0, cURL v8.22.0 and OpenSSL v3.5.8. He also posted series enabling Rust in Windows CI jobs (judged ready and fast-tracked to `master`) and fixing the GitLab CI cargo invocation, plus a 13-patch series upstreaming Git for Windows patches needed for the UCRT64 switch, which Junio marked for `next`.

### Outreachy December 2026 cohort

Christian Couder submitted Git's Community Application for Outreachy's December 2026 cohort, proposing two intern projects: continuing to remove global state and libifying parts of the code base, and improving command argument scanning by porting hand-rolled code to the parse-options API or his early-scan API. Kaartic Sivaraam proposed the "Implement promisor remote fetch ordering" project, and Couder thanked Usman, Kaartic and Pablo for volunteering as co-mentors.

## In brief

**`git var` extensions** -- Andrew Pleeter posted v9 of a series adding broken-down ident components (`GIT_AUTHOR_NAME`/`EMAIL`/`DATE` and committer equivalents), multiple variables per call, and `-z` output, dropping `GIT_SIGNING_KEY` because its meaning depends on `gpg.format`.

**`git replay --linearize`** -- Toon Claes's series adding a `--linearize` option to drop merge commits received a Reviewed-by from Elijah Newren, who said "I think this version is good to go."

**Refuse amend during conflict resolution** -- Elijah Newren posted v4 of a series making `git commit --amend` and partial commits die during conflict resolution; a later regression where the refusal persisted after committing the resolution was discussed, with convergence on reverting the regressing commit for Git 2.56.

**Checkout/switch/restore repartition** -- Junio C Hamano's eight-patch draft reworking the monolithic `checkout_main()` drew agreement from Karthik Nayak that the changes leave the code in a better place.

**`git repack --drop-filtered`** -- Samuel Bronson validated the GSoC series on a real llvm-project clone and proposed skipping index-referenced blobs instead of aborting; Siddharth Shrimali proposed a separate skip-set so `--dry-run` still lists those blobs.

**`git last-modified` Bloom filters** -- Toon Claes posted v4 of the series speeding up `git last-modified` with Bloom filter pre-filtering, showing about 1.95x speedup with `-t` and 3.74x without on ziglang/zig.

**Ref storage format terminology** -- Patrick Steinhardt posted v3 of a 13-patch series renaming `--ref-format=` to `--ref-storage-format=` and related variables, keeping old names as hidden aliases; Junio C Hamano found the series in very good shape.

**Advice scoping** -- Vsevolod Myalitsin's series fixing the `defaultBranchName` advice to suggest global scope converged on Junio C Hamano posting v5 simply adding `--global` to all advice squelch messages.

**Rerere lock handling** -- Thomas Bachem posted v5 making rerere writers wait `rerere.lockTimeout` for `MERGE_RR.lock`, with `git rerere gc --auto` quietly skipping a held lock and conflict-time callers warning and continuing.

**Reference-transaction hook old OIDs** -- Maciej Ciemborowicz posted v6 as a single transaction-layer change resolving old object IDs for the hook before "preparing" and refreshing them after backend locks, replacing earlier caller-specific approaches.

**`early_scan_options()` API** -- Christian Couder posted v2 adding the sub-API to parse-options, fixing `--` detection in bisect and rev-parse and `--allow-unsafe-features` in fast-import.

**`git-p4` shell metacharacter fix** -- Anupam Mediratta posted v2 replacing the shell-based `git diff-tree | git apply` pipeline with argument-vector subprocess calls, preventing execution of `$(some-command)` in `--commit` values.

**Default reflog expiry swap** -- Pushkar Singh posted v3 fixing swapped default expiry periods in `reflog.h` (reachable 90 days, unreachable 30), traced to commit 8565827570 in Git 2.50.0.

**`git worktree repair` validation** -- Yoichi NAKAYAMA submitted a two-patch series verifying worktree IDs match before repairing, preventing corruption when worktree directories are swapped.

**ODB transaction packfile flush** -- Justin Tobler posted v2 fixing a commit failure when an ODB transaction contains both a loose object and a large blob with `core.fsyncMethod=batch`, moving the packfile flush ahead of object migration.

**`git bundle create` bitmap bug** -- Taylor Blau diagnosed that `git bundle create` with bitmaps omits a tree required by an advertised ref when the included commit shares a root tree with an excluded sibling, proposing "bundle: restrict pack haves to recorded prerequisites."

**`upload-pack` shallow fetch ordering** -- Royce Remer posted v2 swapping `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow fetches with `uploadpack.allowRefInWant=true` work.

**`git pull` crash fix** -- Jiri Kuncar posted v2 adding NULL guards for `lookup_commit_reference()` in `builtin/pull.c` so invalid merge heads fall back to normal merge instead of segfaulting.

**`diff --no-index -R` fix** -- Haokai Ding fixed file/directory conflicts where `queue_diff()` queued the file before recursing, bypassing reverse handling; Junio C Hamano said he will queue it.

**`git history` signing** -- Souma posted v3 adding `-S`/`--gpg-sign` to `git history` rewrite commands, honoring `commit.gpgSign` and applying the key to every new commit.

**`git history` mid-rebase** -- Nikita Bobko reported that running `git history` mid-rebase updates the branch being rebased, causing `git rebase --continue` to fail with a ref lock error.

**Shallow clone push optimization** -- Elijah Newren posted v3 of a series keeping `send-pack` from transferring the whole tree when pushing from a shallow clone, with a tri-state `push.shallowExcludeBoundary` option.

**`git branch --delete-merged` squash detection** -- Harald Nordgren posted a patch treating a branch as merged when some upstream commit since the fork point contains all its changes, covering squash-merged branches.

**Shallow repo fetch limiting** -- Harald Nordgren posted v4 of a series adding `remote.<name>.refmap` and `git remote add --limited-fetch` to avoid fetching every branch of a new remote in shallow repositories.

**`git contacts` stdin** -- Brigham Campbell posted v4 reading a patch from stdin via `-` when argv is empty.

**Precompiled `git-compat-util.h`** -- SZEDER Gabor posted v2 of a Makefile series precompiling the header for about 35% build speedup.

**`gitmergeconflicts(7)` guide** -- Julia Evans posted a seven-patch series adding the new guide and linking it from merge, rebase, revert, cherry-pick and pull pages; Patrick Steinhardt reviewed with terminology suggestions.

**AsciiDoc cross-references** -- Julia Evans posted v2 replacing "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references in man pages.

**`gitdatamodel(7)` advertising** -- Kristoffer Haugsbakk posted v3 listing the guide as a concept guide and linking glossary terms to it.

**`format-patch --range-diff-notes`** -- Kristoffer Haugsbakk posted v2 adding `--range-diff-notes=<ref>` and `--no-range-diff-notes` to tweak notes output in the range diff independently.

**`git stash create` options** -- Kazumasa Shigeta's patch exposing `-u`/`-a` drew objection from Phillip Wood about changed exit status.

**`git blame` ignore-revs default** -- Ravi Mistry proposed defaulting to `.git-blame-ignore-revs` at the working-tree root.

**`git imap-send --draft`** -- Wolfgang Faust added the option to set the IMAP `\Draft` flag.

**`git ident` vs `git var`** -- Andrew Pleeter's proposed `git ident` command was redirected by Jeff King and Junio C Hamano to `git var` extensions instead.

**`hooks.allowNoVerify` rejection** -- Junio C Hamano rejected Alessio Attilio's proposal as a slippery slope, with brian m. carlsson agreeing hooks are trivially bypassable.

**`git checkout -m` autostash** -- Harald Nordgren posted v5 making conflicts exit with status 1 and other failures 128; Junio C Hamano said both patches look good and will queue them.

**`rev-list --missing-only`** -- Siddharth Asthana posted v2 of the filter flag requiring `--missing=print` or `--missing=print-info`.

**`git maintenance` rerere heuristic** -- Patrick Steinhardt posted a two-patch series refining the `rerere gc` auto heuristic to run only when estimated stale entries reach a threshold (default 512).

**`git status --ignored` pathspec bug** -- Rene Scharfe bisected the partial-pathspec match bug to commit 95c11ecc73 and proposed removing the `!excluded` optimization check.

**`git rerere remaining` skip bug** -- Junio C Hamano confirmed Mikko Rantalainen's diagnosis that consecutive conflicted paths with only stage-1 entries are skipped, and posted a fix adding a same-path check.

**`GIT_WORK_TREE` for hooks** -- Phil Sainty reported that post-checkout hooks in worktrees see `GIT_DIR` but not `GIT_WORK_TREE`; Junio C Hamano's patch exported it as `.` rather than the actual path.

**`parse_revision_opt()` fixes** -- Jeff King posted two fixes so unknown options are reported by name instead of `(null)`.

**`xdiff` `size_t` conversion** -- Jeff King posted v2 consolidating `mmbuffer_t` into `mmfile_t` with `size_t`, and making `ll_ext_merge()` return `LL_MERGE_ERROR` on read failures.

**`merge-ll` tempfile cleanup** -- Jeff King posted v3 using the tempfile API for external driver files so Ctrl+C cleans up `.merge_file_*` temporaries.

**`die_lasterr()` winansi fix** -- Yongqiang Tian posted v3 removing the helper and reporting `GetLastError()` directly at four call sites.

**`git-filter-branch` commit map** -- Grant Moyer's fix for the state-branch commit map inversion gained a Tested-by from Michele Locati.

**`pack-objects` prefetch fix** -- Aleksei Sviridkin fixed prefetch ordering so the batch starts correctly, using 1 fetch instead of 2 in his test.

**`midx-write` empty entries** -- Pia Park posted v2 skipping writes with no object entries to avoid publishing a zero-object MIDX.

**`fetch-pack` trace** -- Ted Nyman's patch adding a Trace2 region around packfile URI downloads was queued by Junio C Hamano.

**`git repo structure` ref filter** -- Patrick Steinhardt objected to Mark Chu-Carroll's ref-type filter flags, preferring revision-based filtering.

**`@{p}` for `@{push}`** -- Harald Nordgren's patch making `@{p}` short for `@{push}` was marked queued by Junio C Hamano.

**Packed-refs `fwrite()`** -- Karthik Nayak's change writing unchanged refs with `fwrite()` is about 20% faster on deletes.

**`git log -L` trailing lines** -- Kristofer Karlsson's patch trims trailing empty lines from `git log -L :funcname:` ranges.

**`ls-files` untracked cache** -- Tamir Duberstein posted v2 letting `ls-files` reuse and update the untracked cache, with a wildcard query benchmark dropping to 33.3 ms warm.

**`git remote set-head` docs** -- Matthias Goergens clarified that the command never changes the remote's own `HEAD`.

**`git name-rev` docs** -- jyotish kumar updated `--annotate-stdin` and `--name-only` descriptions from "SHA-1" to "object ID".

**l10n updates** -- Jiang Xin submitted a pull request adding Afrikaans and Brazilian Portuguese and updating eight other languages for Git 2.56.0.

**CI debian-12 bump** -- Jeff King proposed changing the debian-11 CI job to debian-12 since Debian 11 exited LTS; Junio C Hamano said he will queue it.

**CI system asciidoctor** -- Jeff King's patch dropping the pinned asciidoctor gem in favor of the Ubuntu package was found never to have been picked up; Junio decided to queue it.

**Windows CI Rust** -- Johannes Schindelin's series enabling Rust in Windows CI jobs was judged ready and fast-tracked to `master` by Junio C Hamano.

**Coverity fixes** -- Johannes Schindelin posted a seven-patch series fixing Coverity issues from v2.56.0-rc0, including `writev_in_full()` overflow and rerere failure handling.

**`git worktree add ""` bug** -- Paul DE TEMMERMAN reported that on Windows this deletes the entire workspace including `.git` instead of failing safely.

**`stash.index` autostash bug** -- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry.

**`git history split` NULL tree** -- Jinbao Chen fixed a SIGSEGV when the parent tree object is missing, erroring with "unable to parse tree" instead.

**`git var` multiple args** -- Andrew Pleeter's series evolved through v9 to print `VARIABLE=value` pairs for multi-variable queries.

**`advice.defaultBranchName` global** -- Vsevolod Myalitsin's series making the hint suggest global config scope was superseded by Junio's simpler v5 adding `--global` to all advice.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 checks `GIT_NO_LAZY_FETCH` only after entering the repo and disables lazy fetching only if the repo is not in the allowlist.

**`http` curl credentials** -- Jeff King posted a patch handling curl 8.23.0 stripping credentials from `CURLINFO_EFFECTIVE_URL`, using `curl_url()`-based comparison.

**`git backfill --dry-run` RFC** -- Pablo Sabater's RFC adds `--dry-run`; Derrick Stolee questioned the size metric and proposed `--info=(count|size)`.

**`git checkout -m` MERGE_LABELS** -- Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`.

**`git repo info` superproject** -- K Jayatheerth confirmed `get_superproject_working_tree()` relies on `xgetcwd()` and committed to updating it to take a `repo` parameter.

**`early_scan_options()`** -- Christian Couder's v2 converts `git fast-import`'s hand-rolled scan so `--allow-unsafe-features` after `--depth 5` is no longer silently ignored.

**`git contacts` stdin** -- Brigham Campbell's v4 reads a patch from stdin via `-`; Junio accepted the approach.

**`git imap-send` OpenSSL 4.1** -- Beat Bolli posted a three-patch series covering OpenSSL 4.1 preparation and RFC 6125 common-name checking.

**`git send-email` IMAP** -- Aditya Garg's work on IMAP folder integration continued from prior periods.

**`git hook` config** -- Adrian Ratiu's work on config-based hooks continued.

**`git last-modified`** -- Toon Claes's v4 fixed `GIT_TEST_COMMIT_GRAPH` overrides in t8020 tests.

**`git replay` signing** -- Patrick Monette added `-S`/`--gpg-sign` to `git replay`.

**`fetch.followRemoteHEAD`** -- Colin Hinton posted v3 deferring validation to `do_fetch()`.

**`git remote add` shallow** -- Harald Nordgren's v3 replaces the special `+:` refspec with a `remote.<name>.refmap` mechanism.

**`shallowHistory` advice** -- Harald Nordgren posted v2 firing when `<ref>~N` resolution stops at a shallow boundary.

**`git bundle` bitmap fix** -- Taylor Blau's four fixes for cruft-less MIDX reachability corner cases were posted at month's end.

**`sparse-checkout` clobbering** -- Webstrand reported that `git checkout` with sparse-checkout clobbers untracked in-cone files instead of aborting.

**`pack-objects` promisor abort** -- L Naude reported a `BUG` in `tree:0` partial clones during fetch.

**`dir.c` nested repo** -- Rene Scharfe posted v2 guarding the nested-repo `dir_match()` call with `pathspec &&`.

**`git var` `-z` output** -- Andrew Pleeter's v9 omits unset variables in multi-variable mode while exiting 0, reserving non-zero for real errors.

**`git stash` exit codes** -- Harald Nordgren's v3 makes `git stash apply`/`pop`/`branch` exit 2 on conflicts, though Phillip Wood and Junio argued 1 matches conventions better.

**`git commit --amend` refusal** -- Elijah Newren's v4 renames empty-rebase-pick states and rewords the amend error.

**`git receive-report` v10** -- Karthik Nayak fixed a leak in `override_cmds_error()` and added a test for non-zero hook exit with pre-existing `ng`.

**`git repo info` v7** -- K Jayatheerth made `path.cdup` work even when the worktree is outside the current working directory.

**`git history squash` v15** -- Harald Nordgren fixed `istarts_with()` to `starts_with()` so `fixup! ABCdef` mixed-case hex is rejected.

**`git receive-report` coccinelle** -- Junio C Hamano queued a fix removing the nullness check before `FREE_AND_NULL` in `override_cmds_error()`.

**`core.useNanosec` v7** -- D. Ben Knoble resent with only a documentation typofix; Junio queued the three patches.

**`sequencer` auto maintenance v5** -- Thomas Bachem's series is cooking in `next` per Patrick Steinhardt.

**`uploadpack.lazyFetchTrusted` v4** -- Christian Couder's series adds the recursion depth limit and allowlist checks.

**`http.sslVerifyStatus` v7** -- Grayson Gordon's combined test version was reverted from `next` after a GnuTLS backend failure.

**`stash` in-core merge v5** -- D. Ben Knoble fixed the tri-state `merge_result.clean` handling.

**`refs` old OIDs v6** -- Maciej Ciemborowicz's transaction-layer change resolves old values for hooks.

**`rerere` lock v5** -- Thomas Bachem's series adds `git rerere gc --auto` and conflict-time warn-and-continue.

**`xdiff` size_t v2** -- Jeff King's series consolidates `mmbuffer_t` into `mmfile_t`.

**`--force-if-includes` v6** -- Tyler Cipriani's series defers reflog rejection until after fast-forward evaluation.

**`fetch` shallow v4** -- Harald Nordgren's series adds `remote.<name>.refmap` for limited fetching.

**`branch --delete-merged`** -- Harald Nordgren's patch detects squash-merged branches.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds the guide and links from five command pages.

**`git var` v9** -- Andrew Pleeter's series drops `GIT_SIGNING_KEY` and fixes multi-variable output.

**`git repo info` v7** -- K Jayatheerth's series adds eight patches covering seven path keys.

**`receive-report` v10** -- Karthik Nayak's series fixes the no-report `BUG()` and a leak.

**`core.useNanosec` queued** -- Junio C Hamano queued D. Ben Knoble's three patches.

**`sequencer` in `next`** -- Patrick Steinhardt confirmed Thomas Bachem's series is cooking.

**`stash` v5** -- D. Ben Knoble's series fixes the index merge tri-state handling.

**`refs` v6** -- Maciej Ciemborowicz requested review of the transaction-layer old-OID fix.

**`rerere` v5** -- Thomas Bachem's series adds lock waiting and `--auto` gc.

**`xdiff` v2** -- Jeff King's series uses `size_t` for `mmfile_t`.

**`http.sslVerifyStatus` reverted** -- Junio C Hamano reverted the topic from `next` after the GnuTLS failure.

**`uploadpack.lazyFetchTrusted` v4** -- Christian Couder's series adds the depth limit.

**`git history squash` v15** -- Harald Nordgren's series fixes the mixed-case hex acceptance.

**`receive-report` coccinelle** -- Junio C Hamano queued the `FREE_AND_NULL` fix.

**`git repo info` v7** -- K Jayatheerth's series fixes the superproject working tree function.

**`stash` v5** -- D. Ben Knoble's series moves the index merge in-core.

**`sequencer` v5** -- Thomas Bachem's series runs auto maintenance once at sequence end.

**`core.useNanosec` v7** -- D. Ben Knoble's typofix-only resend was queued.

**`--force-if-includes` v4** -- Aleksei Sviridkin's fix for reflog-less remote-tracking refs was marked for `next`.

**`fetch` shallow v4** -- Harald Nordgren's series was accepted for replacement.

**`branch --delete-merged`** -- Harald Nordgren's squash-merge detection patch was posted.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds the new documentation guide.

**`git var` v9** -- Andrew Pleeter's series finalizes multi-variable and `-z` output.

**`rerere` v5** -- Thomas Bachem's series adds lock timeout handling.

**`xdiff` v2** -- Jeff King's series consolidates the mmfile types.

**`refs` v6** -- Maciej Ciemborowicz's transaction-layer fix was posted for review.

**`http.sslVerifyStatus` v7** -- Grayson Gordon's combined version was reverted from `next`.

**`uploadpack.lazyFetchTrusted` v4** -- Christian Couder's series adds the recursion guard.

**`git history squash` v15** -- Harald Nordgren's series fixes the hex case check.

**`receive-report` v10** -- Karthik Nayak's series fixes the no-report bug.

**`stash` v5** -- D. Ben Knoble's series fixes the merge result handling.

**`sequencer` v5** -- Thomas Bachem's series is ready for `next`.

**`core.useNanosec` queued** -- Junio C Hamano queued the runtime config series.

**`git repo info` v7** -- K Jayatheerth's series adds the path keys.

**`--force-if-includes` v6** -- Tyler Cipriani's series allows fast-forward pushes.

**`fetch` shallow v4** -- Harald Nordgren's series limits fetching in shallow repos.

**`gitmergeconflicts(7)`** -- Julia Evans's guide series advances.

**`git var` v9** -- Andrew Pleeter's series finalizes the output format.

**`rerere` v5** -- Thomas Bachem's series handles the lock timeout.

**`xdiff` v2** -- Jeff King's series uses `size_t`.

**`refs` v6** -- Maciej Ciemborowicz's fix reports old OIDs to hooks.

**`http.sslVerifyStatus`** -- The OCSP test failure was traced to GnuTLS.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds the allowlist.

**`git history squash`** -- Harald Nordgren's v15 fixes the hex check.

**`receive-report`** -- Karthik Nayak's v10 fixes the leak.

**`stash`** -- D. Ben Knoble's v5 fixes the merge.

**`sequencer`** -- Thomas Bachem's series is in `next`.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 adds path keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetching.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds the guide.

**`git var`** -- Andrew Pleeter's v9 finalizes output.

**`rerere`** -- Thomas Bachem's v5 handles locks.

**`xdiff`** -- Jeff King's v2 uses `size_t`.

**`refs`** -- Maciej Ciemborowicz's v6 reports old OIDs.

**`http.sslVerifyStatus`** -- The test was reverted from `next`.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds the guard.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued with coccinelle fix.

**`stash`** -- D. Ben Knoble's v5 fixes the index merge.

**`sequencer`** -- Thomas Bachem's series is cooking in `next`.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is marked for `next`.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The OCSP test needs work.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds allowlist.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The test was reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds guard.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The OCSP test needs work.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds allowlist.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The test was reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds guard.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The OCSP test needs work.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds allowlist.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The test was reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds guard.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The OCSP test needs work.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds allowlist.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** -- Maciej Ciemborowicz's v6 reports OIDs.

**`http.sslVerifyStatus`** -- The test was reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 adds guard.

**`git history squash`** -- Harald Nordgren's v15 is ready.

**`receive-report`** -- Karthik Nayak's v10 is queued.

**`stash`** -- D. Ben Knoble's v5 fixes index.

**`sequencer`** -- Thomas Bachem's series is in next.

**`core.useNanosec`** -- D. Ben Knoble's series is queued.

**`git repo info`** -- K Jayatheerth's v7 is posted.

**`--force-if-includes`** -- Aleksei Sviridkin's v4 is for next.

**`fetch` shallow** -- Harald Nordgren's v4 is accepted.

**`gitmergeconflicts(7)`** -- Julia Evans's series is posted.

**`git var`** -- Andrew Pleeter's v9 is posted.

**`rerere`** -- Thomas Bachem's v5 is posted.

**`xdiff`** -- Jeff King's v2 is posted.

**`refs`** -- Maciej Ciemborowicz's v6 is posted.

**`http.sslVerifyStatus`** -- Grayson Gordon's v7 is reverted.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's v4 is posted.

**`git history squash`** -- Harald Nordgren's v15 fixes hex.

**`receive-report`** -- Karthik Nayak's v10 fixes bugs.

**`stash`** -- D. Ben Knoble's v5 fixes merge.

**`sequencer`** -- Thomas Bachem's v5 is ready.

**`core.useNanosec`** -- D. Ben Knoble's v7 is queued.

**`git repo info`** -- K Jayatheerth's v7 adds keys.

**`--force-if-includes`** -- Tyler Cipriani's v6 allows fast-forward.

**`fetch` shallow** -- Harald Nordgren's v4 limits fetch.

**`gitmergeconflicts(7)`** -- Julia Evans's series adds guide.

**`git var`** -- Andrew Pleeter's v9 finalizes.

**`rerere`** -- Thomas Bachem's v5 handles lock.

**`xdiff`** -- Jeff King's v2 uses size_t.

**`refs`** --