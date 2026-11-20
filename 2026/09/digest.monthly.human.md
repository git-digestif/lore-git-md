## Long threads

### history: add squash subcommand to fold a range
*Long thread: 91 replies so far from 9 participants over 87 days.*
Harald Nordgren's series adding `git history squash <revision-range>` dominated September discussion, reaching v15 with fixes for mixed-case hex rejection in `fixup!` subjects and completion test updates. Junio C Hamano noted that with the "complete 'git history'" topic merged, an evil merge for the completion test change was no longer needed. The series remained the most-discussed topic of the month, still in active review at period's end.

### receive-report hook for git-receive-pack
*Long thread: 58 replies so far from 7 participants over 29 days.*
Karthik Nayak's series adds a `receive-report` hook running after the reference transaction but before the client report, reading the pkt-line report on stdin with stdout replacing it and non-zero exit rejecting all refs. The month saw the series progress through v10 with fixes for a `BUG()` when no report was requested, a leak in `override_cmds_error()`, and a coccinelle fix for a redundant nullness check, leaving it in a stable state awaiting integration.

### stash: in-core index merge for `stash apply --index`
*Long thread: 53 replies so far from 5 participants over 13 days.*
D. Ben Knoble's series replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing a non-re-entrant lock bug when autostashing with `stash.index=true`. After evolving from v4 to v5 to fix tri-state handling of `merge_result.clean`, Phillip Wood confirmed the range-diff, and Junio C Hamano marked the series for `next` on 10/01.

### Convert build-time `USE_NSEC` into runtime `core.useNanosec`
*Long thread: 46 replies so far from 6 participants over 36 days.*
D. Ben Knoble's series exposes `USE_NSEC` as the runtime `core.useNanosec` config, letting distributed builds use nanosecond index timestamps to avoid racy-git false positives. After a v7 resend differing only in a documentation typofix, Junio C Hamano queued the three patches, noting they matched what he had already queued from v6.

### git-replay(1) gains a `--linearize` option
*Long thread: 39 replies so far from 7 participants over 86 days.*
Toon Claes's series adds a `--linearize` option to `git-replay(1)` that drops merge commits and replays all reachable commits into a single linear history. The month brought v9 with rephrased error messages, and Elijah Newren reviewed the series, stating "I think this version is good to go" and adding a `Reviewed-by` trailer, effectively clearing the series for integration.

### repo: add path keys to `git repo info`
*Long thread: 37 replies so far from 5 participants over 74 days.*
K Jayatheerth's GSoC series adds eight `path.*` keys to `git repo info` covering toplevel, superproject-root, hooks, index, grafts, git-prefix and cdup, each with absolute and relative suffixes. The series reached v7, splitting the `get_superproject_working_tree()` fix into its own commit and fixing `path.cdup` to return an absolute path when the worktree lies outside the current directory.

### repack: add `--drop-filtered` to reclaim space in partial clones
*Long thread: 37 replies so far from 5 participants over 51 days.*
This GSoC series lets users reclaim space in partial clones by enumerating promisor blobs over a size threshold, dropping them locally, and relying on lazy-fetch to restore them. Samuel Bronson reported testing `--drop-filtered` on a real llvm-project clone, finding it worked well, then proposed an RFC change to skip index-referenced blobs with a warning rather than aborting.

### Write object-database alternates at creation time
*Long thread: 36 replies so far from 5 participants over 18 days.*
Patrick Steinhardt's nine-patch series refactors `init_db()` into `create_repository()` and moves alternate writing into `odb_source_create_on_disk()`, removing the ad-hoc `odb_source_write_alternate()` callback. The month saw v4 and v5, with Justin Tobler reviewing v4 as good and Steinhardt addressing comments on sticky file-stream error indicators.

### packfile recovery when a MIDX names a removed pack
*Long thread: 36 replies so far from 5 participants over 15 days.*
Elijah Newren's series fixes objects being reported missing despite being present when a stale multi-pack-index names a pack removed by a concurrent geometric repack. Derrick Stolee reviewed patch 4/4 with "Good patch. LGTM." and Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`.

### uploadpack.lazyFetchTrusted server-side lazy fetch control
*Long thread: 34 replies so far from 3 participants over 85 days.*
Christian Couder's series introduces the protected `uploadpack.lazyFetchTrusted` config variable, letting server operators mark repositories trusted for lazy fetching during clones. V5 was rebased with no functional changes from v4, only documentation and test updates, including a NEEDSWORK comment about silently ignoring misspelled allowlist paths.

