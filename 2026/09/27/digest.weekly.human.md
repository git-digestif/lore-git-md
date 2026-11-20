## Long threads

### `git repo info` path keys series
*Long thread: 37 replies so far from 5 participants over 74 days.*

K Jayatheerth's GSoC series adds eight path keys to `git repo info` (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, `path.cdup`), each with absolute and relative suffixes. The series reached v7 this week, splitting the `get_superproject_working_tree()` fix into its own commit, removing dead `if (!index_file)` checks, and fixing `path.cdup` to return an absolute path when the worktree lies outside the current directory, matching `git rev-parse --show-cdup`. Earlier revisions had drawn attention to the superproject bug relying on the process working directory rather than the repository being inspected.

### Sequencer auto-maintenance timing
*Long thread: 32 replies so far from 5 participants over 20 days.*

Thomas Bachem's series moves `git maintenance --auto` from individual sequencer-spawned commands to a single call at the end of a rebase, cherry-pick, or revert sequence, implemented via `disable_auto_maintenance()` in `sequencer.c` and calls in `builtin/rebase.c` and `builtin/revert.c`. The week centered on placement: Bachem asked Patrick Steinhardt to choose between the builtins and the sequencer internals, and Phillip Wood reviewed the v5 tests and declared the series "looks ready for next to me."

### http.sslVerifyStatus OCSP stapling in the HTTP transport
*Long thread: 31 replies so far from 5 participants over 46 days.*

Grayson Gordon's series adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to false. After cooking in `next`, the topic was reverted when SZEDER Gábor reported a t5585 failure; this week he traced the failure to his GnuTLS curl backend lacking OCSP support, confirmed the test passes with OpenSSL, and proposed improving the `SSL_VERIFYSTATUS` prerequisite to check for the specific error message rather than the config setting. Patrick Steinhardt and Junio C Hamano endorsed the diagnosis.

### stash `--index` in-core merge cleanup
*Long thread: 28 replies so far from 3 participants over 9 days.*

D. Ben Knoble's v3 series replaces the shell-out `diff-tree`/`apply --cached`/`reset` sequence in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing the autostash/`stash.index=true` bug where `git reset` triggered a non-re-entrant lock. The week saw the tree-argument order bug diagnosed by Junio C Hamano (transposed arguments differing from `merge_ort_nonrecursive()`), a memory leak fix, and a new content-level test from Phillip Wood. On Sunday Junio identified a further bug in patch 5/5 where `o.verbosity = 0` is set unconditionally and never restored for a later merge, and ejected the topic from `seen`.

### reference-transaction hooks reporting old values
*Long thread: 25 replies so far from 5 participants over 6 days.*

Maciej Ciemborowicz posted v6 of the series fixing the reference-transaction hook reporting all-zero old OIDs for batched deletions. The new approach resolves the old value in the common hook layer before the "preparing" hook and refreshes it after the backend locks refs, keeping the hook value separate from caller-supplied `old_oid`. Patrick Steinhardt had argued for exactly this transaction-layer approach, and Ciemborowicz agreed the earlier batch-deletion API changes were the wrong layer, leaving `refs_delete_refs()` and callers unchanged.

### --force-if-includes with a reflog-less remote-tracking ref
*Long thread: 20 replies so far from 6 participants over 23 days.*

Aleksei Sviridkin's patch fixes `push --force-if-includes` rejecting valid pushes when the remote-tracking ref has no reflog by initializing the cutoff timestamp to 0 in `is_reachable_in_reflog()`. Tyler Cipriani argued for `date=0` as always correct, giving a clone/reset/wait/commit/push scenario where a gc-based cutoff produces the wrong answer, and Junio C Hamano agreed that with a never-expired reflog, using all available information works better than any cutoff.

### AsciiDoc cross-references and the tutorial rewrite in Documentation/
*Long thread: 17 replies so far from 4 participants over 4 days.*

Julia Evans reposted v2 of her patch replacing "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references; Jeff King approved it. She also announced a planned rewrite of `gittutorial.adoc` covering only basic commands, and Junio C Hamano agreed the current tutorial has outlived its time but stressed that new guides must be sufficient to replace it.

### GitLab CI Windows cargo invocation fix
*Long thread: 12 replies so far from 3 participants over 6 days.*

Johannes Schindelin posted v2 of the four-patch series fixing the GitLab CI cargo invocation in Windows jobs, rewording commit messages to clarify "Rust toolchain that targets GCC" and correcting "GNU MSI" to "Rust MSI", and answering Patrick Steinhardt's questions about the MinGW build and PATH quoting.

### New gitmergeconflicts(7) guide
*Long thread: 12 replies so far from 5 participants over 2 days.*

