# Git mailing list daily digest for 2026/11/16

## Long threads

### `git repo info` path keys (GSoC series v9)

*Long thread (root 2026/07/16/01-21-31): 64 replies so far from 6 participants over 124 days; 1 message that day.*

This GSoC series exposes repository paths and metadata via the `git repo info` command, adding keys such as `path.workingtree.absolute`/`path.workingtree.relative`, `path.superprojectworkingtree.*`, `path.index.*`, and `path.prefix`, and refactoring `get_superproject_working_tree()` to operate on a `struct repository *`. Earlier revisions renamed `path.toplevel.*` to `path.workingtree.*`, dropped `path.hooks` after reviewers argued it encouraged an outdated interface, and dropped `path.cdup` as redundant with `path.working-tree.relative`. Today K Jayatheerth stated that the easiest approach would be to directly combine the tests and optimize them in the follow-up series he is already planning to consolidate overlapping logic between `repo` and `rev-parse`.

### `git branch --delete-integrated` for squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 58 replies so far from 7 participants over 49 days; 1 message that day.*

This series adds `--delete-integrated` to `git branch`, which deletes a branch when some upstream commit since the fork point contains all of its changes--covering GitHub "Squash and merge" and "Rebase and merge" cases whose tips are unreachable--and names that commit in the output. Earlier revisions moved the index read out of the per-file loop in `find_landed_commits()` and added a test that a branch whose changes did not land is kept. Today Jasper Mayone confirmed the new v7 test covers the same three cases he had proposed tests for, reported building v7 and running t3200 with his four extra tests on top with all 209 tests passing, confirmed v7 addresses his three earlier points, and added a `Tested-by:` trailer.

### New detached HEAD state guide

*Long thread (root 2026/10/07/14-52-01): 46 replies so far from 5 participants over 41 days; 2 messages that day.*

Julia Evans is adding a new `git help detached-head` guide to help users who accidentally end up in detached HEAD state, dual-purpose with the DETACHED HEAD section of `git-checkout.adoc`, fixing issues like not explaining why detached HEAD is useful or why it happens accidentally. Earlier discussion settled wording around how `HEAD` stores a branch or commit ID and the quote style in the `advice.c` message. Today Julia Evans posted v7, adding the sentence "You can also use this to make experimental commits without being on a branch" and updating the commit message to explain the dual purpose. Junio C Hamano said the v7 commit message makes the change clearer but asked for someone else to give a hopefully final pass with fresh eyes before merging it down to `next`.

### `credentialStore.lockTimeout` config option

*Long thread (root 2026/09/02/08-31-37): 43 replies so far from 5 participants over 76 days; 2 messages that day.*

This Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for the credential storage lock retry timeout, matching other file-lock retry settings (`rerere.lockTimeout`, `reftable.lockTimeout`, `core.filesRefLockTimeout`, `core.packedRefsTimeout`, `core.configLockTimeout`) that are measured in milliseconds without the unit in the name, while keeping `lockTimeoutMS` as a deprecated synonym. Earlier review consolidated the zero-timeout tests into a loop over both names and dropped a test that never held the lock. Today Okhuomon Ajayi resent the v3 patch with that justification in the message, and Junio C Hamano reviewed it, said it looks good, and marked the topic ready for `next`.

### Worktrees rebasing or bisecting a branch

*Long thread (root 2026/10/21/17-33-36): 41 replies so far from 8 participants over 27 days; 3 messages that day.*

This series makes Git treat a branch being rebased or bisected in a worktree as checked out: `git worktree remove` refuses such a worktree without `--force`, `%(worktreepath)` names the worktree, and a map in `branch.c` makes the lookup a single hash lookup. Today Jasper Mayone answered Junio C Hamano's question about how the new `check_no_operation_in_progress()` interacts with the existing checked-out-branch protection--the two checks answer different questions and should stay separate--and agreed with Junio's objection to the final patch, accepting dropping patches 4/5 and `%(currentworktree)` and stopping the series at 3/5. Phillip Wood separately responded on `%(currentworktree)`, suggesting the name `%(currentworktreepath)` if a path is printed and noting a hashmap is unnecessary for a single entry.

## The day in brief
Junio C Hamano marked two topics ready for `next`: the `credentialStore.lockTimeout` rename and the `log --follow` optionality change. Patrick Steinhardt posted a 12-patch series making commit graphs pluggable across object database backends, and two partial-clone lazy-fetch bug reports were filed, one of which already has a fix verified in flight.

## Notable threads

