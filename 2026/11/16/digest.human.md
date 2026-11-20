# Git mailing list daily digest for 2026/11/16

## Long threads

### `git repo info` path keys (GSoC series v9)

*Long thread: 64 replies so far from 6 participants over 124 days; 1 message that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.workingtree.*`, `path.superprojectworkingtree.*`, `path.index.*` and `path.prefix` -- and refactors `get_superproject_working_tree()` to take a `struct repository *`. Earlier revisions renamed the keys and dropped `path.hooks` and `path.cdup` after review. Today K Jayatheerth said the easiest approach is to combine and optimize the per-test repository setup in the planned follow-up series consolidating `repo` and `rev-parse` logic.

### `git branch --delete-integrated` for squash-merged branches

*Long thread: 58 replies so far from 7 participants over 49 days; 1 message that day.*

Harald Nordgren's series adds `--delete-integrated` to `git branch`, deleting a branch when an upstream commit contains all its changes -- covering GitHub squash and rebase merges -- and naming the landing commit. v7 added a test that the option keeps branches whose changes did not land and reads the index once before the walk. Today Jasper Mayone confirmed the v7 test covers the three cases he had proposed, ran t3200 with his extra tests (all 209 pass), and added a Tested-by trailer.

### New detached HEAD guide

*Long thread: 46 replies so far from 5 participants over 41 days; 2 messages that day.*

Julia Evans's patch adds a `git help detached-head` guide, dual-purpose with the DETACHED HEAD section of `git-checkout.adoc`, explaining what detached HEAD state is and why users land there by accident. Today she posted v7 adding the sentence about making experimental commits without being on a branch and a clearer commit message; Junio C Hamano asked for a fresh-eyes final pass before merging it down to 'next'.

### `credentialStore.lockTimeout` config name

*Long thread: 43 replies so far from 5 participants over 76 days; 2 messages that day.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for credential lock retries, matching other millisecond file-lock settings that omit the unit, with `lockTimeoutMS` kept as a deprecated synonym. Today she resent v3 with that rationale in the message; Junio C Hamano reviewed it, said it looks good, and marked the topic ready for 'next'.

### Worktrees rebasing or bisecting a branch

*Long thread: 41 replies so far from 8 participants over 27 days; 3 messages that day.*

Jasper Mayone's series makes `git worktree remove` refuse a worktree mid-operation, documents reading `%(worktreepath)` safely, and names the worktree rebasing or bisecting a branch. Today he answered Junio C Hamano's question that the new `check_no_operation_in_progress()` and the existing checked-out-branch check answer different questions and should stay separate, with a shared helper proposed as a separate series. He also accepted Junio's objection and agreed to drop patches 4/5, stopping the series at 3/5; Phillip Wood separately suggested renaming `%(currentworktree)` to `%(currentworktreepath)`.

## The day in brief
Three topics were marked ready for 'next' today: the `credentialStore.lockTimeout` rename, optional `--follow` rename tracking in non-linear history, and the `git repo structure` object-counting speedup. Jasper Mayone added a Tested-by to the `--delete-integrated` series and agreed to trim the worktree series to three patches.

## Notable threads

### `git rerere` subcommand conversion

*Long thread: 32 replies so far from 6 participants over 21 days; 3 messages that day.*

Patrick Steinhardt's series converts `builtin/rerere` to proper subcommands using `OPT_SUBCOMMAND()`. Today he confirmed the conversion makes option placement backwards-incompatible -- `git rerere --skip-locked gc` must become `git rerere gc --skip-locked` -- and Junio C Hamano accepted breaking `--skip-locked` placement, telling users to fix their scripts.

### Optional `--follow` rename tracking in non-linear history
Miklos Vajna's patch makes `--follow` rename tracking in non-linear history optional via `--follow=merges`, addressing a performance regression. Today Patrick Steinhardt judged v2 "looks good to me", and Junio C Hamano marked the topic ready for 'next'.

### Faster object counting in `git repo structure`

*Long thread: 9 replies so far from 4 participants over 4 days; 8 messages that day.*

Patrick Steinhardt's two-patch series makes `builtin/repo` parse commits only as needed and avoid fully parsing trees when counting objects. Today he posted v2 with a reworded comment; Kristofer Karlsson said it addresses his only nit, Derrick Stolee called the memory reduction from 3.4 GiB to 0.7 GiB impressive, and Junio C Hamano marked the topic ready for 'next'.

## In brief
- Google - Mergify reported that since Git 2.54.0 a cherry-pick in a blob:none partial clone lazy-fetches a blob it just wrote; Patrick Steinhardt pointed to a known fix in flight and acknowledged verification that it resolves the case.
- לירון עדן reported that `index-pack` in a blobless partial clone starts nested lazy fetches for REF_DELTA bases already in the pack being indexed; prefetching the commit with `GIT_NO_LAZY_FETCH=1` avoids it.
- Andreas Wendleder reported that with `submodule.recurse=true`, switching branches in a superproject worktree whose submodule is a linked worktree aborts with BUG() in `validate_submodule_legacy_git_dir()`.
- Jelle van der Waa will respin the `git history --verbose` RFC with verbose on by default, following `commit.verbose`, for split, reword and fixup.
- **Root refs vs pseudorefs**: Patrick Steinhardt agreed Andy Koppe's patch series looked sensible and ready from his point of view. *Long thread: 31 replies so far from 4 participants over 26 days; 1 message that day.*

- Patrick Steinhardt and Junio C Hamano discussed the `ls-remote` promisor-remote crash fix, preferring to pass the repository down and check `!repo` while accepting the easier NULL-`gitdir` check for now.
