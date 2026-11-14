# Git mailing list daily digest for 2026/11/13

## The day in brief
The Git mailing list saw significant progress on several key fronts today: a regression fix for `git cat-file --batch` with invalid reflog references, resolution of a long-standing design question in the promisor-remote fetch ordering RFC, and multiple performance optimizations for repository analysis tools. Documentation improvements for detached HEAD state and governance processes also advanced, while several bugfixes and feature implementations moved closer to integration.

## Notable threads

### Bugfix series for `git cat-file --batch` inconsistency
**What changed**: Caspar Duregger posted a four-patch series addressing inconsistent error handling in `git cat-file --batch` when processing invalid reflog references. The series systematically applies `GET_OID_GENTLY` to make error handling uniform, fixing a regression introduced by commit 393bbb21 where out-of-bounds reflog entries triggered fatal errors instead of graceful failures.

**Problem/goal**: The regression caused `git cat-file --batch` to crash on references like `branch@{20}` when only 5 entries exist, while other invalid references were handled gracefully. The series also fixes related edge cases where failed peel suffixes could be misinterpreted as reflog selectors.

**Subsystem**: Object name lookup (`object-name.c`, `refs.c`, `builtin/cat-file.c`)

**Kind of change**: Bugfix with test coverage

**Technical impact**: The series restores Git's philosophy of graceful error reporting in batch operations while maintaining backward compatibility. Patch 1 prevents parsing ambiguities, Patch 2 refines `GET_OID_GENTLY` behavior, Patch 3 handles empty reflogs gracefully, and Patch 4 enables the flag in `cat-file`.

**Today's development**: The author necroposted the complete series with thorough commit messages and test coverage for reflog selectors by index, date, and peel suffixes. The patches are uncontroversial and address a real regression with clear technical improvements.

---

### `git history` merge-commit RFC: boundary commit behavior resolved
**What changed**: Philippe Blain retracted his earlier demand for a code fix after discovering that the pre-seeding logic mapping negative revisions to the `--onto` target is intentional and documented in Elijah Newren's upstream branch.

**Problem/goal**: The thread addresses how `git history` handles boundary commits (e.g., `M^!`) during merge commit replay. The behavior was initially perceived as a regression but is now recognized as a design choice that forces explicit ancestry constraints.

**Subsystem**: Rebase/replay logic (`replay.c`)

**Kind of change**: Documentation improvement and design clarification

**Technical impact**: The resolution removes a blocker for the RFC series while maintaining the existing behavior. Blain now proposes documentation improvements to `git replay`'s man page explaining that negative revisions are mapped to `onto` and that `--ancestry-path` is required for partial-parent rewrites.

**Today's development**: Blain's retraction and documentation proposal effectively resolve the regression concern, allowing the thread to focus on implementation details rather than design questions. The author, Johannes Schindelen, is now free to proceed with the technical work of integrating Phillip Wood's single-parent-rebase optimization.

---

### `git fetch` refspec matching: override semantics resolved
**What changed**: Phillip Wood explicitly confirmed that command-line refspecs must override `remote.<name>.fetchRefmap` entirely, not coexist with it, resolving the ambiguity that blocked the v13 series.

**Problem/goal**: The series implements a "matching-inspired" fetch mode that dynamically fetches only branches tracked by local branches at the remote. The design question centered on how command-line refspecs should interact with the new `fetchRefmap` configuration.

**Subsystem**: Remote/fetch logic (`remote.c`)

**Kind of change**: Feature implementation with architectural implications

**Technical impact**: The resolution enables the series to proceed with a clear design direction. The v14 implementation will update `remote.c` and tests to reflect override semantics, where command-line refspecs completely replace `fetchRefmap` rather than merging with it.

**Today's development**: Phillip's confirmation removes the last major ambiguity before graduation. The series remains under strategic evaluation for partial merge proposals, but the core design question is now settled.

---

### Detached HEAD documentation improvements
**What changed**: Julia Evans proposed targeted wording changes to address Junio C Hamano's concern that the documentation omits the "most important consequence" of detached HEAD: its utility for making experimental commits without affecting any branch.

**Problem/goal**: Improve Git's documentation for detached HEAD state to better serve users who accidentally enter this state and are confused about their repository's status.

**Subsystem**: Documentation (`gitdetached-head(7)`)

**Kind of change**: Documentation improvement

**Technical impact**: The proposed addition—"You can also use this to make experimental commits without being on a branch."—directly incorporates maintainer feedback while maintaining the document's accessibility focus. Evans also proposed reframing the series as a standalone guide (`git help detached-head`) to resolve tensions between beginner and advanced documentation needs.

**Today's development**: The proposed changes are minimal, uncontroversial, and ready for merging. They address a specific gap in the existing documentation without reopening broader philosophical debates about documentation scope.

---

### `git worktree` safety checks and ref-filter atoms
**What changed**: Junio C Hamano raised substantive objections to the final patch in Jasper Mayone's series, questioning the value of suppressing `+` markers and paths in `git branch` output for rebasing/bisecting branches in the current worktree.

**Problem/goal**: The series extends `git worktree list` filtering capabilities and adds safety checks to prevent deletion of worktrees in the middle of operations like rebase or bisect. The final patch uses the new `%(currentworktree)` atom to clean up `git branch` output.

