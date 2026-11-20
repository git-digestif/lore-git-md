## Long threads

### `git repo info` gains more `path.*` keys

*Long thread: 37 replies so far from 5 participants over 74 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info` in `builtin/repo.c`, covering `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, each with absolute and relative suffixes. The series reached v7 on 2026/09/27 with eight patches, splitting the `get_superproject_working_tree()` fix into its own commit, removing dead code in the index path handlers, and fixing `path.cdup` to return an absolute path when the worktree lies outside the current directory, matching `git rev-parse --show-cdup`. Earlier revisions had addressed dead `if (!index_file)` checks and the discrepancy between `git repo` and `git rev-parse` output for `path.cdup`.

### Sequencer auto-maintenance timing

*Long thread: 32 replies so far from 5 participants over 20 days.*

Thomas Bachem's series makes merge-backend sequences run `git maintenance --auto` once when the sequence completes instead of inside the spawned `git commit`, `git merge`, and exec commands. By 2026/09/23 the series reached v5, with Phillip Wood reviewing the test comments and declaring it "looks ready for next to me". The placement of the call in `builtin/rebase.c` and `builtin/revert.c` rather than inside the sequencer was settled after Bachem spelled out that single picks return through paths that bypass `pick_commits()`.

### http.sslVerifyStatus OCSP stapling in the HTTP transport

*Long thread: 31 replies so far from 5 participants over 46 days.*

Grayson Gordon's series adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off because enabling it would block servers that do not staple. The topic was reverted out of `next` on 2026/09/23 after SZEDER Gábor reported a t5585 failure; by 2026/09/25 he confirmed his setup used the GnuTLS curl backend, which lacks proper OCSP support, and that switching to OpenSSL makes the test pass. The test prerequisite still needs refinement to check for the absence of "could not enable OCSP status verification" rather than merely testing the config setting.

### stash `--index` in-core merge cleanup

*Long thread: 28 replies so far from 3 participants over 9 days.*

D. Ben Knoble's series fixes the interaction between autostashing, staged index entries, and `stash.index=true` by replacing the shell-out sequence of `diff-tree`/`apply --cached`/`reset` in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. v3 posted on 2026/09/26 corrected the tree argument order to `(merge_base, head, merge)`, fixed a `merge_result` memory leak, and relabeled the conflict output. On 2026/09/27 Junio C Hamano identified a new bug in patch 5/5 where `o.verbosity = 0` is set unconditionally for the index merge but never restored for a later three-way merge reusing the same `merge_options`, and said he would eject the topic from `seen` until fixed.

### reference-transaction hooks reporting old values

*Long thread: 25 replies so far from 5 participants over 6 days.*

Maciej Ciemborowicz's series addresses the reference-transaction hook reporting all-zero old object IDs for unconditional updates. After earlier iterations extended `refs_delete_refs()` and its callers, v6 posted on 2026/09/24 moved the fix to the common transaction layer: the hook layer resolves the old value before the "preparing" hook and refreshes it after the backend locks the references, only when a hook exists, leaving `refs_delete_refs()`, `ref_transaction_delete()`, and all callers unchanged. Patrick Steinhardt had argued for exactly this common-layer approach earlier that day.

### --force-if-includes with a reflog-less remote-tracking ref

*Long thread: 20 replies so far from 6 participants over 23 days.*

Aleksei Sviridkin's patch fixes `push --force-if-includes` rejecting valid pushes when the remote-tracking ref has no reflog, by initializing the cutoff timestamp to 0 in `is_reachable_in_reflog()`. On 2026/09/25 Tyler Cipriani argued for `date=0` as always correct, giving a clone/reset/wait/commit/push scenario where a gc-based cutoff produces the wrong answer, and Junio C Hamano responded that with a never-expired reflog, using all available information works better than any cutoff.

### AsciiDoc cross-references and the tutorial rewrite in Documentation/

*Long thread: 17 replies so far from 4 participants over 4 days.*

Julia Evans's patch replaces "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references in the man pages so rendered text is controlled across asciidoc implementations. She reposted v2 on 2026/09/25 with a corrected commit message, and Jeff King said it looks good. She also announced a planned rewrite of `gittutorial.adoc` covering only basic commands; Junio C Hamano agreed the current tutorial has outlived its time but stressed that new guides must be sufficient to replace it.

### GitLab CI Windows cargo invocation fix

*Long thread: 12 replies so far from 3 participants over 6 days.*

