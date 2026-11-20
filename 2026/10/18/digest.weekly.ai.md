## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 64 replies so far from 6 participants over 21 days.*
Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from the git-merge, git-rebase, git-revert, git-cherry-pick and git-pull docs. The series progressed through multiple revisions absorbing review feedback from Junio C Hamano and Patrick Steinhardt, including corrected diff3 examples, consistent bullet punctuation, and switching `Reviewed-by` to `Helped-by` trailers. By the end of the week Evans had accepted the final wording fixes and the series stood ready for integration.

### CI housekeeping and modernizations
*Long thread: 39 replies so far from 6 participants over 9 days.*
Patrick Steinhardt's eight-patch series cleans up Git's CI configuration, fixing smaller issues and moving jobs off EOL Docker images, including renaming the "linux-exotic" job to "linux-feature-testbed" and reworking the reftable job. The series reached v4 during the week, addressing Jeff King's review points on the t5004 SHA-256 test and locale handling, and Junio C Hamano marked it for `next`. Discussion also covered resurrecting a no-Rust build job.

### credentialStore.lockTimeout config option
*Long thread: 38 replies so far from 5 participants over 45 days.*
Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, retaining `lockTimeoutMS` as a deprecated alias. Junio C Hamano reviewed the patch, suggesting a loop over both variable names to keep tests in sync and questioning a test case; Ajayi posted v2 incorporating the feedback, wrapping long lines and dropping an invalid test. The topic remained under active review at week's end.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control
*Long thread: 37 replies so far from 3 participants over 96 days.*
Christian Couder's five-patch series introduces the protected `uploadpack.lazyFetchTrusted` configuration variable so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones and fetches. After earlier revisions added recursion protection and documentation fixes, Junio C Hamano marked the topic ready for `next` on October 13, concluding a long-running effort.

### rerere waiting for `MERGE_RR.lock`
*Long thread: 36 replies so far from 5 participants over 43 days.*
Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` and adds a hidden `--skip-locked` option to `git rerere gc`. The series reached v8 during the week, with Patrick Steinhardt calling it "in a good enough state" and agreeing to mark it ready for `next`, resolving a 43-day discussion about lock contention during rebase.

### `branch --delete-merged` detecting squash-merged branches
*Long thread: 32 replies so far from 5 participants over 20 days.*
Harald Nordgren's series teaches `git branch --delete-merged` to treat a branch as merged when some upstream commit contains all of its changes, naming that commit as "landed as", with a `--no-squashed` option to skip the search. The week saw continued debate between Nordgren and Junio C Hamano over whether this belongs in `--delete-merged` or a separate `--delete-irrelevant` option, with Nordgren reporting that a range-diff approach was slower and no more accurate than the current implementation.

### Limited fetch for new remotes in shallow repositories
*Long thread: 31 replies so far from 4 participants over 26 days.*
Harald Nordgren's five-patch series avoids fetching every branch of a new remote in a shallow repo, adding `remote.<name>.refmap`, a `--limited-fetch` option to `git remote add`, and defaulting to it in shallow repositories. The series reached v9 during the week, moving the `do_fetch()` if/else split into the refmap-inference commit after earlier reviews shaped the helper extraction.

### Incremental connectivity check
*Long thread: 27 replies so far from 3 participants over 33 days.*
Kristofer Karlsson's four-patch series adds an incremental mode for the connectivity check in `connected.c`, gated behind `transfer.connectivityCheck=incremental`. Patrick Steinhardt reviewed the documentation and implementation, raising concerns about `die()` calls in library code and counter types; Karlsson benchmarked the approach, finding it up to 2.8x faster on large repos, and proposed a three-mode configuration with a 1000-commit cutoff.

### CI annotations naming the failing test script
*Long thread: 25 replies so far from 4 participants over 19 days.*
Harald Nordgren's series links failure and leak annotations in CI output to the test script, showing the file and line as plain text. After earlier revisions reverted direct file/line linking, Junio C Hamano marked the topic ready for `next` on October 13.

### `checkout -m` recreating original conflict labels
*Long thread: 22 replies so far from 3 participants over 14 days.*
Phillip Wood's three-patch series teaches the merge-ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` can reuse the original merge's labels. The series reached v4 during the week, adding a patch that caches merge config settings and labels once per invocation; Junio C Hamano reviewed it and said he would replace the previous version.

