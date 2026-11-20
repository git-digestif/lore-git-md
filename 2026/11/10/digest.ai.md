# Git mailing list daily digest for 2026/11/10

## Long threads

### `git repo info` path keys naming debate

*Long thread (root 2026/07/16/01-21-31): 57 replies so far from 6 participants over 118 days; 2 messages that day.*

The GSoC series by K Jayatheerth adds path keys to `git repo info` (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, `path.cdup`) and refactors `get_superproject_working_tree()` to operate on a `struct repository *`. Earlier, Phillip Wood had suggested renaming keys to `path.working-tree.*`, `path.superproject-working-tree.*`, and `path.prefix`, and K Jayatheerth agreed on the superproject-working-tree name while defending `path.hooks` for scripts migrating from `git rev-parse --git-path hooks`. Today Phillip Wood argued that `git repo info` should provide a better interface than `rev-parse` rather than porting its path options unchanged, so adding `path.hooks.*` would encourage an outdated interface that ignores config-based hooks; he also questioned the practical benefit of `path.cdup` over `path.working-tree.relative`, and noted that while repo-info keys are not config keys, existing keys like `path.gitdir.*` and `path.commondir.*` use no dashes, so dropping dashes would be consistent.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread (root 2026/10/21/17-33-36): 32 replies so far from 8 participants over 21 days; 6 messages that day.*

Jasper Mayone's v4 series makes `git worktree remove` refuse a worktree mid-operation (rebase, am, bisect, cherry-pick/revert) without `--force`, documents reading `%(worktreepath)` safely, makes `%(worktreepath)` name the worktree rebasing or bisecting a branch, and stops `git branch` from marking such a branch with "+". Today Phillip Wood reviewed the patches: he approved patch 2/4, suggested list-formatting tweaks to patch 1/4's new bullet list in git-worktree.adoc, objected that `branch_checked_out_worktree()` iterates over all worktrees for every listed ref and should use a hash map as the existing ref-filter code does, and for patch 4/4 proposed a `%(currentworktree)` format specifier instead of the patch's approach. Ben Knoble disagreed with the list-formatting suggestion, saying the "or" construction fits as-is.

### test-tool genbytes to avoid slow tr(1) on macOS

*Long thread (root 2026/11/07/14-59-22): 26 replies so far from 5 participants over 4 days; 4 messages that day.*

Kristofer Karlsson's patch adds `test-tool genbytes <char> [<count>]` next to `genzeros` because macOS `tr(1)` processes input one character at a time through getwchar()/putwchar(), making t4205 spend 95-123 seconds of CPU in tr alone on macOS CI. Earlier discussion covered printf-based alternatives, which fail on macOS, and buffer and parsing choices. Today Kristofer Karlsson posted v3, which takes a literal character instead of two hex digits, keeps `genzeros` and its NUL-byte callers, keeps the file unrenamed, uses a static buffer, and parses the count with `git_parse_signed()` so an invalid count is a usage error; he also confirmed macOS `tr` has no single-byte fast path. Junio C Hamano reviewed the v3 diff, called the conversions trivially correct, and marked the topic ready for `next`.

### doc: rewrite detached HEAD state explanation

*Long thread (root 2026/10/07/14-52-01): 25 replies so far from 4 participants over 35 days; 2 messages that day.*

Julia Evans's patch rewrites the detached HEAD explanation into a standalone `gitdetached-head` guide wired into advice.c, command-list.txt, and the build files, addressing issues like not explaining why detached HEAD is useful and not advertising that `git switch` makes accidental detachment harder. Earlier, Junio C Hamano and D. Ben Knoble discussed how to describe HEAD, with Junio objecting to the "useful for HEAD to be a commit ID" phrasing. Today Junio proposed a four-point explanation built on HEAD recording a branch or commit object name to refer to the checked-out commit, with the direct reference being "detached HEAD" state; Julia Evans called the list great, proposed communicating its first three points, and objected that the verb "record" is not well understood by users, asking whether it means "references" or "points to".

### doc: git-diff EXAMPLES section rewrite

*Long thread (root 2026/11/03/15-33-27): 23 replies so far from 6 participants over 8 days; 4 messages that day.*

Julia Evans's v2 patch rewrites the git-diff EXAMPLES section into sub-sections with a definition list, mentions `--cached` alongside `--staged`, restores the `git diff AUTO_MERGE` example reworded to "merge conflict", and adds a DESCRIPTION note pointing to EXAMPLES. Earlier, Jeff King flagged AsciiDoc anchor syntax that works in both Asciidoctor and python AsciiDoc, and Junio queued SQUASH??? fixups for those. Today Julia Evans retracted an accidental swap of `--cached` for `--staged` in the synopsis, affirmed the rest of the patch, and said she would learn how to apply the queued squash commits; Kaartic Sivaraam explained the SQUASH??? commits are already in Junio's topic branch and gave commands to view and cherry-pick them, and Junio suggested fetching from him and cherry-picking the squash commits from `seen`.

