# Git mailing list daily digest for 2026/11/05

## The day in brief
The Git mailing list saw significant activity around performance optimizations for `git push`, with Junio C Hamano reviewing and approving multiple patches in Jon Simons' 12-patch series that delivers 10-100x speedups in refspec matching. Philippe Blain identified a critical regression in Johannes Schindelin's merge commit support for `git history`, while Derrick Stolee withdrew his competing patch for an fsmonitor/untracked-cache bug in favor of Ilia Kurenkov's more targeted solution. Outreachy proposals and internship project discussions continued, with several contributors refining their plans for libification and command-line interface improvements.

## Notable threads

### [RFC PATCH 0/5] Add merge commit support to "git history"
**What changed**: Philippe Blain identified a regression in boundary commit handling when replaying a merge whose second parent is specified as a negative revision (e.g., `M^!`). The pre-seeding logic incorrectly maps the second parent to the `--onto` target, altering the merge base and triggering unnecessary conflicts.

**Problem/goal**: Extend "git history" to handle merge commits by using three-way merge composition. The regression breaks expected behavior when replaying merges with unchanged trees.

**Subsystem**: Replay/merge logic in `replay.c`

**Change type**: Bugfix (regression)

**Impact**: High for users of `git history` with merge commits. The bug affects core use cases like `git replay --onto HEAD M^!` and could block adoption if unresolved.

**Today's developments**: Blain provided a concrete reproduction scenario and suggested two actionable improvements: adjust the pre-seeding logic to exclude negative revisions that are not boundary commits, and document the pre-seeding behavior. This is a high-signal review that directly impacts the series' viability.

---

### [PATCH 0/15] push: speed up refspec matching for many deletions
**What changed**: Junio C Hamano reviewed and approved multiple patches in Jon Simons' 12-patch series (v2) that optimizes `git push` refspec matching. Key approvals include:
- Patch 3: Eliminates `mkpath()` overhead in `refname_match()`, delivering 55-82% of the total speedup
- Patch 5: Fixes a memory leak in `send-pack`
- Patch 8: Corrects client-side rejection of duplicate destinations
- Patch 9: Optimizes `match_explicit_refs()` via `strmap` lookups
- Patch 12: Optimizes `--force-with-lease` overhead with `strmap`-based lookups

**Problem/goal**: Client-side refspec matching becomes a scalability bottleneck when pushing many branch deletions to servers with large numbers of refs (e.g., 100k), leading to O(n²) behavior.

**Subsystem**: Push/transport, refspec matching

**Change type**: Performance optimization

**Impact**: High for users pushing many deletions. The series delivers 10-100x speedups in benchmarked scenarios, with `git push --dry-run` of 100 delete refspecs dropping from 2.5-8.9s to 0.15-0.19s.

**Today's developments**: Junio's reviews effectively approve 5 of the 12 patches for integration, with the remaining patches awaiting broader review. The series is now technically complete and addresses all prior feedback, including SZEDER Gábor's endorsement of the `mkpath()` removal.

---

### fsmonitor/untracked-cache integration bug
**What changed**: Derrick Stolee withdrew his competing patch (`ds/fsmonitor-untracked-cache-fix`) in favor of Ilia Kurenkov's older, unreviewed patch (`ik/fsmonitor-untracked-cache-trivial-response`).

**Problem/goal**: Fix a long-standing bug where the untracked cache remains incorrectly marked as valid after fsmonitor fails or returns a trivial response, causing new untracked files to be invisible to `git status`.

**Subsystem**: fsmonitor, untracked cache

**Change type**: Bugfix

**Impact**: High for users of fsmonitor with untracked files. The bug affects core functionality and has been present since 2017.

**Today's developments**: Stolee's withdrawal resolves the procedural question about which patch to prioritize. Ilia's approach preserves parts of the untracked cache rather than invalidating it entirely, offering better performance in edge cases. The patch now awaits formal review from Stolee and broader testing confirmation.

---

### [PATCH] promisor-remote: add promisor.fetchOrder config
**What changed**: Yopa Nelly posted a v2 patch addressing architectural flaws in the initial implementation of `promisor.fetchOrder`. The new version:
- Fixes a crash triggered by `string_list_split_in_place_f()` on a `STRING_LIST_INIT_DUP` list
- Resolves the file-static `fetch_order` list that was shared across repositories
- Adds documentation in `Documentation/config/promisor.adoc`
- Implements a two-pass approach in `try_promisor_remotes()` for explicit ordering

