# Git mailing list daily digest for 2026/09/30

## Long threads

### stash: in-core index merge for `stash apply --index`

*Long thread (root 2026/09/19/21-26-34): 51 replies so far from 5 participants over 12 days; 3 messages that day.*

D. Ben Knoble's series fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true` in `builtin/stash.c`. After Junio C Hamano diagnosed that the v4 code mishandled the tri-state `merge_result.clean` (where `-1` signals a hard error and would dereference a NULL `result.tree`), Knoble acknowledged the diagnosis and posted [PATCH v5 0/4] "stash: clean up index-mode test merge", rebasing on a synthetic merge for the t5520 interaction and fixing the tri-state handling. Patch 4/4 replaces the subprocess-based index merge (`git diff-tree`, `git apply --cached`, `git reset`) with an in-core `merge_incore_nonrecursive()` call, distinguishing `result.clean < 0` (hard error) from `!result.clean` (conflict).

### refs: report old values to reference-transaction hooks

*Long thread (root 2026/09/19/13-34-15): 28 replies so far from 5 participants over 12 days; 1 message that day.*

Maciej Ciemborowicz's patch addresses the reference-transaction hook reporting an all-zero old object ID for unconditional updates, so batched branch, tag, and remote-ref deletions report zero for both old and new values. The v6 approach resolves the old value in the common transaction hook layer in `refs.c`, before the "preparing" hook and refreshed after the backend locks the refs, keeping it separate from caller-supplied values so it cannot constrain the update. Today Ciemborowicz requested a review of the v6 patch, saying he would like to close the chapter and move on to the next bug.

### Documentation: new `gitmergeconflicts(7)` guide

*Long thread (root 2026/09/24/14-44-15): 21 replies so far from 6 participants over 7 days; 4 messages that day.*

Julia Evans's 7-patch series introduces a new `gitmergeconflicts(7)` manual page explaining merge-conflict resolution with examples, linking to it from the `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull` man pages. Patrick Steinhardt reviewed the new page with suggestions including `linkgit:` references, fixing a "the the" typo, and correcting the "stage N" terminology; Evans responded accepting most, declining a 3-way-merge explanation, and agreeing to rewrite the "ours"/"theirs" section. Junio C Hamano agreed that showing the commit message in conflict markers would help resolution and that `merge.conflictstyle=diff3` should be encouraged strongly enough to become the default.

### rerere: waiting for `MERGE_RR.lock` and `gc --auto`

*Long thread (root 2026/09/02/08-31-37): 21 replies so far from 4 participants over 29 days; 1 message that day.*

Thomas Bachem's v5 three-patch series makes every rerere writer wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds `git rerere gc --auto` (used by `git maintenance run --auto` and `git gc --auto`) that quietly skips a held lock while a manual `git rerere gc` waits and fails, and lets commands that stop at a conflict warn and go on without rerere. Patrick Steinhardt questioned whether `--auto` is misnamed since it only skips garbage collection when the lock is held, proposed renaming it to `--skip-locked` or calling `rerere_gc()` directly, and noted `builtin/rerere.c` needs subcommand refactoring outside this series' scope.

### xdiff/merge-ll: `size_t` for `mmfile_t` and error reporting

*Long thread (root 2026/09/29/06-49-35): 17 replies so far from 4 participants over 2 days; 10 messages that day.*

Jeff King's v2 seven-patch series switches xdiff `mmfile_t` buffer sizes to `size_t`, consolidates `mmbuffer_t` into `mmfile_t`, NUL-terminates buffers read by `read_mmfile()`, and makes `merge-ll` report an error when reading external merge results fails. Patrick Steinhardt reviewed the consolidation, leaning toward keeping the NUL-termination patch but reordered first; Junio C Hamano suggested squashing the merge-ll fixes into the `read_mmfile()` patch. King posted [PATCH v2 0/7], moving the `xmallocz()` NUL-termination patch earlier and documenting the new stderr error reporting in the `merge-ll` commit message.

