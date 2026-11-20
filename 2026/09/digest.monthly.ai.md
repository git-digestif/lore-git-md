## Long threads

### history: add squash subcommand to fold a range
*Long thread: 91 replies so far from 9 participants over 87 days.*
Harald Nordgren's series adding `git history squash <revision-range>` dominated the month, reaching v15 with fixes for mixed-case hex in `fixup!` subjects and completion tests. Junio C Hamano noted that with the "complete 'git history'" topic merged, an evil merge for the completion test change was no longer needed. The series remained the most-discussed topic of the period, still in active review at month's end.

### receive-report hook for git-receive-pack
*Long thread: 58 replies so far from 7 participants over 29 days.*
Karthik Nayak's series adds a `receive-report` hook running after the reference transaction but before the client report, reading the pkt-line report on stdin with stdout replacing it and non-zero exit rejecting all refs. The thread progressed through v10 with fixes for a `BUG()` when no report was requested and a leak in `override_cmds_error()`, plus a coccinelle fix from Junio C Hamano removing a redundant nullness check. Patrick Steinhardt expressed satisfaction with the direction, leaving the series in a stable state awaiting integration.

### stash: in-core index merge for `stash apply --index`
*Long thread: 53 replies so far from 5 participants over 13 days.*
D. Ben Knoble's series replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing the autostash/`stash.index=true` bug where `git reset` triggered a non-re-entrant lock. After v5 fixed tri-state handling of `merge_result.clean` and Junio C Hamano diagnosed a transposed tree-argument order, Phillip Wood confirmed the range-diff and welcomed the removal of subprocesses. Hamano marked the series for `next` on 10/01.

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`
*Long thread: 46 replies so far from 6 participants over 36 days.*
D. Ben Knoble's series exposes `USE_NSEC` as the runtime `core.useNanosec` config, letting distributed builds use nanosecond index timestamps to avoid racy-git false positives. After a v7 resend differing only in a documentation typofix, Junio C Hamano queued the three patches, noting they matched what he had already queued from v6, effectively concluding the thread.

### git-replay(1) gains a `--linearize` option
*Long thread: 39 replies so far from 7 participants over 86 days.*
Toon Claes's series adds a `--linearize` option to `git-replay(1)` that drops merge commits and replays all reachable commits into a single linear history. The period brought v9 with rephrased error messages, and Elijah Newren reviewed the series, stating "I think this version is good to go" and adding a `Reviewed-by` trailer, effectively clearing the series for integration.

### repo: add path keys to `git repo info`
*Long thread: 37 replies so far from 5 participants over 74 days.*
K Jayatheerth's GSoC series adds eight `path.*` keys to `git repo info` covering toplevel, superproject-root, hooks, index, grafts, git-prefix and cdup, each with absolute and relative suffixes. The series reached v7, splitting the `get_superproject_working_tree()` fix into its own commit and fixing `path.cdup` to return an absolute path when the worktree lies outside the current directory. Earlier revisions had drawn attention to the superproject bug relying on the process working directory rather than the repository being inspected.

### repack: add `--drop-filtered` to reclaim space in partial clones
*Long thread: 37 replies so far from 5 participants over 51 days.*
This GSoC series lets users reclaim space in partial clones by enumerating promisor blobs over a size threshold, dropping them locally, and relying on lazy-fetch to restore them. Samuel Bronson reported testing `--drop-filtered` on a real llvm-project clone, finding it worked well, then proposed an RFC change to skip index-referenced blobs with a warning rather than aborting. Siddharth Shrimali agreed that aborting after the full walk is a poor trade-off but proposed a separate skip-set so `--dry-run` still lists those blobs.

### packfile recovery when a MIDX names a removed pack
*Long thread: 36 replies so far from 5 participants over 15 days.*
Elijah Newren's series fixes objects being reported missing despite being present when a stale multi-pack-index names a pack removed by a concurrent geometric repack. Derrick Stolee reviewed patch 4/4 with "Good patch. LGTM." and Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`, while Newren clarified that the fallback scan runs over the packs in the MIDX rather than all packs.

### Write object-database alternates at creation time
*Long thread: 36 replies so far from 5 participants over 18 days.*
Patrick Steinhardt's nine-patch series refactors `init_db()` into `create_repository()` and moves alternate writing into `odb_source_create_on_disk()`, removing the ad-hoc `odb_source_write_alternate()` callback. The period saw v4 and v5, with Justin Tobler reviewing v4 as good and Steinhardt addressing comments on sticky file-stream error indicators and declining to formalize backend errors for alternates.

### Introduce uploadpack.lazyFetchTrusted
*Long thread: 34 replies so far from 3 participants over 85 days.*
Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config (global/system scope only) setting `GIT_NO_LAZY_FETCH=0` for trusted repos, motivated by large-object promisors. V5 was rebased with no functional changes from v4, only documentation and test updates, including a NEEDSWORK comment about silently ignoring misspelled allowlist paths. Junio C Hamano reviewed the control logic as trivially clear and suggested doc wording fixes.