**Problem/goal**: Allow users to configure the order in which Git queries multiple promisor remotes during lazy fetches, replacing the current implicit configuration order.

**Subsystem**: Promisor remote, partial clone

**Change type**: Feature

**Impact**: Medium for users with multiple promisor remotes. The feature addresses a concrete interoperability gap but remains controversial due to process concerns.

**Today's developments**: The v2 patch addresses all technical feedback from Tanga Joris's substantive review, including the crash and architectural flaws. However, Kaartic Sivaraam's process concerns about bypassing the RFC phase remain unresolved. Junio C Hamano also raised a question about interaction with server-side promisor remote advertisement.

---

### Loose object database cache invalidation during `git merge`
**What changed**: Jeff King (Peff) raised fundamental concerns about Patrick Steinhardt's bugfix for loose object cache invalidation during `git merge`. Peff identified that the current patch is a band-aid that leaves other code paths (e.g., `prefetch_cache_entries()`) and racy staleness unaddressed.

**Problem/goal**: Fix a bug where newly written merged blobs in `git merge` cannot be read back due to stale loose object cache entries, causing Git to incorrectly assume the objects are missing locally.

**Subsystem**: Loose object database cache, promisor remote backfill logic

**Change type**: Bugfix

**Impact**: High for users of partial clones with `git merge`. The bug causes unnecessary, failing backfill fetches from promisor remotes.

**Today's developments**: Peff's review shifts the thread from "ready to merge" to an open design discussion. He proposes two alternatives:
1. Omit `QUICK` in prefetch paths, forcing a full re-scan of the ODB
2. Reconsider the cache-flushing logic removed in commit `5946a564cdd`

The discussion now centers on whether to prioritize correctness (Peff's preference) or minimal disruption (the current patch).

---

## In brief
- **[PATCH] builtin/annotate.c: fix command-line completion**: Junio approved Tanga Joris's patch that fixes `git annotate --<TAB>` offering `git diff` options instead of `git annotate`'s own. The patch removes redundant `-c` prepending and adds regression tests.
- **[PATCH] hash: prevent panic when formatting ObjectID with invalid algorithm**: Tanga Joris identified a potential inconsistency with the C codebase where the Rust implementation treats algorithm ID 0 as invalid while the C code silently falls back to the default hash algorithm.
- **[PATCH] rev-list: fix `--grep` and related option parsing**: Azuama Odinaka Maximus expanded the fix to cover all 18 `parse_long_opt()`-style options (including `--grep`, `--author`, `--committer`, `--since`, `--until`) that previously misparsed their arguments when followed by another option-like string.
- **[PATCH] odb.c: skip self-referential alternates during quarantine**: Junio confirmed the patch is ready for integration, marking the topic as ready for `next`. The patch fixes a 65.5% performance regression in repositories with 200,000 hidden refs.
- **Outreachy proposals**: Srujith Poojary and Tanga Joris submitted detailed proposals for libification and command-line interface improvements, respectively. Both proposals demonstrate strong technical preparation and alignment with project goals.
- **[PATCH] bundle-uri: handle relative URIs with excessive parent traversals**: Aniket Gupta posted v3 addressing feedback from Patrick Steinhardt, including splitting the patch into three commits and tightening test coverage.
- **[PATCH] environment: move `pack.packsizelimit` into `repo_config_values`**: Tanga Joris identified a procedural blocker: Junio's `jc/freeze-repo-config-values` topic prohibits adding new members to `struct repo_config_values`, making integration impossible in its current form.
- **Geometric maintenance strategy regression**: Christoph Weiss reported that the `geometric` maintenance strategy (now the default in Git 2.54) prunes unreachable commits but fails to update the commit-graph, leaving it in an inconsistent state.
- **[PATCH] completion: remove NO_PARSEOPT for commands using parse-options**: Junio suggested rewording a comment in `parse-options.h` to focus on the function's general purpose rather than over-specifying the `fast-import` edge case.
- **[PATCH] path-walk: fix object loss when tagged tree revisits path**: Junio acknowledged Taylor Blau's "LGTM" but noted the patch still lacks substantive review from domain experts. The patch fixes a regression in Git 2.55.0 where path-walk machinery drops objects when a tagged tree contains a blob at a path already visited by a prior tree batch.