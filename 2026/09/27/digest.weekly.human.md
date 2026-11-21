## Long threads

### `git repo info` gains more `path.*` keys

*Long thread: 37 replies so far from 5 participants over 74 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info` in `builtin/repo.c` -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, each with absolute and relative suffixes. The series reached v7 on 2026/09/27, splitting the `get_superproject_working_tree()` fix into its own commit, removing dead `if (!index_file)` checks, and making `path.cdup` return an absolute path outside the work tree to match `git rev-parse --show-cdup`. Earlier in the week Jayatheerth agreed to drop the dead checks and objected to testing the superproject fix in `t1500-rev-parse.sh` since the bug also affects `git repo`.

### Sequencer auto-maintenance timing

*Long thread: 32 replies so far from 5 participants over 20 days.*

Thomas Bachem's series makes merge-backend sequences run `git maintenance --auto` once when the sequence completes instead of inside the spawned `git commit`, `git merge`, and exec commands. On 2026/09/22 he explained why the call sits in `builtin/rebase.c` and `builtin/revert.c` rather than inside the sequencer: single picks return through paths that bypass `pick_commits()`, so a sequencer-internal call would need to sit at multiple return points. Phillip Wood reviewed the v5 test comments on 2026/09/23 and said the series "looks ready for next to me".

### `http.sslVerifyStatus` OCSP stapling in the HTTP transport

*Long thread: 31 replies so far from 5 participants over 46 days.*

Grayson Gordon's series adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off because enabling it would block servers that do not staple. The topic was reverted out of `next` on 2026/09/23 after SZEDER Gábor reported a t5585 failure; by 2026/09/25 he confirmed his setup used the GnuTLS curl backend, which does not properly support OCSP, and that switching to OpenSSL makes the test pass. SZEDER also argued the `SSL_VERIFYSTATUS` prereq only checks libcurl support, not server capability, and should instead check for the absence of "could not enable OCSP status verification"; Junio C Hamano affirmed both suggestions while noting the prerequisite still needs improvement.

### stash `--index` in-core merge cleanup

*Long thread: 28 replies so far from 3 participants over 9 days.*

D. Ben Knoble's series fixes the interaction between autostashing, staged index entries, and `stash.index=true` by replacing the shell-out sequence of `diff-tree`/`apply --cached`/`reset` in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. v3 on 2026/09/26 fixed the tree argument order to `(merge_base, head, merge)`, addressed a `merge_result` leak, and relabeled the conflict output. On 2026/09/27 Junio C Hamano identified a new bug in patch 5/5: the patch unconditionally sets `o.verbosity = 0` for the index merge but reuses the same `merge_options` for a later three-way merge whose verbosity is then never restored; he proposed moving the quiet handling next to `init_ui_merge_options()` and said he would eject the topic from `seen` for the time being.

### reference-transaction hooks reporting old values

*Long thread: 25 replies so far from 5 participants over 6 days.*

The reference-transaction hook reports an all-zero old object ID for unconditional updates, so batched branch, tag, and remote-ref deletions report zero for both old and new values. On 2026/09/24 Maciej Ciemborowicz posted v6 as a single transaction-layer change: the common hook layer resolves the old value before the "preparing" hook and refreshes it after the backend locks the references, only when a hook exists, keeping the hook value separate from caller-supplied `old_oid`. Patrick Steinhardt had argued for exactly this common-layer approach earlier that day, and Maciej agreed the earlier batch-deletion API changes were the wrong layer, leaving `refs_delete_refs()` and all callers unchanged.

### --force-if-includes with a reflog-less remote-tracking ref

*Long thread: 20 replies so far from 6 participants over 23 days.*

Aleksei Sviridkin's patch fixes `push --force-if-includes` rejecting valid pushes when the remote-tracking ref has no reflog, by initializing the cutoff timestamp to 0 in `is_reachable_in_reflog()`. On 2026/09/25 Tyler Cipriani argued for `date=0` as always correct, giving a clone/reset/wait/commit/push scenario where a gc-based cutoff produces the wrong answer; Junio C Hamano responded that with a never-expired reflog, using all available information works better than any cutoff.

### AsciiDoc cross-references and the tutorial rewrite in Documentation/

*Long thread: 17 replies so far from 4 participants over 4 days.*

Julia Evans's patch replaces "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references in the man pages so rendered text is controlled across asciidoc implementations. She reposted v2 on 2026/09/25 with a corrected commit message, and Jeff King said it looks good. She also announced a planned rewrite of `gittutorial.adoc` covering only `git init`, `git add`, `git commit`, `git status`, `git diff`, `git push`, `git config`, and `git remote add`; Junio C Hamano agreed the current tutorial has outlived its time but stressed that new guides must be sufficient to replace it.

### GitLab CI Windows cargo invocation fix

*Long thread: 12 replies so far from 3 participants over 6 days.*

A four-patch series fixes the GitLab CI cargo invocation in Windows jobs, broken since Rust was enabled there. On 2026/09/24 Johannes Schindelin posted v2, rewording commit messages in patches 1 and 4 and answering Patrick Steinhardt's questions: "GNU Rust" meant the GCC-targeting toolchain, and patch 4 installs no second GCC -- the `Gcc` feature only ensures the installed Rust can produce libraries GCC can link to.

## The period in brief

The week of 2026/09/21 through 2026/09/27 saw seven active days of heavy, eventful traffic. Readers should not miss Junio C Hamano's proposed Git v3.0 timeline, the reference-transaction hook fix reaching v6 as a single transaction-layer change, and the diagnosis that the `http.sslVerifyStatus` test failure stemmed from a GnuTLS curl backend without OCSP support.

## Key developments

### Git v3.0 timeline proposal

Junio C Hamano laid out a proposed v3.0 timeline on 2026/09/24: 2.98 ending this year, 2.99 in March 2027 as a signal that something big is coming, 2.99.1 in April 2027 carrying only accumulated bugfixes, and 3.0 identical to 2.99.1 except with breaking changes enabled via the `WITH_BREAKING_CHANGES` Makefile knob. He cautioned that anything beyond this year is "play it by ear" territory. Johannes Schindelin thanked him for the roadmap clarity, and Ramsay Jones reported a successful Cygwin build of master with the experimental Rust package. In the related version-number thread, Harald Nordgren argued Git need not signal 3.0 in advance and proposed changing several default configuration values first; Junio rejected that interpretation, explaining an advance version jump warns downstream projects, and said Harald's proposed changes would not be part of 3.0 without prior draft code.

### Rebase `--amend` regression after conflict commit

Patrick Steinhardt posted a regression fix on 2026/09/23 for `builtin/rebase`: since commit 6257588252 made `git commit` refuse `--amend` at a conflicted rebase stop, the refusal persisted after the user committed the resolution. Phillip Wood asked whether cherry-pick, merge, and revert share the problem, and Elijah Newren warned `MERGE_MSG` is unreliable because `git reset` removes it without moving HEAD. The discussion converged on reverting 6257588252 for Git 2.56, which Junio C Hamano preferred at -rc2.

### git-p4 shell metacharacter injection fix

Anupam Mediratta submitted a patch replacing the shell-based `git diff-tree | git apply` pipeline in `P4Submit.applyCommit()` with two argument-vector subprocess calls connected directly through a pipe, so a `--commit` value like `$(some-command)` is no longer executed by the shell. He posted v2 on 2026/09/24 adding an `ignore_error` parameter to the new `diffTreeApply()` helper to preserve the old behavior where a failed apply aborts the submit. A regression test in `t/t9803-git-p4-shell-metachars.sh` verifies that shell metacharacters are never executed.

### Repack kept-pack snapshot

Qin ShiCheng's series stops `git repack -d` from deleting a pack whose objects were never copied when a `.keep` appears mid-run, by handing pack-objects the snapshot repack took at startup. Junio C Hamano reviewed v2 and questioned whether the unchecked downcast to the files backend is safe long-term, noting that until another ODB backend materializes the change might be sufficient but relying on the downcast feels like "laying mines for our future selves."

### Submodule merge reads wrong repository

Guillaume CHAUVEL reported that merging a superproject with submodules sometimes makes Git v2.56.0-rc2 read from submodule A a commit existing only in submodule B, failing with "repository corrupt" instead of a submodule conflict; his reproducer triggered it in all 50 Ubuntu and 43 of 50 Alpine runs. Patrick Steinhardt endorsed passing the repository explicitly into `can_all_from_reach()` to fix the superproject/submodule commit-graph merge failure.

### Early option scanning API

Christian Couder posted v2 on 2026/09/23 adding an `early_scan_options()` sub-API to parse-options that reuses the command's `struct option` array via a `PARSE_OPT_EARLY` flag, and converted `git fast-import`'s hand-rolled scan so `--allow-unsafe-features` after `--depth 5` is no longer silently ignored. Junio C Hamano agreed that converting `bisect_start()` to parse-options would make `--` detection come for free.

### pack-objects aborts on missing promised tree

L Naudé reported on 2026/09/26 that `git pack-objects --exclude-promisor-objects-best-effort` aborts with a BUG in `should_include_obj` during `git fetch origin main` in a `tree:0` partial clone of `NousResearch/hermes-agent` on Git 2.55.0, traced to `is_not_in_promisor_pack_obj()` called from `process_tree()` for a missing tree.

### Changing default config values

In the thread on what comes after Git 2.56, Phillip Wood proposed on 2026/09/27 a consequentialist approach to changing defaults for cosmetic UI settings such as `diff.algorithm`, `diff.colorWords`, `commit.verbose`, and `merge.conflictStyle`, plus a `core.defaults` setting offering evolving "modern defaults" enabled by default with a single opt-out. Junio C Hamano responded that Git already takes such an approach with repeated changes to the default merge strategy, but argued that net benefit and downside remain speculative until a change has been released for months, and questioned the need to poll `core.defaults` when it would be easier to measure how many users already set a target configuration.

## In brief

- **refs batched deletions old OIDs** -- Maciej Ciemborowicz posted v3 and v4 of the series teaching `refs_delete_refs()` to accept expected old OIDs, then v6 as a transaction-layer change; Karthik Nayak reviewed earlier iterations and questioned the all-or-nothing pruning semantics, while Junio C Hamano argued the flag should always be passed and partial failures counted.

- **safe strbuf API for trace2** -- Junio C Hamano dissected an RFC proposing a `strbuf-safe` API to prevent recursive `die()` loops in trace2, identified a memory leak in `srealloc()`, and suggested renaming functions that merely use a strbuf.

- **pull segfault fix** -- Jiri Kuncar posted v2 adding NULL guards in `get_can_ff()` and `already_up_to_date()` so a failed `lookup_commit_reference()` falls through to the normal merge path instead of segfaulting.

- **dir.c nested repo pathspec** -- René Scharfe posted v2 guarding the nested-repo `dir_match()` call with `pathspec &&` to fix the NULL pathspec crash Junio C Hamano had flagged in v1.

- **git var extension** -- Andrew Pleeter posted v9 of a four-patch series adding `-z` output, multiple variables per invocation, and broken-out `GIT_AUTHOR_*`/`GIT_COMMITTER_*` idents to `git var`.

- **git-history during interactive rebase** -- Nikita Bobko reported that running `git history` mid-rebase updates the branch being rebased, causing `git rebase --continue` to fail with a ref lock error, and proposed five possible fixes.

- **GIT_WORK_TREE post-checkout hook** -- Phil Sainty reported that in a worktree the post-checkout hook sees `GIT_DIR` but `GIT_WORK_TREE` is exported as `.` rather than the actual path; he tested Junio C Hamano's patch and found the problem persists in setup.c.

- **parse_revision_opt() bugfixes** -- Jeff King posted two fixes so unknown options are reported by name instead of `(null)` and malformed options are no longer called "unknown"; Junio C Hamano said both patches make sense.

- **credential/libsecret explicit loading** -- Daniel Martí posted v2 dropping the `SECRET_SEARCH_LOAD_SECRETS` flag and always loading the one secret via `secret_item_load_secret_sync()`, so an inaccessible item produces a useful error instead of assertion failures.

- **Pushkar Singh's reflog expiry fix** -- v3 of the fix for swapped default reflog expiry periods splits the regression test into four independent cases; Junio C Hamano judged the tests much better organized and answered "Probably" to fast-tracking for Git 2.56.

- **Early option scanning and fast-import** -- Christian Couder's v2 converts `git fast-import`'s hand-rolled scan to the new `early_scan_options()` API.

- **Julia Evans's gitmergeconflicts(7)** -- Her seven-patch series adds the new guide and links to it from merge, rebase, revert, cherry-pick, and pull pages; D. Ben Knoble flagged that the `git-merge.adoc` rewrite dropped the `git log --merge -p <path>` tip, and Junio C Hamano objected that the `git-cherry-pick.adoc` rewrite removed cherry-pick-specific details.

- **Tamir Duberstein's ls-files untracked cache** -- v2 lets `ls-files` reuse and update the untracked cache, with a wildcard query benchmark dropping to 33.3 ms warm.

- **Justin Tobler's packfile flush fix** -- v2 fixes the packfile flush during ODB transaction commit with batch loose-object fsync; Patrick Steinhardt said the version looks good.

- **Royce Remer's tempfile registration** -- Registers temporary pack files from `pack-write.c` and `pack-bitmap-write.c` with the tempfile subsystem so killed `git gc` runs no longer leave orphaned files.

- **Patrick Monette's git replay signing** -- Adds `-S`/`--gpg-sign` to `git replay` and fixes `pick_regular_commit()` to mark commit-creation failures.

- **Colin Hinton's fetch.followRemoteHEAD** -- v3 defers validation to `do_fetch()`.

- **Kristoffer Haugsbakk's range-diff-notes** -- His series adds `--range-diff-notes=<ref>` and `--no-range-diff-notes` to `format-patch`; Junio C Hamano noted a redundant `test_when_finished` in one test.

- **l10n updates** -- Jiang Xin submitted a pull request adding Afrikaans and Brazilian Portuguese and updating eight other languages for Git 2.56.0, and said he will rebase l10n commits onto the new upstream branch to prevent reverted commits from being reintroduced.

- **Documentation patches** -- jyotish kumar updated `git name-rev` descriptions from "SHA-1" to "object ID"; Matthias Goergens clarified that `git remote set-head` never changes the remote's own `HEAD`; Kristoffer Haugsbakk proposed fixing `trailer.<key-alias>.cmd` examples; Tamir Duberstein posted v5 clarifying `--follow`'s single-file limitation in `git-log.adoc`.