### sequencer: leave auto maintenance to the end of a sequence
*Long thread: 33 replies so far from 5 participants over 25 days.*
Thomas Bachem's series makes merge-backend rebase sequences run auto maintenance once at completion, matching the apply backend, by calling `run_auto_maintenance()` from `builtin/rebase.c` and `builtin/revert.c`. Patrick Steinhardt noted on 9/28 that the series is already cooking in `next` and agreed it looks ready, effectively concluding the thread's active development phase.

### Documentation: new `gitmergeconflicts(7)` guide
*Long thread: 32 replies so far from 6 participants over 10 days.*
Julia Evans's seven-patch series adds a `gitmergeconflicts` manual page linked from merge, rebase, revert, cherry-pick, and pull. The month saw v2 add a "WHAT IS A MERGE CONFLICT?" section explaining 3-way merge, with Junio C Hamano offering an illustration of why `diff3` style is essential, and Patrick Steinhardt proposing extending conflict-marker labels to include commit ID, message, and date.

### unifying ref storage format terminology
*Long thread: 29 replies so far from 4 participants over 12 days.*
Patrick Steinhardt's 13-patch v3 series renames inconsistent ref storage format interfaces to consistent `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases. Reviewers debated whether `OPT_ALIAS()` entries should be hidden, with Junio C Hamano explaining that `OPT_HIDDEN` belongs to a later deprecation stage.

### refs: report old values to reference-transaction hooks
*Long thread: 29 replies so far from 5 participants over 13 days.*
Maciej Ciemborowicz's v6 single-patch series resolves old object IDs in the common transaction layer before the "preparing" hook and after backend locks, fixing batched deletions reporting all-zero old OIDs. Patrick Steinhardt had argued for this transaction-layer approach over caller-specific fixes, and Ciemborowicz agreed the earlier batch-deletion API changes were the wrong layer.

### Repartition checkout/switch/restore away from monolithic `checkout_main()`
*Long thread: 27 replies so far from 5 participants over 8 days.*
Junio C Hamano's eight-patch draft reworks `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` to dispatch directly to `checkout_branch()` or `checkout_paths()` instead of the monolithic `checkout_main()`. Karthik Nayak agreed that patch 6 makes patch 1's removal of `cb_option` coherent, while Hamano noted the series "merely scratches the surface".

### repack: corner cases for cruft-less MIDXs
*Long thread: 27 replies so far from 5 participants over 4 days.*
Taylor Blau's v2 eight-patch series fixes cases where `repack.midxMustContainCruft=false` produces a multi-pack-index not closed under reachability, which can fail bitmap generation. Elijah Newren objected that oidset iteration order can regress pack quality by visiting subtrees first, causing blobs to be written in full instead of delta-compressed.

### checkout -m: refine autostash fallback
*Long thread: 26 replies so far from 4 participants over 42 days.*
Harald Nordgren's series makes autostash conflicts in `sequencer.c` exit with status 1 and separates conflict advice from the branch-switch message. v5 was queued after Hamano said both patches look good, with remaining discussion centering on CI style tooling and clang-format limitations.

### Refuse `git commit --amend` and partial commits during conflict resolution
*Long thread: 26 replies so far from 3 participants over 8 days.*
Elijah Newren's series makes `git commit --amend` and partial commits die during conflict resolution in merge, cherry-pick, revert, am, or rebase. Phillip Wood reviewed v4, finding the `FROM_REBASE_NOW_EMPTY` rename clearer, but expressed mixed feelings about permitting a partial commit when a rebase pick becomes empty.

### Git 3.0: restrict hex object IDs to lowercase only
*Long thread: 26 replies so far from 6 participants over 41 days.*
brian m. carlsson's series removes Git's acceptance of uppercase hex in object IDs for Git 3.0. v2 fixed a hex-range comment, updated commit messages, moved t5324 fixes to a separate commit, and restored lowercasing in `parse_oid_prefix`. Phillip Wood questioned whether refnames ending in object-ID-like strings should also be forbidden.

