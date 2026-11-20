## Long threads

### Convert build-time USE_NSEC into runtime core.useNanosec
*Long thread: 44 replies so far from 6 participants over 28 days.*
D. Ben Knoble's series converts the compile-time `USE_NSEC` option into the runtime `core.useNanosec` configuration, allowing distributed builds to use nanosecond timestamps to reduce racy-git issues in index operations. The week saw the series reach v6 with Patrick Steinhardt's approval for `next`, though Junio C Hamano raised concerns about `repo_config_values()` potentially `BUG()`ing on non-`the_repository` repositories during submodule recursion. Knoble declined to remove remaining `NO_NSEC` `#ifdef`s and indicated no reroll is planned absent strong demand, while Hamano suggested the test apparatus set the config globally to surface potential issues.

### git-replay(1) gains a --linearize option
*Long thread: 39 replies so far from 7 participants over 86 days.*
Toon Claes's v9 series adds `--linearize` to `git-replay(1)`, dropping merge commits to create linear history similar to `git rebase --no-rebase-merges`. Elijah Newren reviewed v9 and added a `Reviewed-by` trailer, calling it "good to go." Earlier revisions rephrased error messages around the multiple-branches restriction and simplified documentation after reviewer feedback. The series now rejects multiple branches and `--contained`, updating refs pointing at merges to their last replayed ancestor.

### Introduce the receive-report hook for git-receive-pack
*Long thread: 37 replies so far from 5 participants over 18 days.*
Karthik Nayak's series adds a `receive-report` hook running after the reference transaction but before the client report, allowing stdout to replace the pkt-line report and non-zero exits to reject all references. The week saw v5 through v7, with v6 introducing `enum report_status_version` to replace static variables and v7 rebasing on `master` while removing redundant `report_v2()`. Junio C Hamano questioned the `bool detailed_report` parameter in favor of the enum, and clarified that when the hook reports failure, the pusher's remote-tracking branches reflect a failed push although the server committed.

### repack: add --drop-filtered to reclaim space in partial clones
*Long thread: 37 replies so far from 5 participants over 51 days.*
This GSoC series lets partial clones drop promisor blobs above a size threshold and lazily re-fetch them. Samuel Bronson validated it on a real llvm-project clone, proposing to skip index-referenced blobs with a warning rather than aborting. Siddharth Shrimali agreed aborting after the full walk is poor but proposed a separate skip-set so `--dry-run` still lists those blobs, noting the drop log must not record them.

### packfile recovery when a MIDX names a removed pack
*Long thread: 36 replies so far from 5 participants over 15 days.*
Elijah Newren's v3 series fixes objects reported missing when a stale multi-pack-index names a pack removed by concurrent geometric repack. Patch 4/4 scans the MIDX's packs for surviving copies on `OBJECT_INFO_SECOND_READ` with a tri-state `midx_fill_entry()`. Derrick Stolee completed review with "Good patch. LGTM," and Jeff King agreed with Junio C Hamano to mark the series for `next`.

### repo: add path keys to git repo info
*Long thread: 31 replies so far from 5 participants over 52 days.*
K Jayatheerth's GSoC series adds `path.*` keys (toplevel, superproject-root, hooks, index, etc.) to `git repo info` with absolute/relative suffixes. Junio C Hamano identified that `get_superproject_working_tree()` uses the process working directory rather than the repository, causing empty values under `--git-dir`. Jayatheerth committed to updating the function to take a `repo` parameter and adding `--git-dir` test coverage in v6.

### Repartition checkout/switch/restore away from monolithic checkout_main()
*Long thread: 27 replies so far from 5 participants over 8 days.*
Junio C Hamano's eight-patch draft reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()`. Karthik Nayak agreed patch 6 makes patch 1's removal coherent and clarified patch 3's guard is safe because `--patch` mode rejects conflicting combinations earlier. Hamano noted the series "merely scratches the surface" since whole commands may still be too coarse as reusable code.

### Refuse git commit --amend and partial commits during conflict resolution
*Long thread: 26 replies so far from 3 participants over 8 days.*
Elijah Newren's series makes `git commit --amend` and partial commits die during conflict resolution in merge, cherry-pick, revert, am, or rebase. Phillip Wood reviewed v4, praising the `FROM_REBASE_NOW_EMPTY` rename and reworded amend error, but had mixed feelings about patch 2 permitting partial commits when a rebase pick becomes empty, since `CHERRY_PICK_HEAD` and `.git/MERGE_MSG` are written at such stops.

### checkout -m: refine autostash fallback
*Long thread: 26 replies so far from 4 participants over 42 days.*
Harald Nordgren's series makes autostash conflicts exit with status 1 (other failures 128) and separates conflict advice from branch-switch messages. v5 was queued earlier in the week. The discussion turned to style tooling, with Nordgren noting CI's clang-format job misses mismatched braces, and Hamano showing `RemoveBracesLLVM` cannot add braces to whole if/else cascades.

