## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 64 replies so far from 6 participants over 21 days.*

Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` docs. After v3 drew mostly positive review from Junio C Hamano, who supplied a corrected diff3 example, and Patrick Steinhardt raised wording nits, Evans posted final fixes on October 14 accepting the alternative example, fixing bullet punctuation, and spelling out `--patch`. The series has absorbed all review feedback but has not yet been marked ready for `next`.

### CI housekeeping and modernizations

*Long thread: 39 replies so far from 6 participants over 9 days.*

Patrick Steinhardt's eight-patch series fixes smaller CI issues and moves jobs off EOL Docker images, renaming the "linux-exotic" job to "linux-feature-testbed" and reworking the reftable-plus-SHA-256 coverage job. After v3 on October 14 adopted the new job name and Jeff King questioned the t5004 SHA-256 test, Steinhardt posted v4 on October 15 forcing the test repo to SHA-1 so it works under both hash algorithms; Junio C Hamano marked the series for `next`. On October 16 Hamano suggested also resurrecting the no-Rust build job in the same spirit.

### `credentialStore.lockTimeout` config option

*Long thread: 38 replies so far from 5 participants over 45 days.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries in `builtin/credential-store.c`, keeping `lockTimeoutMS` as a deprecated alias. Junio C Hamano reviewed the initial version on October 15, suggesting a loop over both variable names to keep tests in sync and questioning one test's behavior with a 0 ms timeout. Ajayi posted v2 on October 16 implementing the loop, wrapping long lines, and dropping the questionable test.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control

*Long thread: 37 replies so far from 3 participants over 96 days.*

Christian Couder's five-patch series introduces the protected `uploadpack.lazyFetchTrusted` configuration variable so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones and fetches, with refactoring in `promisor-remote.c`, `setup.c`, and `builtin/upload-pack.c`. After earlier revisions added recursion protection and documentation fixes, Junio C Hamano marked the topic ready for `next` on October 13.

### rerere waiting for `MERGE_RR.lock`

*Long thread: 36 replies so far from 5 participants over 43 days.*

Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` so a rebase hitting a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` option to `git rerere gc`. After Bachem posted v8 on October 13 incorporating earlier review, Patrick Steinhardt called it "in a good enough state by now," and on October 14 agreed to mark the topic ready for `next`.

### `branch --delete-merged` detecting squash-merged branches

*Long thread: 32 replies so far from 5 participants over 20 days.*

Harald Nordgren's series teaches `git branch --delete-merged` to treat a branch as merged when some upstream commit since the fork point contains all of its changes, naming that commit as "landed as", and adds `--no-squashed` to skip the slower search. Junio C Hamano proposed on October 13 that `--delete-merged` remain a strict reachability check with a separate `--delete-irrelevant` for squash-merged work, but Nordgren argued on October 17 and 18 that the feature should delete branches once their work lands upstream regardless of how it arrived. A range-diff approach tested on October 18 proved slower (10s vs 0.5s) with no better matching.

### Limited fetch for new remotes in shallow repositories

*Long thread: 31 replies so far from 4 participants over 26 days.*

Harald Nordgren's five-patch series avoids fetching every branch of a new remote in a shallow repo, adding `remote.<name>.refmap`, a `--limited-fetch` option to `git remote add`, and defaulting to it in shallow repositories. After earlier reviews shaped the helper extraction and refmap inference, Nordgren posted v9 on October 14 moving the `do_fetch()` if/else split into the refmap-inference commit.

### Incremental connectivity check

*Long thread: 27 replies so far from 3 participants over 33 days.*

Kristofer Karlsson's four-patch series adds an incremental mode for the connectivity check in `connected.c`, gated behind `transfer.connectivityCheck=incremental`, running the verifier inside the existing `rev-list` subprocess via a new `--verify-trees-incremental` flag. Patrick Steinhardt reviewed v3 on October 15, raising points about `die()` in library code, counter types, and fallback thresholds. On October 16 Karlsson benchmarked full vs. incremental checks, finding incremental up to ~2.8x faster on large repos, and proposed three config modes ("full", "incremental", "auto") with a 1000-commit cutoff.

## The period in brief

The week of 2026/10/12 through 2026/10/18 saw seven active days of steady traffic, with three topics marked ready for `next` and several long-running series reaching decisive review stages. Readers should not miss the `gitmergeconflicts(7)` documentation nearing completion, the `core.indexLockTimeout` proposal to stop rebase failures from background `git status`, and the fix for seven-minute protocol v2 fetches from shallow repositories.

## Key developments

### Signing commits created by `git history` rewrites

A two-patch series adds a signing-key option to the replay API and exposes `-S`/`--gpg-sign` and `--no-gpg-sign` on `history drop`, `fixup`, `reword`, `split`, and `squash`, so rewritten commits honor `commit.gpgSign` like rebase and cherry-pick do. Junio C Hamano marked the topic ready for `next` on October 13, explaining that it had already received a "looks good enough" at v3 that he had missed.

### Faster client-side refspec matching in push

Jon Simons posted v2 of a 12-patch series speeding up client-side refspec matching in `git push`, cutting a 100-delete push against 100k advertised refs from ~2.5s/~6.6s to ~0.15s/~0.19s by converting linear ref traversals in remote.c to strmap lookups; v2 also plugs a send-pack memory leak.

### `git log --follow` performance on non-linear history

Miklos Vajna submitted a patch adding two shortcuts in `propagate_follow_pathspec_to_parents()`: skip expensive rename detection unless the path was added relative to a parent, and follow a parent's history directly when the path is unchanged. The linux.git `rsc_table.h` case drops from 6m43s to 29s. Junio C Hamano noted the followed-path clue is lost after `log_tree_diff()` flushes its output and suggested inspecting the diff queue in `log_tree_diff_flush()` instead.

### Fetch committing references before backfilling tags

Karthik Nayak posted v3 of "fetch: commit references fetched before backfilling tags", fixing shallow fetches where newly fetched refs were unavailable as 'haves' during tag backfill because batched updates were not yet committed. The patch commits the fetched refs first, uses a separate `commit_ref_ret` error variable, and adds tests for 'haves' advertisement and upstream configuration when the main ref update is rejected.

### Wait for the index lock instead of failing

Jun Wu posted v2 of a four-patch series adding `core.indexLockTimeout` (default 1s; 0 disables retry, -1 waits indefinitely) so commands like `git add`, `git checkout` and `git commit` wait briefly for the index lock instead of failing immediately when another process holds it. The series adds `repo_try_hold_locked_index()` for optional-lock users such as `git status`, and fixes write races in `commit` and `reset` by reordering from "read -> lock -> write" to "lock -> read -> write".

### upload-pack: fix slow fetches from a shallow repository

Harald Nordgren submitted a patch fixing fetches from a shallow repository hanging over protocol v2: in a clone with 900k commits, fetching one new commit took 7 minutes over v2 versus about a second over v0, because the server walks its whole history to work out which shallow commits to advertise. The patch makes `upload-pack` advertise all shallow grafts when the client does not request a depth, as v0 does.

### worktree repair hardening

Yoichi NAKAYAMA posted v3 of a five-patch series hardening `git worktree repair` in `worktree.c`, which previously could modify the wrong `.git` file or gitdir. V3 validates the worktree ID in `repair_gitfile()` to report "unrelated .git file" and skip repair on mismatch, stops `repair_worktree_at_path()` when the gitdir path cannot be inferred, and rewrites `get_worktree_id()` to handle separators more leniently.

### sigchain concurrent double-pop recovery

Ian Hu submitted a patch to `sigchain.c` fixing a race where two concurrent handlers popping the same signal both read the same chain depth, leaving the dispatcher stuck and spinning at 100% CPU until SIGKILL. The patch adds a `pushed` counter and reinstalls the pre-chain disposition when popping an already-empty chain; fault-injected A/B testing showed the unpatched code froze in 121 of 300 runs while the patched code never froze in 300 runs.

## In brief

- **parse-options early-scan API** -- Christian Couder posted v3 of his four-patch "Standardize early option scanning" series, adding `early_scan_options()` and the `PARSE_OPT_EARLY` flag so `git fast-import` can safely skip option values during early scans.
- **blame default-ignore-revs** -- Ravi Mistry posted v5 and v6 of the series teaching `git blame` to read `.git-blame-ignore-revs` from the working-tree root by default; Junio C Hamano approved the v6 design as "Very cleanly done".
- **`checkout -m` conflict labels** -- Phillip Wood posted v4 of his series teaching the ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` reuses them; Junio C Hamano said he will replace the previous version.
- **CI annotations naming failing test** -- Harald Nordgren's series linking CI failure annotations to the test script was marked ready for `next` on October 13.
- **odb/files alternates cleanups** -- Patrick Steinhardt's six-patch v2 series cleaning up the files object database backend was marked ready for `next` on October 16.
- **stash creation index loads** -- Phillip Wood posted v2 of a seven-patch series avoiding unnecessary index loads in `git stash push`/`create`, cutting lstat() calls per changed path by eight for push and four for create.
- **cache-tree with locked index** -- Szymon Wilczek submitted a patch dropping `LOCK_DIE_ON_ERROR` from `write_index_as_tree()` so `git write-tree` and `git am` proceed while a background `git status` holds the lock.
- **HTTP push with token-only URL** -- Dan reported that `git push` over smart HTTP fails when the remote URL carries a token-only userinfo after libcurl 8.23.0 stopped returning credentials in `CURLINFO_EFFECTIVE_URL`, and proposed comparing URLs after stripping userinfo.
- **repack --dry-run** -- Siddharth Shrimali posted v3 fixing `git repack --drop-filtered --dry-run` writing a new promisor pack instead of just listing candidates; Junio C Hamano said he will queue it as-is.
- **read-cache CRLF size change** -- Torsten Bögershausen posted v3/RFC of a fix so `ie_modified()` does not trust a size change for paths subject to CRLF conversion.
- **fast-import 'from' by oid** -- Marc Becker posted v4 allowing 'from' to reference new commits/tags by oid; Junio said "Will queue."
- **imap-send TLS updates** -- Beat Bolli posted v3 replacing deprecated SSLv23_method() with TLS_method() and using OpenSSL's built-in hostname verification.
- **worktree repair v2** -- Yoichi NAKAYAMA posted v2 making `git worktree repair` validate cross-references so it cannot break an unrelated `.git` file.
- **connectivity check with alternates** -- Igor Wiedler posted v2 fixing the connectivity check with alternates in deepening fetches by negating `--alternate-refs` and ignoring refs of shallow/grafted alternates; Jeff King called the approach exactly what he would have expected.
- **fsck bigFileThreshold** -- Patrick Steinhardt fixed a bogus fsck corruption report when a packed blob's size equals `core.bigFileThreshold`; Junio C Hamano queued it.
- **sequencer HEAD ref** -- Andy Koppe fixed the sequencer clobbering the HEAD ref name in worktree lookup by using `refs_resolve_refdup()`; queued.
- **bundle uninitialized memory** -- Olof Fairbrook fixed `git bundle create` freeing uninitialized stack memory on an unrecognized argument.
- **config blob 0xff** -- Olof Fairbrook made `config_buf_fgetc()` return blob bytes as `unsigned char` so `0xff` is no longer treated as EOF.
- **reflog --operations** -- Harald Nordgren posted v2 adding `--operations` and `--historical-branch` to `git reflog` and `git log -g`.
- **root refs in loose cache** -- Andy Koppe posted v2 keeping root refs such as `HEAD` out of the files backend's loose ref cache, fixing `git for-each-ref --include-root-refs --merged=main` silently omitting merged root refs.
- **test modernization** -- Srujith Poojary posted v3 replacing old-style `test -f` checks in `t/t2030-unresolve-info.sh` with `test_path_is_file` and `test_path_is_missing`.