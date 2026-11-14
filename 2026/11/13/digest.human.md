# Git mailing list daily digest for 2026/11/13

## The day in brief
The Git mailing list saw significant progress on several key topics today. A critical bugfix for `git cat-file --batch` with invalid reflog references was necroposted, addressing a long-standing regression. The `git ls-remote` promisor-remote crash fix sparked a design discussion about repository context handling. Documentation improvements for detached HEAD state continued to evolve, with a proposed minimal wording change to address maintainer concerns. Several performance optimizations and refactoring efforts made progress, including memory usage improvements in the commit subsystem and optimizations for `git repo structure`.

## Notable threads

### Bugfix for `git cat-file --batch` with invalid reflog references
**What changed:** Caspar Duregger necroposted a four-patch series that systematically applies `GET_OID_GENTLY` to make `git cat-file --batch` error handling uniform for invalid reflog references, addressing a regression introduced by commit 393bbb21.

**Problem/goal:** The regression caused `git cat-file --batch` to fatally error on out-of-bounds reflog entries (e.g., `branch@{20}` when only 5 exist), while other invalid references were handled gracefully.

**Subsystem:** Object name lookup machinery (`object-name.c`, `refs.c`, `builtin/cat-file.c`)

**Kind of work:** Bugfix with test coverage

**Impact:** This fix restores consistent error handling behavior across reference lookups, preventing crashes in batch processing scenarios. The series also addresses related edge cases like parsing ambiguities and empty reflogs.

**Today's development:** The author posted the complete series with thorough commit messages and test coverage. The patches are well-structured, addressing both the immediate regression and related inconsistencies in the object-name lookup machinery.

### `git ls-remote` promisor-remote crash fix
**What changed:** Christian Couder posted a patch to fix a crash in `git ls-remote` when invoked outside any Git repository with promisor-remote configuration enabled.

**Problem/goal:** The crash occurs because `filter_promisor_remote()` dereferences `repo->gitdir` without checking for NULL, a regression introduced in 2026.

**Subsystem:** Promisor-remote subsystem

**Kind of work:** Bugfix

**Impact:** This fix prevents a segmentation fault that could occur when running `git ls-remote` outside a repository with certain promisor-remote configuration options enabled.

**Today's development:** Junio C Hamano proposed an alternative approach that makes `promisor_remote_reply()` a no-op when no repository is present. The discussion now centers on which approach is preferable: the minimal fix at the crash site or the higher-level no-op. Both solutions prevent the crash, but Junio's alternative avoids calling `filter_promisor_remote()` entirely when outside a repository.

### Detached HEAD documentation improvements
**What changed:** Julia Evans proposed several documentation improvements for the detached HEAD state explanation.

**Problem/goal:** The documentation needed to better explain the practical consequences of detached HEAD, particularly its utility for making experimental commits without affecting any branch.

**Subsystem:** Documentation

**Kind of work:** Documentation improvement

**Impact:** These changes will help users understand when and why to use detached HEAD, particularly for temporary experimental work.

**Today's development:** Julia proposed a minimal wording change - "You can also use this to make experimental commits without being on a branch." - to address Junio's concern that the documentation omitted the most important consequence of detached HEAD. She also proposed reframing the series as a standalone guide (`git help detached-head`) specifically aimed at users confused by detached HEAD, rather than trying to serve both the `git-checkout` manual page and a beginner-friendly guide simultaneously.

### Memory optimization for `struct commit`
**What changed:** Kristofer Karlsson's patch to reduce memory usage in the commit subsystem by reordering fields in `struct commit` received final maintainer approval.

**Problem/goal:** Eliminate padding in `struct commit` on 64-bit platforms to reduce memory usage by 10% (from 80 to 72 bytes).

**Subsystem:** Commit subsystem

**Kind of work:** Performance optimization

**Impact:** This change reduces memory usage for repositories with many commits, with a measured 32 MiB reduction in peak RSS for `git rev-list --all` on a 4.5-million-commit monorepo.

**Today's development:** Junio C Hamano gave the final "Will queue" approval, indicating the patch is ready for integration. The patch has been thoroughly reviewed and all feedback incorporated, including explanatory comments for the `index` and `date` fields and clarification of casting directions.

### `git repo structure` performance improvements
**What changed:** Patrick Steinhardt posted a two-patch series improving performance and reducing memory usage of `git repo structure`.

**Problem/goal:** The current implementation parses every object it encounters, which is prohibitively expensive in repositories with tens of billions of tree entries.

**Subsystem:** `git-repo` built-in

**Kind of work:** Performance optimization

**Impact:** The patches achieve an 80% memory reduction (5.1× less memory) and 20% wall-time improvement by delaying parsing until necessary and skipping full tree parsing when only the object type is required.

**Today's development:** Kristofer Karlsson provided substantive review, endorsing both patches and noting the significant performance improvements. The review confirmed that the optimization aligns with prior work on incremental connectivity checks.

## In brief

- **[`git fetch` refspec override semantics]**: Phillip Wood explicitly confirmed that command-line refspecs must override `remote.<name>.fetchRefmap` entirely, resolving the ambiguity that blocked the v13 series.
- **[`git worktree list` improvements]**: Junio questioned the value of suppressing `+` markers and paths in `git branch` output for rebasing/bisecting branches, suggesting the series might be better off stopping at patch 3/5.
- **[`git rerere` subcommand handling]**: Post-merge discussion continued about workflow improvements, with Phillip Wood providing a tip for Junio's rebase workflow using `--keep-base`.
- **[`promisor.fetchOrder` design discussion]**: Christian Couder argued that the current patch's static client-side order cannot handle multi-remote scenarios where different remotes advertise different sets of promisor remotes.
- **[`--show-object-format` for `git ls-remote`]**: Patrick Steinhardt approved the v2 patch, signaling readiness for queueing.
- **[Git v3 default object format]**: brian m. carlson posted a revised patch that ensures `git init` correctly defaults to SHA-256 when Git is built with `WITH_BREAKING_CHANGES=1`.
- **[`git-config` documentation]**: YikunWei added the `fmt-merge-msg` configuration section to the main `git-config(1)` man page to improve discoverability.
- **[`for-each-ref --no-format` bugfix]**: Junio leaned toward making `--no-format` consistently restore the default format across all ref-listing commands.
- **[`git repack --filter-to-loose`]**: Adithya Chakilam posted a two-patch series introducing a new option to improve interoperability with key-value object stores.
- **[Loose object database cache invalidation]**: Junio confirmed integration of the fix for stale cache entries that were causing existence check failures in partial clones.