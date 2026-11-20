# Git mailing list daily digest for 2026/10/30

## Long threads

### fetch refmap and @{push} resolution for limited-fetch remotes

*Long thread: 66 replies so far from 5 participants over 42 days; 2 messages that day.*

Harald Nordgren posted v13 of the series that avoids fetching every branch of a new remote in a shallow repository. The new sixth commit gives `tracking_for_push_dest()` the same fallback to `remote.<name>.fetchRefmap` that `get_ref_map()` uses on the fetch side, so `@{push}` resolves correctly for a `--limited-fetch` remote, and the first commit adds a regression test confirming that an explicit-destination `remote.<name>.fetch` entry is stored both at that destination and through the refmap. Earlier, Phillip Wood and Junio C Hamano had debated how `remote.<name>.fetch` and the refmap should interact when both are configured.

### branch: --delete-integrated for squash- and rebase-merged branches

*Long thread: 51 replies so far from 6 participants over 32 days; 3 messages that day.*

Harald Nordgren posted v5 of the series adding `git branch --delete-integrated <pattern>`, which deletes branches whose changes landed upstream via squash or rebase merges and names the landing commit in the output. V5 fixes the two issues Jasper Mayone had reported: `git branch -h` now lists `--delete-integrated`, and combining it with `--delete-merged` dies with a short "cannot be used together" message. Junio C Hamano reviewed patches 3/4 and 4/4, suggesting a variable rename and asking whether ignoring gitattributes in `merge_keeps_upstream()` is intentional.

### doc: new Git tutorial for beginners

*Long thread: 38 replies so far from 6 participants over 25 days; 6 messages that day.*

Julia Evans posted v5 of the series replacing `gittutorial` with a new beginner tutorial; the only changes are replacing an ambiguous "it" with "GCM" in the commit message and removing trailing whitespace. She explained she left other references to the old tutorial in place to avoid merge conflicts. Junio C Hamano squashed fixups into the series commenting out `gittutorial` in `Documentation/Makefile` and `command-list.txt`, then supplied a fuller fixup removing references from `git.adoc`, `gitcore-tutorial.adoc`, and several other documents, suggesting the two patches could eventually be squashed into one.

### Documentation: community voting process

*Long thread: 29 replies so far from 13 participants over 25 days; 8 messages that day.*

Discussion of Emily Shaffer's proposed voting-process document continued. Toon Claes argued eligible voters should be able to abstain and that voting should be reserved for spread opinions. Emily Shaffer summarized reroll changes--subject line, secret-vote requirement, votes referencing a proposal's commit ID--and listed open questions on the platform (leaning toward CIVS) and voter roll. Junio C Hamano asked to drop the word "code", tally non-patch contributions, and use Message-ID instead of commit ID; Jeff King proposed a human-input exception for extra voters; Jonathan Corbet reported using CIVS for TAB elections.

### loose: fix migrating "loose-object-idx" with batch fsync

*Long thread: 22 replies so far from 4 participants over 9 days; 9 messages that day.*

Justin Tobler's v3 fixes ODB transactions failing on commit when a compatibility hash is configured, by merging the temporary directory's loose-object-idx mapping into the main one and writing it atomically. Patrick Steinhardt reviewed v3 and said he is happy with it. Karthik Nayak suggested renaming `insert_loose_map()` and `loose_object_idx_merge()` for consistency and adjusting an error message; Justin Tobler declined the renames as not worth a reroll and plans to leave the series as-is.

## The day in brief
Harald Nordgren advanced two long-running series, posting v13 that resolves `@{push}` through `remote.<name>.fetchRefmap` and v5 of `git branch --delete-integrated`. Julia Evans's replacement tutorial reached v5, and Justin Tobler's loose-object-idx migration fix drew a positive review from Patrick Steinhardt.

## Notable threads

### loose object size checks and LLP64 size truncations
Elia Pinto posted a six-patch series fixing uninitialized-memory exposure from short loose-object zlib streams, missing size checks in the streaming reader, and 32-bit `long` truncations on Windows affecting `git cat-file --batch`, notes concatenation, and diffing large text files; the loose-object reading path now uses `size_t` throughout.

### builtin/grep: read submodule trees via their repository

Patrick Steinhardt posted a three-patch series fixing `git grep --recurse-submodules` failing with "unable to read tree" on nested submodule trees; only the first patch fixes the bug by reading trees via the submodule's repository, while the other two are belt-and-suspenders for sparse directory entries and `.gitmodules`. Toon Claes approved all three.

### notes: do not drop notes written by a concurrent process
Farid Zakaria's patch makes concurrent `git notes add` fail with "cannot lock ref" instead of silently dropping the other process's note. Kristoffer Haugsbakk argued against a three-way merge resolution for such races, and Farid Zakaria stated the thread should count as his GenAI disclosure.

## In brief
- Miklos Vajna posted v4 speeding up `log --follow` for non-linear history, splitting the `-m --follow` fix into its own patch and adding a fix for history lost when one merge side renamed a path the other side edited.
- Jasper Mayone posted a series making `git worktree remove` refuse a worktree mid-rebase or bisect without `--force`, and making `%(worktreepath)` name such worktrees; Junio C Hamano asked that it be labeled v3.
- Johannes Schindelin posted `Documentation/gitsecurityboundary.adoc`, defining Git's security boundary around unexpected execution of attacker-controlled code.
- Elia Pinto posted v2 widening object-write, word-diff, and convert.c buffer sizes to `size_t` for 64-bit Windows; Patrick Steinhardt reviewed and asked for a human touch in one commit message.
- Kaartic Sivaraam reviewed v3 of Christian Couder's `early_scan_options()` series for parse-options, leaving minor comments.
