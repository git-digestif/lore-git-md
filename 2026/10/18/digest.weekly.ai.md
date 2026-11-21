## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 64 replies so far from 6 participants over 21 days.*
Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` docs. The series progressed through multiple revisions, with Junio C Hamano supplying a corrected diff3 example and Patrick Steinhardt raising wording nits. By the end of the week Evans had accepted the alternative example, agreed to fix bullet punctuation and spelling, and proposed deleting the `zdiff3` note, leaving the series close to final.

### CI housekeeping and modernizations
*Long thread: 39 replies so far from 6 participants over 9 days.*
Patrick Steinhardt's eight-patch series fixes smaller CI issues and moves jobs off EOL Docker images, renaming the "linux-exotic" job to "linux-feature-testbed" and reworking the reftable job. Jeff King reviewed the t5004 SHA-256 test rework, and Junio C Hamano marked v4 for `next`. Later in the week Hamano suggested adding a no-Rust build check to the feature-testbed job until Rust becomes required.

### credentialStore.lockTimeout config option
*Long thread: 38 replies so far from 5 participants over 45 days.*
Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. Junio C Hamano reviewed the initial version, suggesting a loop over both variable names for tests and questioning a zero-timeout test. Ajayi posted v2 incorporating the feedback, looping the tests and dropping the problematic case.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control
*Long thread: 37 replies so far from 3 participants over 96 days.*
Christian Couder's five-patch series introduces the protected `uploadpack.lazyFetchTrusted` configuration variable so server operators can mark repositories whose promisor remotes may be lazy-fetched. After earlier revisions added recursion protection and documentation fixes, Junio C Hamano marked the topic ready for `next` on October 13.

### rerere waiting for `MERGE_RR.lock`
*Long thread: 36 replies so far from 5 participants over 43 days.*
Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` and adds a hidden `--skip-locked` option to `git rerere gc`. Patrick Steinhardt called v8 "in a good enough state" and agreed to mark it ready for `next`, which Hamano confirmed.

### `branch --delete-merged` detecting squash-merged branches
*Long thread: 32 replies so far from 5 participants over 20 days.*
Harald Nordgren's series teaches `git branch --delete-merged` to treat a branch as merged when an upstream commit contains all its changes, naming that commit as "landed as", with a `--no-squashed` option to skip the search. Junio C Hamano proposed keeping `--delete-merged` strict and adding a separate `--delete-irrelevant`, but Nordgren argued for deleting any branch whose work landed upstream regardless of how. Nordgren also reported that a range-diff approach was slower and no better at matching.

### Limited fetch for new remotes in shallow repositories
*Long thread: 31 replies so far from 4 participants over 26 days.*
Harald Nordgren's five-patch series avoids fetching every branch of a new remote in a shallow repo, adding `remote.<name>.refmap`, a `--limited-fetch` option to `git remote add`, and defaulting to it in shallow repositories. He posted v9 moving the `do_fetch()` split into the refmap-inference commit.

### Incremental connectivity check
*Long thread: 27 replies so far from 3 participants over 33 days.*
Kristofer Karlsson's four-patch series adds an incremental mode for the connectivity check gated behind `transfer.connectivityCheck=incremental`. Patrick Steinhardt reviewed the implementation, raising points about `die()` in library code and counter types. Karlsson benchmarked the modes, finding incremental up to 2.8x faster on large repos, and proposed three config modes with a 1000-commit cutoff.

## The period in brief
The week of 2026/10/12 through 2026/10/18 saw seven active days of heavy, eventful traffic. Three topics were marked ready for `next`: `uploadpack.lazyFetchTrusted`, `history` commit signing, and the CI annotation series. Readers should not miss Julia Evans's near-final `gitmergeconflicts(7)` documentation, Ravi Mistry's approved blame default-ignore-revs series, and Jun Wu's `core.indexLockTimeout` proposal.

## Key developments

### `git blame` using `.git-blame-ignore-revs` by default
Ravi Mistry's series teaches `git blame` and `git annotate` to read `.git-blame-ignore-revs` from the working-tree root (or `HEAD:.git-blame-ignore-revs` in bare repositories) without per-clone configuration. After v4 added a 100MB blob cap, Mistry posted v5 reading the working-tree copy and v6 fixing review points; Junio C Hamano approved v6 as "Very cleanly done" and said he would replace the previous version. Phillip Wood also indicated he would be happy to see it included.

