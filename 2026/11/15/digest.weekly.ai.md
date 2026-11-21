## Long threads

### fetch: limited fetching for new remotes in shallow clones
*Long thread: 78 replies so far from 6 participants over 57 days.*

Harald Nordgren's series separates which refs a remote fetches from where they are stored, adding `remote.<name>.fetchRefmap` and `--limited-fetch` so shallow clones of a new remote skip downloading every branch. The series reached v14 on November 14, squashing the `@{push}` fix into the refmap-only patch and shrinking the series from six patches to five. Phillip Wood urged dropping the refmap-only patch in favor of combining `.fetchRefmap` with `.fetch`, and Junio C Hamano agreed the `@{push}` failure looked like a bug the patch introduced. Matt Hunter identified `ch/remote-defer-follow-remote-head-validation` as a topic blocked by textual conflicts that merging up to the earlier patches could resolve.

### `git repo info` path keys series reaches v9 review
*Long thread: 63 replies so far from 6 participants over 123 days.*

K Jayatheerth's GSoC series extends `git repo info` with path keys and refactors `get_superproject_working_tree()` onto a `struct repository *`. Version 9, posted November 14, renamed keys to `path.workingtree.*`, `path.superprojectworkingtree.*`, and `path.prefix`, and dropped the questioned `path.hooks` and `path.cdup` keys after Phillip Wood argued they encourage an outdated interface. On November 15 Phillip Wood reviewed the v9 range-diff and said he would be happy for the series to be merged as-is, with test consolidation deferred to a planned follow-up series that will also combine overlapping logic between `repo` and `rev-parse`.

### branch: `--delete-integrated` for squash-merged branches
*Long thread: 57 replies so far from 7 participants over 47 days.*

Harald Nordgren's series adds `git branch --delete-integrated <pattern>`, deleting branches whose changes landed upstream via squash or rebase merges and naming the commit with "landed as". Version 7, posted November 14, reads the index once before the walk in `find_landed_commits()` instead of at every file merge, adds a test that a branch whose changes did not land is kept, and states in the commit message that the merge uses the repository's attributes from the working tree and index. Jasper Mayone reviewed v6 and concluded the patch looks good to him.

### doc: detached HEAD guide rewrite
*Long thread: 45 replies so far from 5 participants over 39 days.*

Julia Evans is rewriting the detached HEAD explanation as a standalone guide referenceable as `git help detached-head` from advice messages. Version 6 on November 12 used the agreed wording ("HEAD can store either") and updated test expectations. The thread debated how much detail to include about commits made in detached HEAD state not landing on a branch: Junio C Hamano argued that not clobbering branches when making a temporary commit is the most important user-impacting consequence, while Evans identified the underlying conflict as writing for two audiences -- users accidentally detached versus advanced users exploring. On November 14 Hamano approved her sentence about making experimental commits without being on a branch, saying the material can serve both the general manual and a survival guide for confused users.

### `credentialStore.lockTimeout` naming defended
*Long thread: 42 replies so far from 5 participants over 75 days.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. On November 15 she defended the unit-less name by noting that other file-lock retry settings -- `rerere.lockTimeout`, `reftable.lockTimeout`, `core.filesRefLockTimeout`, `core.packedRefsTimeout`, and `core.configLockTimeout` -- are measured in milliseconds without the unit in the name, unlike `credentialStore.lockTimeoutMS`. She said she will resend with that rationale in the commit message.

### worktree: treat a branch being rebased or bisected as checked out
*Long thread: 38 replies so far from 8 participants over 24 days.*

Jasper Mayone's series makes `git worktree remove` refuse a worktree mid-operation without `--force`, documents reading `%(worktreepath)` safely, names the rebasing or bisecting worktree in `%(worktreepath)`, adds `%(currentworktree)`, and stops `git branch` marking the rebased branch with `+`. Version 5 on November 11 used a map in `branch.c` for one-lookup `branch_checked_out_worktree()`. On November 13 Junio C Hamano praised the central `check_no_operation_in_progress()` design as very good and extensible, called patch 3 a "nice loss of duplicated logic", but said he is "a bit negative" on patch 5, suggesting the series might stop at 3/5 if 4/5 exists only to support it.

### t: add `test-tool genbytes` to avoid slow `tr(1)` on macOS
*Long thread: 30 replies so far from 5 participants over 5 days.*

Kristofer Karlsson's patch adds `test-tool genbytes <char> [<count>]` so tests stop piping `genzeros` through macOS's slow `tr(1)`, which burns over 95 seconds of CPU on t4205. Version 4 on November 11 rebased to catch a new `genzeros | tr` callsite in `t/t1600-index.sh`. Junio C Hamano called the conversions trivially correct and marked the topic ready for `next` on November 10.

### builtin/rerere: convert to subcommands
*Long thread: 29 replies so far from 6 participants over 19 days.*

Patrick Steinhardt's 12-patch series replaces `git rerere`'s hand-rolled subcommand parsing with `OPT_SUBCOMMAND()`, making forget, clear, status, remaining, diff, and gc proper subcommands. On November 14 Junio C Hamano reported a possible UI regression: options before the subcommand are no longer accepted, so `git rerere --skip-locked gc` must be spelled `git rerere gc --skip-locked`.

## The period in brief

