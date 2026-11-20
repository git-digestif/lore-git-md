# Git mailing list daily digest for 2026/11/14

## Long threads

### fetch: limited fetching for new remotes in shallow clones

*Long thread (root 2026/09/19/14-47-26): 78 replies so far from 6 participants over 57 days; 2 messages that day.*

Harald Nordgren posted v14 of the series that keeps a shallow single-branch clone from downloading every branch of a newly added remote. The series adds `remote.<name>.fetchRefmap` as a configured default for `--refmap`, makes a refmap-only remote work on its own so plain `git fetch` fetches the branches local branches track, adds `git remote add --limited-fetch`, and defaults to `--limited-fetch` in shallow repositories. Today's v14 squashes the `@{push}` resolution fix into "fetch: infer branches to fetch from a refmap-only remote", moves its test to `t/t5586-fetch-refmap.sh`, and drops the separate "remote: resolve @{push} through remote.<name>.fetchRefmap" commit, shrinking the series from six patches to five. Earlier, reviewers had debated the cover letter, the overlap between `.fetch` and `.fetchRefmap`, and whether the `@{push}` failure was a bugfix belonging with the earlier patch; Nordgren agreed it was and said he would fix it.

### repo: path keys for `git repo info`

*Long thread (root 2026/07/16/01-21-31): 60 replies so far from 6 participants over 122 days; 1 message that day.*

K Jayatheerth posted v9 of the GSoC series exposing repository paths via `git repo info`. The five patches add `path.workingtree.absolute`/`.relative`, refactor `get_superproject_working_tree()` to take a `struct repository *`, add `path.superprojectworkingtree.absolute`/`.relative`, add `path.index.absolute`/`.relative`, and add `path.prefix` for the path from the worktree root to the current directory. The revision renames `path.toplevel.*` to `path.workingtree.*`, `path.superproject-root.*` to `path.superprojectworkingtree.*`, and `path.git-prefix` to `path.prefix`, and drops `path.hooks` and `path.cdup`, which earlier reviewers had argued encouraged an outdated interface or added nothing over the relative working-tree path.

### branch: `--delete-integrated` for squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 57 replies so far from 7 participants over 47 days; 3 messages that day.*

Harald Nordgren posted v7 of the series letting `git branch --delete-integrated` delete branches whose changes landed via squash or rebase merges, naming the upstream commit in output as "landed as <commit>". Changes in v7: a new test that the option keeps a branch whose changes did not land, reading the index once before the walk in `find_landed_commits()` instead of at every file merge, and a commit message stating the merge uses the repository's attributes from the working tree and the index. Tuomas Ahola reviewed patch 4/4 with wording nits on the commit message and documentation.

### doc: rewrite of the detached HEAD explanation

*Long thread (root 2026/10/07/14-52-01): 45 replies so far from 5 participants over 39 days; 3 messages that day.*

Julia Evans's series rewrites the detached HEAD explanation into a standalone guide referenceable as `git help detached-head` in advice. Today Junio C Hamano approved her proposed sentence "You can also use this to make experimental commits without being on a branch", calling it sufficient to hint at detached HEAD's usefulness, and called the material short enough to be shared between the manual and a survival guide. Evans said she would submit the change with a commit message stating the document serves both purposes. Hamano noted `gitdetached-head.adoc` need not merely include `detached-head.adoc`; the rest of the file can serve as survival-guide material.

### builtin/rerere: convert to subcommands

*Long thread (root 2026/10/27/10-46-43): 29 replies so far from 6 participants over 19 days; 2 messages that day.*

Patrick Steinhardt's 12-patch v4 series converts `git rerere`'s hand-rolled subcommand parsing to `OPT_SUBCOMMAND()`, giving each subcommand its own usage string. Today Junio C Hamano reported a possible UI regression in the new syntax: options placed before the subcommand are no longer accepted, so `git rerere --skip-locked gc` now fails and must be spelled `git rerere gc --skip-locked`. Tuomas Ahola also asked whether `git history fixup HEAD~10` is what Hamano's wished-for single `git fixup HEAD~10` command would do.