### last-modified Bloom filter pre-filtering
*Long thread: 26 replies so far from 5 participants over 56 days.*
Toon Claes's series speeds up `git last-modified` by reusing the revision machinery's Bloom filter checks, showing about 1.95x speedup with `-t` and 3.74x without on ziglang/zig. Patrick Steinhardt reviewed v4, noting `bloom_filter_contains()` can return -1 for empty filters, suggesting a rename and proposing to store `bloom_filter_settings` in `struct last_modified`.

### format-patch `--range-diff-notes` options
*Long thread: 25 replies so far from 4 participants over 42 days.*
Kristoffer Haugsbakk's series teaches `git format-patch` `--[no-]range-diff-notes=<ref>` so the range diff can use a different notes ref than the patches. The month saw v4 trim expository text and refactor the callback, then v5 adopt Junio C Hamano's condensed documentation wording.

### trace2 stops allowing die()
*Long thread: 25 replies so far from 6 participants over 49 days.*
Derrick Stolee's series bans die()-capable helpers from the low-level trace2 API via a root-level `banned-die.h` header. Jeff King argued that chasing transitive die() calls would need static analysis or a ground-up rewrite, though he is not opposed to the header in the meantime.

### Advice scoping for the defaultBranchName hint
*Long thread: 25 replies so far from 4 participants over 9 days.*
Vsevolod Myalitsin's series addresses the `defaultBranchName` advice, whose suggested `git config set` writes to local scope and thus only affects the repository where the advice was shown. The thread converged on Jeff King's simpler suggestion of recommending `--global` in all advice squelch messages, with Hamano posting a v5 patch adding `--global` across `advice.c` and 11 test files.

### push: fix `--force-if-includes` when the remote-tracking ref has no reflog
*Long thread: 23 replies so far from 6 participants over 27 days.*
Aleksei Sviridkin's v4 initializes the timestamp cutoff to 0 when the remote-tracking ref has no reflog, fixing spurious "remote ref updated since checkout" rejections. Junio C Hamano said on 9/29 that the v4 changes look sensible, will replace the patch and mark it for `next`.

### shallow-repo fetch of new remotes via refmap
*Long thread: 23 replies so far from 4 participants over 16 days.*
Harald Nordgren's four-patch series avoids fetching every branch of a new remote in shallow repositories by adding `remote.<name>.refmap`, inferring branches from local upstreams, and adding `git remote add --limited-fetch`. On 10/04 he posted v6 removing a leftover reference to deleted default-branch logic, with Hamano flagging a die when a remote has `refmap` and no tracked branches.

### rerere: wait for `MERGE_RR.lock` and skip in auto gc
*Long thread: 23 replies so far from 4 participants over 31 days.*
Thomas Bachem's series makes rerere writers wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds a hidden `--skip-locked` option to `git rerere gc` for auto-maintenance, and lets conflict-time callers warn and continue. On 10/02 he posted v6 renaming the option to `--skip-locked` and the flag to `RERERE_WARN_LOCKED`.

### Make t/lib-httpd CGI test helpers concurrency-safe
*Long thread: 23 replies so far from 3 participants over 58 days.*
Michael Montalbo's series fixes race conditions in the `apply-one-time-script.sh` and `http-429.sh` test helpers seen in flaky t5616 failures on macOS CI. Patrick Steinhardt reviewed v5 and said the version looks good to him.

### git var `-z`, multiple variables, broken-out idents
*Long thread: 22 replies so far from 6 participants over 39 days.*
Andrew Pleeter's v9 series splits into four patches adding string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, support for multiple variable arguments, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables. Phillip Wood reviewed the series as good with only minor test-style nits.

### xdiff: use `size_t` for `mmfile_t`
*Long thread: 21 replies so far from 4 participants over 3 days.*
Jeff King's seven-patch v2 consolidates `mmbuffer_t` into `mmfile_t`, switches buffer sizes to `size_t`, NUL-terminates buffers read by `read_mmfile()`, and makes `merge-ll` report errors when reading external merge results fails. Patrick Steinhardt flagged a commit-message nit and okayed the `merge-ll` change.

### CI annotations linked to test scripts
*Long thread: 20 replies so far from 4 participants over 10 days.*
Harald Nordgren's two-patch series links GitHub Actions failure and leak annotations to the test script's file and line, and stops leak-sanitizer jobs at the first failure. On 10/04 he asked whether the inability to link annotations to unchanged test files is enough to call the change a regression.

