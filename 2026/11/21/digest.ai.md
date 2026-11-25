# Git mailing list daily digest for 2026/11/21

## Long threads

### branch: let --delete-integrated find squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 68 replies so far from 7 participants over 54 days; 3 messages that day.*

This series adds `git branch --delete-integrated <pattern>` to the branch builtin, which works like `--delete-merged` but also deletes a branch when some upstream commit contains all of its changes, as happens with GitHub "Squash and merge" or "Rebase and merge"; it names that commit in the output ("Deleted branch topic (was ..., landed as ...)"). Earlier revisions fixed a crash when a path was a submodule at the fork point and both the branch and upstream replaced it with different regular files, and earlier review questioned the opt-out helper name, the `--dry-run` handling, and the message wording. Today Harald Nordgren posted v10 (five patches): the file merge is now a plain text merge that ignores merge drivers set in `.gitattributes` (dropping the index read added in v6), a branch that replaced a file or submodule with a directory is now found as landed, `--dry-run` works with `-d` and `-D`, the "Deleted"/"Would delete" messages are printed from one table of templates in `delete_branches()`, the helper is renamed to `opted_out_of_delete_merged()`, the `--delete-integrated` commit subject is reworded, and new tests cover `--dry-run`, `--quiet`, and `branch.<name>.deleteMerged=false` with `--delete-integrated`. He also endorsed the earlier suggestion to select the message templates from a single table based on `--dry-run`.

## The day in brief

Harald Nordgren posted v10 of the series adding `git branch --delete-integrated` for squash- and rebase-merged branches, with a plain-text merge that ignores `.gitattributes` drivers and a unified message-template table. Aniket Gupta fixed the combined-diff column corruption for merges with more than 32 parents and made oversized octopus merges report an error instead of printing wrong hunks. Orgad Shaneh rerolled the fix making `fetch --prune-tags` clobber changed tags by adding "+" to its implied refspec.

## Notable threads

### combine-diff: fix patches for more than 32 parents

Aniket Gupta posted a two-patch series in `combine-diff.c` fixing a bug reported by Sami Jawhar where the combined diff of a merge with more than 32 parents put lines in wrong columns and showed no hunks at all with 64 parents. The first patch changes the int shift `1<<parent` in `coalesce_lines()` to `1UL<<parent`, which had set the bit for parent minus 32 for the 33rd parent and later, and adds a test with forty parents where two lose the same line. The second makes combine-diff report an error and skip the patch for merges with more parents than the masks can hold (more than 62 where `long` is 64 bits, 30 where it is 32 bits) instead of showing missing or wrongly columned hunks, without dying so that `git log --cc` is not cut short; raw format output is unchanged, with tests for sixty-two and sixty-three parents.

### fetch: make --prune-tags clobber changed tags

Orgad Shaneh posted v2 of the series fixing `fetch --prune-tags`, which since 0bc8d71b99 (Git 2.20) rejects changed tags with "would clobber existing tag" instead of force-updating them as documented. Rather than treating `--prune-tags` like `--force` for tags as in v1, v2 adds "+" to the refspec that `--prune-tags` implies, changing `refs/tags/*:refs/tags/*` to `+refs/tags/*:refs/tags/*` in `builtin/fetch.c` and the docs that quote it. The tests now verify the local tag was actually updated after each forced fetch and that `git fetch origin tag <name>` is still refused with `fetch.pruneTags` set, and the series is rebased on current master.

### fast-import: catch missing data after inline M at EOF

Wonseok Oh posted a patch to `builtin/fast-import.c` so that `M <mode> inline <path>` at the end of input no longer puts an uninitialized stack object ID into the tree and exits 0, leaving a tree entry for a missing blob that `git fsck` flags; the patch replaces the `while (read_next_command(state) != EOF)` loop in `file_change_m()` with a `for (;;)` loop and adds a test that fast-import fails when inline data is cut off at end of input. Junio C Hamano objected that the reasoning is wrong because `note_change_n()` can call `parse_and_store_blob()` at EOF only since it does not loop, and warned that the proposed change would loop forever if a `cat-blob` command appears on an incomplete final line without a trailing LF, since the updated code ignores the return value of `read_next_command()`.

