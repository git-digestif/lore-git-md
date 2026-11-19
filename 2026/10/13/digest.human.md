# Git mailing list daily digest for 2026/10/13

## Long threads

### New `gitmergeconflicts(7)` manual page linked from merge commands

*Long thread: 61 replies so far from 6 participants over 20 days; 2 messages that day.*

Julia Evans's six-patch documentation series adds a new `gitmergeconflicts(7)` man page explaining how to handle merge conflicts, and links to it from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` docs instead of repeating the guidance. Earlier rounds reworked the page's examples and switched `Reviewed-by` trailers to `Helped-by`. Today Patrick Steinhardt reviewed the page and said he is happy with it aside from a handful of nits, including inconsistent bullet punctuation and spelling out `-p` as `--patch`; he also noted that the commit message for the `git-cherry-pick` patch is inaccurate because it claims to remove the merge-conflict discussion when the patch actually replaces it with a link to the new guide.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control

*Long thread: 37 replies so far from 3 participants over 96 days; 1 message that day.*

Christian Couder's five-patch series introduces the protected `uploadpack.lazyFetchTrusted` configuration variable so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones and fetches, with refactoring in `promisor-remote.c`, `setup.c`, and `builtin/upload-pack.c` plus tests. Earlier revisions added recursion protection and documentation fixes. Today Junio C Hamano marked the topic ready for `next`.

### `rerere` waiting for `MERGE_RR.lock`

*Long thread: 33 replies so far from 4 participants over 42 days; 4 messages that day.*

Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` so a rebase hitting a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` option to `git rerere gc` that auto-maintenance passes. Today Bachem posted v8 incorporating earlier review, Patrick Steinhardt retracted his suggestion of a separate `rerere_gc_flags` enum and called v8 "in a good enough state by now", and Junio C Hamano asked whether the topic should now be marked ready for `next`.

### `git branch --delete-merged` detecting squash-merged branches

*Long thread: 30 replies so far from 5 participants over 15 days; 1 message that day.*

Harald Nordgren's series teaches `git branch --delete-merged` to treat a branch as merged when some upstream commit contains all of its changes, naming that commit as "landed as", and adds `--no-squashed` to skip the slower search. Earlier discussion covered detection tradeoffs and partial clones. Today Junio C Hamano said he was no longer sure of his earlier suggestion, arguing `--delete-merged` should remain a strict check that deletes only branches whose commits were merged upstream, and proposed a separate `--delete-irrelevant` for branches made irrelevant by rebased, cherry-picked, or squash-merged work.

### CI annotations naming the failing test script

*Long thread: 25 replies so far from 4 participants over 19 days; 1 message that day.*

Harald Nordgren's series links failure and leak annotations in CI output to the test script, showing the file and line as plain text in the annotation. Earlier revisions reverted direct file/line linking after it failed for files unchanged in a PR. Today Junio C Hamano marked the topic ready for `next`.

## The day in brief
Three topics were marked ready for `next` today: `uploadpack.lazyFetchTrusted`, `history: sign rewritten commits`, and the CI annotation series. Junio C Hamano also proposed splitting squash-merge detection out of `git branch --delete-merged` into a separate `--delete-irrelevant` option.

## Notable threads

### Signing commits created by `git history` rewrites
This two-patch series adds a signing-key option to the replay API and exposes `-S`/`--gpg-sign` and `--no-gpg-sign` on `history drop`, `fixup`, `reword`, `split`, and `squash`, so rewritten commits honor `commit.gpgSign` like rebase and cherry-pick do. Today Junio C Hamano marked the topic ready for `next`, explaining that it had already received a "looks good enough" at v3 that he had missed.

### `git blame` using `.git-blame-ignore-revs` by default

*Long thread: 23 replies so far from 4 participants over 33 days; 6 messages that day.*

Ravi Mistry's series makes `git blame` and `git annotate` automatically read `.git-blame-ignore-revs` from the working-tree root (or `HEAD:.git-blame-ignore-revs` in bare repositories) without per-clone configuration. Today Mistry posted v5 reading the working-tree copy in non-bare repositories and v6 fixing Junio C Hamano's review points; Hamano approved the v6 design as "Very cleanly done" and said he would replace the previous version.

### `checkout -m` recreating original conflict labels

*Long thread: 22 replies so far from 3 participants over 14 days; 3 messages that day.*

Phillip Wood's three-patch series teaches the merge-ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` can reuse the original merge's labels instead of "base"/"ours"/"theirs". Today Wood posted v4, adding a patch that caches merge config settings and labels once per invocation; Junio C Hamano reviewed it and said he will replace the previous version.

## In brief
- Curtis Smith posted v5 of a fix so `ie_modified()` in `read-cache` does not trust a size change for paths subject to CRLF conversion; Junio C Hamano corrected the commit message and warned the new tests may hit racy-git paths. *Long thread: 10 replies so far from 7 participants over 276 days; 10 messages that day.*
- Erik Cervin-Edin reported that the Linux fsmonitor daemon deterministically misses edits in subdirectories of a renamed directory because `add_watch()` in `compat/fsmonitor/fsm-listen-linux.c` returns early on EEXIST.
- SZEDER Gábor reproduced a `git log --follow` performance regression from commit 304812ed33 (2.69x slower) and traced most of the cost to `propagate_follow_pathspec_to_parent()` on merge commits' later parents.
- Rong Bao fixed `git fetch --all --depth=` and `--all --unshallow` not deepening shallow clones by passing `--depth` to the fetcher subprocess in `builtin/fetch.c`.
- Jun Wu posted a series adding `core.indexLockTimeout` so commands like `add` and `commit` wait for the index lock; D. Ben Knoble and Junio C Hamano questioned the trade-offs.
- Patrick Steinhardt posted five `odb/files` cleanup patches for alternates performance, all reviewed positively by Justin Tobler.