### AsciiDoc cross-references and the tutorial rewrite in Documentation/
*Long thread: 17 replies so far from 4 participants over 4 days.*
Julia Evans reposted v2 of her patch replacing "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references; Jeff King approved it. She also announced a planned rewrite of `gittutorial.adoc`, and Junio C Hamano agreed the current tutorial has outlived its time but stressed that new guides must be sufficient to replace it.

### GitLab CI Windows cargo invocation fix
*Long thread: 12 replies so far from 3 participants over 6 days.*
Johannes Schindelin posted v2 of the four-patch series fixing the GitLab CI cargo invocation in Windows jobs, rewording commit messages to clarify "Rust toolchain that targets GCC" and correcting "GNU MSI" to "Rust MSI".

### RFC for `git-brebase`
*Long thread: 12 replies so far from 2 participants over 1 day.*
Alejandro Colomar posted an RFC for `git-brebase`, a 164-line shell script that rebases onto the first conflicting commit of a target branch using `git bisect` internally. Nico Williams preferred a `--first-conflict` option on `git rebase` itself.

### doc: cherry-pick: note `--no-commit` skips `CHERRY_PICK_HEAD`
*Long thread: 11 replies so far from 4 participants over 2 days.*
Aleksei Sviridkin's documentation-only change notes that `cherry-pick --no-commit` skips `CHERRY_PICK_HEAD`, so a plain `git commit` records the committer as author. v2 dropped the t3507 test after Patrick Steinhardt argued tests need stronger justification.

### Rebase `--amend` regression after conflict commit
*Long thread: 10 replies so far from 4 participants over 1 day.*
Patrick Steinhardt posted a fix for `builtin/rebase` where the refusal of `git commit --amend` at a conflicted stop persisted after the user committed the resolution. The discussion converged on reverting commit 6257588252 for Git 2.56, which Junio C Hamano preferred at -rc2.

