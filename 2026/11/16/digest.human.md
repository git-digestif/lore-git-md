# Git mailing list daily digest for 2026/11/16

## The day in brief

The Git mailing list saw significant progress on several key features and bugfixes today. The most consequential developments include: final approval for the `git repo info` path-related keys feature, confirmation that the `git branch --delete-integrated` squash-merge detection is ready for integration, and resolution of a critical regression in partial clones affecting `git index-pack`. Additionally, several long-running refactoring efforts (ODB abstraction, `the_repository` removal) took important steps forward.

## Notable threads

### 1. `[PATCH v9]` series adding path-related keys to `git repo info` (2026/07/16/01-21-31)

**What changed**: The final confirmation exchange in this series took place today, with K Jayatheerth acknowledging Phillip Wood's feedback about test efficiency but deferring optimization to a follow-up series. Phillip explicitly accepted this plan and reiterated his approval for merging the series as-is.

**Why it matters**: This feature exposes filesystem locations of repository components (working tree, index file, superproject working tree, etc.) in a user-facing, scriptable format. The new keys (`path.working_tree`, `path.superproject_working_tree`, `path.index`, and `path.prefix`) are particularly valuable for tooling that needs to interact with Git's internal paths without requiring direct access to Git's internals.

### Technical details

- The series is now fully approved and ready for merging
- All design objections have been resolved
- The implementation touches `builtin/repo.c`, `submodule.c`, `submodule.h`, `builtin/rev-parse.c`, and test files
- Keys support suffix-based formatting (e.g., `path.working_tree.absolute` vs `path.working_tree.relative`)
- The series includes a critical correctness fix for `get_superproject_working_tree()`

**Next steps**: Junio can queue this for merging without further changes.

---

### 2. `git branch --delete-integrated` squash-merge detection (2026/09/29/07-30-30)

**What changed**: Jasper Mayone confirmed that all three substantive issues raised in v6—performance inefficiency, correctness ambiguity, and edge-case robustness—are now resolved in v7. The patch builds cleanly, passes all tests, and has received a `Tested-by` trailer.

**Why it matters**: This feature extends `git branch --delete-merged` to recognize branches whose changes (but not their tips) have been integrated via squash merges, rebases, or patch-based workflows. This is a significant usability improvement for repository maintenance, particularly in workflows that use squash merges (common in many Git hosting platforms).

### Technical details

- The series provides `--delete-integrated` flag for change-based detection
- Handles upstream modifications via path-by-path three-way merge approximation
- Includes correct `.gitattributes` support and performance optimizations
- New test cases cover edge cases: unmerged, partially landed, and conflicting branches
- The implementation uses Git's low-level merge machinery (`ll_merge`)

**Next steps**: The patch is now technically complete and ready for integration.

---

### 3. Partial clone regression in `git index-pack` (2026/11/16/17-38-38)

**What changed**: A critical regression was reported in Git 2.56.0 where `git index-pack` triggers unnecessary nested lazy fetches for `REF_DELTA` bases already present in the thin pack. The author provided a reproduction script and identified the root cause in `fix_unresolved_deltas()`.

**Why it matters**: This bug affects blobless partial clones when the server doesn't support `OFS_DELTA`, leading to redundant network round-trips and potential unbounded recursion. The issue impacts production workflows and has been observed in real-world scenarios (e.g., Azure DevOps).

### Technical details

- Regression introduced in Git 2.56.0
- Affects `index-pack` in partial clones with `REF_DELTA` (no `OFS_DELTA`)
- Root cause in `fix_unresolved_deltas()` in `builtin/index-pack.c`
- Commit `4360734e90` caps recursion at 5 levels but doesn't fix the redundant fetch
- Workaround: explicitly fetch the commit before checkout

**Next steps**: This is a high-priority bug that will likely receive immediate attention from maintainers.

---

### 4. ODB abstraction: pluggable commit graphs (2026/11/16/13-07-36)

**What changed**: Junio C Hamano provided substantive review of the first patch in this 12-patch series, raising questions about error reporting conventions, the role of the `oid` field in `struct commit_graph`, and potential validation gaps.

**Why it matters**: This series is a foundational step in the ODB (object database) abstraction effort, which aims to make Git's storage layer pluggable. The ability to support alternative ODB implementations is crucial for Git's scalability and adaptability to different use cases.

