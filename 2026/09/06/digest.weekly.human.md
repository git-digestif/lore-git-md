## Long threads

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 44 replies so far from 6 participants over 28 days.*

D. Ben Knoble's series converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration, replacing `#ifdef` checks in `read-cache.c` and `statinfo.c` so distributed builds can use nanosecond timestamps to reduce racy-git problems. The week saw the series reach v6 with Patrick Steinhardt declaring it ready for `next`, while Jeff King clarified that `NO_NSEC` builds fake nsec entries as zero, making the setting a noop there. Junio C Hamano raised a concern about `repo_config_values()` potentially hitting a `BUG()` on repositories other than `the_repository` during submodule recursion, and Knoble argued that recent work in those areas should instead lift the restriction rather than paper over it with a safe default. By week's end Knoble declined a final reroll absent strong demand, leaving the series in a holding pattern.

### git-replay(1) gains a `--linearize` option

*Long thread: 39 replies so far from 7 participants over 86 days.*

Toon Claes's series adds a `--linearize` option to `git-replay(1)` that drops merge commits and replays all reachable commits into a single linear history, mimicking `git rebase --no-rebase-merges`. The week brought v9 with rephrased error messages around the multiple-branches restriction, and Elijah Newren reviewed the series, stating "I think this version is good to go" and adding a `Reviewed-by` trailer, effectively clearing the series for integration.

### repack: add `--drop-filtered` to reclaim space in partial clones

*Long thread: 37 replies so far from 5 participants over 51 days.*

This GSoC series lets users reclaim space in partial clones by enumerating promisor blobs over a size threshold, dropping them locally, and relying on lazy-fetch to restore them. Samuel Bronson reported testing `--drop-filtered` on a real llvm-project clone, finding it worked well and greatly reduced object database size, then proposed an RFC change to skip index-referenced blobs with a warning rather than aborting. Siddharth Shrimali agreed that aborting after the full walk is a poor trade-off but proposed a separate skip-set so `--dry-run` still lists those blobs, noting the planned drop log must not record them.

### hook: introduce the `receive-report` hook

*Long thread: 37 replies so far from 5 participants over 18 days.*

Karthik Nayak's series adds a `receive-report` hook for `git-receive-pack` that runs after the reference transaction completes but before the report reaches the client, receiving the pkt-line report on stdin and replacing it with its stdout; a non-zero exit marks all references rejected. The week saw v5 through v7, with Nayak adopting an `enum report_status_version` to replace a `bool detailed_report` parameter after Junio C Hamano's suggestion, and Patrick Steinhardt offering nits on switch statements and redundant functions that were folded into v7.

### packfile recovery when a MIDX names a removed pack

*Long thread: 36 replies so far from 5 participants over 15 days.*

Elijah Newren's series fixes objects being reported missing despite being present when a stale multi-pack-index names a pack removed by a concurrent geometric repack. The week brought Derrick Stolee's review of patch 4/4 with "Good patch. LGTM." and Jeff King agreeing with Junio C Hamano's proposal to mark the series for `next`, while Newren clarified that the fallback scan runs over the packs in the MIDX rather than all packs.

### repo: add path keys to `git repo info`

*Long thread: 31 replies so far from 5 participants over 52 days.*

K Jayatheerth's GSoC series extends `git repo info` with `path.*` keys outputting repository component paths with absolute/relative suffixes. Junio C Hamano identified that `get_superproject_working_tree()` relies on the process working directory rather than the target repository, causing `git --git-dir=... repo info path.superproject-root.absolute` to return empty. Jayatheerth confirmed the function uses `xgetcwd()` and committed to updating it to take a `repo` parameter in v6 along with `--git-dir` test coverage.

### Repartition checkout/switch/restore away from monolithic `checkout_main()`

*Long thread: 27 replies so far from 5 participants over 8 days.*

Junio C Hamano's eight-patch draft reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()` instead of the monolithic `checkout_main()`. Karthik Nayak agreed that patch 6 makes patch 1's removal of `cb_option` coherent and clarified that patch 3's changed guard is safe because `--patch` mode already rejects conflicting combinations earlier. Hamano confirmed those combinations were already rejected but noted the series "merely scratches the surface" since a whole command like `git restore` may still be too coarse as reusable code.