### trace2 stops allowing die()
*Long thread: 25 replies so far from 6 participants over 49 days.*
Derrick Stolee's series bans die()-capable helpers from the low-level trace2 API via a root-level `banned-die.h` header. v3 fixed the `BANNED()` macro case, `tr2_sysenv_cb()` return values, and a `":<REDACTED>"` length bug with new t0212 coverage. Jeff King argued chasing transitive die() calls would need static analysis, though he is not opposed to the header meanwhile; Stolee said it could be dropped in v4.

### Make t/lib-httpd CGI test helpers concurrency-safe
*Long thread: 23 replies so far from 3 participants over 58 days.*
Michael Montalbo's series fixes race conditions in `apply-one-time-script.sh` and `http-429.sh` seen in flaky t5616 failures on macOS CI. Patrick Steinhardt reviewed v5 and said it looks good. Earlier, Hamano and Steinhardt criticized verbose AI-generated commit messages, and Montalbo committed to writing prose by hand.

### doc: cherry-pick: note --no-commit skips CHERRY_PICK_HEAD
*Long thread: 11 replies so far from 4 participants over 2 days.*
Aleksei Sviridkin's documentation change notes that `--no-commit` skips `CHERRY_PICK_HEAD`, so plain `git commit` records the committer as author. v2 dropped the t3507 test after Patrick Steinhardt argued tests need stronger justification; Phillip Wood disputed that existing tests cover the conflicting case and urged a `test_ref_missing()` check, while Hamano sided with Steinhardt.

## The period in brief
The week of 2026/08/31 through 2026/09/06 saw heavy traffic across seven active days, with twelve long-running threads dominating discussion. Key developments included Elijah Newren's shallow-clone push optimization drawing a reported multi-ref flaw, a heated debate over Thomas Bachem's rerere gc versus rebase lock race, and fixes for `--force-if-includes` consulting the wrong ref. Readers should not miss the `receive-report` hook reaching v7, the runtime `core.useNanosec` series nearing readiness for `next`, and brian m. carlson's weighing of Git 3.0 versus 2.97 for the post-2.56 release.

## Key developments

### Shallow clone push optimization draws multi-ref flaw report
Elijah Newren posted v3 of a six-patch series preventing `send-pack` from transferring the whole tree when pushing from a shallow clone, introducing `push.shallowExcludeBoundary` as a tri-state option. Derrick Stolee reported that an AI-agent review found a multi-ref flaw where `append_reachable_shallow_grafts` treats shallow grafts as reachable without checking advertised refs, supplying a failing test. Newren countered that the multi-ref case is not corruption since receive-pack checks connectivity, and that `push.negotiate=true` can also fail when concurrent repacks defeat upload-pack's quick checks.

### rerere gc versus rebase lock race
Thomas Bachem's patch addresses a race where background `git rerere gc` spawned by auto-maintenance collides with the sequencer's `MERGE_RR.lock` during rebase. After several iterations, Bachem conceded in v4 that proceeding without recording the preimage after `rerere.lockTimeout` is "the wrong trade," proposing instead that every caller wait the timeout then fail, with only `git rerere gc` giving up at once. Junio C Hamano argued proceeding without recording is a hard-to-accept regression and proposed loudly warning when `git rerere` fails to record a preimage.

### push --force-if-includes fixes
Tyler Cipriani posted a two-patch series fixing `--force-if-includes` to check the reflog of the ref being pushed rather than a same-named local branch, a bug behind false rejections and potential data loss. Aleksei Sviridkin separately fixed the case where the remote-tracking ref has no reflog by initializing the timestamp to 0. Hamano questioned whether 0 is a reasonable fallback while Sviridkin argued reflog entries older than 90 days survive, defending zero over "now" or a fixed age.

### use-after-free in setup_revisions()
Jeff King posted a patch fixing a use-after-free where `setup_revisions()` freed argv strings that options like `--src-prefix` still referenced, garbling `git stash show` output. The fix collects overwritten strings in an `argv_to_free` strvec released in `release_revisions()`. He also posted a cleanup simplifying `mark_argv_for_free()` callers, approved by Patrick Steinhardt and Hamano.

### git var extensions versus builtin ident command
Andrew Pleeter proposed a new `git ident` command, but Jeff King argued the features belong in `git var` as broken-down identity components, an assessment Hamano endorsed. Pleeter instead posted v3 of `git var` extensions adding broken-down idents, `GIT_DEFAULT_KEY`, multiple arguments, and `-z` output. Phillip Wood reviewed, questioning the `GIT_SIGNING_KEY` alias and proposing `-z` match `git config list -z`.

