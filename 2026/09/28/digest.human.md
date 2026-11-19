# Git mailing list daily digest for 2026/09/28

## Long threads

### stash: clean up index-mode test merge

*Long thread: 42 replies so far from 4 participants over 10 days; 14 messages that day.*

D. Ben Knoble's series fixes a bug where autostashing with staged index entries and `stash.index=true` hit a non-re-entrant `save_autostash()` lock, replacing the shell-out sequence in `builtin/stash.c` with an in-core `merge_incore_nonrecursive` call. Today Junio C Hamano flagged a NULL-dereference risk in patch 5/5 if the merge fails without populating `result.tree`, which Knoble acknowledged; Hamano later confirmed all existing callers check `result.clean < 0` first. Phillip Wood investigated the t5520.69 failure in `seen` (reported by Hamano on 9/27), showing the rebase upstream changed; Knoble bisected it to commit 2d1fa0323f (auto maintenance after rebase), and Thomas Bachem confirmed the root cause: the series removes eight HEAD reflog entries because `stash apply --index` no longer spawns `git reset --quiet --refresh`, shifting auto reflog expiry past the entry the test needs. Phillip Wood also suggested squashing the new t3903 test into the final patch, since it fails without it.

### sequencer: leave auto maintenance to the end of a sequence

*Long thread: 33 replies so far from 5 participants over 25 days; 1 message that day.*

Thomas Bachem's v5 series makes merge-backend rebase sequences run auto maintenance once at completion, like the apply backend, by calling `run_auto_maintenance()` from `builtin/rebase.c` and `builtin/revert.c` and passing `maintenance.auto=false` via `GIT_CONFIG_PARAMETERS` to spawned commit, merge and exec commands. Today Patrick Steinhardt noted the series is already cooking in `next` and agreed it looks ready.

### Introduce 'uploadpack.lazyFetchTrusted'

*Long thread: 29 replies so far from 3 participants over 81 days; 5 messages that day.*

Christian Couder posted v4 of the series adding a protected config variable that lets server operators mark repositories whose git dir is trusted for lazy fetching during `upload-pack`, preventing clone failures when the server has moved large blobs to a promisor remote. The most significant change in patch 5/5 is that `builtin/upload-pack.c` now checks `GIT_NO_LAZY_FETCH` only after entering the repo and disables lazy fetching only if the repo is not in the allowlist; the other patches refactor `lazy_fetch_objects()` out of `promisor-remote.c`, extract `path_allowlist_apply()` helpers in `setup.c`, and add a `GIT_INTERNAL_LAZY_FETCH_DEPTH` limit of 5 against infinite recursion.

### refs: report old values to transaction hooks

*Long thread: 27 replies so far from 5 participants over 10 days; 2 messages that day.*

Maciej Ciemborowicz's v6 replaces the earlier caller-specific approach with a single transaction-layer change so the reference-transaction hook always receives real old object IDs, resolved before the "preparing" hook and refreshed after the backend locks the refs. Today Patrick Steinhardt clarified that the proper fix is for reference transactions to always resolve the status quo and supply old OIDs to the reftx hook regardless of caller input, avoiding interface changes at any callers.

### rerere: wait for MERGE_RR.lock, and go on at a conflict

*Long thread: 20 replies so far from 4 participants over 27 days; 3 messages that day.*

Thomas Bachem posted v5 of the three-patch series: every rerere writer waits `rerere.lockTimeout` (default 1000 ms) for `MERGE_RR.lock` before failing; a new `git rerere gc --auto`, passed by `git maintenance run --auto` and `git gc --auto`, quietly skips a held lock while a manual `git rerere gc` waits and fails; and commands that stop at a conflict warn and go on without rerere, with the hint routed through `advise_if_enabled()`.

## The day in brief
Christian Couder posted v4 of `uploadpack.lazyFetchTrusted`, Thomas Bachem posted v5 of the rerere lock series, and Patrick Steinhardt confirmed the sequencer auto-maintenance series is cooking in `next`. A t5520 failure traced to the stash topic was bisected to the auto-maintenance commit, and Patrick Steinhardt called the reftable reflog timezone storage "plain wrong" and argued Git should be fixed to match the specification.

## Notable threads

### Reftable reflog timezone encoding differs from specification
Josh McKinney reported that Git 2.55.0 stores reftable reflog timezone offsets as signed HHMM integers (530 for +05:30) rather than the signed minutes (330) the reftable specification requires, reproduced on macOS arm64. Patrick Steinhardt confirmed the issue, called the current storage "plain wrong", and argued the specification's minutes representation should be used and Git fixed, since JGit reads the value as minutes; Junio C Hamano agreed Git should be fixed, noting reflogs are not shared across repositories.

### http: handle curl stripping creds from effective url
Jeff King posted a patch because upcoming curl 8.23.0 strips credentials from `CURLINFO_EFFECTIVE_URL`, which would make Git's direct string comparison falsely detect a redirect and flush `http_auth` credentials, causing re-prompting. Patrick Steinhardt suggested `CURLINFO_REDIRECT_COUNT` as a more direct check, but Jeff King reported it does not work because curl counts the automatic auth retry as a redirect; Junio C Hamano called the `curl_url()`-based `match_effective_url()` approach "very nice".

### connected: add incremental connectivity check via rev-list
Kristofer Karlsson posted v2 of the opt-in `transfer.connectivityCheck=incremental` mode, which runs the verifier inside the rev-list subprocess in topological order against trusted parent trees. The cover letter reports up to 190x speedup and 14 MB RSS versus 3.4 GB for single-commit pushes to a 3.4 GB boundary-closure repo, with deepening fetches falling back to the full check.

## In brief
- Johannes Schindelin proposed a compile-time option to use the new Rust `sha1dc` crate, reporting roughly three times faster `git index-pack` on Windows, with a `core.sha1dcBackend=c` escape hatch.
- Patrick Steinhardt said the `history: sign rewritten commits` series is fine as-is; Junio C Hamano asked whether "Souma" is a real name, and Souma confirmed it is their legal first name.
- Patrick Steinhardt suggested the CI script should fail outright when perforce, git-lfs or JGit fail to install rather than warn; Harald Nordgren agreed.
- Maya Barriault reported that `git -C other_repo` run via an alias from a worktree sets `GIT_DIR` and `GIT_COMMON_DIR` to the worktree's values instead of the target repository's.
- Junio C Hamano's "What's cooking" notes Git 2.56 has been tagged and the next version will be Git 2.98, followed by 2.99 in preparation for Git 3.0.