### branch: let `--delete-merged` find squash-merged branches
*Long thread: 10 replies so far from 3 participants over 1 day.*
Harald Nordgren posted a patch making `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all its changes, covering squash- and rebase-merged branches. D. Ben Knoble noted the check can be expensive and floated a `--delete-merged=squashed,rebased` option value.

## The period in brief

September 2026 saw heavy, eventful traffic across all 28 active days, anchored by the Git v2.56.0 release cycle (rc0, rc1, and final tag) and an extended debate over whether the next version should be 3.0. Long-running series dominated the month, with the `git history squash` command, the `receive-report` hook, and the runtime `core.useNanosec` config all drawing sustained review attention. Readers should not miss Junio C Hamano's proposed Git 3.0 timeline (2.98, 2.99, then 3.0), the multiple iterations of Tyler Cipriani's `--force-if-includes` push safety fixes, and the resolution of the `http.sslVerifyStatus` test failure to a GnuTLS backend limitation.

## Key developments

### Git v2.56 release and the road to 3.0
Junio C Hamano announced Git v2.56.0-rc0 on 2026/09/10 (635 non-merge commits from 82 people), v2.56.0-rc1 on 2026/09/16 (721 commits from 90 people), and tagged v2.56.0 by month's end. The same period saw Hamano propose a release schedule: Git 2.98 at year's end, 2.99 in March 2027 as a signal that something major is coming, 2.99.1 in April 2027 carrying only bugfixes, and 3.0 identical to 2.99.1 except with breaking changes enabled via the `WITH_BREAKING_CHANGES` knob. Patrick Steinhardt identified GitHub's missing SHA-256 support as the biggest question mark for a 3.0 bump, while brian m. carlsson argued that software not already far along on SHA-256 is likely not worth considering.

### push `--force-if-includes` consulting the wrong ref
Tyler Cipriani drove multiple iterations (v2 through v6) of a series making `git push --force-if-includes` consult the reflog of the ref actually being pushed rather than a same-named local branch, fixing false rejections and potential data loss. The series evolved to reject pushes whose source ref is not a branch, defer reflog verification until after fast-forward rules are evaluated, and add a `pushRefUnverifiable` advice message suggesting `--force-with-lease=<ref>:<expect>`. Aleksei Sviridkin separately fixed the case where the remote-tracking ref has no reflog by initializing the cutoff timestamp to 0, which Hamano marked for `next` on 9/29.

### repack race conditions and MIDX consistency
Qin ShiCheng drove a series fixing a production-hit race where `git repack -d` deletes a pack whose objects were never copied because a concurrent push links a `.keep` file between repack's initial scan and pack-objects' rescan, with the fix handing pack-objects a startup snapshot via `--keep-pack-from-file`. Taylor Blau posted v2 of a series fixing cruft-less MIDX corner cases where `repack.midxMustContainCruft=false` produces an index not closed under reachability, though Elijah Newren objected to oidset iteration order regressing pack quality. Elijah Newren also fixed objects being reported missing when a stale MIDX names a pack removed by a concurrent geometric repack.

### Rust integration and Windows CI
Johannes Schindelin's series enabling Rust in Windows CI jobs reached v4 using `CARGO_BUILD_TARGET`, and Hamano judged it ready, merging it to `next` and fast-tracking to `master` on 2026/09/13. Mike Hommey posted v5 moving the Rust `gitcore` crate into a `rust/` subdirectory so Git can be vendored as a submodule without a top-level `Cargo.toml`. Schindelin also posted v4 of a 13-patch series upstreaming Git for Windows patches needed for the UCRT64 switch, marked for `next`.

### sequencer auto-maintenance and rerere lock handling
Thomas Bachem's series making rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes reached v5 and was noted by Patrick Steinhardt as already cooking in `next` on 9/28. Bachem also advanced a separate series making rerere writers wait `rerere.lockTimeout` for `MERGE_RR.lock` and adding a hidden `--skip-locked` option to `git rerere gc`, reaching v6 by month's end.

### HTTP transport and OCSP stapling
Grayson Gordon's series adding `http.sslVerifyStatus` to check stapled OCSP responses reached v7 as a combined version splitting tests between `t5551` and a new `t5585`. After cooking in `next`, the topic was reverted when SZEDER Gábor reported a t5585 failure; he traced it to his GnuTLS curl backend lacking OCSP support, confirmed the test passes with OpenSSL, and proposed improving the `SSL_VERIFYSTATUS` prerequisite. Jeff King separately posted a patch for upcoming curl 8.23.0 stripping credentials from `CURLINFO_EFFECTIVE_URL`, proposing a `curl_url()`-based `match_effective_url()` approach that Hamano called "very nice".

### Outreachy December 2026 cohort
Christian Couder submitted Git's Outreachy Community Application for the December 2026 cohort, with Usman, Kaartic and Pablo volunteering as co-mentors. Proposed projects included continuing to remove global state, libifying parts of the code base, improving command argument scanning via `early_scan_options()`, and implementing promisor remote fetch ordering.

## In brief

**shallow-clone push optimization** -- Elijah Newren posted v3 of a six-patch series keeping `send-pack` from transferring the entire toplevel tree when pushing from a shallow clone, introducing a tri-state `push.shallowExcludeBoundary` option.

**ref storage terminology** -- Patrick Steinhardt's 11-patch series renaming `--ref-format=` to `--ref-storage=` and related variables drew objection from Hamano that neither term is clear unless everything becomes "ref-storage-format".

**use-after-free in `setup_revisions()`** -- Jeff King posted a patch fixing a use-after-free where `setup_revisions()` freed argv strings that options like `--src-prefix` still referenced, garbling `git stash show` prefix output.

**worktree_basename() fixes** -- A four-patch series fixing an out-of-bounds read in `builtin/worktree.c` was marked for `next`.

**proposed builtin `ident` command** -- Andrew Pleeter's `git ident` proposal was rejected in favor of extending `git var` with broken-down identity components, reaching v9 with `-z` output and multiple variables.

**pathspec out-of-bounds fix** -- Yannik Tausch's patch fixes an out-of-bounds read in `dir.c` when a negative pathspec is shorter than the positive prefix; disclosed to the Git security mailing list and confirmed not to require an embargo.

**hooks.allowNoVerify rejected** -- Junio C Hamano rejected Alessio Attilio's proposal as a slippery slope.

**early_scan_options() series** -- Christian Couder posted a six-patch series adding an `early_scan_options()` sub-API to parse-options, fixing `--` detection in bisect and rev-parse.

**git history split SIGSEGV** -- Jinbao Chen fixed a crash when the parent tree object is missing in `builtin/history.c`.

**git imap-send --draft** -- Wolfgang Faust added a `--draft` option to set the IMAP `\Draft` flag.

**debian CI bump** -- Jeff King proposed changing the debian-11 CI job to debian-12 since Debian 11 exited LTS.

**pack-objects prefetch fix** -- Aleksei Sviridkin fixed prefetch ordering so the batch starts correctly.

**sequencer --empty fix** -- Phillip Wood reviewed Farid Zakaria's v4 making the sequencer honor `--empty` when a `fixup!`/`squash!` empties its target.

**rev-list --missing-only** -- Siddharth Asthana posted v2 of a filter flag requiring `--missing=print` or `--missing=print-info`.

**ODB source removal** -- Patrick Steinhardt posted a 12-patch series removing in-memory source registration so the source list is exactly the alternates list.

**gitdatamodel(7) docs** -- Kristoffer Haugsbakk posted v3 advertising `gitdatamodel(7)` as a concept guide.

**test modernization** -- Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals.

**rerere gc heuristic** -- Steinhardt posted a two-patch series refining the auto heuristic to run only when estimated stale entries reach a threshold.

**worktree lifecycle hooks** -- Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks; later v3 consolidated to a single `post-worktree` hook.

**precompiling git-compat-util.h** -- SZEDER Gábor posted a four-patch Makefile series precompiling `git-compat-util.h` for about a 35% build speedup.

**history: sign rewritten commits** -- Souma posted v3 teaching `git history` rewrite commands to honor `commit.gpgSign` and `-S`/`--gpg-sign`.

**refs/files: avoid packed-refs lock for root ref deletion** -- Ariel Keselman posted v3 skipping the packed-refs lock when deleting root refs like `AUTO_MERGE`.

**Use the system asciidoctor in CI** -- Jeff King's patch drops the pinned asciidoctor 1.5.8 gem in favor of the Ubuntu package.

**midx-write: skip writes with no object entries** -- Pia Park posted v2 moving the empty-entries check so all write modes return silent success.

**imap-send OpenSSL 4.1 preparation** -- Beat Bolli posted a three-patch series covering OpenSSL 4.1 preparation and RFC 6125 common-name checking.

**git worktree add "" on Windows** -- Paul DE TEMMERMAN reported that `git worktree add "" HEAD` on Windows deletes the entire workspace including `.git`.

**stash.index redundant entry** -- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry.

**range-diff-notes redesign** -- Kristoffer Haugsbakk proposed only `--no-RD-notes` and `--RD-notes=<ref>` for his redesign.

**bash completion for worktree repair** -- Yoichi NAKAYAMA added completion for `git worktree repair`.

**documentation lint and quoting** -- Todd Zullinger posted a series fixing backtick-quoting in `git-refs` and `git-pack-refs` docs.

**merge-ll temporary cleanup** -- Michal Koutný submitted a patch cleaning up `.merge_file_*` temporaries after Ctrl+C; Jeff King suggested using the tempfile cleanup handler.

**git-blame ignore-revs default** -- Ravi Mistry proposed making `git blame` default to a `.git-blame-ignore-revs` file at the working-tree root.

**CARGO_BUILD_TARGET for Rust build** -- James Le Cuirot posted v3 making the Rust build respect `CARGO_BUILD_TARGET` when locating `libgitcore.a`.

**fetch-pack trace2 region** -- Hamano decided to queue Ted Nyman's "fetch-pack: trace packfile URI downloads".

**history reword memory leak** -- Kaartic Sivaraam posted v3 adding the missing `repo_unuse_commit_buffer()` call fixing an AddressSanitizer leak.

**git pull overwrites local directory with symlink** -- AIKSXD ax reported that `git pull` silently replaces a local directory with a committed symlink when a `.gitignore` pattern like `dataset/` is present.

**dir pathspec prefixes** -- Yannik Tausch's v4 series fixes pathspec prefix handling with leading exclude items; Hamano marked it ready for `next`.

**upload-pack shallow fetch ordering** -- Royce Remer's v2 swaps `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow fetches with `uploadpack.allowRefInWant=true` work.

