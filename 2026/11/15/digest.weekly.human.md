## Long threads

### fetch: limited fetching for new remotes in shallow clones
*Long thread: 78 replies so far from 6 participants over 57 days.*
Harald Nordgren's series separates which refs a remote fetches from where they are stored, adding `remote.<name>.fetchRefmap` and `--limited-fetch` so shallow clones of a new remote skip fetching every branch. The series reached v14, which squashed the `@{push}` fix into the refmap-only patch and shrank the series from six patches to five. Phillip Wood repeatedly urged dropping the refmap-only patch in favor of combining `.fetchRefmap` with `.fetch`, arguing that after patch 1 the refmap covers storage while `fetch` should cover selection; Junio C Hamano agreed the `@{push}` failure looked like a bug the patch introduced. Matt Hunter identified `ch/remote-defer-follow-remote-head-validation` as the topic blocked by textual conflicts with the early patches.

### `git repo info` path keys series reaches v9 review
*Long thread: 63 replies so far from 6 participants over 123 days.*
K Jayatheerth's GSoC series extends `git repo info` with path keys, reaching v9 which renamed `path.toplevel.*` to `path.workingtree.*`, `path.superproject-root.*` to `path.superprojectworkingtree.*`, and `path.git-prefix` to `path.prefix`, while dropping the questioned `path.hooks` and `path.cdup` keys after Phillip Wood argued they encourage an outdated interface. The series also refactors `get_superproject_working_tree()` to take a `struct repository *`. Phillip Wood reviewed the v9 range-diff and said he would be happy for the series to be merged as-is, with test consolidation deferred to a planned follow-up series that will combine and clean up the overlapping logic between `repo` and `rev-parse`.

### branch: `--delete-integrated` for squash-merged branches
*Long thread: 57 replies so far from 7 participants over 47 days.*
Harald Nordgren's series adds `git branch --delete-integrated <pattern>`, deleting branches whose changes landed upstream via squash or rebase merges and naming the commit with "landed as". Version 7 added a test that a branch whose changes did not land is kept, moved the index read out of the per-file loop in `find_landed_commits()` so it happens once before the walk, and clarified in the commit message that the merge uses the repository's attributes from the working tree and index. Jasper Mayone reviewed v6 and concluded the patch looks good, while Tuomas Ahola reviewed patch 4/4 with documentation wording nits.

### doc: detached HEAD guide rewrite
*Long thread: 45 replies so far from 5 participants over 39 days.*
Julia Evans is rewriting the detached HEAD explanation as a standalone guide referenceable as `git help detached-head` from advice messages. The series reached v6 with agreed wording ("HEAD can store either") and updated test expectations. The discussion centered on serving two audiences -- users who detached accidentally versus advanced users exploring -- with Junio C Hamano arguing that a manual page must cater to both and approving Evans's sentence about making experimental commits without being on a branch. Evans identified the underlying conflict and offered to reframe the series around confused users, but Hamano countered that the material can serve both the general manual and a survival guide.

### credentialStore.lockTimeout naming defended
*Long thread: 42 replies so far from 5 participants over 75 days.*
Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. Responding to Junio C Hamano's question about the unit-less name, she justified it by noting that other file-lock retry settings -- `rerere.lockTimeout`, `reftable.lockTimeout`, `core.filesRefLockTimeout`, `core.packedRefsTimeout`, and `core.configLockTimeout` -- are measured in milliseconds without the unit in the name, unlike `credentialStore.lockTimeoutMS`, while `credential-cache --timeout` is a different setting in seconds. She said she will resend with that rationale in the commit message.

