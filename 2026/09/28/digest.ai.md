# Git mailing list daily digest for 2026/09/28

## Long threads

### stash: clean up index-mode test merge

*Long thread (root 2026/09/19/21-26-34): 42 replies so far from 4 participants over 10 days; 14 messages that day.*

D. Ben Knoble's v3 series fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true` in `builtin/stash.c`, with the final patch replacing a shell-out sequence of `diff-tree`/`apply --cached`/`reset` with an in-core `merge_incore_nonrecursive()` call. Today Junio C Hamano flagged a NULL-dereference risk in patch 5/5: if `merge_incore_nonrecursive()` fails catastrophically it may set `result.clean` to -1 without populating `result.tree`, so the following `oidcpy()` would dereference NULL; he later noted all current callers check `result.clean < 0` before touching other result members. Phillip Wood investigated the t5520.69 failure Junio had reported when merging the topic into `seen`, noting the test does not stash anything and tracing a changed upstream commit via GIT_TRACE. D. Ben Knoble's bisect traced the failure to commit 2d1fa0323f (auto maintenance after rebase): the series removes eight HEAD reflog entries because `stash apply --index` no longer spawns `git reset --quiet --refresh`, shifting the 100-entry auto reflog expiry past the entry the test needs. Thomas Bachem confirmed the root cause and proposed pinning `gc.reflogExpire`/`gc.reflogExpireUnreachable` to never. D. Ben Knoble acknowledged the NULL-dereference concern and a verbosity-restoration issue, and Phillip Wood suggested squashing the 3/5 test into the final patch since it fails without patch 5.

### sequencer: leave auto maintenance to the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 33 replies so far from 5 participants over 25 days; 1 message that day.*

Thomas Bachem's v5 series moves auto-maintenance invocation out of the sequencer's spawned commands: `run_auto_maintenance()` is called from `builtin/rebase.c` and `builtin/revert.c` when a sequence completes, while `disable_auto_maintenance()` appends `maintenance.auto=false` to `GIT_CONFIG_PARAMETERS` for spawned `git commit`, `git merge`, and exec commands. Today Patrick Steinhardt noted the series is already in `next` and agreed it looks ready.

### Introduce 'uploadpack.lazyFetchTrusted'

*Long thread (root 2026/07/10/08-51-34): 29 replies so far from 3 participants over 81 days; 5 messages that day.*

Christian Couder posted v4 of the series adding the protected config variable `uploadpack.lazyFetchTrusted`, which lets server operators mark repositories whose git dir is trusted for lazy fetching when serving clones and fetches via `builtin/upload-pack.c`. The most significant change in patch 5/5 is that `GIT_NO_LAZY_FETCH` is now checked after entering the repo and lazy fetching is disabled only if the repo is not trusted; patch 4/5 adds `GIT_INTERNAL_LAZY_FETCH_DEPTH` with a limit of 5 to prevent infinite recursion when a promisor remote resolves back to the served repository.

### refs: report old values to transaction hooks

*Long thread (root 2026/09/19/13-34-15): 27 replies so far from 5 participants over 10 days; 2 messages that day.*

Maciej Ciemborowicz's v6 replaces the earlier caller-specific approach with a single transaction-layer change in `refs.c`: old values for the reference-transaction hook are resolved before the "preparing" hook and refreshed after the backend locks the refs, only when such a hook exists, kept in separate fields so they cannot constrain the update. Today Patrick Steinhardt clarified that the proper fix is for reference transactions to always resolve the status quo and supply old object IDs to the reftx hook regardless of caller input, calling the common-hook-layer approach sensible.

### rerere: wait for MERGE_RR.lock, and go on at a conflict

*Long thread (root 2026/09/02/08-31-37): 20 replies so far from 4 participants over 27 days; 3 messages that day.*

Thomas Bachem posted v5 as three patches: every rerere writer waits `rerere.lockTimeout` (default 1000 ms) for `MERGE_RR.lock` before failing; a new `git rerere gc --auto` (passed by `git maintenance run --auto` and `git gc --auto`) quietly skips a held lock while a manual `git rerere gc` waits and fails; and commands that stop at a conflict warn and go on without rerere, with the hint routed through `advise_if_enabled()` under `advice.mergeConflict`.