### archive: reject remote --add-file in callback

Dominik Maier posted a patch to `archive.c` fixing that `parse_archive_args()` only checks the `--remote` and `--add-file` incompatibility after `parse_options()` returns, so `add_file_cb()` runs first and calls `stat()` on the server filesystem, reporting "File not found" before the incompatibility check is reached; the patch passes `is_remote` into `add_file_cb()` and rejects `--add-file` and `--add-virtual-file` immediately before touching the filesystem. René Scharfe called the bug report a "good find", noted that rejecting `--add-virtual-file` with `--remote` is arguably also a bug since the option's value brings file name and content to the remote end, observed that the patch would also reject `--remote --add-file=foo --no-add-file`, and suggested moving the `stat(2)` call to after applying any `--no-add-file` but before writing archive content.

### submodule: support worktree-specific configuration

Yoichi NAKAYAMA posted v2 of a three-patch series making submodule settings worktree-specific when `extensions.worktreeConfig` is enabled, since settings in the repository config are shared between all worktrees and sharing `submodule.<name>.gitdir` makes submodule worktrees share the same HEAD and index. Patch 1/3 keeps `submodule.<name>.gitdir` relative for linked worktrees by deriving the modules directory from `worktree_git_path()`, so it keeps working when the worktree is moved with the main worktree. Patch 2/3 stores settings such as `submodule.<name>.active` and `submodule.<name>.url` in the worktree config, excluding `submodule.<name>.gitdir` from the config copied by `git worktree add`. Patch 3/3 makes `git submodule init` copy an existing `submodule.<name>.gitdir` to the worktree config only if it belongs to the current worktree, otherwise generating a new setting and warning that the existing config was not copied.

## In brief

- Laveesh Bansal posted v3 of "builtin/reflog: expire each reflog only once", collecting `--all` and named refs into one loop and skipping reflogs already processed in the same run with a strset, with tests for repeated output and `--updateref` when a ref is named twice.
- Tanga Joris tested the `%w()` indent fix and confirmed indented list continuations keep their spaces, but reported that removing the `space++` makes a paragraph after a blank line start with an extra space, and noted the documented newline replacement only occurs when wrapping is enabled, not with `%w(0,2,2)`.
- Wonseok Oh submitted a sequencer patch clearing the saved HEAD object ID to the null OID when `repo_get_oid()` fails on an unborn branch, so `git cherry-pick --abort` reports "cannot abort from a branch yet to be born" instead of "Could not parse object".
- brian m. carlson posted a documentation patch clarifying in `Documentation/config/http.adoc`, `Documentation/git.adoc`, and `Documentation/gitfaq.adoc` that `http.extraheader` should not be used for credentials, since it can expose them to every server or other users and breaks Git LFS with duplicate Authorization headers.
- **ls-files untracked-cache v4 update**: Tamir Duberstein updated the series to v4: with 6b6d1d56eb landed on master enabling detection of concurrent index changes even with index.skipHash enabled, v4 takes the lock after fill_directory() via repo_lock_and_update_index_if_able(), removes the early-lock assertion, and adds concurrent-update coverage.

- William Hatfield asked whether the `--reverse-traversal` name in his v2 "submodule: foreach in reverse order, with the superproject" series is acceptable before resending, arguing it reverses the existing order literally and means the same with or without `--recursive`.
- Tanga Joris noted that `git credential fill --` now runs the subcommand instead of failing with usage output under the OPT_SUBCOMMAND conversion, called the change harmless, and said the patch looks good to him.
- Junio C Hamano retracted his earlier test-chainlint objection to a bare `grep` in the compatObjectFormat v2 test, clarifying it is data filtering rather than an assertion, and said the v2 changes looked reasonable to him.
- **Aniket Gupta on status check UX and cost**: Aniket Gupta acknowledged a user-experience concern about the check suddenly disappearing, then explained the extra cost only comes from commits outside the upstream, so a normal rebased branch should be fine.

- Hussain Sitamouwala posted draft v2 of the Outreachy proposal "Implement promisor remote fetch ordering", correcting the merged-to-`next` status of patch 1, the call chain to `promisor_remote_get_direct()`, and the precedence section so user and server-supplied values stay separate.
