# Git mailing list daily digest for 2026/10/29

## Long threads

### fetch: `remote.<name>.fetchRefmap` for shallow-repo branch selection

*Long thread (root 2026/09/19/14-47-26): 65 replies so far from 5 participants over 41 days; 5 messages that day.*

The series avoids fetching every branch of a new remote in a shallow repo by adding a `remote.<name>.fetchRefmap` config (renamed from `remote.<name>.refmap` in v12) that remaps where fetched refs are stored while `remote.<name>.fetch` still decides what is fetched. Today Phillip Wood pressed on the interaction between the two: he was surprised a command-line refspec with a colon does not override the refmap, called it problematic that `fetchRefmap` cannot be overridden without `--refmap`, and laid out two interpretations -- rejecting colons in `fetch`, or treating `fetch` values as command-line refspecs -- leaning toward the latter, under which an explicit destination must update only that destination. Junio C Hamano explained the command-line refmap was meant mainly for "what to fetch" without a colon, and said he had not planned `--refmap` support on the push side. Ben Knoble agreed a future `pushRefmap` on top of `fetchRefmap` would ease his work with notes, and Harald Nordgren noted the rename to `fetchRefmap` had already landed in v12.

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread (root 2026/09/29/07-30-30): 49 replies so far from 6 participants over 31 days; 2 messages that day.*

The v4 series restores `--delete-merged` to deleting only branches whose tips are reachable from their upstream, and adds `--delete-integrated <pattern>`, which deletes branches whose changes landed upstream via squash or rebase merges -- comparing object ids of changed files, and merging the branch's version into upstream's to catch squashes onto files upstream also changed -- naming the landing commit as "landed as". Today Jasper Mayone tested v4 against master: t3200 passes, `--delete-merged` now matches master on every case including the never-merged false positive, and `--delete-integrated` deletes squash merges, GitHub-style squashes, rebase merges, renames, a reverted squash, and a squash after upstream changed the same file, while keeping partially landed and never-merged branches. He reported that `git branch -h` omits `--delete-integrated` from the usage lines, that combining both options dies with full usage text, and that the cover letter still says `--squashed`; Harald Nordgren acknowledged the usage-line omission and said he will fix the combined-options error message.

### repack corner cases for cruft-less MIDXs

*Long thread (root 2026/09/30/01-28-34): 30 replies so far from 5 participants over 30 days; 1 message that day.*

The series fixes cases where geometric repacks with `repack.midxMustContainCruft=false` can omit packs needed for bitmap generation, adding tree and tag roots, retaining cruft for ordinary incremental repacks, and including required packs in ordinary and incremental MIDX writes. Earlier, Taylor Blau said the series should be ready to go as-is; today Junio C Hamano remarked that his earlier question about the topic's viability was not meant to be answered by the series author.

### fetch: advertise fetched refs when backfilling tags

*Long thread (root 2026/10/08/22-10-37): 24 replies so far from 4 participants over 22 days; 4 messages that day.*

The series fixes a regression where, in shallow repositories, refs fetched before tag backfill could not be used as "haves" during negotiation, so the server re-sent objects the client already had; v4 onward appends the already-fetched tips to the transport's `negotiation_include_tips` instead of committing the first transaction. Today Karthik Nayak posted v6, reordering the upstream-info test commit to the front, fixing grammar, and updating the commit message to note the new tests catch the remote-helper leak fixed in the second patch; Patrick Steinhardt reviewed v6 and said he is happy with the changes.

### loose: merge `loose-object-idx` when migrating tmp-objdir with batch fsync

*Long thread (root 2026/10/22/23-40-07): 13 replies so far from 3 participants over 8 days; 11 messages that day.*

The series fixes ODB transaction commit failures when a compatibility hash is configured, because tmp-objdir migration did not handle the loose object index; it adds `loose_object_idx_merge()` to merge mapping entries and atomically write a new index. Today Justin Tobler posted v2, making the merged write atomic, adding a fixed lock timeout, and dropping duplicate entries; Patrick Steinhardt reviewed the four patches with questions about the optional cache, the file-descriptor-based writer, and lock scope, and Justin agreed to read the source mapping before acquiring the lock and add a comment. Junio C Hamano reported that the topic in `seen` has a funny interaction -- `repo_write_loose_object_map()` loses all callers while the new helper remains -- and Justin confirmed the resolution is correct but noted a leftover declaration in `loose.h`, then posted v3 with the source read before locking, a new comment, and clearer commit messages.

## The day in brief

Karthik Nayak's v6 of the shallow-fetch tag-backfill fix drew a happy review from Patrick Steinhardt, while Pia Park's v3 of the empty-MIDX-write fix was approved by Taylor Blau, with Junio C Hamano saying he will queue it. Jasper Mayone's testing confirmed that v4 of `--delete-integrated` works correctly against master, and a new bug report now traced `git grep --recurse-submodules <tree>` failures on submodule directories to commit 93cd344e34 in builtin/grep.c.