### sequencer: run auto maintenance once a sequence is done
*Long thread: 33 replies so far from 5 participants over 25 days.*
Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, passing `maintenance.auto=false` to spawned commands. The period centered on placement: Bachem asked Patrick Steinhardt to choose between the builtins and the sequencer internals, and Phillip Wood reviewed the v5 tests and declared the series "looks ready for next to me." Steinhardt later noted the series was already cooking in `next`.

### Documentation: new `gitmergeconflicts(7)` guide
*Long thread: 32 replies so far from 6 participants over 10 days.*
Julia Evans's seven-patch series adds a `gitmergeconflicts` manual page linked from merge, rebase, revert, cherry-pick, and pull. V2 added a "WHAT IS A MERGE CONFLICT?" section explaining 3-way merge, with Junio C Hamano offering review reactions and an illustration of why `diff3` style is essential. D. Ben Knoble endorsed the section, and Patrick Steinhardt proposed extending conflict-marker labels to include commit ID, message, and date.

### http.sslVerifyStatus for stapled OCSP responses
*Long thread: 31 replies so far from 5 participants over 46 days.*
Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so Git can check stapled OCSP responses, defaulting to false. After cooking in `next`, the topic was reverted when SZEDER Gabor reported a t5585 failure; he traced the failure to his GnuTLS curl backend lacking OCSP support, confirmed the test passes with OpenSSL, and proposed improving the `SSL_VERIFYSTATUS` prerequisite. Patrick Steinhardt and Junio C Hamano endorsed the diagnosis.

### refs: report old values to reference-transaction hooks
*Long thread: 29 replies so far from 5 participants over 13 days.*
Maciej Ciemborowicz's v6 single-patch series resolves old object IDs in the common transaction layer before the "preparing" hook and after backend locks, fixing batched deletions reporting all-zero old OIDs. Patrick Steinhardt had argued for this transaction-layer approach over caller-specific fixes. On 10/01 Ciemborowicz lowered the priority of the underlying bug after resolving his branch and tag deletion issues, naming branch renaming as the remaining case needing this fix.

### repack: corner cases for cruft-less MIDXs
*Long thread: 27 replies so far from 5 participants over 4 days.*
Taylor Blau's v2 eight-patch series fixes cases where `repack.midxMustContainCruft=false` produces a multi-pack-index not closed under reachability, which can fail bitmap generation. The period saw detailed review of the oidset conversion for extra roots, kept-pack traversal, and the `--write-midx=incremental` gap. Elijah Newren objected that oidset iteration order can regress pack quality by visiting subtrees first, causing blobs to be written in full instead of delta-compressed.

### Repartition checkout/switch/restore away from monolithic `checkout_main()`
*Long thread: 27 replies so far from 5 participants over 8 days.*
Junio C Hamano's eight-patch draft reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()` instead of the monolithic `checkout_main()`. Karthik Nayak agreed that patch 6 makes patch 1's removal of `cb_option` coherent and clarified that patch 3's changed guard is safe because `--patch` mode already rejects conflicting combinations earlier. Hamano confirmed those combinations were already rejected but noted the series "merely scratches the surface."

### Git 3.0: restrict hex object IDs to lowercase only
*Long thread: 26 replies so far from 6 participants over 41 days.*
brian m. carlsson's series removes Git's acceptance of uppercase hex in object IDs for Git 3.0. V2 fixed a hex-range comment, updated commit messages, moved t5324 fixes to a separate commit, and restored lowercasing in `parse_oid_prefix`. Phillip Wood questioned whether refnames ending in object-ID-like strings should also be forbidden.

### last-modified Bloom filter pre-filtering
*Long thread: 26 replies so far from 5 participants over 56 days.*
Toon Claes's series speeds up `git last-modified` by reusing the revision machinery's Bloom filter checks, showing about 1.95x speedup with `-t` and 3.74x without on ziglang/zig. Patrick Steinhardt reviewed v4, noting `bloom_filter_contains()` can return -1 for empty filters, suggesting a rename to `..._with_leading_dirs()`, and proposing to store `bloom_filter_settings` in `struct last_modified` since `prepare_revision_walk()` clears them.

### checkout -m: refine autostash fallback
*Long thread: 26 replies so far from 4 participants over 42 days.*
Harald Nordgren's series makes autostash conflicts in `sequencer.c` exit with status 1 (other failures 128) and separates conflict advice from the branch-switch message. V5 was queued after Hamano said both patches look good, and the remaining discussion centered on CI style tooling, with Nordgren noting the clang-format job misses mismatched-brace issues.

### Refuse `git commit --amend` and partial commits during conflict resolution
*Long thread: 26 replies so far from 3 participants over 8 days.*
Elijah Newren's series makes `git commit --amend` and partial commits die during conflict resolution in merge, cherry-pick, revert, am, or rebase. Phillip Wood reviewed v4, finding the `FROM_REBASE_NOW_EMPTY` rename and reworded amend error clearer, but expressed mixed feelings about patch 2, which permits a partial commit when a rebase pick becomes empty.