### Technical details

- The series moves commit-graph storage logic into the ODB backend layer
- Introduces pluggable commit-graph callbacks (`read_commit_graph` and `write_commit_graph`)
- 12 patches refactor internal commit-graph handling
- Junio's review focuses on the first patch's error handling and validation
- The series is infrastructure-only with no user-facing features yet

**Next steps**: A v2 update is expected to address Junio's feedback.

---

### 5. `git worktree list` enhancements (2026/10/17/15-17-02)

**What changed**: Jasper Mayone agreed to drop the controversial `%(currentworktree)` patch and stop the series at patch 3/5, resolving the main point of contention. Phillip Wood provided substantive review of the dropped patch, questioning its necessity and implementation.

**Why it matters**: This series adds filtering capabilities to `git worktree list` and addresses safety issues with `git worktree remove`. The changes improve repository maintenance workflows, particularly for users with multiple worktrees.

### Technical details

- The series now consists of three uncontroversial patches:
  1. Safety fix for `git worktree remove` (refuses deletion of in-progress worktrees)
  2. Documentation update for `git-for-each-ref`
  3. Enhancement to `%(worktreepath)` (O(1) performance, excludes `rebase --update-refs` branches)
- The dropped `%(currentworktree)` patch was deemed redundant with `%(worktreepath)`
- Phillip Wood criticized the implementation as inefficient (using a hashmap for a single entry)

**Next steps**: The series is now effectively complete and ready for integration.

---

## In brief

- **[PATCH v3] credentialStore: align timeout naming with established convention** (2026/09/02/08-31-37): The follow-up patch for timeout naming consistency is uncontroversial and ready for integration, with all feedback incorporated.

- **`git imap-send` bugfix series** (2026/10/19/19-19-16): Patrick Steinhardt suggests a minor readability improvement to the CRLF-handling logic in the `has_required_headers()` helper function.

- **`git rerere` subcommand modernization** (2026/10/27/10-46-43): Junio has accepted the UI regression (enforcing option ordering) as a deliberate trade-off to fix what the author considers a design flaw.

- **Loose object database cache invalidation** (2026/11/04/12-16-33): Patrick Steinhardt acknowledges the performance impact of removing the `QUICK` flag but concludes that correctness must take precedence.

- **Git v3 object format bugfix** (2026/11/06/16-50-03): Patrick Steinhardt suggests minor documentation and test improvements for the already-merged patch.

- **`git history` verbose support** (2026/11/13/00-06-01): The author announces a pivot to make diff display the default behavior for `git history reword`, `split`, and `fixup`, controlled by the existing `commit.verbose` configuration.

- **`compatObjectFormat` interoperability fixes** (2026/11/12/19-57-37): Patrick Steinhardt provides substantive review of the third patch, questioning the handling of malformed commit headers and proposing a standardized prefix for custom headers.

- **Promisor-remote crash fix** (2026/11/13/00-06-01): Junio agrees with Patrick's preference for checking `!repo` rather than `!repo->gitdir` but accepts the minimal fix as a short-term solution.

- **`git repo structure` performance improvements** (2026/11/13/11-46-16): The series is marked ready for `next` after addressing all review feedback, with impressive memory reduction (80%) and performance improvements (20%).

- **`for-each-ref --no-format` segfault fix** (2026/11/13/16-14-10): The patch is reviewed and approved, fixing a long-standing crash that has existed since 2007.

- **Test modernization** (2026/11/15/18-15-21): Junio approves the patch to replace `!` with `test_must_fail` in test scripts, noting that other instances remain for future work.

- **ANSI color handling in `git log --format`** (2026/11/15/21-35-03): Junio endorses the runtime approach to fix color loss across line breaks, acknowledging the API gap in `%w` and `%b`.

- **`git log --follow=merges` option** (2026/11/16/07-35-50): The v2 patch incorporates feedback, improving error handling and documenting the performance trade-off.

- **UTF-8 handling fixes** (2026/11/16/18-56-59): A two-patch series fixes bounds handling and documentation in `utf8_strnwidth()`.

- **Submodule linked worktree crash** (2026/11/16/22-51-48): A crash (BUG()) is reported in `validate_submodule_legacy_git_dir()` when a submodule is itself a linked worktree.