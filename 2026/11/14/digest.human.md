# Git mailing list daily digest for 2026/11/14

## Long threads

### fetch: limited fetching for new remotes in shallow clones

*Long thread: 78 replies so far from 6 participants over 57 days; 2 messages that day.*

Harald Nordgren's series fixes shallow single-branch clones, where `git remote add` installs the default `+refs/heads/*` refspec and the next fetch downloads every branch of the new remote. It adds `remote.<name>.fetchRefmap`, makes a refmap-only remote fetch the branches local branches track, and adds `git remote add --limited-fetch`, default in shallow repositories. Today Nordgren posted v14, squashing the `@{push}` fix into the refmap-only patch and moving its test to `t/t5586-fetch-refmap.sh`, shrinking the series from six patches to five. Earlier, Phillip Wood had urged dropping the refmap-only patch, and Junio C Hamano agreed the `@{push}` failure looked like a bug it introduced.

### repo: path keys for `git repo info`

*Long thread: 60 replies so far from 6 participants over 122 days; 1 message that day.*

K Jayatheerth's GSoC v9 series exposes repository paths via `git repo info`. It renames `path.toplevel.*` to `path.workingtree.*`, `path.superproject-root.*` to `path.superprojectworkingtree.*`, and `path.git-prefix` to `path.prefix`, dropping the questioned `path.hooks` and `path.cdup` keys. The five patches add `path.workingtree.*`, `path.superprojectworkingtree.*`, and `path.index.*` in absolute and relative forms plus a single `path.prefix` key, and make `get_superproject_working_tree()` take a `struct repository *` so `--git-dir` elsewhere gives correct results.

### branch: `--delete-integrated` for squash-merged branches

*Long thread: 57 replies so far from 7 participants over 47 days; 3 messages that day.*

This series adds `git branch --delete-integrated`, which also deletes branches whose changes landed through GitHub squash or rebase merges, naming the commit where they landed. Today Harald Nordgren posted v7: a new test that a branch whose changes did not land is kept, the index read once before the walk in `find_landed_commits()` instead of at every file merge, and a commit message stating the merge uses the repository's attributes from the working tree and index. Tuomas Ahola reviewed patch 4/4 with documentation wording nits.

### doc: detached HEAD guide rewrite

*Long thread: 45 replies so far from 5 participants over 39 days; 3 messages that day.*

Julia Evans is rewriting the detached HEAD explanation as a standalone guide referenceable as `git help detached-head` from advice output. Today Junio C Hamano approved her sentence about making experimental commits without being on a branch, saying the material can serve both the general manual and a survival guide for users who detached by accident; Evans will note both purposes in the commit message.

### builtin/rerere: convert to subcommands

*Long thread: 29 replies so far from 6 participants over 19 days; 2 messages that day.*

Patrick Steinhardt's 12-patch series replaces `git rerere`'s hand-rolled subcommand parsing with `OPT_SUBCOMMAND()`, making forget, clear, status, remaining, diff, and gc proper subcommands. Today Junio C Hamano reported a possible UI regression: options before the subcommand are no longer accepted, so `git rerere --skip-locked gc` must be spelled `git rerere gc --skip-locked`. Tuomas Ahola asked whether `git history fixup HEAD~10` is what Hamano's wished-for `git fixup` command would do.

## The day in brief
Harald Nordgren posted v14 of the shallow-clone limited-fetch series, folding the `@{push}` fix into the earlier patch, and v7 of `--delete-integrated`, which reads the index once and tests the keep case. K Jayatheerth's v9 of the `git repo info` path keys renamed several keys and dropped two, and Abhishek Agarwal's v2 fixes the `git for-each-ref --no-format` segfault.

## Notable threads

### apply: reject malformed new/deleted file headers
Tanga Joris posted v2 of an Outreachy fix for a crash in `git apply --check`, which hit an assertion in `check_preimage()` on a patch whose new-file entry also carried an old name. Patch 2/2 rejects in `parse_git_diff_header()` a new file with an old name and a deleted file with a new name; patch 1/2 reports an empty "diff --git" header as a corrupt-patch error.

### for-each-ref: fix segfault on `--no-format`
Abhishek Agarwal posted v2 fixing a segfault in `git for-each-ref --no-format` (also `git refs list`): `--format` uses `OPT_STRING()`, so `--no-format` sets the format to NULL, which `verify_ref_format()` dereferences. The fix assigns the default format after `parse_options()` when none remains, so `--no-format` cancels an earlier `--format`; v2 changes only the sign-off name.

### repack: `--filter-to-loose` for filtered-out objects
Adithya Chakilam's series adds `--filter-to-loose=<dir>` to `git repack`, writing objects excluded by `--filter` directly as loose objects for key-value stores that handle objects individually. Today Junio C Hamano asked for a test that a non-existent destination errors out before any work, suggested `OPT_FILENAME` so relative paths resolve from subdirectories, and asked to verify `close(unpack.in)` cannot double-close a descriptor when `start_command()` fails.

## In brief
- Yoichi NAKAYAMA corrected his worktree `submodule.<name>.gitdir` fix: `relative_path()` does not always store into the given `strbuf`, so its return value must be used directly in the absolute-path branch.
- Wasswa Edson posted v2 converting `git credential` dispatch to `OPT_SUBCOMMAND()`, split into three patches with a helper rejecting extra arguments and new tests.
- Laveesh Bansal reviewed both t0410 patches as good, confirming the test passes under SHA-256 as well.
- Junio C Hamano called the restructured hash-algorithm detection in `setup` simpler and said he will replace the patch.