**rerere remaining skips paths** -- Hamano confirmed Mikko Rantalainen's diagnosis that `check_one_conflict()` skips consecutive conflicted paths with only stage-1 entries and posted a fix.

**reflog expire defaults swapped** -- r.norouzi reported that `git reflog expire` prunes reachable entries older than 30 days instead of 90 because the defaults were swapped in commit `85658275702b`; Pushkar Singh's v3 fixes this.

**diff --no-index -R** -- Haokai Ding fixed `diff --no-index -R` for file/directory conflicts by swapping filespecs of the early queue entry when `reverse_diff` is set.

**Coverity fixes** -- Johannes Schindelin posted a seven-patch series fixing Coverity issues from v2.56.0-rc0.

**Outreachy project** -- Kaartic Sivaraam and Christian Couder proposed "Implement promisor remote fetch ordering" for the December 2026 cohort.

**Test modernization** -- Mark C. Chu-Carroll posted v4 of the "t40*: modernize" series.

**fetch.shallow config** -- Harald Nordgren submitted a patch adding a `fetch.shallow` option for shallow repositories.

**shallowHistory advice** -- Harald Nordgren posted a patch adding advice when `<ref>~N` resolution stops at a shallow boundary.

**OpenBSD RUNTIME_PREFIX** -- Brad Smith submitted a patch enabling `RUNTIME_PREFIX` via `getexecpath()`.