### What's cooking and fetch-packfile-uris eviction
In the August "What's cooking" thread, Patrick Steinhardt asked to evict the `ps/fetch-packfile-uris-parallel` topic because it conflicts with Justin Tobler's upcoming object-transaction series for `git-fetch-pack`; Hamano replied "Will do." Steinhardt agreed to drop his two-patch series preparing fetch and clone to fetch announced packfile URIs in parallel until Tobler's series lands.

### Naming the release after Git 2.56
brian m. carlson weighed options for the release following Git 2.56, listing remaining SHA-256 considerations including forge support and libgit2's reftable support. He argued JGit and Gitoxide should not block Git 3.0 since they were informed a year ago, leaning toward Git 3.0, 2.97, or 2.98, or a careful 2.95 followed by 3.0 when ready.

## In brief

**worktree_basename() fixes** -- Junio C Hamano marked the four-patch series fixing out-of-bounds reads and string handling in `builtin/worktree.c` for `next`.

**last-modified Bloom filter speedups** -- Toon Claes posted v3 and v4 of the series reusing `git-log`'s Bloom-filter machinery for `git-last-modified(1)`, but CI failed reproducibly under `ci/run-build-and-tests.sh` and the topic was ejected from `seen`.

**pathspec out-of-bounds fix** -- Yannik Tausch fixed an out-of-bounds read in `dir.c` when a negative pathspec is shorter than the positive prefix, passing 0 as the prefix for negative pathspecs; disclosed to the security list and confirmed not to require embargo.

**hooks.allowNoVerify rejected** -- Junio C Hamano rejected Alessio Attilio's proposal as a slippery slope; brian m. carlson agreed, noting hooks are bypassable via `core.hooksPath=/dev/null`.

**early_scan_options() series** -- Christian Couder posted six patches adding an `early_scan_options()` sub-API to parse-options, fixing `--` detection in bisect and rev-parse; Hamano called the fast-import fix "Nicely done."

**git history split SIGSEGV** -- Jinbao Chen fixed a crash when the parent tree object is missing, erroring with "unable to parse tree" instead of dereferencing NULL.

**git imap-send --draft** -- Wolfgang Faust added a `--draft` option to set the IMAP `\Draft` flag; Hamano affirmed the flag-first approach.

**rev-list --missing-only** -- Siddharth Asthana posted v2 of a filter flag requiring `--missing=print`, explaining Gitaly uses the missing list as a one-shot dependency set.

**Rust in Windows CI** -- Johannes Schindelin posted a two-patch series enabling Rust in Windows CI jobs by selecting a GCC-compatible Cargo target under MSYS2/MinGW.

**ref storage terminology** -- Patrick Steinhardt posted an 11-patch series renaming `--ref-format=` to `--ref-storage=` and related variables; Hamano objected that neither term is clear unless everything becomes "ref-storage-format."

**debian CI bump** -- Jeff King proposed changing the debian-11 CI job to debian-12 since Debian 11 exited LTS; Hamano will queue it.

**Outreachy December 2026** -- Christian Couder announced Git's Community Application for Outreachy's December 2026 cohort with two intern project ideas, approaching GitLab and others for sponsorship.

**pack-objects prefetch fix** -- Aleksei Sviridkin fixed prefetch ordering so the batch starts correctly, reducing fetches from 2 to 1 in his test.

**gitdatamodel(7) documentation** -- Kristoffer Haugsbakk posted v3 advertising `gitdatamodel(7)` as a concept guide with links from `git(1)` and the glossary.

**--range-diff-notes docs** -- Haugsbakk circulated a draft rewrite; Hamano proposed `--no-range-diff-notes` should mean no notes are used, mirroring `--no-notes`.

**alternates at ODB creation** -- Patrick Steinhardt posted v2 writing alternates via `odb_source_create_on_disk()` using the lockfile API; Justin Tobler reviewed, asking whether `--shared` should be restricted to the files backend.

**sequencer --empty fixup** -- Phillip Wood reviewed Farid Zakaria's v4 making the sequencer honor `--empty` when `fixup!`/`squash!` empties its target, calling it sound.

**test modernization objection** -- Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals in Mark C. Chu-Carroll's cleanup.

## Looking ahead
The `receive-report` hook series reached v7 and appears close to convergence, while the runtime `core.useNanosec` series awaits a final decision on whether to reroll for the `repo_config_values()` concern. Toon Claes's `git-replay --linearize` series received a `Reviewed-by` and should land soon. K Jayatheerth's `git repo info` path keys need the `get_superproject_working_tree()` fix before queuing. Christian Couder's Outreachy application is due within a week, and the community will soon decide whether the post-2.56 release is Git 3.0 or a 2.x version.