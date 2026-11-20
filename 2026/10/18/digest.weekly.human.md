## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 64 replies so far from 6 participants over 21 days.*
Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick` and `git-pull` docs. Early rounds switched `Reviewed-by` to `Helped-by` and reworked examples; Junio C Hamano objected that the "cherry/banana" example was ambiguous and supplied a corrected diff3 example, while Patrick Steinhardt raised wording nits and a commit-message inaccuracy. By midweek Evans accepted the diff3 fix, agreed to punctuation and wording changes, and the series absorbed final review feedback.

### CI housekeeping and modernizations
*Long thread: 39 replies so far from 6 participants over 9 days.*
Patrick Steinhardt's eight-patch series fixes smaller CI issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed" and reworking the reftable job. Version 3 adopted the new job name after Junio C Hamano's suggestion; version 4 reworked t5004 to run under both SHA-1 and SHA-256 by forcing the test repo to SHA-1. Jeff King approved v4 and Junio marked the series for `next`. Later discussion considered resurrecting the no-Rust build job within the feature-testbed.

### credentialStore.lockTimeout config option
*Long thread: 38 replies so far from 5 participants over 45 days.*
Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. Junio C Hamano reviewed the initial version, suggesting a loop over both variable names to keep tests in sync and questioning a 0 ms timeout test. Ajayi posted v2, which loops the two timeout tests over both names, wraps long lines, adds an explanatory comment, and drops the problematic "allows a store" test.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control
*Long thread: 37 replies so far from 3 participants over 96 days.*
Christian Couder's five-patch series introduces the protected `uploadpack.lazyFetchTrusted` configuration variable so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones and fetches, with refactoring in `promisor-remote.c`, `setup.c`, and `builtin/upload-pack.c`. Earlier revisions added recursion protection and documentation fixes. Junio C Hamano marked the topic ready for `next` on October 13.

### rerere waiting for `MERGE_RR.lock`
*Long thread: 36 replies so far from 5 participants over 43 days.*
Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` before failing, so a rebase hitting a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` option to `git rerere gc`. Version 8 incorporated earlier review; Patrick Steinhardt retracted a suggestion for a separate flags enum and called v8 good enough, agreeing to mark the topic ready for `next`.

### `branch --delete-merged` detecting squash-merged branches
*Long thread: 32 replies so far from 5 participants over 20 days.*
Harald Nordgren's series makes `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all of its changes, naming that commit as "landed as", and adds `--no-squashed` to skip the slower search. Phillip Wood asked for a detailed account of detection tradeoffs; Junio C Hamano proposed keeping `--delete-merged` a strict reachability check and adding a separate `--delete-irrelevant` for squash-merged work. Nordgren argued the feature should delete branches once picked up by mainline regardless of method, and reported that a range-diff approach was slower (10s vs 0.5s on 65 branches) with identical results.

### Limited fetch for new remotes in shallow repositories
*Long thread: 31 replies so far from 4 participants over 26 days.*
Harald Nordgren's five-patch series avoids fetching every branch of a new remote in a shallow repo, adding `remote.<name>.refmap`, a `--limited-fetch` option to `git remote add`, and defaulting to it in shallow repositories. Version 9 moved the `do_fetch()` if/else split into the refmap-inference commit after earlier reviews shaped the helper extraction.

### Incremental connectivity check
*Long thread: 27 replies so far from 3 participants over 33 days.*
Kristofer Karlsson's four-patch series adds an incremental mode for the connectivity check in `connected.c`, gated behind `transfer.connectivityCheck=incremental`, running the verifier inside the existing `rev-list` subprocess via a new `--verify-trees-incremental` flag. Patrick Steinhardt raised concerns about `die()` calls in library code, counter types, and recursion depth. Karlsson benchmarked full vs. incremental checks, finding incremental up to 2.8x faster on large repos, and proposed three config modes with a 1000-commit cutoff.

### CI annotations naming the failing test script
*Long thread: 25 replies so far from 4 participants over 19 days.*
Harald Nordgren's series links failure and leak annotations in CI output to the test script, showing the file and line as plain text. Earlier revisions reverted direct file/line linking after it failed for files unchanged in a PR. Junio C Hamano marked the topic ready for `next` on October 13.