Johannes Schindelin's four-patch series fixes the GitLab CI cargo invocation in Windows jobs, broken since Rust was enabled there. He posted v2 on 2026/09/24, rewording commit messages to clarify that "GNU Rust" means the GCC-targeting toolchain and correcting "GNU MSI" to "Rust MSI", and answered Patrick Steinhardt's questions about the `Gcc` feature and PATH quoting.

### New gitmergeconflicts(7) guide

*Long thread: 12 replies so far from 5 participants over 2 days.*

Julia Evans's seven-patch series adds `Documentation/gitmergeconflicts.adoc` explaining how to handle merge conflicts and links to it from the `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull` pages. On 2026/09/25 D. Ben Knoble flagged that the `git-merge.adoc` rewrite dropped both an explanation of what a conflict is and the `git log --merge -p <path>` tip, and Junio C Hamano objected that the `git-cherry-pick.adoc` rewrite removed cherry-pick-specific details such as `CHERRY_PICK_HEAD`.

### Rebase `--amend` regression after conflict commit

*Long thread: 10 replies so far from 4 participants over 1 day.*

Patrick Steinhardt posted a regression fix for `builtin/rebase`: since commit 6257588252 made `git commit` refuse `--amend` at a conflicted rebase stop, the refusal persisted after the user committed the resolution. Phillip Wood asked whether cherry-pick, merge, and revert share the problem, and Elijah Newren warned that `MERGE_MSG` is unreliable because `git reset` removes it without moving HEAD. The discussion converged on reverting 6257588252 for Git 2.56, which Junio C Hamano preferred at -rc2.

## The period in brief

The week of 2026/09/21 through 2026/09/27 saw active traffic across all seven days, with several long-running series reaching new iterations and one major topic reverted from `next` after a test failure. Junio C Hamano proposed a v3.0 release timeline, and contributors advanced work on refs transactions, the sequencer, stash, and documentation.

## Key developments

### Git v3.0 timeline proposal

Junio C Hamano laid out a proposed timeline: Git 2.98 at year's end, 2.99 in March 2027 as a signal that something major is coming, 2.99.1 in April 2027 carrying only accumulated bugfixes, and 3.0 identical to 2.99.1 except with breaking changes enabled via the `WITH_BREAKING_CHANGES` Makefile knob. He cautioned that anything beyond this year is "play it by ear" territory. Johannes Schindelin thanked him for the roadmap clarity, saying it helps plan the Git for Windows release schedule, while Harald Nordgren argued against signaling 3.0 in advance, a position Junio rejected by explaining that an advance version jump warns downstream distributions that something major is coming.

### refs batched deletions and transaction hooks

Maciej Ciemborowicz progressed his series from v4 to v6 during the week, ultimately moving the old-OID reporting for batched deletions to the common transaction layer after Patrick Steinhardt suggested that approach, leaving `refs_delete_refs()` and its callers unchanged. Earlier iterations had extended the API to accept parallel old-OID arrays, with Junio C Hamano reviewing v3 and arguing for always passing `REF_TRANSACTION_ALLOW_FAILURE` and counting individual failures rather than collapsing them to one.

### Sequencer auto-maintenance

Thomas Bachem's series reached v5 with Phillip Wood declaring it ready for `next`, settling on running `git maintenance --auto` once at sequence completion from `builtin/rebase.c` and `builtin/revert.c` rather than inside the sequencer, since single picks return through paths that bypass `pick_commits()`.

### OCSP stapling reverted from next

Grayson Gordon's `http.sslVerifyStatus` series was reverted out of `next` after SZEDER Gábor reported t5585 failures, later traced to a GnuTLS curl backend lacking OCSP support; the test prerequisite still needs to check for the absence of "could not enable OCSP status verification" rather than merely testing the config setting.

### Rebase --amend regression

Patrick Steinhardt's fix for the regression introduced by 6257588252 converged on reverting that commit for Git 2.56, with Elijah Newren warning that `MERGE_MSG` is unreliable because `git reset` removes it without moving HEAD.

### git-p4 shell injection fix

Anupam Mediratta's v2 replaced the shell pipeline in `applyCommit()` with two argument-vector subprocess calls connected by a pipe, removing shell interpretation of `--commit` values, and added an `ignore_error` parameter to `diffTreeApply()` so the two `tryPatch` callers can preserve the old behavior of ignoring apply failures while the main path still aborts on non-zero exit.

### Submodule merge corruption

Guillaume CHAUVEL reported that merging a superproject with submodules in Git v2.56.0-rc2 sometimes reads from submodule A a commit existing only in submodule B, failing with "repository corrupt"; his reproducer triggered it in all 50 Ubuntu and 43 of 50 Alpine runs, with an AI analysis pointing to a stale delta-base cache entry outliving its pack.