### Advice scoping for the defaultBranchName hint
*Long thread: 25 replies so far from 4 participants over 9 days.*
Vsevolod Myalitsin's series addresses the `defaultBranchName` advice, whose suggested `git config set advice.defaultBranchName false` writes to local scope and thus only affects the repository where the advice was shown. Jeff King suggested simply recommending `--global` in all advice squelch messages, which Junio C Hamano agreed with, and Hamano posted a v5 patch adding `--global` to the `git config set` command in the advice message across `advice.c` and 11 test files.

### trace2 stops allowing die()
*Long thread: 25 replies so far from 6 participants over 49 days.*
Derrick Stolee's series bans die()-capable helpers from the low-level trace2 API via a root-level `banned-die.h` header. Jeff King argued that chasing transitive die() calls would need static analysis or a ground-up rewrite, though he is not opposed to the header in the meantime, and Stolee said it helps demonstrate progress during the refactor but could be dropped in a v4.

### format-patch `--range-diff-notes` options
*Long thread: 25 replies so far from 4 participants over 42 days.*
Kristoffer Haugsbakk's series teaches `git format-patch` `--[no-]range-diff-notes=<ref>` so the range diff can use a different notes ref than the patches. The period saw v4 trim expository text and refactor the callback, then v5 adopt Junio C Hamano's condensed documentation wording. Hamano reviewed v4 2/2 as otherwise good.

### push: fix `--force-if-includes` when the remote-tracking ref has no reflog
*Long thread: 23 replies so far from 6 participants over 27 days.*
Aleksei Sviridkin's v4 initializes the timestamp cutoff to 0 when the remote-tracking ref has no reflog, fixing spurious "remote ref updated since checkout" rejections. Junio C Hamano said on 9/29 that the v4 changes look sensible, will replace the patch and mark it for `next`, noting trivial t5533 conflicts with another topic.

### shallow-repo fetch of new remotes via refmap
*Long thread: 23 replies so far from 4 participants over 16 days.*
Harald Nordgren's four-patch series avoids fetching every branch of a new remote in shallow repositories by adding `remote.<name>.refmap`, inferring branches from local upstreams, and adding `git remote add --limited-fetch`. On 10/04 he posted v6 removing a leftover reference to deleted default-branch logic. Junio C Hamano reviewed v6 as much better but flagged that with only patch 1/4 applied, a remote with `refmap` and no tracked branches makes `git fetch` die.

### rerere: wait for `MERGE_RR.lock` and skip in auto gc
*Long thread: 23 replies so far from 4 participants over 31 days.*
Thomas Bachem's series makes rerere writers wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds a hidden `--skip-locked` option to `git rerere gc` for auto-maintenance, and lets conflict-time callers warn and continue. On 10/02 he posted v6 renaming the option to `--skip-locked` and the flag to `RERERE_WARN_LOCKED`, after finding that calling `rerere_gc()` directly from `builtin/gc.c` would die with exit code 128 on a held lock.

### Make t/lib-httpd CGI test helpers concurrency-safe
*Long thread: 23 replies so far from 3 participants over 58 days.*
Michael Montalbo's series fixes race conditions in the `apply-one-time-script.sh` and `http-429.sh` test helpers seen in flaky t5616 failures on macOS CI. Patrick Steinhardt reviewed v5 and said the version looks good to him, while earlier criticism of verbose AI-generated commit messages led Montalbo to commit to writing prose by hand.

### git var `-z`, multiple variables, broken-out idents
*Long thread: 22 replies so far from 6 participants over 39 days.*
Andrew Pleeter's v9 series splits into four patches adding string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, support for multiple variable arguments, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables. Phillip Wood reviewed the series as good with only minor test-style nits about `test_env()` and collapsing per-component tests.

### xdiff: use `size_t` for `mmfile_t`
*Long thread: 21 replies so far from 4 participants over 3 days.*
Jeff King's seven-patch v2 consolidates `mmbuffer_t` into `mmfile_t`, switches buffer sizes to `size_t`, NUL-terminates buffers read by `read_mmfile()`, and makes `merge-ll` report errors when reading external merge results fails. Patrick Steinhardt flagged a commit-message nit in the NUL-termination patch and okayed the `merge-ll` change.

### CI annotations linked to test scripts
*Long thread: 20 replies so far from 4 participants over 10 days.*
Harald Nordgren's two-patch series links GitHub Actions failure and leak annotations to the test script's file and line, and stops leak-sanitizer jobs at the first failure. On 10/04 he asked whether the inability to link annotations to unchanged test files is enough to call the change a regression, suggesting that part might not be worth doing.

### AsciiDoc cross-references and the tutorial rewrite in Documentation/
*Long thread: 17 replies so far from 4 participants over 4 days.*
Julia Evans reposted v2 of her patch replacing "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references; Jeff King approved it. She also announced a planned rewrite of `gittutorial.adoc` covering only basic commands, and Junio C Hamano agreed the current tutorial has outlived its time but stressed that new guides must be sufficient to replace it.

