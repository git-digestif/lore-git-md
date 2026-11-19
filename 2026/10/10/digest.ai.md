# Git mailing list daily digest for 2026/10/10

## Long threads

### New `gitmergeconflicts(7)` manual page and cross-links from merge commands

*Long thread (root 2026/09/24/14-44-15): 54 replies so far from 6 participants over 17 days; 1 message that day.*

This documentation series by Julia Evans adds a new `gitmergeconflicts` man page explaining how to navigate merge conflicts with examples, and links to it from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of reexplaining the process in each command's page. Earlier rounds revised the "ours"/"theirs" section, debated whether `git status` should recommend `git merge --continue` over `git commit` during a merge, and resolved a `Reviewed-by` versus `Helped-by` trailer question. Today Junio C Hamano stated two style preferences for the technical manual: list all merge-conflict-causing commands explicitly (including `git am -3` and `git checkout -m`) rather than ending with "and other operations", with a note where a description is incomplete, and use formal constructions ("here is", "there is") over contractions ("here's", "there's") consistently throughout, since the document is not a personal diary.

### Fetch behavior for new remotes in shallow repositories

*Long thread (root 2026/09/19/14-47-26): 31 replies so far from 4 participants over 22 days; 4 messages that day.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repo, adding `git remote add --limited-fetch` and defaulting to limited fetch in shallow repositories. Earlier revisions fixed a fatal `--refmap` error when only `remote.<name>.refmap` was configured and clarified that `remote.<name>.fetch` wins when both are set. Today Harald Nordgren posted v8 of the now five-patch series, splitting out a preliminary `collect_upstream_from_remote()` helper in `remote.c`, fixing a regression that could drop the current branch's `@{upstream}` when `remote.<name>.fetch` was narrowed, and threading `struct repository *` through instead of hardcoding `the_repository`. Junio C Hamano reviewed patch 2/5, objecting that duplicating the current-branch upstream logic into the `remote.*.fetch` branch belongs in patch 3/5 rather than this refactoring, and asked whether dropping the original NULL-branch guard in `collect_upstream_from_remote()` was intended; he called patch 4/5 (`git remote add --limited-fetch`) straightforward and nicely done. Harald Nordgren replied that the commit-story point was good and he would look into it, and that the missing guard was overlooked but he does not think the segfault could be reproduced by a test.

### Rerere waiting for `MERGE_RR.lock` instead of dying

*Long thread (root 2026/09/02/08-31-37): 27 replies so far from 4 participants over 39 days; 3 messages that day.*

Thomas Bachem's series addresses a rebase dying at a conflict when a background `git rerere gc` holds `MERGE_RR.lock`. Earlier rounds renamed the auto-maintenance option to a hidden `--skip-locked` and debated whether commands stopping at a conflict should go on without rerere. Today Thomas Bachem posted v7 as a two-patch series: patch 1/2 makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for `MERGE_RR.lock` before failing, and patch 2/2 adds the hidden `git rerere gc --skip-locked` option, implemented via a new `enum rerere_gc_flags` with `RERERE_GC_NOWAIT`, which auto-maintenance passes so a background gc does nothing while the lock is held. The third patch that would have let conflict-stopping commands go on without rerere was dropped in v7.

### Routing reference copy and rename through transactions

*Long thread (root 2026/09/19/13-33-44): 22 replies so far from 6 participants over 22 days; 2 messages that day.*

Maciej Ciemborowicz's four-patch series makes reference copy and rename go through ordinary transactions so the reference-transaction hook can observe and reject the complete operation; previously the files backend saw only the source deletion and reftable saw neither endpoint. Earlier rounds fixed a v3 regression in t0600/t5510 caused by a packed-refs preparation error being overwritten, and drew discussion about AI-assisted patch authorship and review engagement. Today Maciej Ciemborowicz offered to let someone else write the patches if that would be more efficient than reviewing his, saying his main goal is to get the issue resolved. brian m. carlsson advised that it is better not to submit a patch at all if one cannot submit something of reasonably good quality or satisfy the SubmittingPatches guidelines, noted that reporting bugs without a patch is valuable, and said the reported problem may be picked up by someone familiar with the code--possibly himself, though with no promises--since it also relates to the expensive `git remote rename` issue he had reported.

## The day in brief

