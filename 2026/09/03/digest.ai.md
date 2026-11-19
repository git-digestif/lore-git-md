# Git mailing list daily digest for 2026/09/03

## Long threads

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread (root 2026/08/07/11-56-23): 44 replies so far from 6 participants over 28 days; 3 messages that day.*

This series exposes the build-time `USE_NSEC` knob as the runtime `core.useNanosec` configuration so distributors can ship one binary, reducing "racy git" hits in index-related commands like `git diff` on supporting filesystems. Today Ben Knoble declined to remove the remaining `NO_NSEC`-related `#ifdef` in a reroll, saying he and Jeff King agree it is not worth the hassle for now, and noted that getting the test apparatus to set the config globally would help scare out potential `BUG()`s from `repo_config_values()` on repositories other than `the_repository`. Junio C Hamano said he is less worried about the current code than about future features such as `diff` or `status` recursing into submodules in-process, noting that `git grep --recurse-submodule` not hitting a `BUG()` is weak assurance because grep does not specially handle racily clean entries. Ben Knoble argued that a safe default could alleviate that concern but that recent work in these areas should instead be impetus to lift the restriction, and that he is not inclined to change the shape of the series now.

### Introduce the `receive-report` hook for `git-receive-pack`

*Long thread (root 2026/08/18/07-55-55): 37 replies so far from 5 participants over 17 days; 7 messages that day.*

This series adds a `receive-report` hook that runs after the reference transaction completes but before the pkt-line report is sent to the client; the hook's stdout replaces the report, and a non-zero exit marks all references rejected. Today Karthik Nayak posted v6, which introduces `enum report_status_version` to replace static variables and rewords the documentation to explain why the reference-transaction hook cannot be used and the hook owner's responsibility to undo changes. Patrick Steinhardt offered nits: a switch statement instead of if-else chains for the enum, noting that `report()` and `report_v2()` are now identical except for the enum value, and unnecessary curly braces around a single-statement loop. Karthik Nayak agreed to fold these into a new version the following day if no other reviews arrive.

### Add `--drop-filtered` to `git repack` for partial clones

*Long thread (root 2026/07/16/13-28-41): 36 replies so far from 5 participants over 50 days; 1 message that day.*

This GSoC series lets users reclaim space in partial clones by enumerating promisor blobs over a size threshold, dropping them locally, and relying on lazy-fetch to restore them on demand, rebuilding the promisor pack crash-safely via the existing repack machinery. Today Samuel Bronson reported testing `--drop-filtered` on a real llvm-project clone not originally cloned with `--filter`, finding it worked well and greatly reduced the object database size. He proposed an RFC change to `builtin/repack` so that instead of dying when a blob referenced by the current index would be dropped, the operation skips it with a warning, arguing this is more convenient than canceling after the slow enumeration step and lets `git repack -a --filter=blob:limit=0 --drop-filtered` work in non-bare repositories.

### Refactor `checkout`/`switch`/`restore` code partitioning

*Long thread (root 2026/08/27/18-29-00): 27 replies so far from 5 participants over 8 days; 1 message that day.*