### GitLab CI Windows cargo invocation fix
*Long thread: 12 replies so far from 3 participants over 6 days.*
Johannes Schindelin posted v2 of the four-patch series fixing the GitLab CI cargo invocation in Windows jobs, rewording commit messages to clarify "Rust toolchain that targets GCC" and correcting "GNU MSI" to "Rust MSI", and answering Patrick Steinhardt's questions about the MinGW build and PATH quoting.

### RFC for `git-brebase`
*Long thread: 12 replies so far from 2 participants over 1 day.*
Alejandro Colomar posted an RFC for `git-brebase`, a 164-line shell script that rebases onto the first conflicting commit of a target branch using `git bisect` internally. Nico Williams preferred a `--first-conflict` option on `git rebase` itself and suggested script improvements including a `trap`.

### doc: cherry-pick: note `--no-commit` skips `CHERRY_PICK_HEAD`
*Long thread: 11 replies so far from 4 participants over 2 days.*
Aleksei Sviridkin's documentation-only change notes that `cherry-pick --no-commit` skips `CHERRY_PICK_HEAD`, so a plain `git commit` records the committer as author. V2 dropped the t3507 test after Patrick Steinhardt argued tests need stronger justification, while Phillip Wood disputed that existing tests cover the conflicting case.

### Rebase `--amend` regression after conflict commit
*Long thread: 10 replies so far from 4 participants over 1 day.*
Patrick Steinhardt posted a fix for `builtin/rebase` where the refusal of `git commit --amend` at a conflicted stop persisted after the user committed the resolution. The discussion converged on reverting commit 6257588252 for Git 2.56, which Junio C Hamano preferred at -rc2.

### branch: let `--delete-merged` find squash-merged branches
*Long thread: 10 replies so far from 3 participants over 1 day.*
Harald Nordgren posted a patch making `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all its changes, covering squash- and rebase-merged branches. D. Ben Knoble reviewed the approach, noting the check can be expensive, and floated a `--delete-merged=squashed,rebased` option value.

## The period in brief

September 2026 saw heavy, eventful traffic across all twenty-eight days, anchored by the Git v2.56.0 release cycle and an active debate over whether the next version should be 3.0. Long-running series dominated the discussion, with the `git history squash` command, the `receive-report` hook, and the runtime `core.useNanosec` config all drawing sustained review attention. Readers should not miss Junio C Hamano's proposed Git v3.0 timeline, the resolution of the `http.sslVerifyStatus` test failure to a GnuTLS backend limitation, and the fix for `git push --force-if-includes` consulting the wrong ref.

## Key developments

### Git 2.56 release and the road to 3.0
Junio C Hamano announced Git v2.56.0-rc0 on 2026/09/10 and v2.56.0-rc1 later in the month, comprising over 700 non-merge commits from 90 people. He laid out a proposed release schedule: Git 2.98 at year's end, 2.99 in March 2027 as a signal that something major is coming, 2.99.1 in April 2027 carrying only bugfixes, and 3.0 identical to 2.99.1 except with breaking changes enabled via the `WITH_BREAKING_CHANGES` knob. Patrick Steinhardt called GitHub's missing SHA-256 support the biggest question mark for a 3.0 bump, noting reftable is upstreamed in libgit2 with SHA-256 default-enabled. brian m. carlsson argued that software not already far along on SHA-256 is likely not worth considering for the 3.0 timeline.

### `--force-if-includes` push safety
Tyler Cipriani drove a multi-week arc fixing `git push --force-if-includes` to consult the reflog of the ref actually being pushed rather than a same-named local branch, a bug behind false rejections and potential data loss. The series progressed through v6, deferring reflog verification until after fast-forward rules are evaluated and adding a `pushRefUnverifiable` advice message suggesting `--force-with-lease=<ref>:<expect>`. Aleksei Sviridkin separately fixed the case where the remote-tracking ref has no reflog by initializing the cutoff timestamp to 0, which Hamano marked for `next` on 9/29.

### repack race with concurrent `.keep` files
Qin ShiCheng drove a series fixing a production-hit race where `git repack -d` deletes a pack whose objects were never copied because a concurrent push links a `.keep` file onto an existing pack between repack's initial scan and pack-objects' rescan. The core change has repack write its startup snapshot of kept packs to a tempfile and pass it to pack-objects via a new `--keep-pack-from-file=<file>` option, ensuring both processes work from one snapshot. Justin Tobler agreed to fix the receive-pack side by having the ODB transaction create the `.keep` itself at commit time.

### Rust integration and Windows CI
Mike Hommey posted v5 moving the Rust `gitcore` crate into a `rust/` subdirectory so Git can be vendored as a submodule without a top-level `Cargo.toml`. Separately, Johannes Schindelin's series enabling Rust in Windows CI jobs reached v4 using `CARGO_BUILD_TARGET`, and Hamano judged it ready, merging it to `next` and fast-tracking to `master` on 2026/09/13. Schindelin also posted a four-patch series fixing the GitLab CI cargo invocation in Windows jobs broken since Rust was enabled there.

### git var extensions for idents, signing key, and multiple args
Andrew Pleeter advanced his series teaching `git var` to expose `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, committer equivalents, `GIT_SIGNING_KEY`, multiple variables per call, and `-z`, reaching v9 by month's end. Phillip Wood pressed for the use case for `GIT_SIGNING_KEY` and objected to non-zero exit when one variable is unset; Junio C Hamano proposed modeling multi-variable output on `git var -l` var=value pairs rather than empty records.

