## Long threads

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 44 replies so far from 6 participants over 28 days.*

D. Ben Knoble's series converts the compile-time `USE_NSEC` option into the runtime `core.useNanosec` configuration, replacing `#ifdef` checks in `read-cache.c` and `statinfo.c` so distributed builds can use nanosecond timestamps to reduce racy-git problems in index-related commands. Patrick Steinhardt indicated the series is ready for `next`, and Jeff King clarified that under `NO_NSEC` builds the setting is a noop since nsec entries are faked as zero. Junio C Hamano raised a concern about `repo_config_values()` potentially hitting a `BUG()` on repositories other than `the_repository` during submodule recursion, while Knoble argued recent work should instead lift that restriction. By week's end Knoble said he would not reroll absent strong demand.

### Add `--linearize` to `git-replay(1)`

*Long thread: 39 replies so far from 7 participants over 86 days.*

Toon Claes's series adds a `--linearize` option to `git-replay(1)` that drops merge commits and replays reachable commits into a single linear history, updating refs that point at merges to their last replayed ancestor. V9 rephrased "multiple revision ranges" as "multiple branches" in error messages and simplified the documentation after reviewer requests. Elijah Newren reviewed v9 and added a Reviewed-by trailer, saying the version is good to go.

### Introduce the `receive-report` hook for `git-receive-pack`

*Long thread: 37 replies so far from 5 participants over 18 days.*

Karthik Nayak's series adds a `receive-report` hook that runs after the reference transaction completes but before the report reaches the client, letting the hook's stdout replace the pkt-line report and a non-zero exit reject all references. V7 rebased on master, removed the redundant `report_v2()`, and switched on a new `report_status_version` enum replacing the `bool detailed_report` parameter. Junio C Hamano questioned the semantics when the hook reports failure, noting the pusher's remote-tracking branches would reflect a failed push although the server already committed.

### Add `--drop-filtered` to `git repack` for partial clones

*Long thread: 37 replies so far from 5 participants over 51 days.*

This GSoC series lets users reclaim space in partial clones by enumerating promisor blobs over a size threshold, dropping them locally, and relying on lazy-fetch to restore them. Samuel Bronson tested it on a real llvm-project clone and proposed skipping index-referenced blobs with a warning instead of aborting; Siddharth Shrimali countered that aborting after the full walk is a poor trade-off but suggested a separate skip-set so `--dry-run` still lists those blobs.

### Packfile recovery when a MIDX names a removed pack

*Long thread: 36 replies so far from 5 participants over 15 days.*

Elijah Newren's v3 series fixes objects being reported missing when a stale multi-pack-index names a pack removed by a concurrent geometric repack. Patch 4/4 scans the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ` with a tri-state return from `midx_fill_entry()`. Derrick Stolee completed his review with "Good patch. LGTM.", and Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`.

### Add `path.*` keys to `git repo info`

*Long thread: 31 replies so far from 5 participants over 52 days.*

K Jayatheerth's GSoC series extends `git repo info` with `path.*` keys for repository components like `path.toplevel`, `path.superproject-root`, and `path.index`, supporting absolute and relative suffixes. Junio C Hamano identified that `get_superproject_working_tree()` uses the process working directory rather than the target repository, so `git --git-dir=... repo info path.superproject-root.absolute` returns empty; Jayatheerth committed to updating the function to take a `repo` parameter in v6.

### Repartition checkout/switch/restore away from monolithic `checkout_main()`

*Long thread: 27 replies so far from 5 participants over 8 days.*

Junio C Hamano's eight-patch draft reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()` instead of the monolithic `checkout_main()`. Karthik Nayak agreed the series leaves the code in a better place, and Junio confirmed the changed guards are safe because conflicting option combinations are already rejected earlier, though he noted the series merely scratches the surface of making commands reusable.

### Refine `checkout -m` autostash fallback

*Long thread: 26 replies so far from 4 participants over 42 days.*

Harald Nordgren's series makes autostash conflicts in `sequencer.c` exit with status 1 while other failures exit 128, separating conflict advice from the branch-switch message. V5 was queued earlier in the week, and the remaining discussion centered on CI style tooling, with Junio C Hamano showing that `RemoveBracesLLVM` cannot add braces to a whole if/else cascade.

## The period in brief

The week of 2026/08/31 through 2026/09/06 saw heavy traffic across seven active days, with several long-running series reaching review milestones. Readers should not miss the runtime `core.useNanosec` conversion nearing `next`, the `receive-report` hook reaching v7, and Elijah Newren's shallow-clone push optimization drawing scrutiny over a multi-ref flaw.

## Key developments

### Shallow-clone push optimization draws multi-ref scrutiny

Elijah Newren posted v3 of a six-patch series keeping `send-pack` from transferring the whole tree when pushing from a shallow clone, introducing a tri-state `push.shallowExcludeBoundary` option and new push advice. Derrick Stolee reported that an AI-agent review found a multi-ref flaw where `append_reachable_shallow_grafts` treats shallow grafts as reachable without checking the remote's advertised refs, supplying a failing test; Newren countered that the case is not corruption because receive-pack checks connectivity regardless, and that `push.negotiate=true` can also fail under concurrent repacks.

### Background `rerere gc` versus rebase lock race