### Pluggable commit graphs
Patrick Steinhardt posted a 12-patch series introducing pluggable commit graphs so that an object database backend is in full control over where the commit graph is read from and written to, moving all storage path derivation logic from `commit-graph.c` into the "files" backend in `odb/source-files.c` and adding `read_commit_graph`/`write_commit_graph` callbacks to `odb_source`. The series also refactors chain linking, split merge strategy, and write paths ahead of the pluggability change. Junio C Hamano reviewed PATCH 01/12, calling the simplification nice and questioning whether `cur_g->oid` becomes write-only now that the OID array comparison was removed.

### index-pack lazy-fetches in-pack REF_DELTA bases
לירון עדן filed a bug report: in a blobless partial clone fetching a thin pack whose in-pack delta bases arrive as REF_DELTA (no ofs-delta negotiated), `index-pack` starts a nested lazy fetch for delta bases already in the pack being indexed, because the prefetch loop in `fix_unresolved_deltas()` in `builtin/index-pack.c` iterates all REF_DELTA entries and the incoming objects are not yet in the object store. The report documents recursive nested lazy fetches--12 levels in about a minute, several hundred in an unattended job--against Azure DevOps Services, and notes the workaround of prefetching the commit with `GIT_NO_LAZY_FETCH=1`.

### Partial clone lazy-fetches a blob it just wrote
Google -- Mergify reported that since Git 2.54.0 a cherry-pick in a `blob:none` partial clone can lazy-fetch a blob the cherry-pick just wrote locally, bisected to 5946a564cd, because the loose object cache is not updated when an object is written; the command dies even though the commit was stored. Patrick Steinhardt identified the issue as known with a fix in flight and asked for testing; after verification that both patches make the cherry-pick succeed and a new regression test pass, he acknowledged the result.

### `git rerere` converted to subcommands

*Long thread (root 2026/10/27/10-46-43): 32 replies so far from 6 participants over 21 days; 3 messages that day.*

Patrick Steinhardt's v4 series modernizes `builtin/rerere` to use `OPT_SUBCOMMAND()` like other Git commands, turning `forget`, `clear`, `status`, `remaining`, `diff`, and `gc` into proper subcommands with their own usage strings. The conversion makes option placement backwards-incompatible--`git rerere --skip-locked gc` must become `git rerere gc --skip-locked`. Today Junio C Hamano acknowledged he had missed the documented incompatible change and decided to accept breaking `--skip-locked` placement: "OK. Let's break it and have the users fix their scripts."

### Optional `--follow` rename following for non-linear history
This series makes `--follow` following renames for non-linear history optional via `--follow=merges`, fixing a performance regression from 304812ed33 and a case where the speedup lost history when one merge parent renamed a path and the other edited the old name. Today Patrick Steinhardt judged the v2 patch "looks good to me", and Junio C Hamano marked the topic ready for `next`.

## In brief
- Patrick Steinhardt posted v2 of "builtin/repo: improve counting of objects", which avoids fully parsing trees in `git repo structure`; Derrick Stolee called the memory reduction (3.4 GiB down to 0.7 GiB) impressive, and Junio C Hamano marked the topic ready for `next`. *Long thread (root 2026/11/13/11-46-16): 9 replies so far from 4 participants over 4 days; 8 messages that day.*
- Andreas Wendleder reported that with `submodule.recurse=true`, switching branches in a superproject worktree whose submodule is a linked worktree aborts with `BUG()` in `validate_submodule_legacy_git_dir()`, and proposed either accepting linked worktrees or returning an error instead of `BUG()`.
- Discussion of the `promisor-remote` crash without a repository: Patrick Steinhardt and Junio C Hamano agreed the easier `repo->gitdir` NULL check is acceptable for now, with passing the repository down and checking `!repo` as the more sensible long-term direction.
- **Tanga Joris to send v3 patch series**: He will add a preparatory patch with a `free()` in `parse_git_diff_header()` for the empty "diff --git" line fix, and name the `---`/`+++` lines in the error for the both-names patch.

- Patrick Steinhardt approved the v2 fix for the `git for-each-ref --no-format` segfault, agreeing that restoring the default format is the better pick over `PARSE_OPT_NONEG`.
- Patrick Steinhardt reviewed the imap-send header-order fix, suggesting a clearer header-block termination check, and said he is otherwise happy with the version. *Long thread (root 2026/10/19/19-19-16): 20 replies so far from 3 participants over 29 days; 1 message that day.*
- Patrick Steinhardt reviewed Adrian Ratiu's three `compatObjectFormat` patches, raising a concern that copying unknown commit headers unchanged also accepts malformed lines, and requesting casing and style fixes.