### Avoid sending the whole tree when pushing from a shallow clone
Elijah Newren posted v3 of a six-patch series keeping `send-pack` from transferring the entire toplevel tree when pushing from a shallow clone. The series introduces a tri-state `push.shallowExcludeBoundary` option (true/false/abort) that omits shallow boundary objects from generated packs, defaulting to true to avoid transferring gigabytes of tree data, and adds `advice.pushShallowBoundary` suggesting one-at-a-time pushes when multi-ref pushes fail.

### RFC: a "safe" strbuf API
Derrick Stolee posted an RFC proposing a `strbuf-safe` API whose methods cannot transitively call `die()` or `exit()`, motivated by recursive die loops in trace2 under allocation failure. Phillip Wood objected to the name "safe" and identified that `sstrbuf_grow()` calls `st_add3()` which dies on overflow, undermining the verification goal. Junio C Hamano dissected the proposal, arguing the longer-term goal should be making service routines free of `die()` and identifying a memory leak in `srealloc()`.

### incremental connectivity check via rev-list
Kristofer Karlsson posted a two-patch series adding an opt-in incremental connectivity check gated behind `transfer.connectivityCheck=incremental`, running inside the rev-list subprocess via a new internal `--verify-trees-incremental` flag with benchmarks reporting up to ~190x speedup and lower RSS for small pushes. Junio C Hamano called the benchmarks "exciting" but raised concerns about a possibly too-late `is_promisor_object()` call and a possible NULL from `get_commit_tree_oid()`.

## In brief

**last-modified Bloom filter pre-filtering** -- Toon Claes posted v3 and v4 of the series reusing `git-log`'s Bloom-filter machinery to speed up `git-last-modified(1)`, but after the series was pushed to `seen`, CI failed reproducibly and the topic was ejected.

**worktree_basename() fixes** -- The four-patch series fixing an out-of-bounds read and string handling in `builtin/worktree.c` was marked for `next` by Junio C Hamano.

**proposed builtin `ident` command** -- Andrew Pleeter's `git ident` proposal was rejected in favor of extending `git var` with broken-down identity components, a view endorsed by Jeff King and Junio C Hamano.

**pathspec out-of-bounds fix** -- Yannik Tausch's patch fixes an out-of-bounds read in `dir.c` when a negative pathspec is shorter than the positive prefix; the issue was disclosed to the Git security mailing list and confirmed not to require an embargo.

**hooks.allowNoVerify rejected** -- Junio C Hamano rejected Alessio Attilio's proposal as a slippery slope; brian m. carlsson agreed, noting hooks are trivially bypassable via `core.hooksPath=/dev/null`.

**early_scan_options() series** -- Christian Couder posted a six-patch series adding an `early_scan_options()` sub-API to parse-options, fixing `--` detection in bisect and rev-parse; Hamano called the `fast-import --allow-unsafe-features` fix "Nicely done".

**git history split SIGSEGV** -- Jinbao Chen fixed a crash when the parent tree object is missing, erroring with "unable to parse tree" instead of dereferencing NULL; v2 guards a NULL tree in `write_ondisk_index()`.

**git imap-send --draft** -- Wolfgang Faust added a `--draft` option to set the IMAP `\Draft` flag; Hamano affirmed the flag-first approach.

**debian CI bump** -- Jeff King proposed changing the debian-11 CI job to debian-12 since Debian 11 exited LTS; Hamano will queue the patch.

**pack-objects prefetch fix** -- Aleksei Sviridkin fixed prefetch ordering so the batch starts correctly, reducing fetches from 2 to 1 in his test.

**sequencer --empty fix** -- Phillip Wood reviewed Farid Zakaria's v4 making the sequencer honor `--empty` when a `fixup!`/`squash!` empties its target, calling the approach sound.

**rev-list --missing-only** -- Siddharth Asthana posted v2 of a filter flag requiring `--missing=print` or `--missing=print-info`, explaining Gitaly uses the missing list as a one-shot dependency set.

**ODB source removal** -- Patrick Steinhardt posted a 12-patch series removing in-memory source registration so the source list is exactly the alternates list.

**Rust in Windows CI** -- Johannes Schindelin posted a two-patch series enabling Rust in Windows CI jobs by selecting a GCC-compatible Cargo target under MSYS2/MinGW.

**gitdatamodel(7) docs** -- Kristoffer Haugsbakk posted v3 advertising `gitdatamodel(7)` as a concept guide with links from `git(1)` and the glossary.

**--range-diff-notes docs** -- Haugsbakk circulated a draft rewrite; Hamano proposed `--no-range-diff-notes` should mean no notes are used for the range diff, mirroring `--no-notes`.