Thomas Bachem refined his patch addressing the race where auto-maintenance starts `git rerere gc` during a rebase, causing the sequencer's `repo_rerere()` to die on `MERGE_RR.lock`. After Junio C Hamano objected that proceeding without recording a preimage is a hard-to-accept regression, Bachem conceded in v4 that every caller should wait the timeout and then fail, with only `git rerere gc` giving up at once, while Phillip Wood endorsed disabling auto-maintenance during rebase as a separate topic.

### `--force-if-includes` fixes consult the correct ref

Tyler Cipriani posted a two-patch series fixing `remote.c` so `--force-if-includes` checks the reflog of the ref being pushed rather than a same-named local branch, resolving false rejections and potential data loss. Aleksei Sviridkin separately posted v3 initializing an uninitialized timestamp to 0 in `is_reachable_in_reflog()` so a remote-tracking ref without a reflog no longer wrongly rejects the push; Junio C Hamano questioned whether 0 is a reasonable fallback while Sviridkin argued for zero over "now".

### `last-modified` Bloom-filter series ejected from `seen`

Toon Claes posted v3 and v4 of a six-patch series reusing `git-log`'s Bloom-filter machinery to speed up `git-last-modified(1)` on directories, with benchmarks showing up to 3.74x speedup. Junio C Hamano reported that after the series was pushed to `seen`, CI failed reproducibly under `ci/run-build-and-tests.sh` test variables and the topic was ejected.

### What's cooking and topic evictions

In the August "What's cooking" thread, Patrick Steinhardt asked to evict the `ps/fetch-packfile-uris-parallel` topic because it conflicts with an upcoming object-transaction series for `git-fetch-pack(1)`, and Junio C Hamano replied "Will do". Steinhardt also agreed with Justin Tobler to drop the fetch-pack parallelizing series for now and resend once Tobler's ODB transaction series lands.

### Ref storage terminology rename draws naming objection

Patrick Steinhardt posted an 11-patch series renaming `--ref-format=` to `--ref-storage=`, `GIT_REFERENCE_BACKEND` to `GIT_REF_STORAGE`, and `init.defaultRefFormat` to `init.defaultRefStorage`, keeping old names as hidden aliases. Junio C Hamano objected that neither "ref-storage" nor "ref-format" is clear unless everything becomes "ref-storage-format".

### Outreachy December 2026 cohort

Christian Couder announced he will submit a Community Application for Git for Outreachy's December 2026 cohort, with Usman, Kaartic and Pablo volunteering as co-mentors, and proposed two intern projects: removing global state and improving command argument scanning via the parse-options API.

## In brief

- **Sequencer `--empty` handling**: Phillip Wood reviewed Farid Zakaria's v4 making the sequencer honor `--empty` when a `fixup!`/`squash!` empties its target, calling the approach sound.
- **`git var` extensions**: Andrew Pleeter posted v3 of extensions adding broken-down idents, `GIT_DEFAULT_KEY`, multiple args, and `-z`; Junio C Hamano endorsed multiple arguments.
- **Worktree basename fixes**: Junio C Hamano marked the four-patch `worktree_basename()` fixes for `next`, saying the topic was in good shape.
- **Debian CI bump**: Jeff King proposed changing the debian-11 CI job to debian-12 since Debian 11 exited LTS; Junio C Hamano will queue the patch.
- **Git version after 2.56**: brian m. carlson weighed naming options after Git 2.56, leaning toward Git 3.0 or 2.97, and listed SHA-256 forge support and libgit2 readiness as considerations.
- **Pathspec out-of-bounds fix**: Yannik Tausch's patch fixes an out-of-bounds read in `dir.c` when a negative pathspec is shorter than the positive prefix; disclosed to the security list and confirmed not to require embargo.
- **`git history split` crash fix**: Jinbao Chen fixed a SIGSEGV when the parent tree object is missing, erroring gracefully instead of dereferencing NULL.
- **`git imap-send --draft`**: Wolfgang Faust added a `--draft` option to set the IMAP `\Draft` flag.
- **`rev-list --missing-only`**: Siddharth Asthana posted v2 of a filter flag requiring `--missing=print`, explaining Gitaly uses the missing list as a one-shot dependency set.
- **Rust in Windows CI**: Johannes Schindelin posted a two-patch series enabling Rust in Windows CI jobs under MSYS2/MinGW.
- **`early_scan_options()`**: Christian Couder posted a six-patch series adding the sub-API to parse-options, fixing `--` detection in bisect and rev-parse; Junio C Hamano called the fast-import fix "Nicely done".
- **Alternates at ODB creation**: Patrick Steinhardt posted v2 writing alternates at object-database creation time via `odb_source_create_on_disk()`.
- **`gitdatamodel(7)` docs**: Kristoffer Haugsbakk posted v3 advertising the data model guide as a concept guide and linking glossary terms to it.
- **Test modernization objection**: Junio C Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals in Mark C. Chu-Carroll's cleanup.
- **`--range-diff-notes` docs**: Kristoffer Haugsbakk circulated a draft rewrite; Junio C Hamano proposed `--no-range-diff-notes` mean no notes are used, mirroring `--no-notes`.
- **Hooks `allowNoVerify`**: Junio C Hamano rejected Alessio Attilio's `hooks.allowNoVerify` proposal as a slippery slope; brian m. carlson agreed hooks are trivially bypassable.
- **Pack-objects prefetch**: Aleksei Sviridkin fixed prefetch ordering so the batch starts correctly, reducing fetches from 2 to 1 in his test.