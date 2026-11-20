# Git mailing list daily digest for 2026/11/05

## Long threads

### Removing gittutorial-2 from the documentation

*Long thread: 32 replies so far from 4 participants over 39 days; 4 messages that day.*

Julia Evans's two-patch series deletes `Documentation/gittutorial-2.adoc` and all references to it, leaving a stub that redirects readers to `gitdatamodel` and `gitcore-tutorial`, to clear obsolete tutorial material. Earlier revisions removed the entry from `command-list.txt` so `git(1)` would not advertise the obsolete guide, and Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out of the series. Today Tuomas Ahola reported that the removal topic has graduated to `master` and asked Junio to pick up his rerolled `ta/command-list-guides-sync-lint`; after confirming the patch was the result of the rebasing he had planned, Junio noted the thread still has an unanswered review comment from Julia Evans and asked whether it is ready to be picked up.

### Hint for `<rev>~N` failures in shallow clones

*Long thread: 23 replies so far from 6 participants over 47 days; 1 message that day.*

Harald Nordgren's patch adds a hint in the `object-name` code explaining why `<rev>~N` fails in a shallow clone when the walk runs out of parents exactly at a recorded shallow boundary, suggesting the exact `--deepen` needed. Earlier, Junio C Hamano had asked for volunteers to review the topic, warning he would discard it if no reviews arrived. Today he observed that after a few days nobody had expressed support for the hint, and that the thread meant to invite supporters had instead been cluttered with messages about the review process.

### Alternate refs during receive-pack quarantine

*Long thread: 21 replies so far from 5 participants over 9 days; 4 messages that day.*

Kristofer Karlsson's patch fixes the object database code so that `rev-list --alternate-refs` no longer lists the repository's own refs--including hidden ones--while receive-pack's quarantine connectivity check runs, where the quarantine directory is swapped in as the primary object directory. Earlier versions tried a "borrowed" flag recognizing the quarantine's parent directory; today's v3 reverts to skipping the repository's own object directory directly in the alternate-refs code, with a perf test showing a push into a repository with ~200,000 hidden refs improving from 0.29s to 0.10s. Jeff King reviewed the v3 patch and said it looks good, and Junio C Hamano acknowledged the submission, said he will replace the previous version, and marked the topic ready for `next`.

## The day in brief
Kristofer Karlsson's quarantine alternate-refs fix reached v3 and was marked ready for `next`, while the gittutorial-2 removal graduated to `master` and its follow-up lint topic awaits a final readiness check. Junio C Hamano noted the shallow-clone `<rev>~N` hint still lacks any expressed support. Christoph Weiss reported that geometric maintenance prunes commits without updating the commit-graph.

## Notable threads

### Geometric maintenance prunes commits without updating commit-graph
Christoph Weiss reported a bug in the maintenance code: after Forgejo switched Git from 2.52 to 2.54, changing the default maintenance strategy from `gc` to `geometric`, repository health checks started failing because under `geometric` maintenance prunes commits without updating the commit-graph. He provided a reproduction script using `git maintenance run --auto` on a bare repository with `core.commitGraph` enabled.

### Replaying 2-parent merges in git history
This series extends the `replay` builtin and `git history` to support replaying 2-parent merge commits, which currently die with "replaying merge commits is not supported yet", wiring Elijah Newren's algorithm into the `replay_revisions()` API. Today Philippe Blain reported that when replaying a merge whose first parent was amended, the pre-seeding incorrectly maps the second parent to the onto commit, changing the merge-base and causing a conflict; he also suggested adding the replay design notes as a technical document and mentioning the pre-seeding in the commit message, while calling the series a massive time saver.

### Stale loose-object cache after merge writes
Patrick Steinhardt's two-patch series fixes `odb/source-loose.c` existence checks so blobs written by `git merge` can be read back without a spurious backfill fetch from a promisor remote. Today Jeff King argued the failure points to a separate underlying issue: callers using `QUICK` should tolerate false negatives, so `prefetch_cache_entries()` and other prefetch callers may need to omit `QUICK`, and noted the fix leaves related stale-cache cases such as objects moved into a pack or written by another process.

## In brief
- Derrick Stolee reviewed Ilia K's fsmonitor/untracked-cache patch, judged the selective invalidation better than his own full-clear approach, and found no issues worth re-rolling.
- Junio C Hamano reviewed several commits of the push refspec-matching speedup series, calling the strmap conversions "clever and apparently very effective" with "very nice" speedups.
- Okhuomon Ajayi posted a replacement six-patch series storing the repository in `struct notes_tree` so `notes-utils.c` can drop `USE_THE_REPOSITORY_VARIABLE`.
- Patrick Steinhardt posted a five-patch series introducing `GIT_TEST_DEFAULT_OBJECT_STORAGE_FORMAT=` to run tests with a non-default object storage backend.
- Kristofer Karlsson reported the describe date-skew fix makes an expected-failure test pass only with a commit-graph; Junio C Hamano suggested freezing the test to always run under `GIT_TEST_COMMIT_GRAPH`.
- Tanga Joris tested v3 of moving `pack.packSizeLimit` into repo config and noted patch 2/2 conflicts with the freeze on new `repo_config_values` members.
- Yopa Nelly posted v2 of `promisor.fetchOrder` with the parsed list per-repository in `struct promisor_remote_config`; Junio C Hamano asked how it meshes with server-side promisor ordering work.