### Blame reads .git-blame-ignore-revs by default
*Long thread: 24 replies so far from 4 participants over 35 days.*
Ravi Mistry's series teaches `git blame` and `git annotate` to use `.git-blame-ignore-revs` at the working-tree root (or `HEAD:.git-blame-ignore-revs` in bare repositories) by default. Version 4 capped the blob at 100MB; v5 read the working-tree copy in non-bare repos; v6 fixed Junio C Hamano's review points. Hamano approved v6 as "Very cleanly done", and Phillip Wood said he would be happy to see it included aside from an `lstat()` question.

### `checkout -m` recreating original conflict labels
*Long thread: 22 replies so far from 3 participants over 14 days.*
Phillip Wood's three-patch series teaches the ort machinery to write conflict labels to `.git/MERGE_LABELS` when switching to a conflicted merge result so `git checkout -m` reuses them. Version 4 added a patch caching merge config settings and labels once per invocation via a new `checkout_merged_data` structure. Junio C Hamano reviewed v4 and said he would replace the previous version.

### push: check pushed ref for `--force-if-includes`
*Long thread: 20 replies so far from 5 participants over 45 days.*
Tyler Cipriani's series fixes `--force-if-includes` to consult the reflog of the ref actually being pushed rather than the local branch named after the destination. Version 7, rebased on maint, resolves the pushed ref via `ref->peer_ref`, rejects non-branch sources as unverifiable, adds a `pushRefUnverifiable` advice, and defers the reflog check until after fast-forward rules so valid fast-forward pushes proceed.

### doc: add more AsciiDoc cross-references
*Long thread: 20 replies so far from 5 participants over 27 days.*
Julia Evans's patch updates `Documentation/` to use the verbose `<<EXAMPLES,EXAMPLES>>` cross-reference form, since the shorthand sometimes renders as `the section called "EXAMPLES"`. Jean-Noël Avila argued for the double-reference form because translated manual pages can require changes incompatible with simply inserting the title.

### odb/files alternates cleanups
*Long thread: 14 replies so far from 3 participants over 2 days.*
Patrick Steinhardt's six-patch v2 series cleans up the files object database backend after alternates migrated into it, adding a patch so corrupt objects are not re-fetched from a promisor remote. Johannes Schindelin noted patch 1/6 fixes the same performance bug Microsoft Git patched independently and asked for a regression test; Steinhardt agreed to fold it in. Justin Tobler accepted the explanations, and Junio marked the topic ready for `next` on October 16.

### read-cache: do not trust a size change when conversion is active
*Long thread: 10 replies so far from 7 participants over 276 days.*
Curtis Smith and Torsten Bögershausen's patch fixes `ie_modified()` so `git status` no longer reports a file as modified when a CRLF rewrite or clean filter converts different working-tree bytes to the same blob. Junio C Hamano corrected the commit message and warned the new tests may hit racy-git paths; Bögershausen later suggested the `would_convert_to_git()` call may belong in `ie_match_stat()`.

### ".keep" lockfile ownership in ODB transactions
*Long thread: 6 replies so far from 2 participants over 1 day.*
Justin Tobler's series fixes a bug in `object-file.c` where a `git-receive-pack` process could delete another concurrent process's ".keep" lockfile for an identical packfile. Version 2, a four-patch series, splits the hash-agnostic comment change, renames to `write_pack_lockfile()`, uses errno-based error messages, and adds a test assertion that the ".pack" file is not migrated.

## The period in brief

The week of 2026/10/12 through 2026/10/18 saw steady traffic across all seven days, with numerous long-running threads reaching resolution and several topics marked ready for `next`. Readers should not miss Julia Evans's `gitmergeconflicts(7)` documentation series absorbing final review fixes, Patrick Steinhardt's CI housekeeping series landing in `next`, and the protracted debate over whether `git branch --delete-merged` should detect squash-merged branches.

## Key developments

### Fetch committing references before backfilling tags
Karthik Nayak posted v2 and v3 of a fix committing the previous batched reference update before tag backfill, so shallow fetches report refs in the 'have' section and avoid requesting all objects. Junio C Hamano asked whether the problem also occurs when the main fetch transaction is empty and whether `--atomic` users should expect a degenerate full clone; Nayak added tests for 'haves' advertisement and upstream configuration when the main ref update is rejected.