Julia Evans's seven-patch series adds `Documentation/gitmergeconflicts.adoc` and links it from merge, rebase, revert, cherry-pick, and pull pages. D. Ben Knoble flagged that the `git-merge.adoc` rewrite dropped the `git log --merge -p <path>` tip, which Evans proposed restoring, while Junio C Hamano objected that the cherry-pick page rewrite removed `CHERRY_PICK_HEAD` details.

### Rebase `--amend` regression after conflict commit
*Long thread: 10 replies so far from 4 participants over 1 day.*

Patrick Steinhardt posted a fix for `builtin/rebase` where the refusal of `git commit --amend` at a conflicted stop persisted after the user committed the resolution. Phillip Wood asked whether cherry-pick, merge, and revert share the problem, and Elijah Newren warned that `MERGE_MSG` is unreliable. The discussion converged on reverting commit 6257588252 for Git 2.56, which Junio C Hamano preferred at -rc2.

## The period in brief

The week of 2026/09/21 through 2026/09/27 saw seven active days of steady, technical traffic dominated by long-running series reaching critical review stages. Readers should not miss Junio C Hamano's proposed Git v3.0 timeline, the resolution of the `http.sslVerifyStatus` test failure to a GnuTLS backend limitation, and the ejection of D. Ben Knoble's stash series from `seen` after a newly spotted verbosity bug.

## Key developments

### Git v3.0 timeline proposal
Junio C Hamano laid out a proposed release schedule: Git 2.98 at year's end, 2.99 in March 2027 as a signal that something major is coming, 2.99.1 in April 2027 carrying only bugfixes, and 3.0 identical to 2.99.1 except with breaking changes enabled via the `WITH_BREAKING_CHANGES` knob. He cautioned that anything beyond this year is "play it by ear." Johannes Schindelin thanked him for the roadmap clarity, and Harald Nordgren argued against advance signaling, which Junio rejected, explaining that the version jump warns downstream distributions. Phillip Wood separately proposed a consequentialist approach to changing default config values and a `core.defaults` setting, which Junio met with skepticism about speculative benefits.

### refs batched deletions and transaction hooks
Maciej Ciemborowicz advanced his series teaching `refs_delete_refs()` to accept old OIDs through v3, v4, and finally v6, which moved the fix to the common transaction layer so the reference-transaction hook receives real old values for batched deletions without changing the public API. Karthik Nayak's early review questioned the all-or-nothing pruning semantics and asked whether an LLM generated the patches; Ciemborowicz disclosed AI assistance and took responsibility. Junio C Hamano argued for per-ref transactions or best-effort mode to preserve the documented "attempt to keep going" semantics.

### "safe" strbuf API for trace2
Junio C Hamano dissected an RFC proposing a `strbuf-safe` API to prevent recursive `die()` loops in trace2's json-writer. He argued the longer-term goal should be making service routines free of `die()`, identified a memory leak in `srealloc()` where the original pointer is clobbered before the failure check, and questioned the semantics of `enum safe_result`, offering alternative implementations.

### git-p4 shell metacharacter injection
Anupam Mediratta submitted v2 of a fix replacing the shell-based `git diff-tree | git apply` pipeline in `P4Submit.applyCommit()` with two argument-vector subprocess calls, preventing execution of shell metacharacters in the unvalidated `--commit` option. The v2 adds an `ignore_error` parameter to preserve the old abort-on-failure behavior for `tryPatch` callers.

### repack kept-pack snapshot
Qin ShiCheng's series fixes a race where a concurrent push installs a `.keep` next to a pack that `git repack -d` has already decided to delete, causing pack-objects to omit objects while repack deletes the pack. The fix hands pack-objects the startup snapshot. Junio C Hamano accepted the point about cache invalidation belonging in `packfile.c` while noting the layering violation of the unchecked downcast to the files backend remains until another ODB backend materializes.

### Submodule merge reads wrong repository
Guillaume CHAUVEL reported that merging a superproject with submodules in Git v2.56.0-rc2 sometimes reads from submodule A a commit existing only in submodule B, failing with "repository corrupt"; his reproducer triggered it in all 50 Ubuntu and 43 of 50 Alpine runs. Patrick Steinhardt endorsed passing the repository explicitly into `can_all_from_reach()` to fix the superproject/submodule commit-graph merge failure.

### Early option scanning API
Christian Couder posted v2 adding an `early_scan_options()` sub-API to parse-options that reuses the command's `struct option` array via a `PARSE_OPT_EARLY` flag, converting `git fast-import`'s hand-rolled scan so `--allow-unsafe-features` after `--depth 5` is no longer silently ignored. Junio C Hamano agreed that converting `bisect_start()` to parse-options would make `--` detection come for free.