### Fetch committing references before backfilling tags
Karthik Nayak posted v3 of a fix committing fetched references before tag backfill so shallow fetches report refs in the 'have' section and avoid requesting all objects. The patch uses a separate `commit_ref_ret` error variable and adds tests for 'haves' advertisement and upstream configuration when the main ref update is rejected. Earlier, Hamano had asked whether the problem occurs when the main transaction is empty and whether `--atomic` users should expect a degenerate full clone.

### Signing commits created by `git history` rewrites
A two-patch series adds a signing-key option to the replay API and exposes `-S`/`--gpg-sign` on `history drop`, `fixup`, `reword`, `split`, and `squash`. Junio C Hamano marked the topic ready for `next` on October 13, explaining it had received a "looks good enough" at v3 that he had missed.

### Faster client-side refspec matching in push
Jon Simons posted v2 of a 12-patch series cutting push refspec-matching time against a 100k-ref server from seconds to ~0.15s by converting linear ref traversals in remote.c to strmap lookups; v2 also plugs a send-pack memory leak.

### Speeding up `git log --follow` on non-linear history
Miklos Vajna submitted a patch adding shortcuts in `propagate_follow_pathspec_to_parents()`, dropping the linux.git `rsc_table.h` case from 6m43s to 29s. Junio C Hamano noted the followed-path clue is lost after `log_tree_diff()` flushes output and suggested inspecting the diff queue instead. SZEDER Gabor had earlier traced the regression to commit 304812ed33.

### Wait for the index lock instead of failing
Jun Wu posted v2 adding `core.indexLockTimeout` (default 1s) so commands like `git add` and `git commit` wait briefly for the index lock instead of failing when an IDE's `git status` holds it. The series adds `repo_try_hold_locked_index()` for optional-lock users and fixes write races in `commit` and `reset` by reordering to "lock -> read -> write".

