# Git mailing list daily digest for 2026/10/29

## Long threads

### fetch refmap configuration for shallow repos

*Long thread: 65 replies so far from 5 participants over 41 days; 5 messages that day.*

The series "fetch: avoid fetching every branch of a new remote in a shallow repo" adds a `remote.<name>.fetchRefmap` config (renamed from `remote.<name>.refmap` in v12) that remaps fetched refs while `remote.<name>.fetch` still decides what is fetched. Today Phillip Wood said he was surprised that a colon-bearing command-line refspec does not override the refmap, called it problematic that `fetchRefmap` cannot be overridden without `--refmap`, and leaned toward treating `fetch` values as command-line refspecs so an explicit destination updates only itself. Junio C Hamano explained that the command-line refmap was meant mainly for specifying "what to fetch" without a colon, and said he had not planned `--refmap` support on the push side. Ben Knoble agreed to a future `pushRefmap` on top of `fetchRefmap`, saying it would ease his work with notes.

### branch --delete-integrated for squash- and rebase-merged branches

*Long thread: 49 replies so far from 6 participants over 31 days; 2 messages that day.*

Harald Nordgren's v4 keeps `git branch --delete-merged` deleting only branches whose tips are reachable from their upstream, and adds `--delete-integrated <pattern>`, which also deletes branches that were rebase- or squash-merged upstream, naming the landing commit as "landed as ...". Today Jasper Mayone reported testing v4 against master: `--delete-merged` now matches master on every case, and `--delete-integrated` deletes squash merges, GitHub-style squashes, rebase merges, renames, a reverted squash, and a squash that landed after upstream changed another line of the same file, while keeping partly landed, differently resolved, and never-merged branches. He also flagged that `git branch -h` omits the new option from the usage lines and that combining both options dies with the full usage text; Harald Nordgren acknowledged both and said he will fix the error message.

### repack corner cases for cruft-less MIDXs

*Long thread: 30 replies so far from 5 participants over 30 days; 1 message that day.*

Taylor Blau's series fixes cases where geometric repacks with `repack.midxMustContainCruft=false` can omit packs needed for bitmap generation, adding tree and tag roots and including required packs in ordinary and incremental MIDX writes. Earlier he stated the topic should be ready to go as-is; today Junio C Hamano remarked that his earlier question about the topic's viability was not meant to be answered by the series author.

### fetch: advertise fetched refs when backfilling tags

*Long thread: 24 replies so far from 4 participants over 22 days; 4 messages that day.*

Karthik Nayak's series fixes a regression in which shallow-repository fetches could resend objects during tag backfill because the already-fetched refs were not advertised as haves; the fix appends the fetched tips to the transport's `negotiation_include_tips` instead of committing the first transaction. Today Karthik Nayak posted v6, reordering the upstream-info test commit to the front and updating commit messages, and Patrick Steinhardt reviewed it and stated he is happy with the changes.

### loose-object-idx migration with batch fsync

*Long thread: 13 replies so far from 3 participants over 8 days; 11 messages that day.*

Justin Tobler's series fixes ODB transaction commit failures when a compatibility hash is configured, because tmp-objdir migration did not merge the "loose-object-idx" mapping file. Today he posted v3, reading the source mapping file before acquiring the lock to shrink the critical section and adding a comment to `loose_object_idx_merge()`. Junio C Hamano noted that merging to `seen` drops `repo_write_loose_object_map()` while the new `loose_object_map_write()` helper remains; Justin Tobler confirmed that and observed the old declaration is still left in `loose.h`.

## The day in brief

Karthik Nayak's v6 of the shallow-fetch tag-backfill fix drew a happy review from Patrick Steinhardt, and Pia Park's MIDX empty-write fix will be queued by Junio C Hamano after Taylor Blau's approval. A new bug report traced `git grep --recurse-submodules <tree>` failures on submodule directories to commit 93cd344e34, and Farid Zakaria's notes fix turned silent note loss into a loud "cannot lock ref" failure.

## Notable threads

### midx-write: skip writes with no object entries
Pia Park's patch stops `git multi-pack-index write` from publishing a zero-object MIDX without a reverse index when a write finds packs but no new objects, a failure mode that appeared when incremental bitmap writes began loading reverse indexes from prior layers. Today she posted v3 with simplified tests and an updated commit message; Taylor Blau said it looks good, and Junio C Hamano said he will queue the series.

### git grep --recurse-submodules <tree> dies on a directory inside a submodule
Root reported that since Git 2.56.0, `git grep --recurse-submodules <pattern> <tree>` dies with "fatal: unable to read tree" when it reaches a directory inside a submodule, while the working-tree form is unaffected. He identified commit 93cd344e34 (builtin/grep: stop registering submodule ODB as source) as the cause: `grep_tree()` reads subdirectory trees from the superproject's object database while `grep_submodule()` switches to the submodule, and noted `grep_cache()` performs the same lookup for sparse-directory index entries.

### notes: do not drop notes written by a concurrent process
Farid Zakaria's patch makes commands that modify notes fail with "cannot lock ref" instead of silently dropping notes written by a concurrent process, by using the commit the notes were read from as both the parent of the new notes commit and the expected old value of the ref update. Kristoffer Haugsbakk called the find "really nice" and judged the change a pure improvement, proposing an ideal-world retry; Junio C Hamano offered a merge-or-rebase alternative; Farid Zakaria clarified he agreed the patch is a positive change as it stands.

## In brief
- Jasper Mayone withdrew the `git worktree list --branch` option and instead proposed making `%(worktreepath)` in `git for-each-ref` name the worktree of a branch being rebased or bisected; Phillip Wood called it a useful improvement, and Ben Knoble questioned whether `git worktree remove` should then refuse such worktrees.
- Mark C. Chu-Carroll posted v4 of degree-of-concern reporting for `git repo structure`; Junio C Hamano spotted a copy-and-paste error computing the largest tree's concern level from the commit size, which Mark acknowledged and will fix.
- Tanga Joris posted v3 dropping `NO_PARSEOPT` from patch-id, diff-pairs, fast-import and var, moving completion-helper handling into `show_usage_with_options_if_asked()` in parse-options.c as Junio C Hamano suggested.
- Patrick Steinhardt agreed that the "odb/files: reload alternates on retry" series should be ready to go.
- Harald Nordgren posted v2 of the Makefile change exporting `CCACHE_SLOPPINESS += pch_defines,time_macros` so ccache caches objects built with the precompiled header.
- Justin Tobler analyzed the quarantine alternate-refs issue as child processes having no concept of an ODB transaction; Junio C Hamano asked whether a `GIT_QUARANTINE_OBJECT_DIRECTORY` variable could identify the real object directory.
- ian4hu plans to block SIGINT, SIGHUP, SIGTERM and SIGQUIT around spawn loops in a replacement sigchain series, leaving SIGPIPE out, and will wait for Peff and Junio to comment before sending v2.