### `git log --follow` performance regression
SZEDER Gábor reproduced a 2.69x slowdown from commit 304812ed33, tracing most cost to `propagate_follow_pathspec_to_parent()` on merge commits' later parents. Patrick Steinhardt confirmed an 8-minute vs 33-second difference on linux.git. Miklos Vajna submitted a patch adding shortcuts in `propagate_follow_pathspec_to_parents()`, dropping the `rsc_table.h` case from 6m43s to 29s; Junio C Hamano suggested inspecting the diff queue in `log_tree_diff_flush()` instead.

### `core.indexLockTimeout` for index lock waits
Jun Wu posted v2 of a four-patch series adding `core.indexLockTimeout` (default 1s; 0 disables retry, -1 waits indefinitely) so commands like `git add`, `git checkout` and `git commit` wait briefly for the index lock instead of failing when an IDE's `git status` holds it. The series adds `repo_try_hold_locked_index()` for optional-lock users and fixes write races in `commit` and `reset` by reordering to "lock -> read -> write".

### Worktree repair hardening
Yoichi NAKAYAMA posted v3 of a five-patch series hardening `git worktree repair` to validate cross-references between a linked working tree and its administrative data, preventing modification of an unrelated `.git` file or gitdir. The final patch validates the worktree ID in `repair_gitfile()` to report "unrelated .git file" and skip repair on mismatch.

### Upload-pack slow fetches from shallow repositories
Harald Nordgren submitted a patch fixing protocol v2 fetches from shallow repositories hanging because the server walks its whole history to determine shallow grafts; in a 900k-commit clone, fetching one new commit took 7 minutes over v2 versus about a second over v0. The patch makes `upload-pack` advertise all shallow grafts when the client does not request a depth, matching v0 behavior.

### Parse-options early-scan API
Christian Couder posted v3 of a four-patch series adding `early_scan_options()` and the `PARSE_OPT_EARLY` flag to the parse-options API so commands can safely skip option values during early scans, recognizing negated forms, abbreviations and aliases. `git fast-import` now uses it for `--allow-unsafe-features`, fixing disagreements with `parse_options()`.

### Signing commits created by `git history` rewrites
A two-patch series adds a signing-key option to the replay API and exposes `-S`/`--gpg-sign` on `history drop`, `fixup`, `reword`, `split`, and `squash`, so rewritten commits honor `commit.gpgSign`. Junio C Hamano marked the topic ready for `next` on October 13.

### Faster client-side refspec matching in push
Jon Simons posted v2 of a 12-patch series converting linear ref traversals in `remote.c` to strmap lookups, cutting a 100-delete push against 100k advertised refs from ~2.5s/~6.6s to ~0.15s/~0.19s; v2 also plugs a send-pack memory leak.

## In brief