This 8-patch series repartitions the code so `cmd_switch()` and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()` instead of funneling through the monolithic `checkout_main()`, including moving option-compatibility checks into `checkout_paths()` and making `post_checkout_hook()` independent of `the_repository`. Today Karthik Nayak stated that there is merit in merging the patch down since it leaves the code in a better place than before, responding to Junio C Hamano's earlier remark that the refactoring only scratches the surface of finer-grained C-level reuse.

### Make `t/lib-httpd` CGI test helpers concurrency-safe

*Long thread (root 2026/07/08/02-59-40): 23 replies so far from 3 participants over 58 days; 1 message that day.*

This series fixes race conditions in the `t/lib-httpd.sh` helpers `apply-one-time-script.sh` and `http-429.sh`, which under concurrent Apache requests could return multiple modified responses or empty responses causing flaky failures such as t5616.47 on macOS CI, by chaining atomic operations like `rm` (without `-f`) with the logic they guard and using `mkdir` to elect a first request. Today Patrick Steinhardt reviewed the v5 series and stated that this version looks good to him.

## The day in brief

Karthik Nayak posted v6 of the `receive-report` hook series with an `enum report_status_version`, and Patrick Steinhardt posted a two-patch series refining the `rerere gc` maintenance heuristic with a default threshold of 512 stale entries. Junio C Hamano approved Harald Nordgren's v5 `checkout -m` autostash series, saying he will queue it, while Kristoffer Haugsbakk reported a 283-byte LeakSanitizer leak in `git history reword` bisected to commit 13763ecf.

## Notable threads

### Refine `checkout -m` autostash fallback and reserve exit status 1 for conflicts

*Long thread (root 2026/07/25/15-34-27): 22 replies so far from 4 participants over 41 days; 8 messages that day.*

This series makes `checkout -m` avoid autostash retries when no tracked local changes exist, separates autostash conflict advice from the branch-switch message, and makes conflicts exit with status 1 like `merge-tree` while other failures exit 128. Today Harald Nordgren posted v5 with improved commit messages and a new `do_stash_apply` helper in `sequencer.c` mapping `run_command()` results to `enum stash_apply_result`. Phillip Wood reviewed v4, keeping the useful return-path audit while calling the commit message overly verbose, and Junio C Hamano reviewed v5, saying both patches look good to him and that he will queue them.

### Improve the heuristic for the `rerere gc` maintenance task
This two-patch series extracts staleness logic in `rerere.c` and refines the `rerere-gc` auto heuristic so the task runs only when the estimated number of stale prunable entries reaches the configured threshold, redefining positive `maintenance.rerere-gc.auto` values as that minimum count with default 512 and sampling only `rr-cache` entries starting with "17" to bound stat overhead. Today Patrick Steinhardt posted the series; Thomas Bachem built it and confirmed the default-512 behavior with a `Tested-by` tag, and Derrick Stolee reviewed both patches, concluding "Both patches LGTM."

### Fix pathspec prefixes with exclusions in `dir.c`

*Long thread (root 2026/09/02/13-04-09): 10 replies so far from 2 participants over 2 days; 12 messages that day.*

This series fixes pathspec prefix optimization so exclude items are matched against the full pathname and the common prefix is found starting from the first non-exclude item, plus a preparatory const-correctness patch for `pathspec_item`. Today Yannik Tausch posted v3 adding Junio C Hamano's const patch as 1/3 to fix a unit-test build error; Junio said the const patch will be queued separately, told Yannik to simply ignore v3 if the other two patches are unchanged, and Yannik replied "Please ignore v3 then."

### `git history reword` surfaces a memory leak
Kristoffer Haugsbakk reported that `git history reword`, with or without `--dry-run`, triggers a 283-byte LeakSanitizer-detected leak allocated in `unpack_compressed_entry` and surfaced via `repo_logmsg_reencode` in `commit_tree_ext` (`builtin/history.c`), reproducing on `master`, `next`, and `seen`. Bisecting identified commit 13763ecf (merge of `ps/receive-pack-shallow-optim`) as the first bad commit.

### Fix `--force-if-includes` when the remote-tracking ref has no reflog
Aleksei Sviridkin posted a patch fixing `is_reachable_in_reflog()` in the push code, where an uninitialized `timestamp_t date` cut off the reflog walk when the remote-tracking ref has no reflog, causing spurious "remote ref updated since checkout" rejections; the fix initializes it to 0 plus a new test in `t/t5533-push-cas.sh`. Junio C Hamano reviewed the code change as good but objected to the bare `Assisted-by: LLM` trailer and asked for the sign-off last; Sviridkin defended the trailer citing kernel practice and his multi-model routing, and said v2 will follow.

## In brief
- Thomas Bachem's patch keeps a background `rerere gc` from killing a rebase by making `setup_rerere()` wait up to `rerere.lockTimeout` then warn and return -1 instead of dying; Patrick Steinhardt and Phillip Wood agreed the grace period is sensible alongside a separate maintenance-heuristic change.
- Michael Montalbo posted v3 of the series scoping `--stat`, `--check`, and `-G` to `-L` line ranges, enabling `git log -L:func:file --stat` and `-G` searches within a single function, with v3 simplifying tests and prose and fixing a line-classification bug.
- Andrew Pleeter posted v3 of `git var` adding broken-down ident variables, `GIT_DEFAULT_KEY` with a `GIT_SIGNING_KEY` alias, multiple variable arguments, and `-z`; Junio C Hamano endorsed multiple arguments but proposed doc rewording and code-level changes including `parse_options()`.
- Patrick Steinhardt reviewed Jinbao Chen's fix for a NULL dereference in `write_ondisk_index()` (`builtin/history.c`) when a parent tree object is missing, saying v2 looked good to him.
- Elijah Newren responded to Derrick Stolee's reported bug in the shallow-clone push optimization, calling it a good discovery but arguing the test-case description is self-contradictory and that re-running it shows no receiver corruption, while agreeing the error messages ought to be improved.
- Aleksei Sviridkin fixed `pack-objects` prefetch ordering so `prefetch_to_pack()` indexes the sorted array, reducing fetches from 8 individual fetches to 1 batched fetch when packing absent blobs alongside locally present loose objects.
- Aleksei Sviridkin patched `contrib/contacts/git-contacts` to skip `git blame` boundary commits when collecting contacts, with measurements showing twenty-seven contact lists lose a name and nine become empty.