**test modernization** -- Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals, since the semantic helpers are meant to shout on failure.

**rerere gc heuristic** -- Steinhardt posted a two-patch series refining the auto heuristic to run only when estimated stale entries reach a threshold (default 512); Derrick Stolee said both look good.

**git pull overwrites local directory with symlink** -- AIKSXD ax reported that `git pull` silently replaces a local directory with a committed symlink of the same name when a `.gitignore` pattern like `dataset/` is present; brian m. carlsson explained the trailing-slash pattern ignores contents rather than the entry itself.

**worktree lifecycle hooks** -- Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks; Kristoffer Haugsbakk proposed a `--id=<string>` ownership alternative, which Kožar argued does not cover tools observing the lifecycle regardless of which tool invokes Git.

**precompiling git-compat-util.h** -- SZEDER Gábor posted a four-patch Makefile series precompiling `git-compat-util.h` for about a 35% build speedup, with patch 3/4 reintroducing `REFTABLE_OBJS` to exclude reftable sources.

**history: sign rewritten commits** -- Souma posted v3 teaching `git history` rewrite commands to honor `commit.gpgSign` and `-S`/`--gpg-sign`, adding a `sign_commit` field to `replay_revisions_options`.

**refs/files: avoid packed-refs lock for root ref deletion** -- Ariel Keselman posted v3 skipping the packed-refs lock when deleting root refs like `AUTO_MERGE`, fixing post-commit cleanup errors in linked worktrees with read-only shared metadata.

**Use the system asciidoctor in CI** -- Jeff King's patch drops the pinned asciidoctor 1.5.8 gem in favor of the Ubuntu package; Hamano decided to queue it after discovering the third patch of the series was never applied.

**midx-write: skip writes with no object entries** -- Pia Park posted v2 moving the empty-entries check so all write modes return silent success instead of publishing a zero-object MIDX.

**imap-send OpenSSL 4.1 preparation** -- Beat Bolli posted a three-patch series covering OpenSSL 4.1 preparation, ASN1_STRING NUL-termination, and RFC 6125 common-name checking.

**Outreachy application** -- Christian Couder submitted Git's Outreachy Community Application for the December 2026 cohort, offering to mentor two projects.

**git worktree add "" on Windows** -- Paul DE TEMMERMAN reported that `git worktree add "" HEAD` on Windows deletes the entire workspace including `.git` instead of failing safely.

**stash.index redundant entry** -- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry.

**range-diff-notes redesign** -- Kristoffer Haugsbakk proposed only `--no-RD-notes` and `--RD-notes=<ref>` for his redesign, accepted by Hamano.

**bash completion for worktree repair** -- Yoichi NAKAYAMA added completion for `git worktree repair`, completing paths to linked worktrees or those with broken links.

**documentation lint and quoting** -- Todd Zullinger posted a series fixing backtick-quoting in `git-refs` and `git-pack-refs` docs; Jean-Noël AVILA argued against extending the lint regex to commands due to false positives in definition lists.

**merge-ll temporary cleanup** -- Michal Koutný submitted a patch cleaning up `.merge_file_*` temporaries after Ctrl+C; Jeff King suggested using the tempfile cleanup handler and flagged an LLP64 narrowing hazard.

**git-blame ignore-revs default** -- Ravi Mistry proposed making `git blame` default to a `.git-blame-ignore-revs` file at the working-tree root.

**CARGO_BUILD_TARGET for Rust build** -- James Le Cuirot posted v3 making the Rust build respect `CARGO_BUILD_TARGET` when locating `libgitcore.a`.

**fetch-pack trace2 region** -- Hamano decided to queue Ted Nyman's "fetch-pack: trace packfile URI downloads" adding a Trace2 region around the download loop.

**history reword memory leak** -- Kaartic Sivaraam posted v3 adding the missing `repo_unuse_commit_buffer()` call fixing an AddressSanitizer leak in `builtin/history.c`.

**dir pathspec prefixes** -- Yannik Tausch's v4 series fixes pathspec prefix handling with leading exclude items by matching exclude items against full paths and deriving the common prefix from the first non-exclude item; Hamano marked it ready for `next`.

**upload-pack shallow fetch ordering** -- Royce Remer's v2 swaps `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow fetches with `uploadpack.allowRefInWant=true` work, fixing a bug present since the original ref-in-want implementation.

**rerere remaining skips paths** -- Junio C Hamano confirmed Mikko Rantalainen's diagnosis that `check_one_conflict()` skips consecutive conflicted paths with only stage-1 entries and posted a fix adding a same-path check to the while-loop condition.

**reflog expire defaults swapped** -- r.norouzi reported that `git reflog expire` prunes reachable entries older than 30 days instead of 90 because the defaults were swapped in commit `85658275702b` first released in 2.50.0.

**diff --no-index -R** -- Haokai Ding fixed `diff --no-index -R` for file/directory conflicts by swapping filespecs of the early queue entry when `reverse_diff` is set, collecting reviews from René Scharfe and Jacob Keller.