### worktree: treat a branch being rebased or bisected as checked out
*Long thread: 38 replies so far from 8 participants over 24 days.*
Jasper Mayone's v5 series makes `git worktree remove` refuse a worktree mid-operation without `--force`, documents reading `%(worktreepath)` safely, names the rebasing or bisecting worktree in `%(worktreepath)`, adds `%(currentworktree)`, and stops `git branch` marking the rebased branch with `+`. Patch 3 now uses a map in `branch.c` for one-lookup `branch_checked_out_worktree()`. Junio C Hamano praised the central `check_no_operation_in_progress()` design as very good and extensible, called patch 3 a "nice loss of duplicated logic", but said he is "a bit negative" on patch 5, suggesting the series might stop at 3/5 if 4/5 exists only to support it.

### Community voting process
*Long thread: 31 replies so far from 14 participants over 35 days.*
Following Contributor's Summit notes and earlier debate over voter eligibility and platforms, Stéphane Glondu proposed Belenios as the voting platform: it supports Condorcet voting, an instance already runs at vote.debian.net for anyone with a Debian Salsa account, and a new instance could be deployed elsewhere.

### t: add `test-tool genbytes` to avoid slow `tr(1)` on macOS
*Long thread: 30 replies so far from 5 participants over 5 days.*
Kristofer Karlsson's series adds `test-tool genbytes <char> [<count>]` so tests stop piping `genzeros` through macOS's slow `tr(1)`, which burns over 95 seconds of CPU on t4205. Version 4 rebased to catch a new `genzeros | tr` use in `t/t1600-index.sh`. Patrick Steinhardt called v3's argument handling great, and Junio C Hamano marked the topic ready for `next`.

## The period in brief

The week of 2026/11/09 through 2026/11/15 saw seven active days of heavy, eventful traffic on the Git mailing list. The period opened with a coordinated security release, Git 2.56.1, addressing six CVEs, and closed with several long-running series -- the `git repo info` path keys, the shallow-clone fetch refmap, and the valueless-config segfault fixes -- reaching review rounds where they were marked ready for `next` or approved for merging. A reader should not miss the security release announcement, the opt-in fix for the `log --follow` performance regression, and the v9 of the `git repo info` path-keys series that Phillip Wood cleared for merging.

## Key developments

### Git 2.56.1 security release
Junio C Hamano announced coordinated security releases Git 2.56.1 and maintenance releases down to maint-2.43 addressing six CVEs. Todd Zullinger noted the cited GitLab work item links appear inaccessible to normal users, guessing that was not the intent, and added Patrick Steinhardt to help open them up.

### Segfaults on valueless settings marked ready for `next`
Thomas Bachem's 15-patch series fixes segfaults when settings such as `rebase.missingCommitsCheck`, `remote.*.tagOpt`, or `color.blame.*` are given without a value. After v3 made `remote.*.tagOpt` and `color.blame.repeatedLines` warn and ignore a missing value while keeping errors elsewhere, Phillip Wood called the range-diff ready for `next`, Derrick Stolee found it reasonable, and Junio C Hamano marked the topic ready for `next`.

### `log --follow` regression addressed with opt-in flag
Justin Tobler benchmarked `git log --stat --follow` on linux.git: 23.5s with the change reverted, 335.9s at the regression, 38.7s with v5 -- still a significant regression. Patrick Steinhardt agreed the cost is too high for GitLab's master deploys, and Junio C Hamano ejected the speedup topic and queued Miklos Vajna's opt-in patch making `--follow` rename following for non-linear history optional via `--follow=merges` and `log.follow=merges`.

### promisor-remote reply missing from fetch
Christian Couder posted a three-patch series fixing a "promisor-remote" protocol v2 bug where the client omits the `promisor-remote=...` reply from `fetch`, failing with stateless servers such as Gerrit and when fetching only object IDs. Junio C Hamano objected to patch 1/3's dedicated parameter in `write_command_and_capabilities()`, suggesting the existing `server_options` list be generalized to hold keys like "promisor-remote".

### Hash algorithm detection on re-init fixed
brian m. carlsson's v2 reworks hash selection in `setup.c` so a command-line choice takes precedence, environment, config, and default are consulted only for new or hash-less repositories, and reinitializing an existing repository with a different hash algorithm is refused. Junio C Hamano called the restructured detection simpler and said he will replace the patch.

