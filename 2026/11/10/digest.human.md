# Git mailing list daily digest for 2026/11/10

## Long threads

### Path keys for `git repo info`

*Long thread: 57 replies so far from 6 participants over 118 days; 2 messages that day.*

K Jayatheerth's GSoC series adds path keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, `path.cdup`) to `git repo info` and refactors `get_superproject_working_tree()` to take a `struct repository *`. Phillip Wood argued `git repo info` should be a better interface than `rev-parse`, not a port of its path options: he said `path.hooks.*` would encourage an outdated interface that ignores config-based hooks, questioned `path.cdup`'s benefit over `path.working-tree.relative`, and noted existing keys like `path.gitdir.*` use no dashes.

### Worktrees rebasing or bisecting a branch

*Long thread: 32 replies so far from 8 participants over 21 days; 6 messages that day.*

Jasper Mayone's v4 series makes `git worktree remove` refuse a worktree mid-rebase, `git am`, bisect, or cherry-pick without `--force`, documents `%(worktreepath)`, names the rebasing worktree in `%(worktreepath)`, and stops `git branch` marking such branches with "+". Phillip Wood approved the doc patch but said `branch_checked_out_worktree()` iterates all worktrees per listed ref and a hash map should be used; for the `git branch` patch he proposed a `%(currentworktree)` specifier instead. On the remove patch he asked for self-contained list sentences; Ben Knoble disagreed.

### `test-tool genbytes` to avoid slow `tr(1)` on macOS

*Long thread: 26 replies so far from 5 participants over 4 days; 4 messages that day.*

Kristofer Karlsson's patch adds `test-tool genbytes <char> [<count>]` so tests stop piping `genzeros` through `tr(1)`, which on macOS spends 95-123 CPU-seconds on t4205. He posted v3 taking a literal character, keeping `genzeros`, using a static buffer and `git_parse_signed()`; Junio C Hamano called the conversions trivially correct and marked the topic ready for `next`.

### Detached HEAD documentation rewrite

*Long thread: 25 replies so far from 4 participants over 35 days; 2 messages that day.*

Julia Evans's patch rewrites the detached HEAD explanation as a standalone `gitdetached-head` guide wired into `advice.c` and the build files. Junio C Hamano reiterated that "HEAD is a branch" versus "HEAD is a commit ID" is misleading -- HEAD records one to refer to the checked-out commit -- and proposed a four-point explanation; Julia Evans called it great but objected that "record" is not a word users understand.

### `git diff` EXAMPLES section rewrite

*Long thread: 23 replies so far from 6 participants over 8 days; 4 messages that day.*

Julia Evans retracted an accidental `--cached`->`--staged` synopsis swap from the v2 patch, saying she "will take this out" to keep the scope small. Kaartic Sivaraam explained that Junio C Hamano's queued SQUASH??? commits can be fetched from his tree and cherry-picked; Junio suggested finding them via `git log seen`.

## The day in brief
Junio C Hamano announced coordinated security releases Git 2.56.1 and older maintenance tracks addressing six CVEs. Two topics -- `test-tool genbytes` and the `git-rebase` doc fix -- were marked ready for `next`, and new work arrived on configurable orphaned-commit checks and symref reflog entries.

## Notable threads

### Git 2.56.1 security release announcement

Junio C Hamano announced Git 2.56.1 and maintenance releases down to maint-2.43 addressing six CVEs. Todd Zullinger noted the cited GitLab work item links appear inaccessible to normal users, guessing that was not the intent, and added Patrick Steinhardt to help open them up.

### Fixing segfaults on settings given without a value

*Long thread: 16 replies so far from 4 participants over 4 days; 10 messages that day.*

Thomas Bachem posted v2 of a 15-patch series fixing segfaults when a setting is given without a value (`git -c foo.bar`), covering `rebase.missingCommitsCheck`, `core.alternateRefsCommand`, `fetch.bundleCreationToken`, `grep.patternType`, `remote.*.tagOpt`, `color.blame.*`, `branch.*.mergeOptions`, `remotes.*` and `safe.bareRepository`. Phillip Wood questioned patches that error on NULL while ignoring invalid values; Derrick Stolee accepted error-over-segfault for bundle URIs; Junio C Hamano disagreed on treating a missing `mergeOptions` as empty. Bachem said v3 will warn-and-ignore for `tagOpt` and `color.blame.repeatedLines` and keep errors elsewhere.

### Configurable orphaned commit checks on checkout
Wyatt Holden's three-patch series adds `checkout.orphanedCommitWarning` (default true) so checkout and switch can skip the orphaned-commit reachability check -- about 193 seconds in an internal mono-repo -- plus advice when it exceeds two seconds, and disables it in Scalar's recommended config. Derrick Stolee flagged a whitespace nit and supported the series; Junio C Hamano questioned some design choices but called it coherent.

## In brief
- tsukasa inoue added `ref_transaction_resolve_new_oid()` so the files and reftable backends log a symref target's new value even when updated in the same transaction.
- Kaartic Sivaraam reviewed the series moving loose-object logic from `object-file.c` into `odb/source-loose.c`, asking that dropped doc comments be kept above the moved functions.
- Srujith A Poojary posted v3 of the archive `USE_THE_REPOSITORY_VARIABLE` cleanup; Junio C Hamano objected that the iteration was sent before the prior review concluded. *Long thread: 7 replies so far from 3 participants over 2 days; 8 messages that day.*
- Derrick Stolee gave an LGTM to the v2 series removing `the_repository` from `unpack-trees.c`.
- Justin Tobler posted a four-patch series creating the ODB transaction temp directory eagerly; `git add` of 10000 files showed about 7% overhead.
- Junio C Hamano marked the `git-rebase` incompatible-options doc fix ready for `next`.
- Azuama Odinaka Maximus posted an RFC adding a generic `early_scan_options()` API to parse-options and updating `rev-list`'s early scan.