**Coverity fixes** -- Johannes Schindelin posted a seven-patch series fixing Coverity issues from v2.56.0-rc0 including `writev_in_full()` overflow and rerere failure handling; Hamano approved the rerere patch.

**Outreachy project** -- Kaartic Sivaraam and Christian Couder proposed "Implement promisor remote fetch ordering" for the December 2026 cohort.

**Test modernization** -- Mark C. Chu-Carroll posted v4 of the "t40*: modernize" series and accepted review suggestions.

**fetch.shallow config** -- Harald Nordgren submitted a patch adding a `fetch.shallow` option for shallow repositories.

**shallowHistory advice** -- Harald Nordgren posted a patch adding advice when `<ref>~N` resolution stops at a shallow boundary.

**worktree lifecycle hooks RFC** -- Maciej Ciemborowicz posted an RFC asking whether native hooks for worktree add/remove/move events should be a single hook or separate post-operation hooks.

**OpenBSD RUNTIME_PREFIX** -- Brad Smith submitted a patch enabling `RUNTIME_PREFIX` via `getexecpath()`, with Hamano asking about version applicability.

**winansi die_lasterr** -- Yongqiang Tian fixed `die_lasterr()` in `compat/winansi.c` passing its `va_list` incorrectly to `die_errno()`.

**odb transaction fsync** -- Justin Tobler's two-patch series fixes packfile flush during ODB transaction commit with `core.fsyncMethod=batch`.

**status --ignored bisect** -- René Scharfe bisected a partial-pathspec match bug to commit 95c11ecc73 and proposed removing an optimization check in `treat_directory()`.

**LESS=FRX documentation** -- brian m. carlsson explained Git sets `LESS=FRX` only when `LESS` is unset, with Todd Zullinger suggesting documentation of `LESS=""` overrides.

**refs copy/rename** -- Maciej Ciemborowicz posted v2 routing ref copy and rename through the transaction API; Junio praised the commit message and asked about avoiding hook-skipping flags.

**--fixup variations** -- Erik Cervin-Edin posted v3 allowing `-m`/`-F`/`-c`/`-C` for all `--fixup` variations.

**dir.c nested repo** -- René Scharfe posted v2 guarding the nested-repo `dir_match()` call with `pathspec &&` to fix a NULL pathspec crash.

**ls-files untracked cache** -- Tamir Duberstein posted v2 letting `ls-files` reuse the untracked cache, dropping a wildcard benchmark to 33.3 ms warm.

**create_repository** -- Patrick Steinhardt posted a 7-patch series making the repository parameter a pure out-parameter.

**Meson improvements** -- Patrick Steinhardt posted a 7-patch series speeding clean builds from ~6.8s to ~5.0s and fixing GitLab MSVC hangs.

**git-history during rebase** -- Nikita Bobko reported `git history` mid-rebase updates the branch being rebased, breaking `git rebase --continue`.

**GIT_WORK_TREE in hooks** -- Phil Sainty reported the post-checkout hook sees `GIT_DIR` but `GIT_WORK_TREE` is exported as `.` rather than the worktree path.

**parse_revision_opt()** -- Jeff King posted two fixes so unknown options are reported by name instead of `(null)`.

**credential/libsecret** -- Daniel Martí posted v2 loading secrets explicitly via `secret_item_load_secret_sync()`, producing useful errors instead of assertion failures.

**tempfile registration** -- Royce Remer registered temporary pack files with the tempfile subsystem so killed `git gc` runs leave no orphans.

**git replay signing** -- Patrick Monette added `-S`/`--gpg-sign` to `git replay`.

**fetch.followRemoteHEAD** -- Colin Hinton posted v3 deferring validation to `do_fetch()`.

**git-contacts stdin** -- Brigham Campbell posted v4 reading a patch from stdin via `-`.

**Documentation updates** -- jyotish kumar updated `git name-rev` docs from "SHA-1" to "object ID"; Matthias Goergens clarified `git remote set-head` is local-only; Kristoffer Haugsbakk proposed fixing `trailer.<key-alias>.cmd` examples; Tamir Duberstein posted v5 clarifying `--follow`'s single-file limitation.

**l10n** -- Jiang Xin submitted a pull request adding Afrikaans and Brazilian Portuguese and updating eight languages for Git 2.56.0, and said he will rebase l10n commits to prevent reverted commits from reappearing.

**.mailmap** -- Kristoffer Haugsbakk mapped his Fastmail and Gmail addresses to `code@khaugsbakk.name`.

**Optional sha1dc Rust backend for SHA-1** -- Johannes Schindelin proposed a compile-time option to use the Rust `sha1dc` crate for roughly three times faster `git index-pack` on Windows, with a `core.sha1dcBackend=c` escape hatch; brian m. carlsson objected that the crate requires Rust 1.87 while Git supports older versions.

**CI hard-fail on optional dependency install failures** -- Patrick Steinhardt suggested the CI script should fail outright when perforce, git-lfs or JGit fail to install rather than warn; Harald Nordgren agreed.