### push: check pushed ref for `--force-if-includes`
*Long thread: 20 replies so far from 5 participants over 45 days.*
Tyler Cipriani's series fixes `--force-if-includes` to consult the reflog of the ref actually being pushed rather than the local branch named after the destination. Cipriani sent v7 rebased on maint, resolving the pushed ref via `ref->peer_ref`, adding a `pushRefUnverifiable` advice, and deferring the reflog check until after fast-forward rules so valid pushes proceed.

### doc: add more AsciiDoc cross-references
*Long thread: 20 replies so far from 5 participants over 27 days.*
Julia Evans's patch updates `Documentation/` to use the verbose `<<EXAMPLES,EXAMPLES>>` cross-reference form for consistent rendering. Jean-Noël AVILA argued for the double-reference form because translated manual pages can require changes incompatible with simply inserting the title, keeping the discussion open at week's end.

### odb/files alternates cleanups
*Long thread: 14 replies so far from 3 participants over 2 days.*
Patrick Steinhardt's six-patch v2 series cleans up the files object database backend after alternates migrated into it, adding a patch so corrupt objects are not re-fetched from a promisor remote. Johannes Schindelin noted patch 1/6 fixes a performance bug Microsoft Git patched independently; Justin Tobler accepted the explanations and agreed the corrupt-object change is good.

### ".keep" lockfile ownership in ODB transactions
*Long thread: 6 replies so far from 2 participants over 1 day.*
Justin Tobler's series fixes a bug in `object-file.c` where a `git-receive-pack` process could delete another concurrent process's ".keep" lockfile for an identical packfile. Tobler posted v2 as a four-patch series, splitting the hash-agnostic comment change, applying Patrick Steinhardt's rename and error-handling suggestions, and adding a test assertion that the ".pack" file is not migrated.

## The period in brief
The week of 2026/10/12 through 2026/10/18 saw seven active days of steady traffic on the Git mailing list, with several long-running topics reaching resolution or `next` status. Readers should not miss the approval of Julia Evans's `gitmergeconflicts(7)` documentation, the marking of `uploadpack.lazyFetchTrusted` and CI annotation series for `next`, and the ongoing debate over squash-merge detection in `git branch --delete-merged`.

## Key developments

### `git blame` reads `.git-blame-ignore-revs` by default
Ravi Mistry's series teaches `git blame` and `git annotate` to automatically read `.git-blame-ignore-revs` from the working-tree root (or `HEAD:.git-blame-ignore-revs` in bare repositories) without per-clone configuration. After progressing through v4, v5, and v6 during the week, Junio C Hamano approved the v6 design as "Very cleanly done" and said he would replace the previous version, with Phillip Wood also expressing support for inclusion.

### Fetch commits references before backfilling tags
Karthik Nayak posted v3 of a fix for shallow fetches where newly fetched refs were unavailable as 'haves' during tag backfill because batched updates were not yet committed. The patch commits the fetched refs first, uses a separate `commit_ref_ret` error variable, and adds tests for 'haves' advertisement and upstream configuration when the main ref update is rejected, addressing a performance issue in shallow clones.

### `git log --follow` performance regression
SZEDER Gábor reported a 2.69x slowdown in `git log --follow` on linux.git, bisected to commit 304812ed33, with Patrick Steinhardt confirming an 8-minute versus 33-second difference. Miklos Vajna submitted a patch adding shortcuts in `propagate_follow_pathspec_to_parents()` that drop the linux.git case from 6m43s to 29s, though Junio C Hamano noted the followed-path clue is lost after `log_tree_diff()` flushes its output and suggested inspecting the diff queue instead.

### `core.indexLockTimeout` for index lock waiting
Jun Wu posted v2 of a series adding `core.indexLockTimeout` (default 1s) so commands like `git add`, `git checkout` and `git commit` wait briefly for the index lock instead of failing immediately when another process holds it. The series adds `repo_try_hold_locked_index()` for optional-lock users and fixes write races in `commit` and `reset` by reordering from "read -> lock -> write" to "lock -> read -> write".

### Worktree repair hardening
Yoichi NAKAYAMA posted v3 of a five-patch series hardening `git worktree repair` to validate cross-references between a linked working tree and its administrative data, preventing modification of the wrong `.git` file or gitdir. The final patch validates the worktree ID in `repair_gitfile()` to report "unrelated .git file" and skip repair on mismatch, and rewrites `get_worktree_id()` to handle separators more leniently.

