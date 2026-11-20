## Long threads

### fetch: limited fetching for new remotes in shallow clones
*Long thread: 78 replies so far from 6 participants over 57 days.*
Harald Nordgren's series separates which refs a remote fetches from where they are stored via `remote.<name>.fetchRefmap` and `--limited-fetch`, fixing shallow clones that otherwise download every branch of a new remote. The series reached v14 this week, shrinking from six patches to five by folding the `@{push}` fix into the refmap-only patch. Phillip Wood urged dropping the refmap-only patch in favor of combining `.fetchRefmap` with `.fetch`, while Junio C Hamano agreed the `@{push}` failure looked like a bug the patch introduced. The series awaits final review.

### repo: path keys for `git repo info`
*Long thread: 63 replies so far from 6 participants over 123 days.*
K Jayatheerth's GSoC series adds path keys to `git repo info`, reaching v9 with renames to `path.workingtree.*`, `path.superprojectworkingtree.*`, and `path.prefix`, and dropping the questioned `path.hooks` and `path.cdup` keys. Phillip Wood reviewed the v9 range-diff and said he would be happy for the series to be merged as-is, with test consolidation deferred to a planned follow-up series. The series also refactors `get_superproject_working_tree()` onto a `struct repository *`.

### branch: `--delete-integrated` for squash-merged branches
*Long thread: 57 replies so far from 7 participants over 47 days.*
Harald Nordgren's series adds `git branch --delete-integrated <pattern>`, deleting branches whose changes landed upstream via squash or rebase merges. v7 arrived this week with the index read once before the walk instead of per file, a test that unlanded branches are kept, and a commit message clarifying that merge attributes come from the working tree. Jasper Mayone reviewed v6 positively, and Tuomas Ahola noted documentation wording nits in v7.

### doc: detached HEAD guide rewrite
*Long thread: 45 replies so far from 5 participants over 39 days.*
Julia Evans is rewriting the detached HEAD explanation as a standalone `git help detached-head` guide. The week saw debate over serving both accidentally-detached and advanced users; Junio C Hamano approved Evans's sentence about making experimental commits without being on a branch, saying the material can serve both the general manual and a survival guide. Evans posted v6 with agreed wording and updated test expectations.

### credential-store: lockTimeout config option
*Long thread: 42 replies so far from 5 participants over 75 days.*
Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. She defended the unit-less name by pointing to other millisecond-based lock timeouts like `rerere.lockTimeout` and `core.filesRefLockTimeout`, and plans to resend with that rationale in the commit message after Junio C Hamano questioned the naming.

### worktree: treat a branch being rebased or bisected as checked out
*Long thread: 38 replies so far from 8 participants over 24 days.*
Jasper Mayone's v5 series makes `git worktree remove` refuse worktrees mid-operation without `--force`, adds `%(currentworktree)`, and stops `git branch` marking rebased branches with `+`. Junio C Hamano praised the central `check_no_operation_in_progress()` design as extensible and called patch 3 a nice loss of duplicated logic, though he was less positive on patch 5, suggesting the series might stop at 3/5.

### Community voting process
*Long thread: 31 replies so far from 14 participants over 35 days.*
Following Contributor's Summit notes, Stéphane Glondu proposed Belenios as the voting platform, noting it supports Condorcet voting and that an instance already runs at vote.debian.net for Debian Salsa account holders. The thread continues debate over voter eligibility and platforms.

### t: add `test-tool genbytes` to avoid slow `tr(1)` on macOS
*Long thread: 30 replies so far from 5 participants over 5 days.*
Kristofer Karlsson's series adds `test-tool genbytes <char> [<count>]` to replace slow `genzeros | tr` pipelines in tests like t4205 on macOS. v4 arrived this week catching a new callsite in t1600-index.sh; Junio C Hamano called the conversions trivially correct and marked the topic ready for `next`.

### root refs vs pseudorefs documentation
*Long thread: 30 replies so far from 4 participants over 22 days.*
Andy Koppe's five-patch series stops calling root refs pseudorefs across the glossary, `git-ls-remote(1)`, `gitdatamodel(7)`, and tests. v4 dropped the pseudoref/root-ref contrasts and reworded the `rev-list --merge` error message; Junio C Hamano wrote that the series "looked all sensible".

### `git diff` EXAMPLES section rewrite
*Long thread: 23 replies so far from 6 participants over 8 days.*
Julia Evans retracted an accidental `--cached`->`--staged` synopsis swap from the v2 patch to keep scope small. Kaartic Sivaraam explained how to fetch Junio C Hamano's queued SQUASH??? commits from his tree, and Junio suggested finding them via `git log seen`.

### Fix segfaults on settings given without a value
*Long thread: 22 replies so far from 5 participants over 6 days.*
Thomas Bachem's 15-patch series fixes segfaults when settings like `rebase.missingCommitsCheck` or `remote.*.tagOpt` are given without a value. v3 makes `tagOpt` and `color.blame.repeatedLines` warn and ignore missing values while keeping errors elsewhere. Phillip Wood called the v3 range-diff ready for `next`, and Junio C Hamano marked the topic ready.

## The period in brief
The week of 2026/11/09 through 2026/11/15 saw steady traffic across seven active days, highlighted by the coordinated Git 2.56.1 security release addressing six CVEs. Several long-running series reached decisive review rounds, including the shallow-fetch refmap work and the `git repo info` path keys, while a performance regression in `git log --follow` was addressed with an opt-in flag.

## Key developments

### Git 2.56.1 security release
Junio C Hamano announced coordinated security releases Git 2.56.1 and maintenance tracks down to maint-2.43 addressing six CVEs. Todd Zullinger noted the cited GitLab work item links appear inaccessible to normal users and added Patrick Steinhardt to help open them up.