The week of 2026/11/09 through 2026/11/15 saw seven active days of heavy, eventful traffic. Junio C Hamano announced coordinated security releases Git 2.56.1 and older maintenance tracks addressing six CVEs. Several long-running series reached review milestones, including v14 of the shallow-clone limited-fetch series, v9 of the `git repo info` path keys, and v6 of the detached HEAD documentation rewrite. A reader should not miss the security release, the ejection of the `--follow` speedup topic in favor of an opt-in `--follow=merges` patch, and the 15-patch segfault fix series being marked ready for `next`.

## Key developments

### Git 2.56.1 security release
Junio C Hamano announced Git 2.56.1 and maintenance releases down to maint-2.43 addressing six CVEs on November 10. Todd Zullinger noted the cited GitLab work item links appear inaccessible to normal users and added Patrick Steinhardt to help open them up.

### Segfault fixes for valueless config settings
Thomas Bachem's 15-patch series fixes segfaults when settings such as `rebase.missingCommitsCheck`, `remote.*.tagOpt`, or `color.blame.*` are given without a value. Version 3 on November 11 made `remote.*.tagOpt` and `color.blame.repeatedLines` warn and ignore a missing value while keeping errors elsewhere. Phillip Wood called the v3 range-diff ready for `next` on November 12, and Junio C Hamano marked the topic ready for `next` the same day.

### log: `--follow` regression and opt-in fix
Justin Tobler benchmarked `git log --stat --follow` on linux.git: 23.5s with the change reverted, 335.9s at the regression, 38.7s with v5 -- still a significant regression. Patrick Steinhardt and Junio C Hamano called for an opt-in toggle. On November 12 Hamano ejected the "speedup" topic and queued Miklos Vajna's opt-in patch making `--follow` rename following for non-linear history optional via `--follow=merges` and `log.follow=merges`.

### promisor-remote reply missing from fetch protocol v2
Christian Couder posted a three-patch series on November 12 fixing a bug where the client omits the `promisor-remote=...` reply from `fetch`, failing with stateless servers such as Gerrit and when fetching only object IDs. Junio C Hamano objected to patch 1/3's dedicated parameter, suggesting the existing `server_options` list be generalized.

### setup: hash algorithm detection on re-init
brian m. carlsson's v2 on November 13 reworks hash selection so a command-line choice takes precedence, and reinitializing an existing repository with a different hash algorithm is refused. Junio C Hamano called the restructured detection simpler on November 14 and said he will replace the patch.

### repack: `--filter-to-loose` for filtered-out objects
Adithya Chakilam's two-patch series adds `--filter-to-loose=<dir>` to `git repack`, writing objects excluded by `--filter` directly as loose objects. On November 14 Junio C Hamano asked for a test that a non-existent destination errors out before any work and suggested `OPT_FILENAME` so relative paths resolve from subdirectories.

### for-each-ref: fix segfault on `--no-format`
Abhishek Agarwal posted v2 on November 14 fixing a segfault in `git for-each-ref --no-format` where `--format` uses `OPT_STRING()`, so `--no-format` sets the format to NULL, which `verify_ref_format()` dereferences. The fix assigns the default format after `parse_options()` when none remains.

### Preserving the `change-id` header through rebase and cherry-pick
Matt Stark's patch on November 15 modifies `sequencer.c` so rebase and cherry-pick preserve a `change-id` header without standardizing the format. brian m. carlsson objected that change-id headers cannot be trusted as a source of unmodified data because users can modify them, warning that a malicious user could reuse change-ids to submit a different version under an existing review.

## In brief

**fsmonitor untracked cache fix re-queued** -- Ilia K's v2 invalidates the whole cache when the worktree cannot be determined; Junio C Hamano reverted the merge to `next` and queued v2 on the original base on November 9.

**Configurable orphaned commit checks** -- Wyatt Holden's three-patch series adds `checkout.orphanedCommitWarning` (default true) so checkout and switch can skip the orphaned-commit reachability check, with advice when it exceeds two seconds.

**ref-filter double free fix** -- Mustafa Bayar fixed a double free in `git for-each-ref` when a ref points to a missing object; Junio C Hamano said he will queue it and mark the topic ready for `next`.

**cat-file out-of-range reflog selectors** -- Caspar Duregger's four-patch series makes `git cat-file --batch-check` report out-of-range reflog selectors as missing instead of dying.

**apply: reject malformed diff headers** -- Tanga Joris posted v2 rejecting in `parse_git_diff_header()` a new file with an old name and a deleted file with a new name, fixing a crash in `git apply --check`.

**pretty.c color fix at line breaks** -- Andy Koppe submitted a patch on November 15 fixing lost and leaked colors at line breaks in custom log formats by making each line self-contained with reset and replay sequences.

**Community voting process** -- Stéphane Glondu proposed Belenios as the voting platform, supporting Condorcet voting with an existing instance at vote.debian.net.

**root refs vs pseudorefs documentation** -- Andy Koppe's five-patch series stops calling root refs pseudorefs across the glossary and man pages; Junio C Hamano wrote that v4 "looked all sensible".

**rerere subcommand conversion** -- Patrick Steinhardt's v4 modernizes `git rerere` parsing; Junio C Hamano applied a capitalization fixup and asked whether the steps could be simplified.

**credential subcommand conversion** -- Wasswa Edson posted v2 converting `git credential` dispatch to `OPT_SUBCOMMAND()`.

**Various small fixes** -- 井上司 fixed the reflog of a symref updated in the same transaction; Yoichi NAKAYAMA corrected a worktree `submodule.<name>.gitdir` fix; Jakub Kulik fixed a Solaris `uname()` check; Hardik Kumar replaced `!` with `test_must_fail` in several tests; Johannes Schindelin updated his Coverity overflow series.