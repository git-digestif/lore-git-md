# Git mailing list daily digest for 2026/10/10

## Long threads

### New `gitmergeconflicts(7)` manual page

*Long thread: 54 replies so far from 6 participants over 17 days; 1 message that day.*

Julia Evans's six-patch documentation series adds a `gitmergeconflicts(7)` man page explaining how to resolve merge conflicts, and links to it from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of repeating the guidance in each command's page. Today Junio C Hamano asked that the new technical manual list every merge-conflict-causing command explicitly -- including `git am -3` and `git checkout -m` -- rather than ending with "and other operations", and asked for formal constructions ("here is", "there is") instead of contractions throughout, since the document is not a personal diary.

### Fetching fewer branches from a new remote in shallow repos

*Long thread: 31 replies so far from 4 participants over 22 days; 4 messages that day.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository. Today he posted v8, a five-patch series splitting the work into a preliminary `collect_upstream_from_remote()` helper in `remote.c` plus a simplified refmap-inference commit, fixing a regression that could drop the current branch's `@{upstream}` when `remote.<name>.fetch` was narrowed, and threading `struct repository *` through instead of hardcoding `the_repository`. Junio C Hamano reviewed patch 2/5, objecting that duplicating the current-branch upstream logic into the `remote.*.fetch` branch belongs to the following step, and asked whether dropping the NULL-branch guard in `collect_upstream_from_remote()` was intended; Harald Nordgren replied that the missing guard was overlooked but that he does not think the segfault could be reproduced by a test. Junio also called patch 4/5 (`git remote add --limited-fetch`) straightforward and nicely done.

### Rerere waiting for `MERGE_RR.lock`

*Long thread: 27 replies so far from 4 participants over 39 days; 3 messages that day.*

Thomas Bachem's series addresses a rebase dying at a conflict while a background `git rerere gc` holds `MERGE_RR.lock`. Today he posted v7, a two-patch series: the first makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for the lock before failing, and the second adds a hidden `--skip-locked` option to `git rerere gc` that auto-maintenance passes so a background gc does nothing while the lock is held, implemented via a new `RERERE_GC_NOWAIT` flag in `enum rerere_gc_flags`. The earlier patch letting conflict-stopping commands go on without rerere was dropped from v7.

### Running ref copy and rename through transactions

*Long thread: 22 replies so far from 6 participants over 22 days; 2 messages that day.*

Maciej Ciemborowicz's four-patch series routes reference copy and rename through ordinary transactions so the reference-transaction hook can observe and reject the complete operation -- previously the files backend saw only the source deletion and reftable saw neither endpoint. Today he offered to let someone else write the patches if that would be more efficient than reviewing his, saying his goal is to get the issue resolved. brian m. carlsson advised that it is better not to submit a patch at all if one cannot meet the SubmittingPatches guidelines, while noting that reporting the bug without a patch is valuable and that the problem may be picked up by someone familiar with the code.

## The day in brief
Harald Nordgren posted v8 of the shallow-repo fetch series and Thomas Bachem posted v7 of the rerere lock-wait series, while Siddharth Shrimali's fix for `git repack --dry-run` rebuilding packs earned a Reviewed-by. Documentation work continued on the new `gitmergeconflicts(7)` page, and Junio C Hamano started the Git 2.98 cycle by flushing `next` down to `master`.

## Notable threads

### `repack --dry-run` no longer rebuilds packs
Siddharth Shrimali's patch fixes `git repack --drop-filtered --dry-run`, which was documented to list objects without rebuilding any pack or deleting anything but did both because the `--dry-run` block fell through into the regular repack code. Today he posted v2, which returns right after listing the drop candidates -- skipping the repack and any deletion even with an explicit `-d` -- and adds a test that the pack directory is unchanged with and without `-d`; the bug has existed since `--dry-run` was introduced in 1e746b00aa. D. Ben Knoble reviewed v2, said he was happy with it, and offered his Reviewed-by.

### `fast-import` lookup of commits created in the same session
Marc Becker submitted a patch to `git fast-import` fixing a crash when the `from` command references a commit created during the same import session, by resolving the requested tag or commit through the object cache before falling back to the ODB layer.

### Revision filtering for `git repo structure`
Mark C. Chu-Carroll posted v4 of a patch adding a `[<revs>...]` argument to `git repo structure`, limiting the set of objects processed based on the revision specification instead of always summarizing everything in the repository; the implementation notes that `setup_revisions` performs an initial ref-walk requiring a second ref-walk in `count_references` when revs are specified.

## In brief
- Marc Becker posted v2 of a wincred refactor that splits credential blob dissection into separate `write_credential` and `write_credential_item` methods, uses `wmemchr` to detect wide-character newlines, and treats sizes as wide-character counts to fix deterministic line splitting and CRLF bleed-over.
- Junio C Hamano noted that `checkout_merged()` is called once per conflicted path, causing `merge.renormalize` and the `MERGE_LABELS` file to be read repeatedly during a single conflicted `checkout -m`, and suggested a preliminary patch to read these once in a prepare phase.
- Curtis Smith proposed a minimal fix for the disagreement between `git status`, `git diff`, and `git add` under active conversion: skip the size shortcut in `ie_modified()` when `would_convert_to_git()` applies, letting the existing `index_fd()` path compare contents.
- Kazumasa Shigeta retracted his earlier reason for leaving options out of `git stash create` and proposed limiting the current patch to `-m`/`-q`/`-u`/`-a`, leaving `--patch`, `--staged`, and pathspec support for follow-ups.
- Junio C Hamano flushed topics waiting in `next` down to `master` to start the Git 2.98 cycle and rewound the `next` branch.