## The day in brief
D. Ben Knoble posted v5 of the stash index-merge series fixing the tri-state `merge_result.clean` handling Junio C Hamano had flagged. Jeff King posted v2 of the xdiff `size_t` series with the NUL-termination patch reordered, and Taylor Blau posted a four-patch series fixing cruft-less MIDX reachability-closure corner cases in repack. Patrick Steinhardt reviewed Julia Evans's new `gitmergeconflicts(7)` guide and questioned the naming of `git rerere gc --auto`.

## Notable threads

### submodule merge reading another submodule's commit
Philippe Blain confirmed reproducing the bug where a superproject merge tries to read from submodule A a commit that exists only in submodule B, reporting "repository corrupt" on v2.56.0-rc2. He bisected the failure to bb5da75d61 ("commit: use commit graph in lookup_commit_reference_gently()") and the older spurious "hash mismatch" behavior to 6f1e9394e2 ("object: fix leaking packfiles when closing object store"), and noted the related commit-graph lookup bug was fixed in 700f7b74de, which is in `next` but not master.

### sparse-checkout clobbering untracked in-cone files
Webstrand reported that `git checkout` with sparse-checkout clobbers untracked in-cone files instead of aborting: the checkout completes with exit status 0 and a misleading warning, overwriting the untracked content, whereas prior to commit 681c637b4a git refused the sparse checkout and preserved the file.

### repack: cruft-less MIDX reachability corner cases

*Long thread (root 2026/09/30/01-28-34): 7 replies so far from 4 participants over 1 day; 8 messages that day.*

Taylor Blau posted a four-patch series fixing corner cases in cruft-less MIDXs where a written MIDX's objects are not closed under reachability, which can make bitmap generation fail. The patches add a `stdin_packs_context` struct in `builtin/pack-objects.c`, add trees and tags from included packs as extra roots in `--stdin-packs=follow` mode, force `midx_must_contain_cruft` for incremental repacks, and include cruft packs when `.keep` packs are present. Jeff King reviewed, agreeing the fixes are strict improvements while noting `write_midx_incremental()` does not check `midx_must_contain_cruft`.

### git-filter-branch: commit map init from state branch

Grant Moyer's patch fixes `git-filter-branch.sh` populating the commit map dir from the state branch with an inverted "to:from" mapping instead of the written "from:to" format, producing errors with empty filenames under `--prune-empty`. Michele Locati pointed out the subject typo and supplied a test performing an incremental rewrite in two runs, verified to fail without the fix and pass with it. Patrick Steinhardt asked for a direct test of swapped entries.

### reftable: on-disk reflog timezone representation
Patrick Steinhardt's three-patch series fixes the reftable backend storing reflog timezones in `[+-]HHMM` format instead of the spec's signed offset in minutes, adding date helpers to convert between the two. Karthik Nayak reviewed, suggesting `minutes_to_tz(offset) * eastwest` and a negative-timezone test case; Steinhardt agreed to both, and Junio C Hamano concurred with the suggested rewrite.

## In brief
- Kaartic Sivaraam reviewed Christian Couder's `early_scan_options()` series, warning that unhandled abbreviations could misinterpret option values and suggesting the scan stop at the first unrecognized argument.
- Karthik Nayak posted a patch writing unchanged refs verbatim with `fwrite()` in `refs/packed-backend.c`, reporting a consistent ~20% speedup when deleting a ref from 100,000 packed refs.
- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee questioned whether a size check making network calls counts as a dry run, and Junio C Hamano agreed with the concerns. *Long thread (root 2026/09/30/00-21-45): 12 replies so far from 4 participants over 1 day; 13 messages that day.*
- Phillip Wood posted a series teaching merge-ort to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` recreates them; Johannes Sixt asked whether an index extension would be a better home.
- Tamir Duberstein posted v3 of the CI series using `cmp` for huge-output comparison and twice the CPU count for jobs on both providers; Junio C Hamano marked it for `next`.
- Harald Nordgren posted v3 linking CI failure and leak annotations to the test script; Junio C Hamano said the patches look good and would mark the topic for `next`.
- Harald Nordgren posted a patch accepting `@{p}` as short for `@{push}`; Junio C Hamano marked it "Queued" after tracing the history of the `@{u}` asymmetry.
