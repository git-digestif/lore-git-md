## Long threads

### `git repo info` gains more `path.*` keys
*Long thread: 37 replies so far from 5 participants over 74 days.*
K Jayatheerth posted v7 of his GSoC series on the 27th, now eight patches adding `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup` to `git repo info`, each with absolute and relative suffixes where applicable. The superproject working-tree bug fix is now its own commit, dead `if (!index_file)` checks are gone, and `path.cdup` works even outside the work tree, fixing the discrepancy with `git rev-parse --show-cdup` that Junio C Hamano had flagged.

### Sequencer auto-maintenance timing
*Long thread: 32 replies so far from 5 participants over 20 days.*
Thomas Bachem's series moves `git maintenance --auto` to the end of rebase, cherry-pick, and revert sequences rather than running it inside spawned commands. On the 22nd he explained why the call sits in `builtin/rebase.c` and `builtin/revert.c` rather than inside the sequencer, and on the 23rd Phillip Wood reviewed the v5 tests and said the series "looks ready for next to me".

### http.sslVerifyStatus OCSP stapling in the HTTP transport
*Long thread: 31 replies so far from 5 participants over 46 days.*
Grayson Gordon's series adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`, defaulting to false. The topic was reverted out of `next` on the 23rd after SZEDER Gábor reported a t5585 failure; on the 25th he confirmed his GnuTLS curl backend lacks OCSP support while the OpenSSL backend passes, and Junio C Hamano noted the test prerequisite still needs improvement to check for the specific error message rather than just the config.

### stash `--index` in-core merge cleanup
*Long thread: 28 replies so far from 3 participants over 9 days.*
D. Ben Knoble's v3 series, posted on the 26th, replaces the shell-out `diff-tree`/`apply --cached`/`reset` sequence in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing the autostash bug with `stash.index=true` and correcting the transposed tree arguments. On the 27th Junio C Hamano identified that patch 5/5 unconditionally sets `o.verbosity = 0` on a reused `merge_options` struct, breaking a later merge, and said he would eject the topic from `seen`.

### reference-transaction hooks reporting old values
*Long thread: 25 replies so far from 5 participants over 6 days.*
Maciej Ciemborowicz posted [PATCH v6] on the 24th as a single transaction-layer change: the common hook layer resolves the old value before the "preparing" hook and refreshes it after the backend locks references, only when a hook exists, keeping the hook value separate from caller-supplied `old_oid`. Patrick Steinhardt had argued for exactly this approach, and the earlier batch-deletion API changes to `refs_delete_refs()` were abandoned.

### --force-if-includes with a reflog-less remote-tracking ref
*Long thread: 20 replies so far from 6 participants over 23 days.*
Aleksei Sviridkin's patch initializes the cutoff timestamp to 0 in `is_reachable_in_reflog()` so `push --force-if-includes` does not reject valid pushes when the remote-tracking ref has no reflog. On the 25th Tyler Cipriani argued `date=0` is always correct, and Junio C Hamano agreed that with a never-expired reflog using all available information works better than any cutoff.

### AsciiDoc cross-references and the tutorial rewrite in Documentation/
*Long thread: 17 replies so far from 4 participants over 4 days.*
Julia Evans reposted v2 of her patch replacing "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references on the 25th; Jeff King said it looks good. She also announced a planned rewrite of `gittutorial.adoc` covering only basic commands, and Junio C Hamano stressed that new guides must be sufficient to replace the stale document, with removal as the primary goal.

### GitLab CI Windows cargo invocation fix
*Long thread: 12 replies so far from 3 participants over 6 days.*
Johannes Schindelin posted [PATCH v2] on the 24th rewording commit messages in the four-patch series fixing the Windows cargo invocation broken since Rust was enabled in GitLab CI, clarifying that "GNU Rust" means the GCC-targeting toolchain and that patch 4 installs no second GCC.

## The period in brief

The week of 2026/09/21 -- 2026/09/27 saw heavy traffic across all seven days, with long-running series reaching key milestones and several new regressions surfacing. Readers should not miss Junio C Hamano's proposed Git v3.0 timeline, Maciej Ciemborowicz's v6 transaction-layer fix for reference-transaction hooks, and the tracing of the `http.sslVerifyStatus` test failure to a GnuTLS curl backend without OCSP support.

## Key developments

### Git v3.0 timeline proposal
On the 24th Junio C Hamano laid out a proposed timeline: Git 2.98 at year's end, 2.99 in March 2027 as a signal that something major is coming, 2.99.1 in April 2027 carrying only bugfixes, and 3.0 identical to 2.99.1 except with breaking changes enabled via the `WITH_BREAKING_CHANGES` Makefile knob. He cautioned that anything beyond this year is "play it by ear" territory. Johannes Schindelin thanked him for the roadmap clarity, and Ramsay Jones reported a successful Cygwin build of master with the experimental Rust package.

### git-p4 shell metacharacter injection in applyCommit
Anupam Mediratta posted v2 on the 24th of his fix replacing the shell-based `git diff-tree | git apply` pipeline in `P4Submit.applyCommit()` with two argument-vector subprocess calls connected by a pipe, so a `--commit` value like `$(some-command)` is no longer executed. The v2 adds an `ignore_error` parameter to the new `diffTreeApply()` helper, preserving the old behavior where a failed apply aborts the submit by default while the `tryPatch` callers pass `ignore_error=True`.

### Repack kept-pack snapshot
Qin ShiCheng's series fixes a race where a concurrent push installs a `.keep` next to a pack that `git repack -d` has already decided to delete, causing pack-objects to omit the pack's objects while repack deletes the pack anyway. The fix hands pack-objects the snapshot repack took at startup. Junio C Hamano reviewed v2 and questioned the unchecked downcast to the files backend, noting that until another ODB backend materializes the change might be sufficient but relying on the downcast feels like "laying mines for our future selves."