Harald Nordgren posted v8 of the shallow-repo fetch series with a `collect_upstream_from_remote()` refactoring, drawing review comments from Junio C Hamano on where the duplicated upstream logic belongs. Thomas Bachem posted v7 of the rerere lock-wait series, now two patches with a hidden `gc --skip-locked` for auto maintenance. Siddharth Shrimali posted v2 fixing `git repack --dry-run` so it no longer rebuilds packs or deletes anything, with D. Ben Knoble offering his Reviewed-by.

## Notable threads

### `git repack --dry-run` no longer rebuilds packs
Siddharth Shrimali's patch fixes `git repack --drop-filtered --dry-run`, which was documented to list droppable objects without rebuilding any pack or deleting anything but actually did both because the `--dry-run` block fell through into the regular repack code. Today he posted v2, which returns right after listing the candidates--skipping the repack and any deletion even with an explicit `-d`--and adds a test that the pack directory is unchanged with and without `-d`; the log message documents that the bug has existed since `--dry-run` was introduced in 1e746b00aa. D. Ben Knoble reviewed v2, said he was happy with it, and offered his Reviewed-by.

### `fast-import` crash when `from` references a commit from the same session
Marc Becker submitted a patch to `builtin/fast-import.c` fixing a crash when the `from` command references a commit created during the same import session, since new objects may not yet be reachable via the ODB layer. The fix resolves the requested tag or commit through the object cache first, dereferencing tags, before falling back to `odb_read_object_peeled()`.

### wincred credential blob processing refactor
Marc Becker posted v2 of a refactor of the Windows credential helper's blob dissection, splitting it into separate `write_credential` and `write_credential_item` methods, using `wmemchr` to detect wide-character newlines in immutable blob data, and consistently treating sizes as wide-character counts. This fixes deterministic line splitting, CRLF bleed-over into consecutive items, and a bytes-versus-characters size check for `wcsncpy_s` that had already led to memory corruption.

### Revision filtering for `git repo structure`
Mark C. Chu-Carroll posted v4 of a patch adding a `[<revs>...]` argument to `git repo structure` so the set of objects processed can be limited by revision expressions instead of always summarizing all branches, remotes, tags, stashes, and notes--useful for diagnosing performance problems in a specific part of a repo. The v4 changelog notes removal of inadvertent edits after an earlier wrong version was sent; the implementation notes the trade-off that `setup_revisions` performs an initial ref-walk requiring a second ref-walk in `count_references` when revs are specified.

### `checkout -m` recreating original merge conflict labels
This series teaches the ort machinery to write conflict labels to `.git/MERGE_LABELS` when switching to a merge result containing conflicts, so `git checkout -m <path>` can reuse the original labels instead of "base"/"ours"/"theirs". Today Junio C Hamano noted that `checkout_merged()` is called once per conflicted path, causing `merge.renormalize` and the `MERGE_LABELS` file to be read repeatedly during a single conflicted `checkout -m`, and suggested a preliminary patch reading these per-invocation constants once in a prepare phase via a new `checkout_merged_data` structure, providing a diff implementing the approach.

## In brief
- Curtis Smith proposed a minimal fix for `git status` disagreeing with `git diff`/`git add` under active conversion: skip the size shortcut in `ie_modified()` when `would_convert_to_git()` applies, letting the existing `index_fd()` path compare content; he reported the new t0020 tests pass with the fix and asked whether to send v2 with only that change.
- Ravi Mistry explained that the `memchr()` NUL check in the blame ignore-revs parser was only for consistency with rejecting other trailing garbage, agreed with Junio C Hamano that rejecting `"<oid>\0garbage"` has no security implication, and offered to drop the check on a reroll.
- Kazumasa Shigeta retracted his earlier reason for leaving other options out of `git stash create` and proposed limiting the current patch to `-m/-q/-u/-a`, leaving `--patch`, `--staged`, and pathspec support for follow-ups.
- **Todd Zullinger detailed Git's mixed licenses**: Todd Zullinger identified imported components and licenses in Git's tree: `reftable/` and `xdiff/xhistogram.c` as BSD-3-Clause, much of `xdiff/` under LGPL-2.1-or-later, `ewah/` as GPL-2.0-or-later, and `compat/obstack.[ch]` and `compat/regex/` as LGPL-2.1-or-later, explaining Fedora's license enumeration.

- Junio C Hamano flushed topics waiting in `next` down to `master` to start the Git 2.98 cycle and rewound the `next` branch.
