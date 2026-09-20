# Git mailing list daily digest for 2026/09/19

## The day in brief
Git’s Outreachy December 2026 cohort added a third project, “Implement promisor remote fetch ordering,” with Kaartic Sivaraam tentatively confirmed as mentor. A long-standing autostash bug with staged index entries saw its fix polished and submitted as a two-patch series. Phillip Wood’s review of Derrick Stolee’s “safe” strbuf API identified a critical bug and proposed usability improvements, while a regression in the reference-transaction hook for branch renames gained a clear diagnosis and a volunteer to fix it.

## Notable threads

### Outreachy December 2026: third project approved
Christian Couder confirmed the “Implement promisor remote fetch ordering” project for Git’s Outreachy December 2026 cohort. The project, which improves partial-clone performance by allowing configurable fetch ordering for promisor remotes, is now visible to approved mentors on the Outreachy site. Kaartic Sivaraam is tentatively confirmed as mentor, with Christian available to co-mentor if needed. The Outreachy application deadline is October 5, 2026 (4pm UTC), leaving roughly two weeks to finalize mentor assignments and project details.

The project joins two others already approved: “Improve how command arguments and options are scanned and parsed” (co-mentors: Christian Couder, Siddharth Asthana) and “Reduce Git’s global state to enable Git's libification” (co-mentors: Christian Couder, Usman Akinyemi). A fourth proposal, “Enhance promisor-remote protocol for better-connected remotes,” remains excluded due to feasibility concerns. Sponsorship outreach is ongoing, with a discussion topic added to the Git Contributor’s Summit 2026.

### Autostash bugfix: staged index entries now handled correctly
D. Ben Knoble submitted a two-patch series fixing a bug in `git stash` where autostashing failed to correctly handle staged index entries when `stash.index=true` is set. The bug, reported by Eli Barzilay, manifested during merge operations that trigger autostash, causing incorrect index state preservation due to a race condition in the subprocess-based logic (`git diff-tree` and `git apply`).

The fix replaces the external process calls with in-core merging via `merge-ort.c`, ensuring staged entries are properly preserved. The series also adds a test case to `t/t7600-merge.sh` covering the bug scenario (staging a file before a fast-forward merge with `--autostash` and `stash.index=true`). Phillip Wood’s earlier feedback—diagnosing why an earlier `reset_tree()`-based approach failed (cleared stat data and `CE_UPTODATE` flags) and suggesting targeted improvements (silence merge verbosity, simplify conflict-label setup, use `oidcpy()`)—has been incorporated.

The patches touch `builtin/stash.c` and `merge-ort.c`, removing 65 lines of subprocess logic and replacing it with 23 lines of in-core merge calls. The author seeks feedback on whether to retain or remove `assert()` calls in `merge-ort.c`, noting the original assertions (added in 2020) lack explanatory context. The series is ready for review, with no open technical questions beyond the `assert()` handling.

### "Safe" strbuf API: bug identified, usability concerns raised
Phillip Wood’s review of Derrick Stolee’s RFC series introducing a “safe” strbuf API that avoids `die()` identified a critical bug and raised usability concerns. The bug: `sstrbuf_grow()` uses `st_add3()`, which calls `die()` on overflow, violating the safety guarantee. Phillip suggested replacing it with `st_add_overflows()` to maintain the no-`die()` invariant.

Beyond the bug, Phillip critiqued the API’s framing and usability. He argued the “safe” label is vague and should instead describe the API as error-returning, with naming aligned to existing patterns (e.g., `strbuf_*_gently`). He also proposed a sticky error bit (like C’s `stdio` functions) to simplify error checking, as checking every `strbuf` call would be cumbersome. The review is constructive, treating the RFC as a starting point rather than a finished proposal.

The series aims to provide a crash-free subset of the `strbuf` API for critical code paths like trace2, motivated by the trace2 subsystem’s requirement to avoid `die()` calls. The current implementation includes `sstrbuf_init()`, `sstrbuf_release()`, and `sstrbuf_grow()`, with `json-writer.c` partially converted to use them. The bug in `sstrbuf_grow()` undermines the CodeQL verification in the series, as the query would not catch this transitive `die()` call.

### Reference-transaction hook: branch rename regression diagnosed
Karthik Nayak diagnosed a regression in the `reference-transaction` hook where branch renames (`git branch -m`) omit the creation of the new ref. The hook only receives the deletion of the old branch name, affecting both the files-based and reftable backends. Karthik traced the issue to backend-specific behavior: the files backend uses `refs_delete_ref()`, which triggers the hook for the deletion, while the reftable backend writes a TOMBSTONE entry directly, skipping the hook entirely.

Karthik offered to submit a patch to fix the issue, noting that branch renames bypass the ref transaction mechanism entirely. The fix will likely involve updating the branch-rename logic to use transactions, ensuring both deletion and creation events are included. The regression affects Git versions 2.28 through 2.55 and is confirmed by a reproducer script provided by the original reporter, Maciej Ciemborowicz.

### Reference-transaction hook: regression fix hits race condition
Karthik Nayak identified a race condition in Maciej Ciemborowicz’s three-patch bugfix series restoring the old OID in the `reference-transaction` hook for deletions. The first patch changes `refs_delete_refs()` from unconditional deletion to conditional deletion based on supplied old OIDs, which could cause transaction failures if the ref’s value changes between resolution and execution.

The series targets a regression introduced in Git 2.31 (commit `8198907795`), where high-level commands like `git branch -d` and `git tag -d` stopped passing the resolved old OID to the hook, instead reporting all-zeros for both old and new OIDs. The fix restores the pre-2.31 behavior while preserving the performance gains of the original optimization (deleting 10,000 packed tags in ~0.85 seconds). The series includes thorough test coverage for both files and reftable backends, SHA-1/SHA-256, and atomic fetch pruning.

The race condition must be addressed before the series can proceed. Alternatives may include reverting to unconditional deletion (losing the safety check) or implementing a retry mechanism. The patches are otherwise uncontroversial, with no other technical objections raised.

## In brief
- **GitLab CI breakage**: Karthik Nayak set up an internal GitLab merge request (MR #671) and pipeline (#2863888081) to validate Johannes Schindelin’s four-patch series fixing Windows CI breakage after Rust support was enabled. The series addresses missing Rust toolchain provisioning, silent data loss in `.git/info/exclude`, and linker unavailability for MinGW builds.
- **Test modernization regression**: Karthik Nayak reported that Mark C. Chu-Carroll’s v4 patch series modernizing `t40* diff-rename` tests breaks `t4001-diff-rename` by replacing `update-index --add a file.` with `update-index --add path0` but still referencing the non-existent file `a`. The patch also introduces an undefined helper function `initial_setup`.
- **`LESS=FRX` documentation**: Todd Zullinger proposed a documentation patch to `Documentation/config/core.adoc` clarifying the `LESS=""` workaround for users of third-party pager wrappers like `delta`. The patch addresses a long-standing usability issue where Git’s automatic `LESS=FRX` injection disrupts tools that internally chain to `less`.
- **`reflog expire` regression**: r.norouzi reported a regression in Git 2.50 where the default expiry times for reachable and unreachable reflog entries were swapped. Reachable entries (documented to default to 90 days) now expire after 30 days, while unreachable entries (documented to default to 30 days) are retained for 90 days. The bug was introduced by commit `85658275702b`.
- **`fetch.shallow` config**: Harald Nordgren submitted a patch introducing a new `fetch.shallow` configuration option (default: `false`) that restricts refspec-less fetches to the current branch’s upstream in shallow repositories. The feature addresses slow or hanging `git fetch`/`git pull` operations when the remote has many branches.