### Hardening `git worktree repair`
Yoichi NAKAYAMA posted v3 of a five-patch series validating cross-references in `git worktree repair` so it cannot break an unrelated `.git` file or gitdir. V3 makes the separator check more lenient, validates the worktree ID in `repair_gitfile()`, and rewrites `get_worktree_id()` to handle trailing separators and both `/` and `\`.

### Fixing slow protocol v2 fetches from shallow repositories
Harald Nordgren submitted a patch making `upload-pack` advertise all shallow grafts when the client does not request a depth, fixing 7-minute fetches over v2 versus one second over v0 in a 900k-commit clone. This also makes `git clone --reject-shallow` refuse shallow repositories over v2 and lets `git fetch --unshallow` visit each commit only once.

## In brief

- **parse-options early-scan API** -- Christian Couder posted v3 adding `early_scan_options()` and `PARSE_OPT_EARLY` so `git fast-import` can safely skip option values during early scans, fixing abbreviation disagreements.
- **odb/files alternates cleanups** -- Patrick Steinhardt's six-patch v2 series cleans up the files backend after alternates migrated, including a fix so corrupt objects are not re-fetched from promisor remotes; Justin Tobler reviewed it as ready.
- **stash creation index loads** -- Phillip Wood posted v2 of a seven-patch series avoiding unnecessary index loads in `git stash push`/`create`, cutting lstat() calls per changed path.
- **cache-tree no longer dies when the index is locked** -- Szymon Wilczek dropped `LOCK_DIE_ON_ERROR` from `write_index_as_tree()` so `git write-tree` and `git am` proceed while a background `git status` holds the lock.
- **sigchain concurrent double-pop recovery** -- Ian Hu fixed a race in `sigchain.c` where concurrent handlers could spin at 100% CPU; fault-injected testing showed the patch eliminates the freeze.
- **HTTP push failure with token-only URL userinfo** -- Dan reported that `git push` over smart HTTP fails with token-only URLs after libcurl 8.23.0, traced to `update_url_from_redirect()` clearing `http_auth`.
- **repack --dry-run rebuilds packs** -- Siddharth Shrimali posted v3 fixing `git repack --drop-filtered --dry-run` to list candidates without rebuilding packs; Hamano said he will queue it.
- **odb: ignore refs of shallow and grafted alternates** -- A two-patch series fixes the connectivity check for deepening fetches with alternates, but Hamano reported the test breaks Windows CI and will eject the topic from `seen`.
- **read-cache: do not trust a size change when conversion is active** -- Torsten Bogershausen and Curtis Smith fixed `ie_modified()` so `git status` does not report CRLF-converted files as modified on size change alone.
- **bundle: do not free uninitialized memory** -- Olof Fairbrook fixed `git bundle create` freeing uninitialized stack memory on bad arguments.
- **imap-send TLS updates** -- Beat Bolli posted v3 replacing deprecated SSL methods with `TLS_method()` and OpenSSL hostname verification.
- **fast-import 'from' by oid** -- Marc Becker posted v4 allowing 'from' to reference new commits/tags by oid; Hamano said "Will queue."
- **fsck bigFileThreshold fix** -- Patrick Steinhardt fixed a bogus corruption report when a packed blob's size equals `core.bigFileThreshold`.
- **sequencer HEAD ref fix** -- Andy Koppe fixed the sequencer clobbering the HEAD ref name in worktree lookup.
- **connectivity check with alternates** -- Igor Wiedler posted v2 negating `--alternate-refs` in deepening fetches; Jeff King called the approach exactly right.
- **detached HEAD guide** -- Julia Evans posted v3/v4 of a rewritten detached HEAD guide; Hamano suggested stronger wording and supplied a test fix.
- **reflog --operations** -- Harald Nordgren posted v2 adding `--operations` and `--historical-branch` to `git reflog`.
- **root refs in loose cache** -- Andy Koppe posted v2 keeping root refs out of the files backend's loose ref cache, fixing `for-each-ref --include-root-refs`.
- **strbuf_add_wrapped_text fix** -- Andy Koppe fixed `%w()` losing indentation after hard line breaks.
- **prune-packed the_repository removal** -- Shashank S removed `USE_THE_REPOSITORY_VARIABLE` from `prune-packed.c`.
- **test modernization** -- Srujith Poojary replaced old-style test checks in `t2030` with `test_path_is_file`/`test_path_is_missing`.
- **config_buf_fgetc unsigned char** -- Olof Fairbrook made `config_buf_fgetc()` return blob bytes as `unsigned char` so `0xff` is not treated as EOF.
- **followRemoteHEAD validation** -- Colin Hinton deferred `remote.<name>.followRemoteHEAD` validation to `do_fetch()`.
- **cargo target-dir pin** -- Andy Koppe pinned `--target-dir` in Makefile cargo recipes.
- **fsmonitor subdirectory misses** -- Erik Cervin-Edin reported the Linux fsmonitor daemon misses edits in subdirectories of renamed directories.
- **fetch --all --depth fix** -- Rong Bao fixed `git fetch --all --depth=` not deepening shallow clones.
- **archive-zip UT field** -- Rene Scharfe's patch omitting the UT extra field hit i386 CI failures; Matthew E. Luallen proposed `TIME_T_IS_64BIT`.
- **fast-import crash fix** -- Marc Becker posted v2 fixing a crash when querying commits/tags created in the same session.
- **Coverity overflow fixes** -- Phillip Wood reviewed Johannes Schindelin's fixes, proposing `intmax_t` for the `approxidate` fix.
- **loose-ref seeking** -- Venkat Ragavan fixed loose-ref seeking across directory boundaries in `cache_ref_iterator_seek()`.
- **git apply --check crash** -- Gingrspacecadet reported an assertion failure on patches creating new files.
- **branch named `@`** -- Kristoffer Haugsbakk and Hamano discussed that `git checkout -b @` creates a branch named `@` despite `check-ref-format` rejecting it.
- **t1901 SHA-256 test** -- Hamano reported the new test in `t1901` breaks with SHA-256 due to hardcoded 40-hex-digit constants.
- **autoSetupMerge=inherit** -- Phillip Wood judged the change not clearly better; Harald Nordgren proposed inheriting the forked branch's upstream.
- **GIT_DEPTH for CI jobs** -- Patrick Steinhardt proposed setting `GIT_DEPTH: 0` for whitespace/style check jobs; Hamano will queue it.