**winansi die_lasterr** -- Yongqiang Tian fixed `die_lasterr()` in `compat/winansi.c` passing its `va_list` incorrectly to `die_errno()`.

**odb transaction fsync** -- Justin Tobler's two-patch series fixes packfile flush during ODB transaction commit with `core.fsyncMethod=batch`.

**status --ignored bisect** -- René Scharfe bisected a partial-pathspec match bug to commit 95c11ecc73.

**LESS=FRX documentation** -- brian m. carlsson explained Git sets `LESS=FRX` only when `LESS` is unset.

**refs copy/rename** -- Maciej Ciemborowicz posted v2 routing ref copy and rename through the transaction API.

**--fixup variations** -- Erik Cervin-Edin posted v3 allowing `-m`/`-F`/`-c`/`-C` for all `--fixup` variations.

**dir.c nested repo** -- René Scharfe posted v2 guarding the nested-repo `dir_match()` call with `pathspec &&`.

**ls-files untracked cache** -- Tamir Duberstein posted v2 letting `ls-files` reuse the untracked cache.

**create_repository** -- Patrick Steinhardt posted a 7-patch series making the repository parameter a pure out-parameter.

**Meson improvements** -- Patrick Steinhardt posted a 7-patch series speeding clean builds from ~6.8s to ~5.0s.

**git-history during rebase** -- Nikita Bobko reported `git history` mid-rebase updates the branch being rebased, breaking `git rebase --continue`.

**GIT_WORK_TREE in hooks** -- Phil Sainty reported the post-checkout hook sees `GIT_DIR` but `GIT_WORK_TREE` is exported as `.`.

**parse_revision_opt()** -- Jeff King posted two fixes so unknown options are reported by name instead of `(null)`.

**credential/libsecret** -- Daniel Martí posted v2 loading secrets explicitly via `secret_item_load_secret_sync()`.

**tempfile registration** -- Royce Remer registered temporary pack files with the tempfile subsystem so killed `git gc` runs leave no orphans.

**git replay signing** -- Patrick Monette added `-S`/`--gpg-sign` to `git replay`.

**fetch.followRemoteHEAD** -- Colin Hinton posted v3 deferring validation to `do_fetch()`.

**git-contacts stdin** -- Brigham Campbell posted v4 reading a patch from stdin via `-`.

**Documentation updates** -- jyotish kumar updated `git name-rev` docs; Matthias Goergens clarified `git remote set-head`; Tamir Duberstein posted v5 clarifying `--follow`'s single-file limitation.

**l10n** -- Jiang Xin submitted a pull request adding Afrikaans and Brazilian Portuguese and updating eight languages for Git 2.56.0.

**.mailmap** -- Kristoffer Haugsbakk mapped his Fastmail and Gmail addresses to `code@khaugsbakk.name`.

**reftable reflog timezone** -- Patrick Steinhardt posted a three-patch series fixing `refs/reftable-backend.c` storing reflog timezones as signed HHMM integers instead of the specification's signed minute offsets; Hamano said v2 looks ready for `next`.

**packfile delta-base cache corruption** -- Philippe Blain bisected a superproject merge failure to commit bb5da75d61; Patrick Steinhardt identified stale delta-base cache entries keyed by `struct packed_git` address and posted a two-patch fix.