## The day in brief
Git v2.56.0 was tagged and announced, with 748 non-merge commits since v2.55.0. Christian Couder posted v4 of `uploadpack.lazyFetchTrusted`, and the stash index-merge series drew fresh review findings including a NULL-dereference risk and a t5520 regression traced to reflog expiry. Thomas Bachem posted v5 of the rerere lock-wait series.

## Notable threads

### Git v2.56.0 release announcement
Junio C Hamano announced Git v2.56.0, comprising 748 non-merge commits since v2.55.0 contributed by 104 people, 39 of them new contributors.

### Reftable reflog timezone encoding differs from specification
Josh McKinney reported that Git 2.55.0 stores reftable reflog timezone offsets as signed HHMM integers (530 for +05:30) rather than the signed minutes (330) the reftable specification requires, reproduced on macOS arm64. Patrick Steinhardt confirmed the issue, called the current storage "plain wrong", and argued the specification's minutes representation should be used with Git fixed, since JGit reads the int16 as minutes; Junio C Hamano agreed Git should be fixed, noting reftable files are not shipped over networks.

### http: handle curl stripping creds from effective url
Jeff King posted a patch for the upcoming curl 8.23.0, which strips credentials from `CURLINFO_EFFECTIVE_URL`: the direct string comparison against the requested URL would falsely detect a redirect and flush `http_auth` credentials, causing re-prompting. The patch adds `match_effective_url()`, which strips credentials from the requested URL via curl's `curl_url()` interface and compares again. Jeff King later reported that Patrick Steinhardt's `CURLINFO_REDIRECT_COUNT` alternative does not work because curl counts the automatic auth retry as a redirect.

### connected: add incremental connectivity check

Kristofer Karlsson posted v2 of the opt-in `transfer.connectivityCheck=incremental` mode, running the verifier in the rev-list subprocess via `--verify-trees-incremental` and processing incoming commits in topological order against trusted parent trees. Benchmarks on a 3.4 GB boundary-closure repo report up to 190x speedup and 14 MB RSS versus 3.4 GB for one incoming commit; synthetic p5412 fixtures show modest regression at 10000 commits. Deepening fetches fall back to full check.

### Optional Rust sha1dc backend
Johannes Schindelin proposed a four-patch series adding a compile-time `DC_SHA1_RS` option to use the new Rust `sha1dc` crate for collision-detecting SHA-1, reporting roughly three times faster `git index-pack` on Windows from SIMD optimizations, with a `core.sha1dcBackend=c` escape hatch enabled by default.

## In brief
- Git for Windows 2.56.0 is available; Windows 8.1 support was dropped, moving `git.exe` from `/mingw64/bin/` to `/ucrt64/bin/`.
- Junio C Hamano's "What's cooking" notes Git 2.56 has been tagged and the next version will be Git 2.98, followed by 2.99 ahead of Git 3.0.
- Patrick Steinhardt said the `history: sign rewritten commits` series is fine as-is; Junio C Hamano asked whether "Souma" is a legal name, and Souma confirmed it is.
- Patrick Steinhardt suggested CI should fail outright when perforce, git-lfs, or JGit fail to install; Harald Nordgren agreed.
- Patrick Steinhardt is happy with the reflog default-expiry fix and agreed it need not be fast-tracked for 2.56.
- Junio C Hamano marked the GitLab Windows CI Rust-setup series for `next`.
- Patrick Steinhardt suggested rebasing the `replay: add signing support` series onto the in-flight commit-signing infrastructure once it lands.
- **v2 of create_repository() stateless series posted**: Patrick Steinhardt posted v2; reviewers debated free(3p) notation. The 'good read' praise was for v1. *Long thread (root 2026/09/24/09-19-18): 16 replies so far from 4 participants over 5 days; 13 messages that day.*

- Maya Barriault reported that `git -C other_repo` run via an alias from a worktree sets `GIT_DIR`/`GIT_COMMON_DIR` to the worktree's values.
- Julia Evans is updating the new `gitmergeconflicts` documentation series, including restoring the `CHERRY_PICK_HEAD` mention in git-cherry-pick.