### Configurable orphaned commit checks on checkout
Wyatt Holden's three-patch series adds `checkout.orphanedCommitWarning` (default true) so checkout and switch can skip the orphaned-commit reachability check -- about 193 seconds in an internal mono-repo -- plus advice when it exceeds two seconds, and disables it in Scalar's recommended config. Derrick Stolee supported the series; Junio C Hamano questioned some design choices but called it coherent.

### cat-file: report out-of-range reflog entries as missing
Caspar Duregger's four-patch series makes `git cat-file --batch-check` report out-of-range reflog selectors as missing instead of dying, fixing peel-suffix misparsing in `object-name.c`, scoping `GET_OID_GENTLY` to only failing lookups, failing gently on an empty reflog in `read_ref_at()`, and passing the flag in cat-file batch modes.

### repack: `--filter-to-loose` for filtered-out objects
Adithya Chakilam's two-patch series adds `--filter-to-loose=<dir>` to `git repack`, streaming the filtered pack into `unpack-objects` to write filtered-out objects directly as loose objects in a destination object directory. Junio C Hamano asked for a test that a non-existent destination errors out before any work, suggested `OPT_FILENAME` so relative paths resolve from subdirectories, and asked to verify `close(unpack.in)` cannot double-close a descriptor when `start_command()` fails.

### apply: reject malformed new/deleted file headers
Tanga Joris posted v2 of an Outreachy fix for a crash in `git apply --check`, which hit an assertion in `check_preimage()` on a patch whose new-file entry also carried an old name. Patch 2/2 rejects in `parse_git_diff_header()` a new file with an old name and a deleted file with a new name; patch 1/2 reports an empty "diff --git" header as a corrupt-patch error.

### for-each-ref: fix segfault on `--no-format`
Abhishek Agarwal posted v2 fixing a segfault in `git for-each-ref --no-format` (also `git refs list`): `--format` uses `OPT_STRING()`, so `--no-format` sets the format to NULL, which `verify_ref_format()` dereferences. The fix assigns the default format after `parse_options()` when none remains, so `--no-format` cancels an earlier `--format`.

### Preserving the `change-id` header through rebase and cherry-pick
Matt Stark's patch modifies `sequencer.c` so that rebase and cherry-pick preserve a `change-id` header in whatever format it previously used. brian m. carlsson objected that change-id headers cannot be trusted as a source of unmodified data because users can modify them, warning that a malicious user could reuse change-ids to submit a different version under an existing review.

## In brief

**fsmonitor untracked cache after a trivial response** -- Ilia K posted v2 invalidating the whole cache when the worktree cannot be determined; Derrick Stolee said it LGTM, and Junio C Hamano reverted the merge to `next`, queued v2 on the original base, and said he will mark the topic ready for `next` again after a few days.

**ref-filter: double free when an object is missing** -- Mustafa Bayar fixed a double free in `git for-each-ref` when a ref points to a missing object by resetting `oi->content` in `get_object()`; Karthik Nayak rubber-stamped it, and Junio C Hamano said he will queue it and mark the topic ready for `next`.

**Default hash algorithm in `setup.c`** -- brian m. carlsson's patch defaults the hash algorithm to `GIT_HASH_DEFAULT` (SHA-256 in breaking-changes mode) instead of SHA-1; Junio C Hamano diagnosed a regression and a latent bug in `repository_format_configure()` where `init.defaultObjectFormat` can corrupt an existing repository on re-init.

**builtin/rerere: convert to use subcommands** -- Patrick Steinhardt's v4 12-patch series converts `git rerere`'s custom subcommand parsing into proper subcommands with usage strings; Junio C Hamano reported a possible UI regression where options before the subcommand are no longer accepted.