### pack-objects abort in partial clones
L Naudé reported that `git pack-objects --exclude-promisor-objects-best-effort` aborts with a BUG in `should_include_obj` during fetch in a `tree:0` partial clone, traced to `is_not_in_promisor_pack_obj()` called for a missing tree.

### pull segfault on unparseable merge heads
Jiri Kuncar posted v2 of a fix for `git pull` segfaulting when `lookup_commit_reference()` returns NULL for a truncated loose object behind HEAD or a merge head, adding NULL guards that treat the failure as "cannot fast-forward" and fall through to the normal merge path.

## In brief

- **diff --no-index -R fix**: Junio C Hamano queued Haokai Ding's fix for `diff --no-index -R` misreporting file/directory conflicts.
- **refs copy/rename**: Maciej Ciemborowicz posted v2 routing ref copy and rename through the transaction API; Junio praised the commit message and asked about avoiding hook-skipping flags.
- **shallow-clone advice**: Harald Nordgren posted v2 of the shallow-history advice hint; Junio retracted his earlier request to advise during `get_revision()` traversal, and later rejected a `+:` refspec in favor of a `remote.<name>.refmap` config.
- **reflog expiry periods**: Pushkar Singh posted v3 fixing swapped default reflog expiry periods (90 days reachable, 30 unreachable); Junio judged the tests much better organized and answered "Probably" to fast-tracking for 2.56.
- **--fixup variations**: Erik Cervin-Edin posted v3 allowing `-m`/`-F`/`-c`/`-C` for all `--fixup` variations.
- **dir.c nested repo**: René Scharfe posted v2 guarding the nested-repo `dir_match()` call with `pathspec &&` to fix a NULL pathspec crash.
- **ls-files untracked cache**: Tamir Duberstein posted v2 letting `ls-files` reuse the untracked cache, dropping a wildcard benchmark to 33.3 ms warm.
- **packfile flush**: Justin Tobler posted v2 fixing the packfile flush during ODB transaction commit; Patrick Steinhardt approved.
- **create_repository**: Patrick Steinhardt posted a 7-patch series making the repository parameter a pure out-parameter.
- **Meson improvements**: Patrick Steinhardt posted a 7-patch series speeding clean builds from ~6.8s to ~5.0s and fixing GitLab MSVC hangs.
- **git-history during rebase**: Nikita Bobko reported `git history` mid-rebase updates the branch being rebased, breaking `git rebase --continue`.
- **GIT_WORK_TREE in hooks**: Phil Sainty reported the post-checkout hook sees `GIT_DIR` but `GIT_WORK_TREE` is exported as `.` rather than the worktree path.
- **parse_revision_opt()**: Jeff King posted two fixes so unknown options are reported by name instead of `(null)`.
- **git var extension**: Andrew Pleeter posted v9 adding `-z` output, multiple variables, and broken-out idents.
- **credential/libsecret**: Daniel Martí posted v2 loading secrets explicitly via `secret_item_load_secret_sync()`, producing useful errors instead of assertion failures.
- **format-patch range-diff-notes**: Kristoffer Haugsbakk resent v2 adding `--range-diff-notes=<ref>`; Junio noted a redundant test.
- **tempfile registration**: Royce Remer registered temporary pack files with the tempfile subsystem so killed `git gc` runs leave no orphans.
- **git replay signing**: Patrick Monette added `-S`/`--gpg-sign` to `git replay`.
- **fetch.followRemoteHEAD**: Colin Hinton posted v3 deferring validation to `do_fetch()`.
- **git-contacts stdin**: Brigham Campbell posted v4 reading a patch from stdin via `-`.
- **Documentation updates**: jyotish kumar updated `git name-rev` docs from "SHA-1" to "object ID"; Matthias Goergens clarified `git remote set-head` is local-only; Kristoffer Haugsbakk proposed fixing `trailer.<key-alias>.cmd` examples; Tamir Duberstein posted v5 clarifying `--follow`'s single-file limitation.
- **l10n**: Jiang Xin submitted a pull request adding Afrikaans and Brazilian Portuguese and updating eight languages for Git 2.56.0, and said he will rebase l10n commits to prevent reverted commits from reappearing.
- **.mailmap**: Kristoffer Haugsbakk mapped his Fastmail and Gmail addresses to `code@khaugsbakk.name`.

## Looking ahead

The stash `--index` series will need rework after Junio C Hamano ejected it from `seen` over the unrestored verbosity bug in patch 5/5. The `http.sslVerifyStatus` topic remains reverted from `next` pending an improved test prerequisite that detects OCSP-capable backends. K Jayatheerth's v7 `git repo info` series and Thomas Bachem's sequencer auto-maintenance series, which Phillip Wood called ready for `next`, will see further review. The Git v3.0 timeline proposal will likely generate discussion about which breaking changes to enable.