## Notable threads

### `git grep --recurse-submodules <tree>` dies on a directory inside a submodule
Root reported that since Git 2.56.0, `git grep --recurse-submodules <pattern> <tree>` dies with "fatal: unable to read tree" when it reaches a directory inside a submodule, while the working-tree form and submodules without directories are unaffected. He identified commit 93cd344e34 (builtin/grep: stop registering submodule ODB as source) as the cause: `grep_tree()` reads subdirectory trees from the superproject's object database while `grep_submodule()` switches to the submodule, so the lookup fails. Reading from `repo->objects` instead at builtin/grep.c:656 makes the reproduction print the expected output, and `grep_cache()` performs the same lookup at line 563 for sparse-directory index entries.

### notes: do not drop notes written by a concurrent process
Farid Zakaria posted a patch fixing silent note loss when two notes-modifying commands run concurrently: the command now remembers the commit the notes were read from and uses it as the parent of the new notes commit and the expected old value of the ref update, so a concurrent update fails with "cannot lock ref" instead of dropping notes, with a new test in t/t3301-notes.sh. Kristoffer Haugsbakk called the find "really nice" and judged the change a pure improvement, proposing an ideal-world retry where a losing writer retries if the winner did not touch its commit; Junio C Hamano offered an alternative where the loser merges or rebases its note commit on the winner's. Farid said the retry proposals make sense but he may leave them to others, and offered to post two other bugs found with the same tool.

### midx-write: skip writes with no object entries
Pia Park posted v3 of the patch that stops MIDX writes from publishing a zero-object MIDX without a reverse index when packs are empty or an incremental write finds only already-indexed objects. The v3 simplifies tests, renames the initial-layer test, and updates the commit message; the implementation is unchanged from v2, and all 885 tests across the eight MIDX, bitmap, and repack scripts pass. Taylor Blau reviewed v3 and said it looks good, and Junio C Hamano said he will queue the series.

### odb/files: reload alternates on retry again
The series restores reloading alternates when an object cannot be found -- a behavior lost in 5946a564cd -- with follow-up patches recovering most of the performance cost of looking up missing objects. Today Patrick Steinhardt agreed the series should be ready to go, responding to Junio C Hamano's suggestion that the topic could be marked ready for `next`.

### `%(worktreepath)` in `git for-each-ref` for rebasing/bisecting branches
Jasper Mayone withdrew the proposed `git worktree list --branch` option and instead proposed making `%(worktreepath)` in `git for-each-ref` use the `branch_checked_out()` information already in branch.c, so it names the worktree of a branch being rebased or bisected, which is currently empty; he asked whether that behavior change is acceptable as a fix. Eric Sunshine clarified that `git worktree list` already guarantees the main worktree is first, a point that only matters once filtering is engaged. Phillip Wood called the change a useful improvement and agreed it can be considered a bug fix, while noting differences from a `--branch` option; Ben Knoble questioned whether `git worktree remove` should remove a worktree mid-rebase, and Phillip agreed `remove` should not, calling that a bug more likely to be triggered if Git starts reporting such worktrees.

## In brief
- Phillip Wood noted that where threads are created in a loop such as `preload_index()`, it would be more efficient to block signals before the loop and unblock after, and clarified that in a multi-threaded process a signal is only delayed if blocked in all threads.
- Junio C Hamano suggested moving the `PUSH_DEFAULT_SIMPLE` case of `branch_get_push_1` into a separate function for readability, agreeing the triangular-workflow `@{push}` result is sensible; Harald Nordgren confirmed both cases resolve to the same branch and agreed with the refactor.
- Justin Tobler analyzed the quarantine alternate-refs issue -- the real ODB passed as an alternate causes the main repository's refs, including hidden ones, to be considered a second time -- and suggested the connectivity check need not use the quarantine as primary object directory; Junio C Hamano asked whether a `GIT_QUARANTINE_OBJECT_DIRECTORY` variable could let children identify the real ODB.
- Mark C. Chu-Carroll posted v4 of degree-of-concern reporting for `git repo structure`; Junio C Hamano found a copy-and-paste error computing the largest tree's concern level from the commit size, which Mark acknowledged and will fix.
- Tanga Joris posted v3 dropping `NO_PARSEOPT` from patch-id, diff-pairs, fast-import, and var, moving the completion-helper handling into `show_usage_with_options_if_asked()` in parse-options.c as Junio C Hamano suggested.
- Harald Nordgren posted v2 of the Makefile patch letting ccache cache objects built with the precompiled header, moving the `CCACHE_SLOPPINESS` export into the `ifndef NO_PRECOMPILED_HEADER` block.
- Patrick Steinhardt gave a positive review of Sami Jawhar's oss-fuzz commit-graph fuzzer fix setting a placeholder gitdir, noting `repo_clear()` already frees it, and suggested restoring a fuzzer smoke test.