## The day in brief

The day's most consequential developments were new revisions of three long-running series: Harald Nordgren's v14 of limited fetching in shallow clones and v7 of `--delete-integrated`, and K Jayatheerth's v9 of `git repo info` path keys. Reviewers also weighed in on `git repack --filter-to-loose` and the `git rerere` subcommand conversion's option-ordering regression, while Abhishek Agarwal posted a segfault fix for `git for-each-ref --no-format`.

## Notable threads

### apply: reject malformed new/deleted file headers
Tanga Joris posted v2 of the Outreachy fix for a crash where `git apply --check` hit an assertion in `check_preimage()` on a patch whose new file also carried an old name. Patch 2/2 adds checks in `parse_git_diff_header()` rejecting a new file that also has an old name and a deleted file that also has a new name; patch 1/2 reports an empty "diff --git" header as a corrupt-patch error.

### for-each-ref: fix segfault on `--no-format`
Abhishek Agarwal posted v2 fixing the segfault when `git for-each-ref --no-format` (also affecting `git refs list`) sets the format string to NULL, which `verify_ref_format()` then dereferences. The fix assigns the default format after `parse_options()` when none remains, so `--no-format` cancels an earlier `--format`; v2 changes only the sign-off name.

### repack: `--filter-to-loose` for filtered-out objects
Adithya Chakilam's series adds `--filter-to-loose=<dir>` to `git repack` so objects excluded by `--filter` are written directly as loose objects into another object directory, for key-value stores that manage objects individually. Junio C Hamano asked for a test that a non-existent destination directory errors before any real work, suggested `OPT_FILENAME` so relative paths resolve when `git repack` runs from a subdirectory, and asked to double-check that `close(unpack.in)` cannot close the same descriptor twice when `start_command()` fails.

### setup: fix hash algorithm detection
The patch restructures hash algorithm selection in `git init` so a re-init no longer changes the repository's hash algorithm: the command line wins, otherwise environment, config, then default are consulted only when the repository does not exist or has no hash set, and a mismatch on re-init is refused loudly. Junio C Hamano said the updated structure looks simpler and straightforward, and that he will replace the patch.

### credential: parse subcommands with OPT_SUBCOMMAND
Wasswa Edson posted v2 of the three-patch series converting `git credential` dispatch from `strcmp()` to `OPT_SUBCOMMAND()`, so each subcommand gets its own `-h` usage and the names become available to completion. The series also makes `-h` output agree with the SYNOPSIS, moves each subcommand into its own function, and adds a helper rejecting leftover arguments with "too many arguments", with tests.

## In brief
- Yoichi NAKAYAMA corrected his proposal for making `submodule.<name>.gitdir` relative in worktrees: `relative_path()` does not always store its result in the given `strbuf`, so the absolute-path branch must assign the return value directly.
- Laveesh Bansal gave "Looks good to me" with Reviewed-by tags on both t0410 patches (rev-parse subtree OID, exit codes in test_grep), noting the test passes under SHA-256 too.
- René Scharfe questioned passing `repo` to `init_zip_archiver()` in archive-zip.c where it is unused, and noted the file still sets USE_THE_REPOSITORY_VARIABLE.
- Dirk Gouders posted v2 of the doc/gitcli fixes, keeping "argument" singular and using "before or after it" per Junio C Hamano's suggestion.
- Harald Nordgren reported using the "push: suggest a force push after a clean rebase" patch from `seen`, sharing `git status` output showing "rebased cleanly on 'upstream/master'" with a `git push --force-with-lease` hint.
- Kaartic Sivaraam confirmed v3 of the partial-clone promisor fetch ordering documentation is queued and agreed the client-side configuration should come first.
- Kaartic Sivaraam gave extensive feedback on Hussain's Outreachy proposal draft for promisor remote fetch ordering, flagging reference errors and a conflict between the precedence rule and the preferred config shape.
- Johannes Sixt explained that Git does not auto-activate diff drivers by extension or shebang; users must configure them (e.g. `*.sh diff=bash`) or the default hunk header pattern applies.