**`git -C` alias worktree environment bug** -- Maya Barriault reported that `git -C other_repo` run via an alias from a worktree sets `GIT_DIR` and `GIT_COMMON_DIR` to the worktree's values instead of the target repository's.

**Default reflog expiry periods swapped back** -- Pushkar Singh's v3 swaps back the default reflog expiry periods in `reflog.h` (reachable 90 days, unreachable 30); Patrick Steinhardt suggested testing at 88 days, which Junio C Hamano accepted.

**sparse-checkout clobbering untracked in-cone files** -- Webstrand reported that `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting, a regression from before 681c637b4a.

**merge-ort `--no-overwrite-ignore` bypass at rename destination** -- Coy Geek reported that `git merge --no-overwrite-ignore` fails to protect an ignored, untracked file when directory rename detection moves a committed file onto that path, reproduced on versions 2.54.0 through 2.56.0.

**Alternates moved into files ODB backend** -- Patrick Steinhardt posted a 13-patch series moving alternates into the "files" object database backend, dropping the `read_alternates` callback from `struct odb_source`.

**Fetch commit-graph from updated refs only** -- Kristofer Karlsson posted a series making fetch write the commit-graph from updated refs only; Patrick Steinhardt reviewed, noting no benchmark was provided.

**RFC: sign SHA-256 digest of tree contents** -- Scott Chacon posted an RFC signing a SHA-256 digest of tree contents in commits and tags; brian m. carlsson rejected it in favor of moving to SHA-256.

**Branch hook gaps for copy and reflog rewrite** -- Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook while `-c`/`-m` invoke it without the new value.

**fetch.followRemoteHEAD deferred validation** -- Colin Hinton posted v5 deferring `fetch.followRemoteHEAD` validation to the point of use in `do_fetch()`, storing the raw string with a `follow_remote_head_seen` flag after Junio C Hamano endorsed dying rather than warning on bad values.

**Documentation synopsis cleanup** -- Julia Evans published v2 removing required SYNOPSIS sections from section 7 man pages, with Tuomas Ahola's declarative lint-script rewrite.

**Zsh completion deduplication** -- Fionn Fitzmaurice patched the Zsh completion to exclude files already present in the argument list.

**interpret-trailers documentation examples** -- Kristoffer Haugsbakk posted v2 rephrasing the `trailer.<key-alias>.cmd` examples; Junio C Hamano marked the topic for `next` on 10/04.

**Precompiled header `__FILE__` expansion** -- SZEDER Gábor noted the precompiled `git-compat-util.h` changes `__FILE__` expansion in error messages, which he judged acceptable.

**repack `--dry-run` writes packs bug** -- Coy Geek reported that on Git 2.56.0, `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` writes new pack files and with `-d` deletes existing ones, contradicting the documented no-write promise.

**fsmonitor untracked cache invalidation** -- Ilia K posted a patch adding `untracked_cache_invalidate_stale_dirs()` so that after the fsmonitor daemon sends a trivial response, stale untracked-cache directories are invalidated before the index is saved.

**post-worktree hook with prune notification** -- Domen Kožar posted v3 of the series adding a single `post-worktree` lifecycle hook for add, move, and remove, extended to pruning with remove events for each pruned entry.

**git stash create `-u`/`-a` options** -- Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`; Junio C Hamano critiqued the commit message's user-visible framing.

**docs: remove gittutorial-2** -- Julia Evans replied that she is not replacing `gittutorial-2` since `gitdatamodel` and `gitcore-tutorial` cover its material; Junio C Hamano asked that `po/` translations be left intact.

**@{p} shorthand for @{push}** -- Harald Nordgren's patch makes `@{p}` short for `@{push}`; Junio C Hamano marked it queued.

**packed-refs fwrite() optimization** -- Karthik Nayak's packed-refs change writes unchanged refs with `fwrite()`, about 20% faster on deletes; Toon Claes listed sanitizations lost by fwrite() and proposed a separate preformatted writer function.

**git checkout -m conflict labels** -- Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`, fixing CRLF parsing with `strbuf_getline()`; Junio C Hamano agreed to document the common repository files separately.

**git repo structure ref-type filters** -- Patrick Steinhardt objected to Mark Chu-Carroll's ref-type filter flags for `git repo structure`, preferring revision-based filtering.

**backfill `--dry-run` RFC** -- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee questioned the size metric and proposed `--info=(count|size)`.

## Looking ahead

Several series posted late in September will dominate review in October. Harald Nordgren's shallow-repo fetch series v6 needs resolution of the precedence question between `remote.<name>.fetch` and `remote.<name>.refmap`, while Thomas Bachem's rerere v6 with the hidden `--skip-locked` option and Christian Couder's `uploadpack.lazyFetchTrusted` v5 await further maintainer feedback. Taylor Blau's cruft-less MIDX v2 faces the oidset pack-quality objection from Elijah Newren, and Julia Evans's `gitmergeconflicts` guide will see whether the proposed conflict-marker extensions are adopted. The Git v3.0 timeline proposal will likely generate discussion about which breaking changes to enable under the `WITH_BREAKING_CHANGES` knob.