### checkout -m: refine autostash fallback

*Long thread: 26 replies so far from 4 participants over 42 days.*

Harald Nordgren's series makes autostash conflicts in `sequencer.c` exit with status 1 (other failures 128) and separates conflict advice from the branch-switch message. v5 was queued earlier in the week after Hamano said both patches look good, and the remaining discussion centered on CI style tooling, with Nordgren noting the clang-format job misses mismatched-brace issues and Hamano showing that `RemoveBracesLLVM` cannot add braces to a whole if/else cascade.

### Refuse `git commit --amend` and partial commits during conflict resolution

*Long thread: 26 replies so far from 3 participants over 8 days.*

Elijah Newren's series makes `git commit --amend` and partial commits die during conflict resolution in merge, cherry-pick, revert, am, or rebase, since recording a resolution captures the whole tree rather than rewriting HEAD. Phillip Wood reviewed v4, finding the `FROM_REBASE_NOW_EMPTY` rename and reworded amend error clearer, but expressed mixed feelings about patch 2, which permits a partial commit when a rebase pick becomes empty, since `CHERRY_PICK_HEAD` and `.git/MERGE_MSG` are written at such stops and `git cherry-pick` still disallows them.

### trace2 stops allowing die()

*Long thread: 25 replies so far from 6 participants over 49 days.*

Derrick Stolee's series bans die()-capable helpers from the low-level trace2 API via a root-level `banned-die.h` header. Jeff King argued that chasing transitive die() calls would need static analysis or a ground-up rewrite, though he is not opposed to the header in the meantime, and Stolee said it helps demonstrate progress during the refactor but could be dropped in a v4.

### Make t/lib-httpd CGI test helpers concurrency-safe

*Long thread: 23 replies so far from 3 participants over 58 days.*

Michael Montalbo's series fixes race conditions in the `apply-one-time-script.sh` and `http-429.sh` test helpers seen in flaky t5616 failures on macOS CI. Patrick Steinhardt reviewed v5 and said the version looks good to him, while earlier criticism of verbose AI-generated commit messages led Montalbo to commit to writing prose by hand.

### doc: cherry-pick: note `--no-commit` skips `CHERRY_PICK_HEAD`

*Long thread: 11 replies so far from 4 participants over 2 days.*

Aleksei Sviridkin's documentation-only change notes that `cherry-pick --no-commit` skips `CHERRY_PICK_HEAD`, so a plain `git commit` records the committer as author. v2 dropped the t3507 test after Patrick Steinhardt argued tests need stronger justification, while Phillip Wood disputed that existing tests cover the conflicting case and urged a `test_ref_missing()` check; Junio C Hamano sided with Steinhardt on needing justification for new test overhead.

## The period in brief

The week of 2026/08/31 through 2026/09/06 saw heavy traffic across seven active days, with multiple long-running series reaching review milestones or integration decisions. Readers should not miss Elijah Newren's v3 of the shallow-clone push optimization, the ongoing debate over Thomas Bachem's rerere-versus-rebase lock fix, and brian m. carlson's thread on naming the release after Git 2.56.

## Key developments

### Avoid sending the whole tree when pushing from a shallow clone

Elijah Newren posted v3 of a six-patch series keeping `send-pack` from transferring the entire toplevel tree when pushing from a shallow clone. The series introduces a tri-state `push.shallowExcludeBoundary` option (true/false/abort) that omits shallow boundary objects from generated packs, defaulting to true in patch 5/6 to avoid transferring gigabytes of tree data, and adds `advice.pushShallowBoundary` suggesting one-at-a-time pushes when multi-ref pushes fail. Derrick Stolee had reported a multi-ref flaw where `append_reachable_shallow_grafts` treats shallow grafts as reachable without checking advertised refs, but Newren argued the case is not corruption because receive-pack checks connectivity regardless of protocol, and supplied improved error messages.

### Keep a background `rerere gc` from killing a rebase