### `git log --follow` performance regression addressed
Justin Tobler's benchmarks showed the `--follow` rename-following change for non-linear history regressed from 23.5s to 335.9s on linux.git. Junio C Hamano ejected the speedup topic and queued Miklos Vajna's opt-in patch making `--follow=merges` optional, restoring prior behavior for plain `--follow` while allowing the new per-parent following behind a flag.

### promisor-remote reply missing from fetch protocol v2
Christian Couder posted a three-patch series fixing a bug where the client omits the `promisor-remote=...` reply from `fetch`, failing with stateless servers like Gerrit and JGit. Junio C Hamano objected to a dedicated parameter in patch 1/3, suggesting the existing `server_options` list be generalized instead.

### Hash algorithm detection on re-init
brian m. carlsson's v2 reworks hash selection in `setup.c` so a command-line choice takes precedence and reinitializing an existing repository with a different hash algorithm is refused. Junio C Hamano called the restructured detection simpler and said he will replace the patch.

### for-each-ref `--no-format` segfault fix
Abhishek Agarwal posted v2 fixing a segfault in `git for-each-ref --no-format` where `OPT_STRING()` sets the format to NULL, which `verify_ref_format()` dereferences. The fix assigns the default format after `parse_options()` when none remains.

### Configurable orphaned commit checks on checkout
Wyatt Holden's three-patch series adds `checkout.orphanedCommitWarning` (default true) so checkout and switch can skip the orphaned-commit reachability check, which took about 193 seconds in an internal mono-repo. Derrick Stolee supported the series, and Junio C Hamano called it coherent despite questioning some design choices.

### apply: reject malformed diff headers
Tanga Joris posted v2 of an Outreachy fix for a crash in `git apply --check`, rejecting in `parse_git_diff_header()` a new file with an old name and a deleted file with a new name, and reporting an empty "diff --git" header as a corrupt-patch error.

## In brief
- **fsmonitor untracked cache**: Ilia K posted v2 invalidating the whole cache when the worktree cannot be determined; Junio C Hamano reverted the `next` merge and queued v2 on the original base.
- **reflog symref fixes**: 井上司 and tsukasa inoue posted patches fixing reflog entries for symrefs updated in the same transaction, proposing helpers like `ref_transaction_resolve_new_oid()`.
- **unpack-trees the_repository removal**: Junio C Hamano posted v2 and Derrick Stolee gave an LGTM to the series removing `the_repository` from `unpack-trees.c`.
- **ODB transaction temp directory**: Justin Tobler posted a four-patch series creating the ODB transaction temporary directory eagerly, showing about 7% overhead on `git add` of 10000 files; Patrick Steinhardt and Karthik Nayak reviewed.
- **early_scan_options RFC**: Azuama Odinaka Maximus posted an RFC adding a generic `early_scan_options()` API to parse-options.
- **CI speedups**: Patrick Steinhardt posted v3 of CI speedups, including reftables for many-refs tests cutting macOS t5551 from 20 minutes to 2.
- **repack --drop-filtered**: kongfankaikai reported `git repack --drop-filtered` aborts fatally when a droppable blob is index-referenced.
- **struct commit padding**: Kristofer Karlsson proposed moving `index` into the padding hole in `struct commit`, shrinking it from 80 to 72 bytes on 64-bit platforms.
- **ls-remote --show-object-format**: Adrian Ratiu posted v2 printing the remote's object format; Junio C Hamano said "Looking good".
- **fsck object count**: Aniket Gupta's patch reports the real object count in the "Checking connectivity" message.
- **Solaris uname fix**: Jakub Kulik reported Solaris `uname()` returns non-negative on success, proposing `< 0` for the check in `version.c`.
- **cat-file out-of-range reflog**: Caspar Duregger's four-patch series makes `git cat-file --batch-check` report out-of-range reflog selectors as missing instead of dying.
- **repack --filter-to-loose**: Adithya Chakilam's two-patch series adds `--filter-to-loose=dir` to write filtered-out objects directly as loose objects; Junio C Hamano asked for tests on non-existent destinations.
- **ref-filter double free**: Mustafa Bayar fixed a double free in `git for-each-ref` when a ref points to a missing object; Junio C Hamano will queue it.
- **ls-remote crash fix**: Christian Couder fixed a crash when `git ls-remote` runs outside a repository with promisor config.
- **worktree submodule fix**: Yoichi NAKAYAMA corrected his fix for `submodule.<name>.gitdir` to use `relative_path()`'s return value directly.
- **credential OPT_SUBCOMMAND**: Wasswa Edson posted v2 converting `git credential` dispatch to `OPT_SUBCOMMAND()`.
- **Coverity overflow series**: Johannes Schindelin updated his series with `write_in_full()` returning 0 on success and manual overflow checks in `date.c`.
- **test modernization**: Hardik Kumar replaced `!` with `test_must_fail` in several test scripts.
- **http-backend**: Yusuke Hayashi posted v2 dropping strbuf pre-sizing in `get_info_packs()`.
- **doc/gitcli fixes**: Dirk Gouders resent v3 of minor fixes with proper Cc.
- **pretty.c color fix**: Andy Koppe submitted a patch fixing lost and leaked colors at line breaks in custom log formats.
- **change-id preservation**: Matt Stark proposed preserving `change-id` headers through rebase; brian m. carlsson objected that they cannot be trusted as unmodified data.
- **fmt-merge-msg doc**: Yikun Wei retracted a patch that would have duplicated config entries already listed in `git-config(1)`.

## Looking ahead
The v14 shallow-fetch series and v9 `git repo info` path-keys series both reached states where reviewers expressed willingness to merge, so they may land in `next` soon. The `git rerere` subcommand conversion faces a possible UI regression where options before the subcommand are no longer accepted, which may need addressing before the series graduates.