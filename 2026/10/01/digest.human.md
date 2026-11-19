# Git mailing list daily digest for 2026/10/01

## Long threads

### stash: in-core index merge for `stash apply --index`

*Long thread: 53 replies so far from 5 participants over 13 days; 2 messages that day.*

D. Ben Knoble's series fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true` by replacing the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, with a tri-state check on `merge_result.clean`. Phillip Wood confirmed the v5 range-diff looks as expected and was pleased to see subprocesses removed from `git stash`, and Junio C Hamano said the series looks very good to him and marked it for `next`.

### refs: report old values to transaction hooks

*Long thread: 29 replies so far from 5 participants over 13 days; 1 message that day.*

Maciej Ciemborowicz's single-patch v6 resolves old object IDs in the common reference-transaction hook layer, before the "preparing" hook and again after the backend locks refs, so batched branch, tag, and remote-ref deletions report real old values without constraining updates. Today he lowered the priority of the underlying bug after resolving his branch and tag deletion issues using data returned by the transaction in the prepared state, naming branch renaming as the remaining issue he cannot address without this fix.

### rerere: wait for MERGE_RR.lock and skip in auto gc

*Long thread: 23 replies so far from 4 participants over 30 days; 2 messages that day.*

Thomas Bachem's three-patch series makes every rerere writer wait `rerere.lockTimeout` for `MERGE_RR.lock`, adds a `git rerere gc --auto` that quietly skips a held lock for `git maintenance run --auto` and `git gc --auto`, and lets conflict-time callers warn and go on. Today he agreed to rename the option to a hidden, undocumented `--skip-locked`, rename patch 3's flag to `RERERE_WARN_LOCKED`, and keep the option after finding that calling `rerere_gc()` directly from `builtin/gc.c` would die with exit code 128 on a held lock; Patrick Steinhardt agreed to keep the hidden option.

### Documentation: new gitmergeconflicts guide

*Long thread: 23 replies so far from 6 participants over 8 days; 2 messages that day.*

Julia Evans's seven-patch documentation series adds a `gitmergeconflicts` manual page explaining merge conflicts with examples and links to it from `git-merge`, `git-cherry-pick`, `git-rebase`, `git-revert`, and `git-pull`. Today Patrick Steinhardt proposed extending conflict-marker labels to include commit ID, message, and date, and agreed `merge.conflictstyle=diff3` should become the default; Julia Evans endorsed including dates and commit messages in markers.

### xdiff: use size_t for mmfile_t

*Long thread: 21 replies so far from 4 participants over 3 days; 4 messages that day.*

Jeff King's seven-patch v2 consolidates `mmbuffer_t` into `mmfile_t`, switches buffer sizes to `size_t`, NUL-terminates buffers read by `read_mmfile()`, and makes `merge-ll` report errors when reading external merge results fails. Today Patrick Steinhardt flagged a commit-message nit in the NUL-termination patch and okayed the `merge-ll` change, and Junio C Hamano accepted keeping the `mmfile_t` name and agreed the error-reporting fix is much easier after the `read_mmfile()` switch.

## The day in brief

Junio Hamano marked D. Ben Knoble's stash merge for `next` and said Patrick Steinhardt's reftable reflog-timezone fix looks ready. Patrick Steinhardt agreed a delta-base cache issue "seems to be the issue indeed" for submodule merge bug, to investigate. Elijah Newren objected that oidset conversion in the cruft-less MIDX series can regress pack quality.

## Notable threads

### submodule merge reads foreign commit from wrong repository

A bug report describes `git merge` on a superproject with submodules sometimes trying to read from submodule A a commit that exists only in submodule B, reporting A as corrupt. Patrick Steinhardt agreed that a delta-base cache issue "seems to be the issue indeed," noting that packfiles should be cleared from the delta-base cache when closing but currently are not, and said he would investigate further.

### merge-ort: `--no-overwrite-ignore` bypassed at rename destination
Coy Geek reported that `git merge --no-overwrite-ignore` fails to protect an ignored, untracked file when directory rename detection moves a committed file onto that path: with `merge.directoryRenames=true` the merge succeeds and overwrites the ignored content, and in the default `conflict` mode the ignored file is already overwritten when the conflict stops the merge. A direct collision is correctly refused, so only rename-detection destinations bypass the guard; reproduced on versions 2.54.0 through 2.56.0.

### repack: corner cases for cruft-less MIDXs

*Long thread: 14 replies so far from 5 participants over 2 days; 8 messages that day.*

Taylor Blau posted v2 of an eight-patch series fixing tree/tag closure and `--write-midx=incremental` gaps when `repack.midxMustContainCruft=false`, using an oidset for extra roots and traversing kept packs during geometric repacks. Elijah Newren objected that the oidset conversion can regress pack quality, since oidset iteration order can visit a subtree first and cause blobs to be written in full instead of delta-compressed.

## In brief
- Patrick Steinhardt posted v2 of the reftable reflog-timezone fix storing signed minute offsets per the spec; Junio C Hamano said it looks really ready and will mark it for `next`.
- Grant Moyer reposted the filter-branch fix for commit-map init from the state branch; Junio C Hamano asked for a commit-log explanation of a test change.
- **Toon Claes reviews packed-refs fwrite() patch**: He listed sanitizations lost by fwrite() and proposed a separate preformatted writer function rather than mixing both methods.

- Patrick Steinhardt's series adds subcommand groups to `git refs`; Junio C Hamano called patch 3 "Nice".
- Harald Nordgren posted v4 linking CI failure and leak annotations to the test script file and line.
- Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`; Junio C Hamano critiqued the commit message's user-visible framing.
- Phillip Wood defended storing conflict labels in `.git/MERGE_LABELS` and fixed CRLF parsing with `strbuf_getline()`; Junio C Hamano agreed to document the common repository files separately.
