# Git mailing list daily digest for 2026/09/20

## The day in brief

The Git mailing list saw significant activity around ref backend fixes, with Maciej Ciemborowicz submitting patches to address long-standing issues in the `reference-transaction` hook. A new advice message for shallow clones was proposed, and an RFC for worktree lifecycle hooks sparked discussion. Meanwhile, the GitLab CI fix for Windows builds saw validation progress.

## Notable threads

### Fix GitLab CI breakage after Rust enabled for Windows
**What changed?** Johannes Schindelin reported that the `build:mingw64` job in GitLab CI succeeded as planned, confirming that the v1 patch series restores pipeline functionality for Windows builds after Rust support was enabled. The series ensures the correct Rust toolchain is provisioned, preserves `.git/info/exclude` contents, and makes `cargo` accessible in the MinGW environment.

**Why it matters:** This CI/build system fix is critical for maintaining Windows build stability in GitLab’s MinGW environment. The successful job run increases confidence in the series, though Johannes noted the job name may be outdated (`ucrt64` would be more accurate). An unrelated timeout in the `build:msvc-meson` job was observed but does not affect the core fix.

**Today’s developments:** The `build:mingw64` job validation ([2026/09/20/15-27-06 by Johannes Schindelin]) marks a key milestone, moving the series from "proposed" to "tested." Karthik Nayak’s internal GitLab MR (#671) and pipeline (#2863888081) remain the primary validation artifacts.

---

### `reference-transaction` hook omits new ref on branch rename
**What changed?** Maciej Ciemborowicz submitted a patch ([2026/09/20/16-50-37]) to represent branch rename and copy operations as transactions, ensuring the `reference-transaction` hook receives both the deletion of the old ref and the creation of the new ref in the same transaction. The fix introduces a new transaction type (`REF_TRANSACTION_TYPE_RENAME`/`COPY`) and a `REF_TRANSACTION_FLAG_SKIP_HOOK` flag to suppress hooks for nested operations.

**Why it matters:** This bugfix addresses a long-standing inconsistency where `git branch -m` (rename) and `git update-ref --stdin` behaved differently, omitting the new ref from the hook’s view. The patch ensures tools relying on the hook (e.g., Gerrit, GitLab) see the full picture of ref changes. Test coverage includes rename, copy, forced updates, D/F conflicts, and concurrent updates.

**Today’s developments:** The patch submission ([2026/09/20/16-50-37 by Maciej Ciemborowicz]) provides a comprehensive solution to the backend-specific behavior (files vs. reftable) that caused the issue. The unified transaction approach aligns with Git’s existing patterns and is likely to be uncontroversial.

---

### Regression in `reference-transaction` hook: all-zero OIDs on deletion
**What changed?** Maciej Ciemborowicz posted v2 of a three-patch series ([2026/09/20/10-54-19]) to restore the correct behavior of the `reference-transaction` hook when branches, tags, or remote refs are deleted. The series addresses a regression introduced in Git 2.31, where the hook received all-zero OIDs instead of the resolved old OID. The v2 patches explicitly document the intentional trade-off of conditional deletion (safety) over unconditional deletion, preventing concurrent updates from being silently overwritten.

**Why it matters:** The regression affected external tools monitoring ref changes, breaking compatibility with Git 2.28–2.30 behavior. The v2 series preserves the performance gains of the original optimization (bulk deletion of 10,000 packed tags remains at ~0.85 seconds) while restoring the pre-regression safety check. The patches touch the refs API (`refs_delete_refs()`), built-ins (`git branch`, `git tag`, `git fetch --prune`), and include comprehensive test coverage.

**Today’s developments:**
- Maciej clarified ([2026/09/20/10-38-55]) that the patch deliberately changes deletions from unconditional to compare-and-delete, providing a reproduction case where the patched version fails the old-OID check when a hook modifies the branch during the "preparing" phase.
- The v2 series ([2026/09/20/10-54-19]) documents the race condition trade-off and adds tests for concurrent updates. The first patch ([2026/09/20/10-54-20]) modifies `refs_delete_refs()` to accept an optional `oid_array` of old OIDs, making deletions conditional if an OID is provided. The second ([2026/09/20/10-54-21]) and third ([2026/09/20/10-54-22]) patches update the callers and fix a UI inconsistency in `git remote prune`.

**Open question:** Whether the project accepts the intentional trade-off of conditional deletion (safety) over unconditional deletion (simplicity) remains unresolved.

---

### Advice for shallow history in `<ref>~N` and `<ref>^N`
**What changed?** Harald Nordgren submitted a patch ([2026/09/20/09-53-33]) adding a new advice message (`advice.shallowHistory`) to explain why `<ref>~N` or `<ref>^N` fails in a shallow clone and suggest the exact `git fetch --deepen=<n>` command needed. The advice fires only when the walk stops at a recorded shallow boundary and provides context-specific `--deepen` values for `~N` and `^N` syntax.

**Why it matters:** This user-experience improvement addresses a common pain point in shallow clones, where users receive a generic "is not a commit" error with no hint about the repository’s shallow state or how to fetch more history. The patch avoids false positives and provides actionable suggestions, such as deepening by the exact number of commits needed.

**Today’s developments:**
- The patch submission ([2026/09/20/09-53-33 by Harald Nordgren]) includes thorough test coverage for `~N`, `^N`, merge parents, partial depth, and disabled advice.
- D. Ben Knoble noted ([2026/09/20/21-59-59]) an inconsistency in the patch’s messaging, where the commit message and documentation use "ref" and "commit-ish" interchangeably, potentially confusing users. The review also suggested using `<rev>~<n>` and `<rev>^[<n>]` in documentation to match `gitrevisions(7)`.

---

### RFC: Native worktree lifecycle hooks
**What changed?** Maciej Ciemborowicz proposed ([2026/09/20/17-33-33]) adding native Git hooks for worktree operations (`add`, `remove`, `move`, `lock`, `unlock`, `prune`) to replace indirect detection via `post-checkout` or external wrappers. The motivation is to enable tools like `git-hooks-ext` to observe worktree changes without wrapping `git worktree` or diffing `git worktree list --porcelain` output.

**Why it matters:** This feature would provide a cleaner, more reliable way for external tools to monitor worktree lifecycle events. The proposal presents two design options: separate post-operation hooks (e.g., `post-worktree-add`) or a unified `worktree-lifecycle` hook (similar to `reference-transaction`). The author leans toward the latter for operations like `prune` that affect multiple worktrees at once.

**Today’s developments:**
- The RFC ([2026/09/20/17-33-33 by Maciej Ciemborowicz]) asks whether a stable worktree identifier (beyond the path) should be exposed, as paths can change during `move` or `repair`.
- Kristoffer Haugsbakk linked ([2026/09/20/18-25-45]) to a prior discussion (2024) where a similar proposal was debated, providing historical context for the current RFC.

## In brief
- **GitLab CI Windows build validation**: The `build:mingw64` job succeeded, confirming the fix for Rust toolchain provisioning in GitLab’s MinGW environment. Johannes Schindelin noted the job name may be outdated and observed an unrelated timeout in the `build:msvc-meson` job ([2026/09/20/15-27-06]).
- **Branch rename/copy transactions**: Maciej Ciemborowicz submitted a patch to represent branch rename and copy operations as transactions, ensuring the `reference-transaction` hook receives both deletion and creation events. The patch introduces a new transaction type and flag for nested operations ([2026/09/20/16-50-37]).
- **Reference-transaction hook regression fix (v2)**: Maciej Ciemborowicz posted v2 of a three-patch series restoring the correct behavior of the hook on ref deletion. The series documents the intentional trade-off of conditional deletion (safety) over unconditional deletion and includes comprehensive test coverage ([2026/09/20/10-54-19]).
- **Shallow clone advice**: Harald Nordgren submitted a patch adding a new advice message for shallow history in `<ref>~N` and `<ref>^N` lookups. D. Ben Knoble noted an inconsistency in the patch’s messaging and documentation ([2026/09/20/09-53-33], [2026/09/20/21-59-59]).
- **Worktree lifecycle hooks RFC**: Maciej Ciemborowicz proposed adding native Git hooks for worktree operations, presenting two design options. Kristoffer Haugsbakk linked to a prior discussion on the topic ([2026/09/20/17-33-33], [2026/09/20/18-25-45]).