**Subsystem**: Worktree management and ref-filter

**Kind of change**: Feature implementation with user-facing behavior changes

**Technical impact**: Junio's objections focus on whether the suppression of `+` markers (used to identify bisected branches) constitutes a regression for users who rely on them. He suggests the series might be better off stopping at patch 3/5, which implements the core safety checks and performance improvements.

**Today's development**: The maintainer's feedback creates a strategic decision point for the series. The author must either provide a stronger rationale for the cleanup or drop the final patch to proceed with the uncontroversial portions.

---

### Promisor-remote fetch ordering RFC: design question resolved
**What changed**: Christian Couder acknowledged that the current patch's static client-side order cannot handle multi-remote scenarios where different remotes advertise different sets of promisor remotes, effectively blocking the RFC until a more dynamic solution is designed.

**Problem/goal**: Allow users to specify the order in which Git tries multiple promisor remotes during lazy fetches, replacing the current implicit order (configuration file order).

**Subsystem**: Partial clone/promisor-remote

**Kind of change**: Feature RFC with protocol implications

**Technical impact**: The acknowledgment shifts the thread from implementation to design. The current approach cannot handle cases where different remotes (A, B) advertise different promisor remotes (A1/A2 vs. B1/B2/B3), making a static client-side order insufficient.

**Today's development**: The author, Yopa Nelly, agreed to treat the patch as an RFC prototype and pause work until Outreachy selections are finalized. The thread now awaits proposals for a more dynamic or context-aware mechanism to handle the multi-remote scenario.

---

## In brief

- **[PATCH] promisor-remote: fix crash in `git ls-remote` outside repository**: Christian Couder posted a fix for a crash when `git ls-remote` is invoked outside a repository with promisor-remote configuration. Junio proposed an alternative approach making `promisor_remote_reply()` a no-op when no repository is present.

- **[PATCH] http-backend: remove unnecessary pre-sizing in get_info_packs()**: Yusuke Hayashi removed an unnecessary and incorrect pre-sizing step in `get_info_packs()`. Patrick Steinhardt challenged the justification, asking whether the performance trade-off (dynamic `strbuf` growth vs. a second loop) has been properly evaluated.

- **[PATCH 0/2] repo structure: performance and memory-usage improvements**: Patrick Steinhardt posted a two-patch series improving `git repo structure` performance by avoiding unnecessary object parsing. Kristofer Karlsson endorsed the approach, noting 80% memory reduction and 20% speedup.

- **[PATCH] for-each-ref: fix segfault with --no-format**: aeron-gh fixed a long-standing crash in `for-each-ref` and `git refs list` when invoked with `--no-format`. Junio confirmed the bug and leaned toward making `--no-format` consistently restore the default format across all ref-listing commands.

- **[PATCH 0/2] promisor-remote: add promisor.fetchOrder config**: The Outreachy project to implement fetch ordering for promisor remotes paused work after Christian Couder identified a fundamental design limitation with the static client-side order approach.

- **[PATCH v2] commit: shrink struct commit by reordering fields**: Kristofer Karlsson's memory optimization patch moved closer to integration after Junio finalized the last clarity nit in the `struct commit` comment block.

- **[PATCH] refs.h: remove unused peel_iterated_oid() declaration**: Yusuke Hayashi's cleanup patch was acknowledged by Patrick Steinhardt and is ready for integration.

- **[PATCH] partial-clone: clarify fetch ordering for promisor remotes**: Hussain Sitamouwala confirmed the documentation patch is queued and outlined plans for the next phase of the Outreachy project: client-side configuration of fetch ordering.

- **[PATCH] fetch: introduce `+:` refspec for matching-inspired fetch mode**: The v14 implementation remains under strategic evaluation for partial merge proposals, with Matt Hunter's clarification that merging Patches 1-3 would resolve conflicts with a dependent topic.

- **[PATCH] doc: rewrite explanation of detached HEAD state**: Julia Evans proposed reframing the series as a standalone guide (`git help detached-head`) to resolve tensions between beginner and advanced documentation needs.

- **[PATCH 0/3] promisor-remote: fix protocol v2 capability reply for fetch**: Christian Couder proposed sending a v2 with a `NEEDSWORK` comment to document the need for future refactoring, while Junio pushed back on deferring the cleanup of client-side capability-writing functions.

- **[PATCH] fmt-merge-msg: add configuration variables to git-config(1)**: YikunWei added the `fmt-merge-msg` configuration section to the main `git-config(1)` man page to address a discoverability gap.

- **[PATCH 0/3] repo structure: performance and memory-usage improvements**: Patrick Steinhardt's two-patch series for `git repo structure` received positive feedback from Kristofer Karlsson, who noted significant performance improvements.

- **[PATCH] for-each-ref: fix segfault with --no-format**: aeron-gh's bugfix for a long-standing crash in `for-each-ref` moved closer to resolution after Junio leaned toward making `--no-format` consistently restore the default format.

- **[PATCH 0/2] promisor-remote: add promisor.fetchOrder config**: The Outreachy project paused work after Christian Couder identified a fundamental design limitation with the static client-side order approach.

- **[PATCH 0/2] repo structure: performance and memory-usage improvements**: Patrick Steinhardt's series for `git repo structure` received substantive review from Kristofer Karlsson, who endorsed both patches and noted measurable performance improvements.