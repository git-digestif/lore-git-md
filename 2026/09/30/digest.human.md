# Git mailing list daily digest for 2026/09/30

## Long threads

### stash: in-core index merge for `stash apply --index`

*Long thread: 51 replies so far from 5 participants over 12 days; 3 messages that day.*

D. Ben Knoble posted [PATCH v5 0/4] "stash: clean up index-mode test merge", rebasing on a synthetic merge for the t5520 interaction and fixing the tri-state `merge_result.clean` handling Junio C Hamano had diagnosed in v4. Patch 4/4 replaces the subprocess index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing the autostash bug with `stash.index=true`, and distinguishes `result.clean < 0` (hard error) from `!result.clean` (conflict). Earlier, the thread traced a t5520.69 failure to reflog entries lost when `stash apply --index` stopped spawning `git reset`.

### refs: report old values to reference-transaction hooks

*Long thread: 28 replies so far from 5 participants over 12 days; 1 message that day.*

Maciej Ciemborowicz requested review of [PATCH v6 0/1] "refs: report old values to transaction hooks", a single change in the common transaction layer in `refs.c` that resolves old object IDs for the hook before "preparing" and refreshes them after the backend locks the refs, only when a reference-transaction hook exists. This replaces the caller-specific approach Patrick Steinhardt argued was the wrong layer; it fixes batched branch, tag, and remote-ref deletions reporting all-zero old OIDs.

### Documentation: new `gitmergeconflicts(7)` guide

*Long thread: 21 replies so far from 6 participants over 7 days; 4 messages that day.*

Julia Evans's 7-patch series adds a `gitmergeconflicts(7)` page and links it from the `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull` man pages. Patrick Steinhardt reviewed the new page -- `linkgit:` references, a "the the" typo, more accurate "stage" terminology -- and Julia Evans said she will adopt most of them. Junio C Hamano agreed that showing the commit message in conflict markers would help resolution and that `merge.conflictstyle=diff3` should become the default.

### rerere: waiting for and skipping `MERGE_RR.lock`

*Long thread: 21 replies so far from 4 participants over 29 days; 1 message that day.*

Thomas Bachem's v5 series makes rerere writers wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds `git rerere gc --auto` (passed by `git maintenance run --auto` and `git gc --auto`) that quietly skips a held lock, and lets conflict-time callers warn and continue. Patrick Steinhardt questioned the `--auto` name, proposed `--skip-locked` or calling `rerere_gc()` directly, and noted `builtin/rerere.c` needs subcommand refactoring outside this series.

### xdiff: `size_t` for `mmfile_t` and merge-ll error reporting

*Long thread: 17 replies so far from 4 participants over 2 days; 10 messages that day.*

Jeff King posted [PATCH v2 0/7] "use size_t for xdiff mmfile_t", moving the `xmallocz()` NUL-termination patch earlier and documenting the `xsize_t()` truncation check and new stderr errors when `read_mmfile()` fails in `merge-ll`. Patrick Steinhardt reviewed the `mmfile_t` consolidation and NUL-termination; Jeff King agreed to reorder and to state the new error-on-failed-read behavior explicitly.

## The day in brief
D. Ben Knoble's v5 stash series fixes the tri-state index-merge handling and moves `stash apply --index` to an in-core merge, while Maciej Ciemborowicz asked for review of the transaction-layer fix reporting old OIDs to reference-transaction hooks. Taylor Blau posted four fixes for cruft-less MIDX reachability corner cases, and Philippe Blain bisected the submodule merge "repository corrupt" failure to a commit-graph lookup change.

## Notable threads

### submodule merge reading another submodule's commit
Philippe Blain reproduced a superproject merge that reads a commit existing only in submodule B from submodule A, failing with "repository corrupt" on v2.56.0-rc2. He bisected it to bb5da75d61 ("commit: use commit graph in lookup_commit_reference_gently()") and an older "hash mismatch" behavior to 6f1e9394e2; the related commit-graph bug is fixed in 700f7b74de, in `next` but not master.

### sparse-checkout clobbering untracked in-cone files
Webstrand reported that `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting: exit 0 with a misleading warning and the file overwritten, whereas before 681c637b4a git refused the checkout and preserved it.

### repack: cruft-less MIDX reachability corner cases

*Long thread: 7 replies so far from 4 participants over 1 day; 8 messages that day.*

Taylor Blau posted [PATCH 0/4] fixing cruft-less MIDX corner cases whose objects are not closed under reachability, which can fail bitmap generation: extra roots for lone trees/tags in `--stdin-packs=follow`, forcing `midx_must_contain_cruft` on incremental repacks, and including cruft packs when `.keep` packs exist. Reviewers suggested an oidset for extra roots and flagged that `write_midx_incremental()` may share the gap.

## In brief
- Grant Moyer's fix for `git-filter-branch` inverting the commit map when populated from the state branch; Michele Locati supplied a direct regression test.
- Kaartic Sivaraam reviewed Christian Couder's `early_scan_options()` series, flagging abbreviation handling and test questions.
- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee questioned the size metric and proposed `--info=(count|size)`. *Long thread: 12 replies so far from 4 participants over 1 day; 13 messages that day.*
- Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`.
- Patrick Steinhardt objected to Mark Chu-Carroll's ref-type filter flags for `git repo structure`, preferring revision-based filtering.
- Harald Nordgren's patch makes `@{p}` short for `@{push}`; Junio C Hamano marked it queued.
- Karthik Nayak's packed-refs change writes unchanged refs with `fwrite()`, about 20% faster on deletes.
