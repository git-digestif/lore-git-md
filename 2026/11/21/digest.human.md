# Git mailing list daily digest for 2026/11/21

## Long threads

### branch: let --delete-merged find squash merged branches

*Long thread: 68 replies so far from 7 participants over 54 days; 3 messages that day.*

This series adds `git branch --delete-integrated <pattern>`, which works like `--delete-merged` but also deletes branches merged via "Squash and merge" or "Rebase and merge", naming the upstream commit containing their changes. Earlier revisions addressed review points including a crash when a submodule path was replaced by regular files. Today Harald Nordgren posted v10 (five patches). Changes in v10 include: the file merge is now a plain text merge ignoring merge drivers in `.gitattributes`; a branch that replaced a file or submodule with a directory is now found as landed; `--dry-run` now works with `-d` and `-D`; "Deleted"/"Would delete" messages print from one table of templates; the opt-out helper was renamed to `opted_out_of_delete_merged()`; and new tests cover `--dry-run`, `--quiet`, and `branch.<name>.deleteMerged=false` with `--delete-integrated`.

## The day in brief

Harald Nordgren posted v10 of the `--delete-integrated` branch-deletion series, Aniket Gupta posted a fix for combined diffs of merges with more than 32 parents, and Orgad Shaneh posted v2 restoring `--prune-tags` clobbering of changed tags.

## Notable threads

### combine-diff: fix patches for more than 32 parents

Aniket Gupta posted a two-patch series in `combine-diff.c` fixing a bug reported by Sami Jawhar where the combined diff of a merge with more than 32 parents put lines in wrong columns and showed no hunks at 64 parents. Patch 1 changes the parent-bit shift in `coalesce_lines()` from `1<<parent` to `1UL<<parent`, fixing the undefined behavior for the 33rd parent and later. Patch 2 makes combine-diff report an error and skip the patch for merges with more parents than the masks can hold (more than 62 where `long` is 64 bits, 30 where it is 32) instead of printing wrong output, without dying so that `git log --cc` is not cut short.

### fetch: make --prune-tags clobber changed tags

Orgad Shaneh posted v2 of a two-patch series in `builtin/fetch.c` restoring the documented behavior of `--prune-tags` force-updating tags that changed on the remote, which has been broken since 0bc8d71b99 (Git 2.20) began rejecting them with "would clobber existing tag". Instead of treating `--prune-tags` like `--force`, v2 adds "+" to the implied refspec (`+refs/tags/*:refs/tags/*`) and updates the docs that quote it; the tests now verify the local tag was actually updated and that `git fetch origin tag <name>` is still refused with `fetch.pruneTags` set.

### submodule: support worktree-specific configuration

Yoichi NAKAYAMA posted v2 of a three-patch series making submodule settings worktree-specific when `extensions.worktreeConfig` is enabled, since shared settings cause submodule worktrees to share the same HEAD and index. Patch 1 keeps `submodule.<name>.gitdir` relative for linked worktrees so it survives moving the worktree along with the main one; patch 2 stores settings such as `submodule.<name>.active` and `submodule.<name>.url` in the worktree config while excluding `.gitdir` from the config copied by `git worktree add`; patch 3 makes `git submodule init` copy an existing gitdir setting to the worktree config only when it belongs to the current worktree, otherwise warning that it was not copied.

## In brief

- Wonseok Oh posted a patch making `fast-import` fail when inline data after `M <mode> inline <path>` is cut off at EOF, but Junio C Hamano objected that the proposed loop change could spin forever on an incomplete final line without a trailing LF.
- Tanga Joris tested the `%w()` indent fix and confirmed it keeps indentation, while reporting that removing `space++` makes a paragraph after a blank line start with an extra space.
- Laveesh Bansal posted v3 of a series making `git reflog expire` expire each reflog only once, skipping already-processed reflogs with a strset and adding tests for `--updateref`.
- Wonseok Oh submitted a patch clearing the saved HEAD OID to the null OID when cherry-picking onto an unborn branch, so `--abort` reports "cannot abort from a branch yet to be born" instead of "Could not parse object".
- **docs: http.extraheader not for credentials**: brian m. carlsson posted a patch clarifying that `http.extraheader` is not for credentials, adding warnings to `http.adoc` and `gitfaq.adoc` and removing outdated text from `git.adoc`.

- William Hatfield asked whether the design of his v2 series adding `--reverse-traversal`, `--include-superproject`, and `--reversive` to `git submodule foreach` is acceptable, questioning the `--reverse-traversal` name.
- René Scharfe called Dominik Maier's report about `git archive --remote --add-file` stat-ing files before the incompatibility check a "good find", and suggested moving the stat to after `--no-add-file` handling.
- Tamir Duberstein updated the ls-files untracked-cache series to v4, taking the index lock after `fill_directory()` so optional writes detect concurrent index changes even with `index.skipHash` enabled.
