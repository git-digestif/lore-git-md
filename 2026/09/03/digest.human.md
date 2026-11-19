# Git mailing list daily digest for 2026/09/03

## Long threads

### Convert build-time USE_NSEC into runtime `core.useNanosec`

*Long thread: 44 replies so far from 6 participants over 28 days; 3 messages that day.*

This series exposes the build-time `USE_NSEC` knob as the runtime `core.useNanosec` config, so distributed Git builds can avoid racy-git problems in index-related commands like `git diff`. Today Ben Knoble declined to remove the remaining `NO_NSEC`-related `#ifdef` in a reroll, agreeing with Jeff King that it is not worth the hassle. Knoble also said that making the test apparatus set the config globally would help scare out potential `BUG()`s from `repo_config_values()` on repositories other than `the_repository`. Junio C Hamano said his concern is future in-process recursion into submodules, as `git grep --recurse-submodule` already does, and Knoble argued that recent work in these areas should instead be impetus to lift the restriction rather than paper over it with a safe default.

### Introduce the receive-report hook

*Long thread: 37 replies so far from 5 participants over 17 days; 7 messages that day.*

This series adds a `receive-report` hook for `git-receive-pack` that runs after the reference transaction completes but before the report reaches the client, receiving the pkt-line report on stdin and replacing it with its stdout; a non-zero exit marks all references rejected. Karthik Nayak posted v6, introducing `enum report_status_version` to replace static variables and rewording the docs to explain why the reference-transaction hook cannot be used. Patrick Steinhardt offered nits -- a switch statement for the enum, noting `report()` and `report_v2()` are now identical except for the enum value, and unnecessary curly braces -- and Nayak agreed to fold these into a new version.

### Add `--drop-filtered` to `git repack` for partial clones

*Long thread: 36 replies so far from 5 participants over 50 days; 1 message that day.*

This GSoC series lets users reclaim space in partial clones by enumerating promisor blobs over a size threshold, dropping them locally, and relying on the existing lazy-fetch machinery to restore them. Samuel Bronson reported testing `--drop-filtered` on a real llvm-project clone not originally cloned with `--filter`, finding it worked well and greatly reduced the object database size. He proposed an RFC change so that instead of dying when a blob referenced by the current index would be dropped, the operation skips it with a warning, allowing `git repack -a --filter=blob:limit=0 --drop-filtered` to work in non-bare repositories.

### Refactor checkout/switch/restore dispatch

*Long thread: 27 replies so far from 5 participants over 8 days; 1 message that day.*

This 8-patch series repartitions the monolithic `checkout_main()` so `cmd_switch()` and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()`. Karthik Nayak stated that there is merit in merging the patch down since it leaves the code in a better place than before.

### Make t/lib-httpd CGI test helpers concurrency-safe

*Long thread: 23 replies so far from 3 participants over 58 days; 1 message that day.*

This series fixes race conditions in the `apply-one-time-script.sh` and `http-429.sh` test helpers, which can misbehave under concurrent Apache requests, as seen in flaky t5616 failures on macOS CI. Patrick Steinhardt reviewed v5 and said the version looks good to him.

## The day in brief
Karthik Nayak posted v6 of the receive-report hook series with an enum-based report version, while Ben Knoble held the line on the `core.useNanosec` series' remaining `#ifdef`. Samuel Bronson validated `git repack --drop-filtered` on a real large clone and proposed softening its index guard, and Patrick Steinhardt both approved the lib-httpd concurrency fixes and posted a new heuristic for `rerere gc` auto-maintenance.

## Notable threads

### Keep a background `rerere gc` from killing a rebase
Thomas Bachem's patch addresses a race where the background `git rerere gc` spawned by auto-maintenance collides with the sequencer's `MERGE_RR.lock` during `git rebase --continue`. Patrick Steinhardt proposed both a locking grace period and a less aggressive `maintenance.rerere-gc.auto` heuristic; Bachem agreed to a wait-then-warn approach in v2, and Phillip Wood endorsed both the grace period and the heuristic change, while supporting disabling auto-maintenance during a rebase as a separate topic.

### Avoid sending the whole tree when pushing from a shallow clone
Elijah Newren's patch stops shallow clones from pushing the entire toplevel tree. Responding to Derrick Stolee's reported bug, Newren called it a good discovery but argued the test description was self-contradictory: re-running it showed no receiver corruption (`git fsck` passes, the ref was safely rejected), though the error message is inscrutable, and he said he has patches to improve those messages.

### Fix `--force-if-includes` when the remote-tracking ref has no reflog
Aleksei Sviridkin posted a patch initializing a previously uninitialized timestamp to 0 in `is_reachable_in_reflog()`, so a remote-tracking ref without a reflog no longer cuts the walk short and wrongly rejects the push. Junio C Hamano reviewed the code as good but flagged the `Assisted-by: LLM` trailer placement; Sviridkin defended the bare trailer, citing kernel practice, and said v2 will put the sign-off last.

## In brief
- Yannik Tausch resubmitted the pathspec-prefix-with-exclusions fix as v3 including Junio C Hamano's const-correctness patch; Hamano said the const patch will be queued separately and v3 can be ignored. *Long thread: 10 replies so far from 2 participants over 2 days; 12 messages that day.*
- Harald Nordgren posted v5 of the `checkout -m` autostash refinement; Junio C Hamano said both patches look good and will queue them. *Long thread: 22 replies so far from 4 participants over 41 days; 8 messages that day.*
- Patrick Steinhardt posted a two-patch series refining the `rerere gc` auto heuristic to run only when estimated stale entries reach a threshold (default 512); Derrick Stolee said both patches look good to him.
- Michael Montalbo posted v3 of the series scoping `--stat`, `--check`, and `-G` to `-L` line ranges, with simplified tests and a bug fix in the line-range filter.
- **pack-objects prefetch fix**: Aleksei Sviridkin fixed `pack-objects` prefetch ordering so the batch starts correctly; his test with 2 absent and 16 loose objects now uses 1 fetch instead of 2.

- Andrew Pleeter posted v3 of `git var` extensions (broken-down idents, `GIT_DEFAULT_KEY`, multiple args, `-z`); Junio C Hamano endorsed multiple arguments and raised doc and code-level points.
- Jinbao Chen posted v2 guarding a NULL tree in `write_ondisk_index()` so `git history split` errors gracefully instead of crashing when the parent tree is missing.