## The day in brief
Coordinated security releases Git 2.56.1 (and older maintenance tracks) and Git for Windows 2.56.1 were announced, addressing six CVEs. Kristofer Karlsson's `test-tool genbytes` v3 was marked ready for `next` after fixing the macOS `tr(1)` slowdown, and Omri Sarig's git-rebase documentation fix was likewise marked ready for `next`.

## Notable threads

### Git 2.56.1 security release announcement
Junio C Hamano announced coordinated security fix releases Git 2.56.1 and others down to maint-2.43, addressing CVE-2026-111760, CVE-2026-113408, CVE-2026-114466, CVE-2026-114467, CVE-2026-114469 and CVE-2026-105437, with tarballs at the usual places and tags in the public repositories. Todd Zullinger reported that the GitLab work item links cited in the announcement appear inaccessible to normal users and added Patrick Steinhardt to the Cc to help get them opened up.

### Fix segfaults on settings given without a value

*Long thread (root 2026/11/07/08-42-32): 16 replies so far from 4 participants over 4 days; 10 messages that day.*

Thomas Bachem posted v2 of the 15-patch series fixing segfaults when a config setting is given without a value (e.g. `git -c foo.bar`), rebased onto 3c60ef1e38 and compatible with `next` and `seen`; patch 1 now warns and falls back to "ignore" for a valueless `rebase.missingCommitsCheck`. Phillip Wood questioned several patches' erroring on NULL values where invalid values are only warned or ignored, and Derrick Stolee accepted upgrading from segfault to error for the bundle URI code. Thomas Bachem replied that v3 will make the `remote.<name>.tagOpt` and `color.blame.repeatedLines` patches warn and ignore a missing value, keep errors for the bundle list and for `color.blame.highlightRecent` and `branch.<name>.mergeOptions`, and adopt Phillip Wood's test suggestions.

### t9300: group and move fast-import tests to other files
Karthik Nayak posted a 20-patch series making tests in the roughly 4000-line t9300 self-contained and moving them into dedicated files (t9301, t9304, and new t9307 through t9311). Junio C Hamano acknowledged the cleanup as long overdue, and noted that the topics cc/early-scan-options and tj/drop-no-parseopt already in `next` touch the same script, suggesting a rebase onto a synthetic base at 1891de46f7 with those topics on top.

### Git for Windows 2.56.1 announcement
Johannes Schindelin announced Git for Windows 2.56.1, addressing the same six CVEs as Git 2.56.1 and shipping with Git v2.56.1.

### object-file: move loose backend logic into odb/source-loose.c
Patrick Steinhardt's 6-patch series moves loose-backend logic from object-file.c into odb/source-loose.c so backend-internal access is greppable. Kaartic Sivaraam reviewed all six patches today: he asked that documentation comments dropped from object-file.h be retained above the static functions in odb/source-loose.c, suggested dropping now-unneeded object-file.h includes from several files, proposed a goto pattern in `map_fd()`, questioned whether the new `odb/streaming.h` include in object-file.h is needed, and otherwise found the patches good.

## In brief
- tsukasa inoue added `ref_transaction_resolve_new_oid()` so both the files and reftable backends record a symref target's new value in the reflog even when the target is created or updated in the same transaction, with tests in t/t1400-update-ref.sh.
- Wyatt Holden submitted a three-patch series making orphaned commit checks configurable via `checkout.orphanedCommitWarning` (default true), advising when the check exceeds two seconds, and disabling it in Scalar's recommended config; Junio C Hamano questioned omitting a command-line option but called the series coherent.
- Omri Sarig's v6 of the git-rebase doc fix for incompatible-options links was marked ready for `next` by Junio C Hamano.
- Derrick Stolee gave an LGTM to the v2 series removing `the_repository` from unpack-trees.c; Patrick Steinhardt and Junio C Hamano agreed that passing a `struct repository *` through helper structs would be a separate cleanup.
- Justin Tobler posted a 4-patch series creating the "files" ODB transaction temporary directory eagerly at transaction start, with a benchmark showing roughly 7% overhead for `git add` of 10000 files.
- Laveesh Bansal posted v2 making `git reflog expire` expire each reflog only once, skipping names already processed in the same run.
- Azuama Odinaka Maximus posted an RFC series adding a generic `early_scan_options()` API to parse-options and updating rev-list's early scan to skip arguments of options like `--grep`.
- **git history reword default diff**: Karthik Nayak agreed to show the diff by default for `git history reword`, with Ben Knoble accepting an opt-out settable via config.
