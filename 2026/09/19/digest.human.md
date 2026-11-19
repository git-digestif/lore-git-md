# Git mailing list daily digest for 2026/09/19

## The day in brief
D. Ben Knoble posted a two-patch series fixing an autostash bug involving `stash.index=true`, and Maciej Ciemborowicz posted a three-patch series restoring old OIDs in batched ref deletions for the reference-transaction hook. Separately, r.norouzi reported that the default reflog expiry times have been swapped since Git 2.50.

## Notable threads

### stash autostash fix with in-core index merge
D. Ben Knoble posted a two-patch series fixing a bug reported by Eli Barzilay in the interaction between autostashing, staged index entries, and `stash.index=true`. The first patch removes an unused `oid-array.h` include from `builtin/stash.c` that Clang flagged. The second, `builtin/stash: merge index in-core`, replaces the subprocess-based index merge (`git-diff-tree`, `git-apply`, `git-reset`) in `git stash apply --index` with an in-core `merge_incore_nonrecursive()` call, fixing an autostash bug where the file-system round-trip was not re-entrant and caused a lock failure on a deleted `MERGE_AUTOSTASH` ref when `stash.index=true`; it also removes `assert()` calls in `merge-ort.c` and adds a test for fast-forward merge with `--autostash` and `stash.index`. Knoble also apologized for the unusual subject line "[PATCH 0/2] Hi all," attributing it to a mishap while editing the branch description.

### old OIDs for batched ref deletions
Maciej Ciemborowicz traced a regression to commit `8198907795`, which before Git 2.31 replaced single-ref `delete_ref()` calls with `delete_refs()`, dropping the old OID so the reference-transaction hook receives an all-zero update when `git branch -d`, `git tag -d`, `git remote prune`, or `git fetch --prune` deletes a ref. He posted [PATCH 0/3] "refs: report old OIDs for batched deletions", teaching `refs_delete_refs()` to accept an optional `oid_array` of old OIDs and passing the values already resolved by the branch, tag, fetch, and remote callers, keeping a single batched transaction with no added ref reads; deleting 10,000 packed tags took a median of 0.86s before and 0.85s after. Karthik Nayak reviewed patch 1/3, noting that verifying old OIDs introduces a possible race for callers that checked a ref's OID beforehand. D. Ben Knoble had earlier suggested `git bisect` to determine whether the loss of old OIDs was intentional.

### reflog expire default expiry times swapped
r.norouzi reported that `git reflog expire` prunes reachable reflog entries older than 30 days instead of the documented 90, because the default expiry values were swapped in commit `85658275702b` (first released in 2.50.0): `reflog.h` now sets `default_expire_total` to `now - 30 * 24 * 3600` and `default_expire_unreachable` to `now - 90 * 24 * 3600`, reversing the previous assignments in `builtin/reflog.c`. Observed with Git 2.55.0 (Git for Windows), not platform-specific.

## In brief
- Phillip Wood endorsed APIs that return errors rather than dying on allocation failure but objected to the name "safe" for the proposed strbuf API, suggesting a sticky error bit instead; he also identified that `sstrbuf_grow()` calls `st_add3()`, which dies on overflow, undermining the RFC's CodeQL verification.
- Maciej Ciemborowicz reported that `git branch -m old new` does not report the destination ref to the reference-transaction hook; Karthik Nayak explained that branch copy and rename do not use a transaction, called the behavior absurd, and offered to look into a fix.
- Johannes Schindelin posted a four-patch series fixing the GitLab CI cargo invocation in the Windows job, which had been broken since Rust was enabled there; Karthik Nayak created a merge request to test the series.
- Harald Nordgren submitted a patch adding a `fetch.shallow` config option, off by default, that makes a refspec-less fetch or pull in a shallow repository fetch only the current branch's tracked upstream.
- Yashwanth Sai posted an RFC warning when a new commit is dated before its parent, then withdrew it after brian m. carlsson objected to its AI co-authorship under the `ai` section of `Documentation/SubmittingPatches`.
- brian m. carlsson explained that Git sets `LESS=FRX` only when `LESS` is unset and cannot detect whether a custom `core.pager` eventually invokes `less`; Todd Zullinger suggested documenting `LESS=""` overrides in `Documentation/config/core.adoc`.