- **imap-send TLS updates** -- Beat Bolli posted v3 replacing deprecated `SSLv23_method()` with `TLS_method()` and using OpenSSL's built-in hostname verification, with defines for OpenSSL 4.0.
- **archive-zip UT field i386 failures** -- René Scharfe's patch omitting the UT extra field for out-of-range mtimes hit i386 CI failures; Matthew E. Luallen proposed `TIME_T_IS_64BIT`.
- **fast-import crash and 'from' by oid** -- Marc Becker posted v2 fixing a crash when querying commits created in the same session, and v4 allowing 'from' to reference new commits by oid.
- **fsmonitor daemon renamed subdirs** -- Erik Cervin-Edin reported the Linux fsmonitor daemon deterministically misses edits in subdirectories of a renamed directory because `add_watch()` returns early on EEXIST.
- **fetch --all --depth fix** -- Rong Bao fixed `git fetch --all --depth=` and `--all --unshallow` not deepening shallow clones by passing `--depth` to the fetcher subprocess.
- **connectivity check with alternates** -- Igor Wiedler posted v2 negating `--alternate-refs` in deepening fetches and ignoring refs of shallow/grafted alternates; Jeff King called the approach exactly right, though the test broke Windows CI and Junio ejected the topic from `seen`.
- **fsck bigFileThreshold fix** -- Patrick Steinhardt fixed a bogus fsck corruption report when a packed blob's size equals `core.bigFileThreshold`.
- **sequencer HEAD ref fix** -- Andy Koppe fixed the sequencer clobbering the HEAD ref name in worktree lookup by using `refs_resolve_refdup()`.
- **stash index loads** -- Phillip Wood posted v2 of a seven-patch series avoiding unnecessary index loads in `git stash push`/`create`, cutting lstat() calls per changed path by eight and four respectively.
- **sigchain race** -- Ian Hu fixed a race where concurrent signal handlers could spin a process at 100% CPU; fault-injected testing showed the unpatched code froze in 121 of 300 runs.
- **HTTP push token-only URL** -- Dan reported `git push` over smart HTTP fails with token-only userinfo after libcurl 8.23.0 stopped returning credentials in `CURLINFO_EFFECTIVE_URL`, proposing to compare URLs after stripping userinfo.
- **repack --dry-run fix** -- Siddharth Shrimali posted v3 fixing `git repack --drop-filtered --dry-run` writing a new promisor pack instead of just listing candidates; Junio will queue it as-is.
- **cache-tree index lock** -- Szymon Wilczek dropped `LOCK_DIE_ON_ERROR` from `write_index_as_tree()` so `git write-tree` and `git am` proceed while `index.lock` is held, fixing a rebase failure.
- **detached HEAD guide** -- Julia Evans posted v3 and v4 of a rewritten detached HEAD guide; Junio suggested stronger wording and supplied a squash patch for a broken `t2020` test.
- **root refs in files backend** -- Andy Koppe posted v2 keeping root refs out of the files backend's loose ref cache, fixing `git for-each-ref --include-root-refs --merged=main` omitting merged root refs.
- **git apply --check crash** -- Gingrspacecadet reported `git apply --check` crashes with an assertion failure on patches creating new files in git 2.54.0.
- **branch named @** -- Kristoffer Haugsbakk and Junio C Hamano agreed that deprecating `@` as a `HEAD` synonym would not happen given Git's inertia around long-standing features.
- **t1901 SHA-256 test** -- Junio C Hamano reported the new test in `t/t1901-repo-structure.sh` is broken with SHA-256 because it hardcodes 40-hex-digit constants.
- **RFC --delete-merged --into** -- DonQuinleone proposed extending `--delete-merged` with a `--into <ref>` modifier and composable `--upstream-gone`/`--merged=` filters.
- **strbuf_add_wrapped_text fix** -- Andy Koppe posted v2 fixing `strbuf_add_wrapped_text()` losing one column of indentation after a hard line break in `%w()`.
- **config_buf_fgetc** -- Olof Fairbrook made `config_buf_fgetc()` return blob bytes as `unsigned char` so `0xff` is not treated as EOF.
- **bundle uninitialized memory** -- Olof Fairbrook fixed `git bundle create` freeing uninitialized stack memory on bad arguments.
- **followRemoteHEAD** -- Colin Hinton deferred `remote.<name>.followRemoteHEAD` validation to `do_fetch()` so `git push` stops warning; `fetch.followRemoteHEAD` now accepts `warn-if-not-<branch>`.
- **cargo target-dir** -- Andy Koppe pinned `--target-dir target` in Makefile cargo recipes, fixing link errors with custom `CARGO_TARGET_DIR`.
- **prune-packed repository variable** -- Shashank S removed `USE_THE_REPOSITORY_VARIABLE` from `prune-packed.c`.
- **test modernization** -- Srujith Poojary posted v3 replacing old-style `test -f` checks in `t/t2030-unresolve-info.sh`.
- **GIT_DEPTH for CI** -- Patrick Steinhardt proposed setting `GIT_DEPTH: 0` for GitLab CI whitespace and style jobs so they work on large merge requests.

## Looking ahead

The `branch --delete-merged` squash-merge detection debate is likely to continue, with Junio C Hamano's proposal for a separate `--delete-irrelevant` option competing against Harald Nordgren's argument for extending the existing flag. The shallow-fetch fixes -- both the tag-backfill ordering and the upload-pack shallow advertisement -- will see further review next week, as will Jun Wu's `core.indexLockTimeout` series and the incremental connectivity check configuration modes.