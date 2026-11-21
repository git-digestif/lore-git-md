## Long threads

### Convert build-time USE_NSEC into runtime `core.useNanosec`
*Long thread: 44 replies so far from 6 participants over 28 days.*
D. Ben Knoble's series converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration, replacing `#ifdef` checks in `read-cache.c` and `statinfo.c` so distributed builds can use nanosecond timestamps to reduce racy-git problems. The series reached v6 with Patrick Steinhardt saying "Works for me" on readiness for `next`. Junio C Hamano raised concerns about `repo_config_values()` potentially `BUG()`ing on repositories other than `the_repository` during submodule recursion, while Jeff King clarified that `NO_NSEC` builds fake nsec entries as zero. Knoble declined to remove the remaining `#ifdef`, agreeing it was not worth the hassle, and said he would not reroll absent strong demand.

### `git-replay(1)` gains a `--linearize` option
*Long thread: 39 replies so far from 7 participants over 86 days.*
Toon Claes's series adds a `--linearize` option to `git-replay(1)` that drops merge commits and replays reachable commits into a single linear history, mimicking `git rebase --no-rebase-merges`. The v9 revision rephrased "multiple revision ranges" as "multiple branches" in error messages after reviewer feedback. Elijah Newren reviewed v9 and said "I think this version is good to go," adding a `Reviewed-by` trailer. The series updates refs pointing at merges to their last replayed ancestor and rejects multiple branches and `--contained`.

### Introduce the `receive-report` hook for `git-receive-pack`
*Long thread: 37 replies so far from 5 participants over 18 days.*
Karthik Nayak's series adds a `receive-report` hook that runs after the reference transaction completes but before the report reaches the client, reading the pkt-line report on stdin and replacing it with its stdout; a non-zero exit marks all references rejected. The series progressed through v7, introducing `enum report_status_version` to replace static variables and switching on the enum. Junio C Hamano questioned the `bool detailed_report` parameter, suggesting a protocol-version enum, and challenged the rationale for rejecting a reference-transaction hook, noting that when the hook reports failure, the pusher's remote-tracking branches reflect a failed push although the server already committed.

### `repack`: add `--drop-filtered` to reclaim space in partial clones
*Long thread: 37 replies so far from 5 participants over 51 days.*
This GSoC series adds `--drop-filtered` to `builtin/repack.c` so partial clones can drop promisor blobs above a size threshold and lazily re-fetch them via existing machinery. Samuel Bronson tested it on a real llvm-project clone and proposed an RFC change to skip index-referenced blobs with a warning instead of dying. Siddharth Shrimali agreed that aborting after the full `enumerate_promisor_blobs()` walk is a poor trade-off but proposed a separate skip-set so `--dry-run` still lists those blobs, noting the planned drop log must not record them.

### packfile recovery when a multi-pack-index names a removed pack
*Long thread: 36 replies so far from 5 participants over 15 days.*
Elijah Newren's v3 series fixes objects being reported missing when a stale multi-pack-index names a pack removed by a concurrent geometric repack. Patch 4/4 scans the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ` with `midx_fill_entry()` returning a tri-state. Derrick Stolee completed his review with "Good patch. LGTM." and Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`.

### Adding `path.*` keys to `git repo info`
*Long thread: 31 replies so far from 5 participants over 52 days.*
K Jayatheerth's GSoC series extends `git repo info` with `path.*` keys like `path.toplevel`, `path.superproject-root`, and `path.hooks`, supporting absolute and relative suffixes. Junio C Hamano found that `get_superproject_working_tree()` uses the process working directory rather than the repository, so `git --git-dir=... repo info path.superproject-root.absolute` returns empty. K Jayatheerth confirmed the function relies on `xgetcwd()` and committed to updating it to take a `repo` parameter in v6 with `--git-dir` test coverage.

### Repartition checkout/switch/restore away from monolithic `checkout_main()`
*Long thread: 27 replies so far from 5 participants over 8 days.*
Junio C Hamano's eight-patch draft reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()` instead of the monolithic `checkout_main()`. Karthik Nayak agreed patch 6 makes patch 1's removal of `cb_option` coherent and clarified that patch 3's changed guard is safe because `--patch` mode already rejects conflicting combinations earlier. Junio confirmed those combinations were already rejected and said the series "merely scratches the surface" since a whole command like "git restore" may still be too coarse as reusable code.

### `checkout -m`: refine autostash fallback
*Long thread: 26 replies so far from 4 participants over 42 days.*
Harald Nordgren's series makes autostash conflicts in `sequencer.c` exit with status 1 while other failures exit 128, separating conflict advice from the branch-switch message. v5 was queued earlier. The thread discussed style tooling, with Nordgren noting the CI style job runs clang-format but misses mismatched-brace issues, and Junio C Hamano showing `RemoveBracesLLVM` cannot add braces to a whole if/else cascade.