Thomas Bachem's patch addresses a lock race where background `git rerere gc` spawned by auto-maintenance collides with the sequencer's `MERGE_RR.lock` during `git rebase --continue`, leaving later continues refusing with "you have staged changes". The week saw Bachem concede that proceeding without recording the preimage after `rerere.lockTimeout` is "the wrong trade", and in v4 every caller waits the timeout then fails as today, with only `git rerere gc` giving up at once. Junio C Hamano argued that a missed preimage during a rebase is a huge loss and proposed loudly warning when `git rerere` fails to record, while Phillip Wood endorsed disabling auto-maintenance during rebase as a separate topic.

### Fix `--force-if-includes` consulting the wrong ref

Tyler Cipriani posted a two-patch series fixing `remote.c` and the push layer so `--force-if-includes` checks the reflog of the ref being pushed rather than a same-named local branch, a bug behind false rejections and potential data loss. Patch 1/2 resolves `HEAD`, skips deletions, and rejects detached-HEAD pushes; patch 2/2 adds a "remote ref unverifiable" rejection advising `--force-with-lease=<ref>:<expect>`. Aleksei Sviridkin separately posted v3 initializing an uninitialized timestamp to 0 in `is_reachable_in_reflog()` so a remote-tracking ref without a reflog no longer cuts the walk short, though Hamano questioned whether 0 is a reasonable fallback.

### What's cooking and the fetch-packfile-uris eviction

In the August "What's cooking" thread, Patrick Steinhardt asked to evict the `ps/fetch-packfile-uris-parallel` topic because it conflicts with an upcoming object-transaction series for `git-fetch-pack(1)`, saying he will resend once that lands; Junio C Hamano replied "Will do." Steinhardt had earlier agreed with Justin Tobler to drop the parallel packfile URI fetch series for now and resend after Tobler's ODB transaction series merges.

### Naming the release after Git 2.56

brian m. carlson opened a thread asking what to call the release after Git 2.56, listing remaining considerations: forge support for SHA-256 on remaining major forges, libgit2's SHA-256 and reftable support, and a re-roll of the lowercase-only object IDs series. He argued that software not already far along on SHA-256 is likely not worth considering, noting JGit and Gitoxide were informed at least a year ago that SHA-256 was coming in Git 3.0. His gut reaction was that Git 3.0 might be the best choice, though 2.97 might be nice, or the project could be more careful with 2.95 and skip ahead to 3.0 when ready.

### use-after-free in `setup_revisions()`

Jeff King posted a patch for a use-after-free where `setup_revisions()` freed argv strings that options like `--src-prefix` still referenced, garbling `git stash show` prefix output; it collects overwritten strings in an `argv_to_free` strvec released in `release_revisions()`. He also posted a cleanup simplifying `mark_argv_for_free()` callers, approved by Patrick Steinhardt and Junio C Hamano.

### Ref storage terminology series

Patrick Steinhardt posted an 11-patch series renaming `--ref-format=` to `--ref-storage=`, `GIT_REFERENCE_BACKEND` to `GIT_REF_STORAGE`, and `init.defaultRefFormat` to `init.defaultRefStorage`, keeping old names as hidden aliases. Karthik Nayak found it easy to read with small questions, but Junio C Hamano objected that neither "ref-storage" nor "ref-format" is clear unless everything becomes "ref-storage-format".

### Outreachy December 2026 cohort

Christian Couder announced that, with no objections, he will submit a Community Application for Git for Outreachy's December 2026 cohort, thanked Usman, Kaartic and Pablo for volunteering as co-mentors, and proposed two intern project ideas: continuing to remove global state and perhaps libifying parts of the code base, and improving how command arguments and options are scanned by porting hand-rolled code to the parse-options API or his early-scan API. Mentors still need to sign up and projects must be submitted before the deadline less than one week away.

## In brief