### Upload-pack slow fetches from shallow repositories
Harald Nordgren submitted a patch fixing fetches from a shallow repository hanging over protocol v2, where fetching one new commit in a 900k-commit clone took 7 minutes versus about a second over v0 because the server walks its whole history to work out shallow grafts. The patch makes `upload-pack` advertise all shallow grafts when the client does not request a depth, also making `git clone --reject-shallow` refuse shallow repositories over v2 consistently.

### Faster client-side refspec matching in push
Jon Simons posted v2 of a 12-patch series speeding up client-side refspec matching in `git push`, cutting a 100-delete push against 100k advertised refs from ~2.5s to ~0.15s by converting linear ref traversals in remote.c to strmap lookups. The v2 also plugs a send-pack memory leak.

### Sigchain concurrent double-pop recovery
Ian Hu submitted a patch to `sigchain.c` fixing a race where two concurrent handlers popping the same signal both read the same chain depth, leaving the dispatcher stuck and spinning at 100% CPU. The patch adds a `pushed` counter and reinstalls the pre-chain disposition when popping an already-empty chain; fault-injected testing showed the unpatched code froze in 121 of 300 runs while the patched code never froze.

## In brief

**parse-options early-scan API** -- Christian Couder posted v3 of a four-patch series adding `early_scan_options()` and the `PARSE_OPT_EARLY` flag so commands can safely skip option values during early scans, with `git fast-import` now using it for `--allow-unsafe-features`.

**read-cache size change with conversion** -- Torsten Bögershausen posted v3/RFC and later updates to `ie_modified()` so `git status` does not trust a size change for paths subject to CRLF conversion, falling through to `ce_modified_check_fs()` instead.

**imap-send TLS updates** -- Beat Bolli posted v2 and v3 letting OpenSSL do hostname verification via `SSL_set1_host()` and replacing deprecated `SSLv23_method()` with `TLS_method()`.

**archive-zip UT field i386 failures** -- René Scharfe's patch omitting the UT extra field for out-of-range mtimes hit i386 CI failures; Matthew E. Luallen proposed `TIME_T_IS_64BIT`.

**fast-import 'from' by oid** -- Marc Becker posted v4 allowing 'from' to reference new commits/tags by oid, and v2 fixing a crash when querying commits or tags created in the same session.

**history signing** -- A two-patch series adding `-S`/`--gpg-sign` to `history drop`, `fixup`, `reword`, `split`, and `squash` was marked ready for `next` by Junio C Hamano.

**fsmonitor daemon renamed subdirs** -- Erik Cervin-Edin reported the Linux fsmonitor daemon deterministically misses edits in subdirectories of a renamed directory because `add_watch()` returns early on EEXIST.

**fetch --all --depth fix** -- Rong Bao fixed `git fetch --all --depth=` and `--all --unshallow` not deepening shallow clones by passing `--depth` to the fetcher subprocess.

**stash creation index loads** -- Phillip Wood posted v2 of a seven-patch series avoiding unnecessary index loads and refreshes in `git stash push`/`create`, cutting lstat() calls per changed path by eight for push and four for create.

**connectivity check with alternates** -- Igor Wiedler posted v2 fixing the connectivity check with alternates in deepening fetches by negating `--alternate-refs` and ignoring refs of shallow/grafted alternates; Jeff King called the approach exactly what he would have expected.

**fsck bigFileThreshold fix** -- Patrick Steinhardt fixed a bogus fsck corruption report when a packed blob's size equals `core.bigFileThreshold`.

**sequencer HEAD ref fix** -- Andy Koppe fixed the sequencer clobbering the HEAD ref name in worktree lookup by using `refs_resolve_refdup()`.

**cache-tree index lock** -- Szymon Wilczek submitted a patch dropping `LOCK_DIE_ON_ERROR` from `write_index_as_tree()` so `git write-tree` and `git am` proceed while a background `git status` holds the index lock.

**detached HEAD guide** -- Julia Evans posted v3 and v4 of a rewritten detached HEAD guide; Junio C Hamano suggested stronger wording and supplied a squash patch for a broken test.