**"Limbo" object-format state** -- Jeff King proposed letting an empty server-side repository advertise a "limbo" object format so first pushes from either SHA-1 or SHA-256 clients succeed, with brian m. carlsson arguing against it.

**filter-branch state-branch fix** -- Grant Moyer posted v3 fixing `git filter-branch --state-branch` map initialization, which since f6d855091e interpreted object pairs in reverse.

**Optional sha1dc Rust backend** -- Johannes Schindelin proposed a compile-time option to use the Rust `sha1dc` crate for faster `git index-pack` on Windows.

**CI hard-fail on optional dependency install failures** -- Patrick Steinhardt suggested the CI script should fail outright when perforce, git-lfs or JGit fail to install.

**`git -C` alias worktree environment bug** -- Maya Barriault reported that `git -C other_repo` run via an alias from a worktree sets `GIT_DIR` and `GIT_COMMON_DIR` to the worktree's values.

**sparse-checkout clobbering untracked in-cone files** -- Webstrand reported that `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting.

**merge-ort `--no-overwrite-ignore` bypass** -- Coy Geek reported that `git merge --no-overwrite-ignore` fails to protect an ignored, untracked file when directory rename detection moves a committed file onto that path.

**Incremental connectivity check via rev-list** -- Kristofer Karlsson posted v2 of the opt-in `transfer.connectivityCheck=incremental` mode, reporting up to 190x speedup for single-commit pushes.

**Alternates moved into files ODB backend** -- Patrick Steinhardt posted a 13-patch series moving alternates into the "files" object database backend.

**Fetch commit-graph from updated refs only** -- Kristofer Karlsson posted a series making fetch write the commit-graph from updated refs only.

**RFC: sign SHA-256 digest of tree contents** -- Scott Chacon posted an RFC signing a SHA-256 digest of tree contents in commits and tags; brian m. carlsson rejected it in favor of moving to SHA-256.

**Branch hook gaps for copy and reflog rewrite** -- Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook.

**fetch.followRemoteHEAD deferred validation** -- Colin Hinton posted v5 deferring `fetch.followRemoteHEAD` validation to the point of use.

**Documentation synopsis cleanup** -- Julia Evans published v2 removing required SYNOPSIS sections from section 7 man pages.

**Zsh completion deduplication** -- Fionn Fitzmaurice patched the Zsh completion to exclude files already present in the argument list.

**interpret-trailers documentation examples** -- Kristoffer Haugsbakk posted v2 rephrasing the `trailer.<key-alias>.cmd` examples; Hamano marked it for `next` on 10/04.

**repack `--dry-run` writes packs bug** -- Coy Geek reported that on Git 2.56.0, `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` writes new pack files.

**fsmonitor untracked cache invalidation** -- Ilia K posted a patch adding `untracked_cache_invalidate_stale_dirs()`.

**git stash create `-u`/`-a` options** -- Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`.

**docs: remove gittutorial-2** -- Julia Evans replied that she is not replacing `gittutorial-2` since `gitdatamodel` and `gitcore-tutorial` cover its material.

**@{p} shorthand for @{push}** -- Harald Nordgren's patch makes `@{p}` short for `@{push}`; Hamano marked it queued.

**packed-refs fwrite() optimization** -- Karthik Nayak's packed-refs change writes unchanged refs with `fwrite()`, about 20% faster on deletes.

**git checkout -m conflict labels** -- Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`.

**git repo structure ref-type filters** -- Patrick Steinhardt objected to Mark Chu-Carroll's ref-type filter flags for `git repo structure`.

**backfill `--dry-run` RFC** -- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee questioned the size metric.

## Looking ahead

Several series posted late in September will dominate October review. Harald Nordgren's shallow-repo fetch series v6 needs resolution of the precedence question between `remote.<name>.fetch` and `remote.<name>.refmap`, while Thomas Bachem's rerere v6 with the hidden `--skip-locked` option and Christian Couder's `uploadpack.lazyFetchTrusted` v5 await further maintainer feedback. Taylor Blau's cruft-less MIDX v2 faces the oidset pack-quality objection from Elijah Newren, and Julia Evans's `gitmergeconflicts` guide will see whether the proposed conflict-marker extensions are adopted. The Git v3.0 timeline proposal will likely generate discussion about which breaking changes to enable under the `WITH_BREAKING_CHANGES` knob, with GitHub SHA-256 support remaining the central open question.