## The period in brief
The week of 2026/08/31 through 2026/09/06 saw seven active days of steady traffic on the Git mailing list, with several long-running series reaching review milestones. Readers should not miss the runtime `core.useNanosec` conversion nearing readiness for `next`, the `receive-report` hook reaching v7, and Elijah Newren's shallow-clone push optimization drawing scrutiny over a multi-ref flaw.

## Key developments

### Shallow-clone push optimization draws multi-ref scrutiny
Elijah Newren posted v3 of a six-patch series keeping `send-pack` from transferring the whole tree when pushing from a shallow clone, introducing `push.shallowExcludeBoundary` as a tri-state option and `advice.pushShallowBoundary` suggesting one-at-a-time pushes when multi-ref pushes fail. Derrick Stolee reported that an AI-agent review found a multi-ref flaw where `append_reachable_shallow_grafts` treats shallow grafts as reachable without checking the remote's advertised refs, supplying a failing test. Newren countered that the multi-ref case is not corruption because receive-pack checks connectivity regardless of protocol, and that `push.negotiate=true` can also fail when a concurrent server repack defeats upload-pack's quick check.

### Background `rerere gc` versus rebase lock race
Thomas Bachem's patch addresses a race where background `git rerere gc` spawned by auto-maintenance collides with the sequencer's `MERGE_RR.lock` during `git rebase --continue`, leaving later continues refusing with "you have staged changes". After several iterations, Bachem settled on v4 where every caller waits `rerere.lockTimeout` and then fails as today, with only `git rerere gc` giving up at once. Junio C Hamano argued that proceeding without recording the preimage is a hard-to-accept regression and proposed loudly warning when `git rerere` fails to record a preimage instead.

### `--force-if-includes` consulting the wrong ref
Tyler Cipriani posted a two-patch series fixing `remote.c` and the push/send-pack layer so `--force-if-includes` checks the reflog of the ref being pushed rather than a same-named local branch, a bug behind false rejections and potential data loss. Aleksei Sviridkin separately posted v3 initializing an uninitialized timestamp to 0 in `is_reachable_in_reflog()` so a remote-tracking ref without a reflog no longer cuts the walk short. Junio C Hamano questioned whether 0 is a reasonable fallback timestamp, while Sviridkin countered that reflog entries older than 90 days survive and argued for zero over "now".

### What's cooking and fetch-pack eviction
In the August "What's cooking" thread, Patrick Steinhardt asked to evict the `ps/fetch-packfile-uris-parallel` topic because it conflicts with an upcoming object-transaction series for `git-fetch-pack(1)`, saying he will resend once that lands; Junio C Hamano replied "Will do." Steinhardt had earlier agreed with Justin Tobler to drop the parallel packfile URI fetch series for now and resend after Tobler's ODB transaction series merges.

### Version naming after Git 2.56
brian m. carlson weighed in on what to call the release after Git 2.56, listing remaining considerations including forge support for SHA-256, libgit2's SHA-256 and reftable support, and a re-roll of the lowercase-only object IDs series. He argued that JGit and Gitoxide were informed at least a year ago that SHA-256 was coming in Git 3.0 and should not be blockers. His gut reaction was that Git 3.0 might be the best choice, though 2.97 might be nice, or the project could be more careful with 2.95 and skip ahead to 3.0 when ready.

### Use-after-free in `setup_revisions()`
Jeff King posted a patch for a use-after-free where `setup_revisions()` freed argv strings that options like `--src-prefix` still referenced, garbling `git stash show` prefix output; it collects overwritten strings in an `argv_to_free` strvec released in `release_revisions()`. He also posted a cleanup simplifying `mark_argv_for_free()` callers, approved by Patrick Steinhardt and Junio C Hamano.

## In brief

**`last-modified` Bloom-filter pre-filtering** -- Toon Claes posted v3 and v4 of the series reusing `git-log`'s Bloom-filter machinery to speed up `git-last-modified(1)` on directories, but Junio C Hamano reported CI failed reproducibly after the series was pushed to `seen` and the topic was ejected.

**fetch-pack parallelizing packfile URIs** -- Patrick Steinhardt agreed with Justin Tobler to drop the two-patch series for now and resend once Tobler's ODB transaction series for fetch-pack has been merged.

**worktree add `worktree_basename()` fixes** -- Junio C Hamano marked the four-patch series fixing an out-of-bounds read and string-copy issues in `builtin/worktree.c` for `next`.

**proposed builtin `ident` command** -- Andrew Pleeter proposed `git ident`, but Jeff King argued the features belong in `git var` as broken-down identity components, an assessment Junio C Hamano endorsed; Pleeter later posted v3 of `git var` extensions with broken-down idents, `GIT_DEFAULT_KEY`, and `-z`.

**sequencer `--empty` with `fixup!`/`squash!`** -- Phillip Wood reviewed Farid Zakaria's v4 making the sequencer honor `--empty` when a `fixup!`/`squash!` empties its target, calling the approach sound.

