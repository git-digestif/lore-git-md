## Long threads

### fetch: limited fetching for new remotes in shallow clones

*Long thread: 78 replies so far from 6 participants over 57 days.*

Harald Nordgren's series addresses shallow clones fetching every branch of a newly added remote by separating which refs to fetch from where they are stored. The v13 iteration introduced `remote.<name>.fetchRefmap` and `--limited-fetch`, but Phillip Wood and Junio C Hamano pressed for dropping the refmap-only patch in favor of combining `.fetchRefmap` with `.fetch`, and questioned the `@{push}` handling. By v14 Nordgren squashed the `@{push}` fix into the refmap patch and shrank the series to five patches, with Wood reiterating that command-line refspecs with explicit destinations should not also update refmap destinations.

### repo: path keys for `git repo info`

*Long thread: 63 replies so far from 6 participants over 123 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info`, evolving through nine versions after sustained review. Early debate centered on key naming: Phillip Wood argued against `path.hooks` and `path.cdup` as encouraging outdated interfaces, while Junio C Hamano noted Git avoids dashes in configuration-style names. The v9 series renamed keys to `path.workingtree.*`, `path.superprojectworkingtree.*`, and `path.prefix`, dropped the contested keys, and refactored `get_superproject_working_tree()` onto a `struct repository *`. Phillip Wood reviewed the final range-diff and said he would be happy for it to merge as-is, with test consolidation deferred to a follow-up.

### branch: `--delete-integrated` for squash-merged branches

*Long thread: 57 replies so far from 7 participants over 47 days.*

Harald Nordgren's series adds `git branch --delete-integrated <pattern>` to delete branches whose changes landed upstream via squash or rebase merges. Jasper Mayone's review of v6 flagged that `repo_read_index()` ran per changed file and proposed reading the index once before the walk, which v7 adopted alongside a test ensuring branches whose changes did not land are kept. The series names the commit where changes landed and uses merge attributes from the worktree's index rather than the upstream commit.

### doc: detached HEAD guide rewrite

*Long thread: 45 replies so far from 5 participants over 39 days.*

Julia Evans is rewriting the detached HEAD explanation as a standalone `gitdetached-head` guide wired into `advice.c`. The thread grappled with audience: Junio C Hamano argued the guide must serve both users who detached accidentally and advanced users making experimental commits, while Evans initially framed it for confused users. By v6 they converged on wording that "HEAD can store either" a branch or commit ID, with Hamano approving a sentence about making experimental commits without being on a branch, noting the material can serve both the general manual and a survival guide.

### credential-store: lockTimeout config option

*Long thread: 42 replies so far from 5 participants over 75 days.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. Junio C Hamano questioned the unit-less name, but Ajayi defended it by citing other millisecond-based lock timeouts (`rerere.lockTimeout`, `reftable.lockTimeout`, `core.filesRefLockTimeout`) that omit the unit, contrasting with `credential-cache --timeout` which uses seconds. She plans to resend with this rationale in the commit message.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread: 38 replies so far from 8 participants over 24 days.*

Jasper Mayone's v5 series makes `git worktree remove` refuse worktrees mid-operation without `--force`, documents `%(worktreepath)`, names the rebasing worktree in that specifier, adds `%(currentworktree)`, and stops `git branch` marking such branches with `+`. Phillip Wood proposed a hash map for `branch_checked_out_worktree()` to avoid iterating all worktrees, which v5 adopted. Junio C Hamano praised the central `check_no_operation_in_progress()` design as extensible but expressed reservations about patch 5, suggesting the series might stop at 3/5 if 4/5 exists only to support it.

### Community voting process

*Long thread: 31 replies so far from 14 participants over 35 days.*

Following Contributor's Summit notes, Stéphane Glondu proposed Belenios as the voting platform, noting it supports Condorcet voting and that an instance already runs at vote.debian.net for Debian Salsa account holders. The thread continues earlier debate over voter eligibility and platform selection for project decisions.

### t: add `test-tool genbytes` to avoid slow `tr(1)` on macOS

*Long thread: 30 replies so far from 5 participants over 5 days.*

Kristofer Karlsson's series adds `test-tool genbytes <char> [<count>]` to replace slow `genzeros | tr` pipelines in tests like t4205, which burn over 95 CPU-seconds on macOS. After v3 adopted a static buffer and `git_parse_signed()`, Junio C Hamano marked the topic ready for `next`; v4 rebased to catch a new callsite in t1600-index.sh.

### root refs vs pseudorefs documentation

*Long thread: 30 replies so far from 4 participants over 22 days.*

Andy Koppe's five-patch series stops calling root refs pseudorefs across the glossary, `git-ls-remote(1)`, `gitdatamodel(7)`, and tests. His v4 dropped pseudoref/root-ref contrasts and reworded the `rev-list --merge` error message; Junio C Hamano wrote that the series "looked all sensible".

### builtin/rerere: convert to use subcommands

*Long thread: 29 replies so far from 6 participants over 19 days.*

Patrick Steinhardt's 12-patch series converts `git rerere`'s custom parsing to `OPT_SUBCOMMAND()`, making forget, clear, status, remaining, diff, and gc proper subcommands. Junio C Hamano noted a possible UI regression where options before the subcommand are no longer accepted, requiring `git rerere gc --skip-locked` instead of `git rerere --skip-locked gc`.

### `git diff` EXAMPLES section rewrite

*Long thread: 23 replies so far from 6 participants over 8 days.*

Julia Evans retracted an accidental `--cached` to `--staged` synopsis swap from her v2 patch to keep scope small. Kaartic Sivaraam explained how to fetch Junio C Hamano's queued SQUASH??? commits from his tree.

### Fix segfaults on settings given without a value

*Long thread: 22 replies so far from 5 participants over 6 days.*

Thomas Bachem's 15-patch series fixes segfaults when settings like `rebase.missingCommitsCheck` or `remote.*.tagOpt` are given without a value. After v3 made `tagOpt` and `color.blame.repeatedLines` warn-and-ignore rather than error, Phillip Wood called the range-diff ready for `next` and Junio C Hamano marked the topic ready.

## The period in brief

The week of 2026/11/09 through 2026/11/15 saw heavy traffic across seven active days, with several long-running series reaching decisive review rounds. Readers should not miss the Git 2.56.1 security release announcement, the resolution of the `log --follow` performance regression via an opt-in flag, and the v9 `git repo info` path-keys series which Phillip Wood approved for merging.

## Key developments

### Git 2.56.1 security release

Junio C Hamano announced coordinated security releases Git 2.56.1 and maintenance tracks down to maint-2.43 addressing six CVEs. Todd Zullinger noted the cited GitLab work item links appear inaccessible to normal users and added Patrick Steinhardt to help open them up.

### log: --follow rename following regression

Justin Tobler benchmarked `git log --stat --follow` on linux.git, finding the v5 speedup still 15 seconds slower than before the regression, risking server-side timeouts. Patrick Steinhardt and Junio C Hamano agreed an opt-in flag was needed; Miklos Vajna posted a patch making per-parent rename diffing opt-in via `--follow=merges` and `log.follow=merges`, and Hamano ejected the speedup topic in favor of this approach.

### promisor-remote reply missing from fetch protocol v2

Christian Couder posted a three-patch series fixing a bug where the client omits the `promisor-remote=...` reply from `fetch`, failing with stateless servers like Gerrit and JGit. Hamano objected to a dedicated parameter in `write_command_and_capabilities()`, suggesting the existing `server_options` list be generalized instead.

### setup: hash algorithm detection on re-init

brian m. carlsson's v2 reworks hash selection so command-line choices take precedence and reinitializing an existing repository with a different hash algorithm is refused, fixing a latent bug where `init.defaultObjectFormat` could corrupt an existing repository. Hamano called the restructured version simpler and said he would replace the patch.

### repack: --filter-to-loose for filtered-out objects

Adithya Chakilam's series adds `--filter-to-loose=<dir>` to `git repack`, streaming filtered packs into `unpack-objects` to write excluded objects directly as loose objects. Hamano asked for tests covering non-existent destinations and potential double-close of descriptors.

### apply: reject malformed diff headers

Tanga Joris posted v2 of an Outreachy fix for a crash in `git apply --check` on patches with corrupt `diff --git` headers, rejecting new files with old names and deleted files with new names in `parse_git_diff_header()`.

### for-each-ref: fix segfault on --no-format

Abhishek Agarwal's v2 fixes a segfault in `git for-each-ref --no-format` by assigning the default format after `parse_options()` when none remains, so `--no-format` cancels an earlier `--format`.

### Preserving change-id header through rebase

Matt Stark's patch modifies `sequencer.c` to preserve `change-id` headers through rebase and cherry-pick. brian m. carlsson objected that change-ids cannot be trusted as unmodified data since users can alter them, warning of potential abuse.

### Fix colors at line breaks in pretty.c

Andy Koppe submitted a patch to `pretty.c` fixing lost and leaked colors at line breaks in custom log formats by scanning for active color escapes and making each line self-contained.

## In brief

**reftable HEAD reflog** -- 井上司 corrected his report, noting the files backend behaves the same, and proposed a `refs.c` helper returning the new oid of a ref updated in the same transaction.

**log --follow v5** -- Miklos Vajna posted v5 speeding up `log --follow` on non-linear history, now following copies and renames across parents with no merge base.

**unpack-trees the_repository removal** -- Junio C Hamano posted v2 and Derrick Stolee gave an LGTM to the series removing `the_repository` from `unpack-trees.c`.

**ODB transaction temp directory** -- Justin Tobler posted a four-patch series creating the ODB transaction temporary directory eagerly, showing about 7% overhead on `git add` of 10000 files.

**Configurable orphaned commit checks** -- Wyatt Holden's three-patch series adds `checkout.orphanedCommitWarning` to skip the reachability check on checkout, which takes 193 seconds in an internal mono-repo.

**fsmonitor untracked cache** -- Ilia K's v2 invalidates the whole cache when the worktree cannot be determined; Hamano reverted the `next` merge and queued v2 on the original base.

**cat-file out-of-range reflog** -- Caspar Duregger's four-patch series makes `git cat-file --batch-check` report out-of-range reflog selectors as missing instead of dying.

**struct commit padding** -- Kristofer Karlsson proposed moving `index` into the padding hole in `struct commit`, shrinking it from 80 to 72 bytes on 64-bit platforms.

**git repo structure speedup** -- Patrick Steinhardt sped up `git repo structure` in huge repositories by only parsing objects as needed.

**credential OPT_SUBCOMMAND** -- Wasswa Edson posted v2 converting `git credential` dispatch to `OPT_SUBCOMMAND()`.

**Test modernization** -- Hardik Kumar replaced `!` with `test_must_fail` in several test scripts so signal-killed commands fail properly.

**Translation and docs** -- Jean-Noël Avila's synopsis-style conversion and various man page fixes continued, alongside Dirk Gouders' `doc/gitcli` fixes and Yikun Wei's retracted `fmt-merge-msg` doc patch.

## Looking ahead

The v9 `git repo info` path-keys series appears ready to merge after Phillip Wood's approval, with test consolidation planned for a follow-up. Harald Nordgren's v14 shallow-fetch series and v7 `--delete-integrated` series await final review, as does Julia Evans's v6 detached HEAD guide. The `change-id` preservation patch and Adithya Chakilam's `--filter-to-loose` series will likely see further discussion next week.