- **last-modified Bloom filter pre-filtering** -- Toon Claes posted v3 and v4 of the series reusing `git-log`'s Bloom-filter machinery to speed up `git-last-modified(1)`, but after the series was pushed to `seen`, CI failed reproducibly and the topic was ejected.
- **worktree_basename() fixes** -- The four-patch series fixing an out-of-bounds read and string handling in `builtin/worktree.c` was marked for `next` by Junio C Hamano.
- **proposed builtin `ident` command** -- Andrew Pleeter's `git ident` proposal was rejected in favor of extending `git var` with broken-down identity components, a view endorsed by Jeff King and Junio C Hamano; Pleeter posted v3 of the `git var` extensions instead.
- **pathspec out-of-bounds fix** -- Yannik Tausch's patch fixes an out-of-bounds read in `dir.c` when a negative pathspec is shorter than the positive prefix; the issue was disclosed to the Git security mailing list and confirmed not to require an embargo.
- **hooks.allowNoVerify rejected** -- Junio C Hamano rejected Alessio Attilio's proposal as a slippery slope; brian m. carlomb agreed, noting hooks are trivially bypassable via `core.hooksPath=/dev/null`.
- **early_scan_options() series** -- Christian Couder posted a six-patch series adding an `early_scan_options()` sub-API to parse-options, fixing `--` detection in bisect and rev-parse; Hamano called the `fast-import --allow-unsafe-features` fix "Nicely done".
- **git history split SIGSEGV** -- Jinbao Chen fixed a crash when the parent tree object is missing, erroring with "unable to parse tree" instead of dereferencing NULL; v2 guards a NULL tree in `write_ondisk_index()`.
- **git imap-send --draft** -- Wolfgang Faust added a `--draft` option to set the IMAP `\Draft` flag; Hamano affirmed the flag-first approach.
- **debian CI bump** -- Jeff King proposed changing the debian-11 CI job to debian-12 since Debian 11 exited LTS; Hamano will queue the patch.
- **pack-objects prefetch fix** -- Aleksei Sviridkin fixed prefetch ordering so the batch starts correctly, reducing fetches from 2 to 1 in his test.
- **alternates at ODB creation** -- Patrick Steinhardt posted v2 writing alternates at object-database creation time via `odb_source_create_on_disk()` using the lockfile API; Justin Tobler reviewed and asked whether clone options like `--shared` should be restricted to the "files" backend.
- **sequencer --empty fix** -- Phillip Wood reviewed Farid Zakaria's v4 making the sequencer honor `--empty` when a `fixup!`/`squash!` empties its target, calling the approach sound.
- **rev-list --missing-only** -- Siddharth Asthana posted v2 of a filter flag requiring `--missing=print` or `--missing=print-info`, explaining Gitaly uses the missing list as a one-shot dependency set.
- **ODB source removal** -- Patrick Steinhardt posted a 12-patch series removing in-memory source registration so the source list is exactly the alternates list.
- **Rust in Windows CI** -- Johannes Schindelin posted a two-patch series enabling Rust in Windows CI jobs by selecting a GCC-compatible Cargo target under MSYS2/MinGW.
- **gitdatamodel(7) docs** -- Kristoffer Haugsbakk posted v3 advertising `gitdatamodel(7)` as a concept guide with links from `git(1)` and the glossary.
- **--range-diff-notes docs** -- Haugsbakk circulated a draft rewrite; Hamano proposed `--no-range-diff-notes` should mean no notes are used for the range diff, mirroring `--no-notes`.
- **test modernization** -- Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals, since the semantic helpers are meant to shout on failure.
- **rerere gc heuristic** -- Steinhardt posted a two-patch series refining the auto heuristic to run only when estimated stale entries reach a threshold (default 512); Derrick Stolee said both look good.

## Looking ahead

Several series posted late in the week will see review in the coming days: Karthik Nayak's v7 of the `receive-report` hook, rebased on master with the enum-based report version, and Toon Claes's `git-replay --linearize` series, which now carries Elijah Newren's `Reviewed-by`. D. Ben Knoble's `core.useNanosec` series remains in limbo pending guidance on the `repo_config_values()` recursion concern, while K Jayatheerth's `git repo info` path series awaits the `get_superproject_working_tree()` fix in v6. Elijah Newren's shallow-push series with the new `push.shallowExcludeBoundary` tri-state will likely draw further review of the multi-ref boundary handling.