**root refs vs pseudorefs documentation** -- Andy Koppe's v4 five-patch series stops calling root refs pseudorefs across the glossary, `git-ls-remote(1)`, `gitdatamodel(7)`, `revision.c` and tests; Junio C Hamano wrote that the series "looked all sensible".

**Fix for lost and leaked colors at line breaks in `pretty.c`** -- Andy Koppe submitted a patch making each line self-contained by inserting a reset before newlines where colors are active, replaying those colors at the start of the next non-empty line, and appending a reset at the end if colors remain active.

**fmt-merge-msg config doc patch retracted** -- Yikun Wei retracted a documentation patch that would have included `config/fmt-merge-msg.adoc` in `Documentation/config.adoc`, explaining that those entries are already listed in `git-config(1)` through existing includes.

**井上司's symref reflog fix** -- tsukasa inoue added `ref_transaction_resolve_new_oid()` so the files and reftable backends log a symref target's new value even when updated in the same transaction, and later posted a patch fixing the reflog of a symref whose target is updated in the same transaction.

**Miklos Vajna's `log --follow` speedup** -- v5 speeds up `log --follow` on non-linear history, now also following copies and renames across parents with no merge base, though benchmarks showed it still far slower than before the regression.

**Tanga Joris's `git apply` corrupt header fix** -- Reviewers agreed `git apply` should reject corrupt `diff --git` headers with an error instead of silently applying them.

**Junio C Hamano's `unpack-trees.c` the_repository removal** -- v2 of the series was posted and Derrick Stolee gave an LGTM to the v2 series removing `the_repository` from `unpack-trees.c`.

**Justin Tobler's ODB transaction temp directory** -- A four-patch series creates the ODB transaction temporary directory eagerly; `git add` of 10000 files showed about 7% overhead, and Patrick Steinhardt and Karthik Nayak reviewed it, debating the prefix name and error wording.

**Kristofer Karlsson's `struct commit` padding removal** -- A patch removes padding from `struct commit` on 64-bit platforms, shrinking it from 80 to 72 bytes; reviewers refined the cast-direction and date comments.

**Patrick Steinhardt's `git repo structure` speedup** -- Sped up `git repo structure` in huge repositories by only parsing objects as needed and not fully parsing trees.

**Adrian Ratiu's `ls-remote --show-object-format`** -- v2 prints the remote's object format as `object-format: <format>`; Junio C Hamano reviewed it and said "Looking good".

**Aniket Gupta's fsck object count** -- A patch reports the real object count in the "Checking connectivity" message; Laveesh Bansal noted partial clones count blobs never downloaded, and Gupta proposed retitling the patch accordingly.

**Jakub Kulik's Solaris `uname()` fix** -- Reported that Solaris `uname()` returns non-negative on success, breaking the `if (uname(...))` check in `version.c`; he proposed `< 0`, which Junio C Hamano confirmed matches POSIX.

**Hardik Kumar's test modernization** -- Replaced the `!` operator with `test_must_fail` in `t/t5564-http-proxy.sh`, `t/t6030-bisect-porcelain.sh`, `t/t7400-submodule-basic.sh`, and `t/t9001-send-email.sh` so that signal-killed or segfaulting commands fail the test.

**Johannes Schindelin's Coverity overflow series** -- Updated so `write_in_full()` returns 0 on success, the GCC-only `mult_overflows` macros were dropped, and `date.c` uses manual overflow checks in the approxidate relative-time multiplication.

**Srujith A Poojary's archive refactor** -- v2 narrowed to passing an explicit `struct repository` to `init_tar_archiver()` and `init_archivers()` and removing `USE_THE_REPOSITORY_VARIABLE` from `archive-tar.c`.

**Yusuke Hayashi's `http-backend` patch** -- v2 drops the strbuf pre-sizing in `get_info_packs()`, with a rewritten commit message and no code changes from v1.

**Dirk Gouders's `doc/gitcli` fixes** -- Resent v3 of the minor fixes with the proper Cc and a range-diff against v1.