### Early option scanning API

Christian Couder posted v2 adding an `early_scan_options()` sub-API to parse-options that reuses the command's `struct option` array via a `PARSE_OPT_EARLY` flag, and converted `git fast-import`'s hand-rolled scan so `--allow-unsafe-features` after `--depth 5` is no longer silently ignored.

## In brief

**pull segfault fix** -- Jiri Kuncar posted v2 adding NULL guards in `get_can_ff()` and `already_up_to_date()` so a failed commit lookup falls through to the merge path instead of crashing.

**"safe" strbuf API** -- Junio C Hamano reviewed an RFC for a `strbuf-safe` API to prevent trace2 `die()` recursion, identifying a memory leak in `srealloc()` where the original pointer is clobbered before the failure check.

**repack kept-pack race** -- Qin ShiCheng's series hands pack-objects the startup snapshot to prevent losing objects when a `.keep` appears mid-run; Junio accepted the kept-pack cache invalidation point while noting the layering concern about the unchecked downcast to the files backend.

**--force-if-includes reflog** -- Aleksei Sviridkin's patch initializes the cutoff to 0 for reflog-less remote-tracking refs; Tyler Cipriani argued `date=0` is always correct with usually equivalent runtime.

**git-history during rebase** -- Nikita Bobko reported that running `git history` mid-rebase updates the branch being rebased, so `git rebase --continue` fails with a ref lock error.

**GIT_WORK_TREE in post-checkout hook** -- Phil Sainty found the hook sees `GIT_WORK_TREE=.` rather than the actual worktree path in worktrees, pointing at setup.c as the source.

**parse_revision_opt() fixes** -- Jeff King fixed two bugs so unknown options are reported by name instead of `(null)` in `git shortlog` and similar commands.

**pack-objects partial clone abort** -- L Naudé reported a BUG abort in `tree:0` partial clones when fetching missing promised trees via `is_not_in_promisor_pack_obj()`.

**dir.c nested repo** -- René Scharfe's v2 guards the `dir_match()` call against NULL pathspec for excluded directories containing nested repositories.

**git var extensions** -- Andrew Pleeter's v9 adds `-z` output, multiple variables per invocation, and broken-out `GIT_AUTHOR_*`/`GIT_COMMITTER_*` ident variables.

**default config values** -- Phillip Wood proposed a `core.defaults` setting for modern defaults; Junio responded that Git already changes defaults when beneficial but benefits remain speculative until released.

**credential/libsecret** -- Daniel Martí's v2 loads secrets explicitly via `secret_item_load_secret_sync()`, producing useful error messages instead of assertion failures.

**format-patch range-diff-notes** -- Kristoffer Haugsbakk's series adds `--range-diff-notes` options; Junio noted a redundant `test_when_finished` in patch 2/2.

**Documentation updates** -- Julia Evans advanced AsciiDoc cross-references and the `gitmergeconflicts(7)` guide; Tamir Duberstein clarified `--follow` in `git-log`; jyotish kumar updated `name-rev` docs from "SHA-1" to "object ID"; Matthias Goergens clarified that `git remote set-head` never changes the remote's own HEAD.

**Test and CI fixes** -- Royce Remer registered temp pack files with the tempfile subsystem so killed `git gc` runs leave no orphans; Jon Simons fixed perf test ref deletion with `--no-deref`; Harald Nordgren fixed CI leak-sanitizer annotations and macOS brew warnings; Tamir Duberstein used `cmp` for the t4205 comparison.

**Reflog expiry fix** -- Pushkar Singh's v3 fixed swapped default expiry periods (90 days reachable, 30 unreachable) with better-organized tests; Junio judged it probably ready for Git 2.56.

**l10n updates** -- Jiang Xin submitted pull requests for Git 2.56.0 adding Afrikaans and Brazilian Portuguese and updating eight other languages.

## Looking ahead

The `http.sslVerifyStatus` topic needs a corrected test prerequisite to distinguish curl backends with and without OCSP support before it can return to `next`. K Jayatheerth's `git repo info` path keys series at v7 and Thomas Bachem's sequencer auto-maintenance series, deemed ready for `next`, will see further review in the coming week. The rebase `--amend` regression revert for Git 2.56 will need confirmation in the release notes, and D. Ben Knoble's stash series awaits a fix for the verbosity bug in patch 5/5 before it can re-enter `seen`.