**Coverity overflow fixes** -- Phillip Wood reviewed Johannes Schindelin's Coverity overflow fixes, proposing `intmax_t` multiplication for the `approxidate` fix.

**reflog --operations/--historical-branch** -- Harald Nordgren posted v2 adding these options to `git reflog` and `git log -g`, moving the branch decoration rightward for alignment.

**loose-ref seeking fix** -- Venkat Ragavan fixed loose-ref seeking across directory boundaries in `cache_ref_iterator_seek()`.

**HTTP push token-only URL** -- Dan reported `git push` over smart HTTP fails with token-only userinfo after libcurl 8.23.0 stopped returning credentials in `CURLINFO_EFFECTIVE_URL`, proposing to compare URLs after stripping userinfo.

**repack --dry-run fix** -- Siddharth Shrimali posted v3 fixing `git repack --drop-filtered --dry-run` writing a new promisor pack instead of just listing candidates; Junio C Hamano said he will queue it.

**odb shallow/grafted alternates** -- A two-patch series fixing the connectivity check for deepening fetches with alternates was ejected from `seen` after its test broke Windows CI.

**root refs in files backend** -- Andy Koppe posted v2 keeping root refs out of the files backend's loose ref cache, fixing `git for-each-ref --include-root-refs --merged=main` silently omitting merged root refs.

**git apply --check crash** -- Gingrspacecadet reported `git apply --check` crashes with an assertion failure when checking a patch that creates a new file on git 2.54.0.

**branch named @** -- On reports that `git checkout -b @` creates a branch named `@` despite `check-ref-format` rejecting it, Kristoffer Haugsbakk and Junio C Hamano agreed deprecating `@` as a `HEAD` synonym would not happen.

**t1901 SHA-256 test** -- Junio C Hamano reported the new test in `t/t1901-repo-structure.sh` is broken with SHA-256 because it hardcodes SHA-1-specific constants.

**RFC --delete-merged --into** -- DonQuinleone proposed an RFC extending `git branch --delete-merged` with a `--into <ref>` modifier and composable filters.

**strbuf_add_wrapped_text fix** -- Andy Koppe posted v2 fixing `strbuf_add_wrapped_text()` losing one column of indentation after a hard line break in `%w()`.

**diff-tree NUL support** -- Kristoffer Haugsbakk corrected his earlier claim that `git diff-tree` lacks NUL support and suggested a `--use-terminator` option for commands using `log-tree.c`.

**config_buf_fgetc unsigned char** -- Olof Fairbrook made `config_buf_fgetc()` return blob bytes as `unsigned char` so `0xff` is no longer treated as EOF.

**bundle uninitialized memory** -- Olof Fairbrook fixed `git bundle create` freeing uninitialized stack memory on unrecognized arguments.

**autoSetupMerge=inherit** -- Phillip Wood judged the change not clearly better; Harald Nordgren proposed inheriting the forked branch's upstream with the start point as fallback.

**remote.followRemoteHEAD** -- Colin Hinton deferred validation to `do_fetch()` so `git push` and `git remote -v` stop warning; `fetch.followRemoteHEAD` now accepts `warn-if-not-<branch>`.

**cargo target-dir pin** -- Andy Koppe pinned `--target-dir target` in the Makefile's cargo recipes, fixing link errors with custom `CARGO_TARGET_DIR`.

**prune-packed the_repository removal** -- Shashank S removed `USE_THE_REPOSITORY_VARIABLE` from `prune-packed.c`, passing a `struct repository *` via a new `struct prune_data`.

**test modernization** -- Srujith Poojary posted v3 replacing old-style `test -f` checks in `t/t2030-unresolve-info.sh` with `test_path_is_file` and `test_path_is_missing`.

**GIT_DEPTH for CI jobs** -- Patrick Steinhardt proposed setting `GIT_DEPTH: 0` for GitLab CI whitespace and style jobs so they work on merge requests over 50 commits.

## Looking ahead
The squash-merge detection debate in `git branch --delete-merged` is likely to continue, with Harald Nordgren and Junio C Hamano at odds over whether the feature belongs in `--delete-merged` or a separate `--delete-irrelevant` option. The incremental connectivity check series remains in review with proposed configuration modes and benchmarks to be finalized. Additionally, the shallow-fetch fixes for `upload-pack` and tag backfill will see further review in the coming week.