**stash apply/pop/branch exit codes** -- Harald Nordgren's v3 makes these commands exit 2 on conflicts, but Phillip Wood and Junio C Hamano argued 1-for-conflicts matches merge-tree, grep and diff conventions better.

**`rev-list --missing-only`** -- Siddharth Asthana posted v2 of a filter flag requiring `--missing=print` or `--missing=print-info`, explaining Gitaly uses the missing list as a one-shot dependency set.

**`commit --amend` during conflict resolution** -- Elijah Newren posted v4 refusing `git commit --amend` and partial commits during conflict resolution in merge, cherry-pick, revert, am, or rebase; Phillip Wood reviewed with mixed feelings about permitting partial commits when a rebase pick becomes empty.

**Rust in Windows CI** -- Johannes Schindelin posted a two-patch series enabling Rust in Windows CI jobs by selecting a GCC-compatible Cargo target under MSYS2/MinGW.

**`hooks.allowNoVerify` rejection** -- Junio C Hamano rejected Alessio Attilio's proposal as a slippery slope; brian m. carlsson agreed, noting hooks are trivially bypassable via `core.hooksPath=/dev/null`.

**`early_scan_options()` sub-API** -- Christian Couder posted a six-patch series adding `early_scan_options()` to parse-options, fixing `--` detection in bisect and rev-parse; Junio C Hamano called the fast-import fix "Nicely done".

**out-of-bounds read with negative pathspec** -- Yannik Tausch's patch fixes `match_pathspec_with_flags()` passing a common prefix derived only from positive pathspecs when matching negative ones; disclosed to the Git security mailing list and confirmed not to require an embargo.

**ref storage terminology** -- Patrick Steinhardt posted an 11-patch series renaming `--ref-format=` to `--ref-storage=` and related variables; Junio C Hamano objected that neither term is clear unless everything becomes "ref-storage-format".

**debian-11 CI bump to debian-12** -- Jeff King proposed changing the debian-11 job to debian-12 in both GitHub and GitLab CI since Debian 11 exited LTS; Junio C Hamano will queue the patch.

**Outreachy December 2026 cohort** -- Christian Couder announced he will submit a Community Application for Git for Outreachy's December 2026 cohort with two intern project ideas, thanking co-mentors and noting the deadline is less than a week away.

**`git history split` SIGSEGV** -- Jinbao Chen fixed a crash when the parent tree object is missing, erroring with "unable to parse tree" instead of dereferencing NULL, with v2 guarding `write_ondisk_index()`.

**`imap-send --draft`** -- Wolfgang Faust added a `--draft` option to `git imap-send` to set the IMAP `\Draft` flag; Junio affirmed the flag-first approach.

**`rerere gc` auto heuristic** -- Patrick Steinhardt posted a two-patch series refining the `rerere gc` auto heuristic to run only when estimated stale entries reach a threshold (default 512); Derrick Stolee said both patches look good.

**`--stat`/`--check`/`-G` with `-L`** -- Michael Montalbo posted v3 scoping these options to `-L` line ranges with simplified tests and a bug fix in the line-range filter.

**pack-objects prefetch fix** -- Aleksei Sviridkin fixed prefetch ordering so the batch starts correctly; his test now uses 1 fetch instead of 2.

**alternates at ODB creation** -- Patrick Steinhardt posted v2 writing alternates at object-database creation time via `odb_source_create_on_disk()`, now using the lockfile API; Justin Tobler reviewed and asked whether clone options like `--shared` should be restricted to the "files" backend.

**ODB source registration removal** -- Patrick Steinhardt posted a 12-patch series removing in-memory source registration so the source list is exactly the alternates list.

**`gitdatamodel(7)` documentation** -- Kristoffer Haugsbakk posted v3 advertising `gitdatamodel(7)` as a concept guide with links from `git(1)` and the glossary.

**`--range-diff-notes` documentation** -- Kristoffer Haugsbakk circulated a draft rewrite; Junio C Hamano proposed `--no-range-diff-notes` should mean no notes are used for the range diff.

**test cleanup** -- Junio C Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals in Mark C. Chu-Carroll's cleanup.

**cherry-pick `--no-commit` documentation** -- Aleksei Sviridkin posted v3 documenting that `--no-commit` skips `CHERRY_PICK_HEAD` by design, restoring a `test_ref_missing` check.

**trace2 die()-free API** -- Derrick Stolee posted v3 of the series making the low-level trace2 API die()-free via a root-level `banned-die.h` header; Jeff King argued chasing transitive die() calls would need static analysis but is not opposed to the header in the meantime.

**lib-httpd CGI concurrency** -- Michael Montalbo's v5 making the `t/lib-httpd` CGI helpers concurrency-safe was judged "good enough" by Patrick Steinhardt, though Junio C Hamano and Steinhardt criticized the verbose AI-generated commit messages.

**autostash conflict advice blank line** -- Harald Nordgren posted v2 printing a blank line after autostash conflict advice in `git-checkout(1)`; Junio C Hamano objected that a non-zero `run_command()` result does not reliably detect conflicts.