### Submodule merge reads wrong repository
Guillaume CHAUVEL reported that merging a superproject with submodules in Git v2.56.0-rc2 sometimes reads from submodule A a commit existing only in submodule B, failing with "repository corrupt" instead of a submodule conflict; his reproducer triggered it in all 50 Ubuntu and 43 of 50 Alpine runs. Patrick Steinhardt endorsed passing the repository explicitly into `can_all_from_reach()` to fix the superproject/submodule commit-graph merge failure.

### Early option scanning API
Christian Couder posted v2 on the 23rd adding an `early_scan_options()` sub-API to parse-options that reuses the command's `struct option` array via a `PARSE_OPT_EARLY` flag, converting `git fast-import`'s hand-rolled scan so `--allow-unsafe-features` after `--depth 5` is no longer silently ignored. Junio C Hamano agreed that converting `bisect_start()` to parse-options would make `--` detection come for free.

### pack-objects aborts on missing promised tree during fetch
L Naudé reported on the 26th that `git pack-objects --exclude-promisor-objects-best-effort` aborts with a BUG in `should_include_obj` during `git fetch origin main` in a `tree:0` partial clone of `NousResearch/hermes-agent` on Git 2.55.0, traced to `is_not_in_promisor_pack_obj()` called from `process_tree()` for a missing tree.

### Changing default config values
In the thread on what comes after Git 2.56, Phillip Wood proposed on the 27th a consequentialist approach to changing defaults for cosmetic UI settings plus a `core.defaults` setting offering evolving "modern defaults" enabled by default with a single opt-out. Junio C Hamano responded that Git already takes such an approach with repeated default merge strategy changes, but argued net benefit remains speculative until a change has been released for months.

### credential/libsecret loads secrets explicitly
Daniel Martí posted v2 on the 27th rewording the commit message to explain that `SECRET_SEARCH_LOAD_SECRETS` is implemented as a separate `GetSecrets` D-Bus call after `SearchItems`, so the patch drops the flag and always loads the one secret via `secret_item_load_secret_sync()`, producing a useful error message for inaccessible items instead of assertion failures, with git still falling back to prompting.

## In brief

- **pull segfault fix**: Jiri Kuncar posted v2 adding NULL guards in `get_can_ff()` and `already_up_to_date()` so a failed `lookup_commit_reference()` falls through to the normal merge path instead of segfaulting.
- **"safe" strbuf API**: Junio C Hamano reviewed the RFC for a `strbuf-safe` API aimed at trace2 `die()` recursion, identified a memory leak in `srealloc()`, and suggested renaming functions that merely use a strbuf.
- **Rebase `--amend` regression**: Patrick Steinhardt posted a fix for the regression where `git commit --amend` refusal persisted after resolving a conflict; the discussion converged on reverting commit 6257588252 for Git 2.56 at -rc2.
- **Reflog expiry periods**: Pushkar Singh posted v3 fixing the swapped default reflog expiry periods in `REFLOG_EXPIRE_OPTIONS_INIT()` (90 days reachable, 30 unreachable), with Junio C Hamano answering "Probably" to fast-tracking for 2.56.
- **git-history during rebase**: Nikita Bobko reported that running `git history` mid-rebase updates the branch being rebased, causing `git rebase --continue` to fail with a ref lock error.
- **GIT_WORK_TREE for post-checkout hook**: Phil Sainty found that in a worktree the post-checkout hook sees `GIT_DIR` but `GIT_WORK_TREE` is exported as `.` rather than the actual path, so `git rev-parse --show-toplevel` returns the CWD.
- **parse_revision_opt() bugfixes**: Jeff King posted two fixes so unknown options are reported by name instead of `(null)` and negative returns from `handle_revision_opt()` are treated as already-reported errors.
- **dir.c nested repo**: René Scharfe posted v2 guarding the nested-repo `dir_match()` call with `pathspec &&` to fix the NULL pathspec crash Junio C Hamano had flagged.
- **git var extension**: Andrew Pleeter posted v9 adding `-z` output, multiple variables per invocation, and broken-out `GIT_AUTHOR_*`/`GIT_COMMITTER_*` idents.
- **Early option scanning**: Christian Couder's v2 converts `git fast-import` to the new `early_scan_options()` API.
- **format-patch range-diff-notes**: Kristoffer Haugsbakk's series adds `--range-diff-notes=<ref>` and `--no-range-diff-notes`; Junio noted a redundant `test_when_finished` in the test.
- **Documentation updates**: Julia Evans posted a seven-patch `gitmergeconflicts(7)` guide; Tamir Duberstein posted v5 clarifying `--follow` in `git-log.adoc`; jyotish kumar updated `git name-rev` docs from "SHA-1" to "object ID"; Matthias Goergens clarified `git remote set-head` in `git-remote.adoc`.
- **Test and CI fixes**: Royce Remer registers temporary pack files with the tempfile subsystem; Jon Simons fixed repeated `p5551` runs with `--no-deref`; Harald Nordgren's CI leak-sanitizer patch stops at the first failure; Jiang Xin submitted l10n updates for Git 2.56.0 adding Afrikaans and Brazilian Portuguese.
- **Miscellaneous**: Erik Cervin-Edin posted v3 allowing `-m`/`-F`/`-c`/`-C` for all `--fixup` variations; Patrick Monette added `-S`/`--gpg-sign` to `git replay`; Colin Hinton posted v3 deferring `fetch.followRemoteHEAD` validation; Brigham Campbell posted v4 of `git-contacts` reading from stdin; Kristoffer Haugsbakk mapped his addresses in `.mailmap` to `code@khaugsbakk.name`.