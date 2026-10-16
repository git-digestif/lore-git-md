# Git mailing list daily digest for 2026/10/15

## The day in brief
Git's CI infrastructure saw significant modernization with Patrick Steinhardt's eight-patch series landing in `next`, while Miklos Vajna proposed a fix for a severe `git log --follow` performance regression. Documentation improvements and usability features also advanced, with Junio marking several series ready for integration. The day's discussions revealed deeper architectural questions about refspec handling, reflog usability, and the ongoing `the_repository` removal effort.

## Notable threads

### CI infrastructure modernization lands in `next`
**What changed**: Patrick Steinhardt's eight-patch series modernizing Git's CI configurations was marked ready for `next` after addressing all substantive feedback. The series removes outdated dependencies (Python 2, EOL Docker images), eliminates redundant jobs, and improves test coverage for reftable and SHA-256 combinations.

**Why it matters**: The changes reduce CI complexity while maintaining or improving coverage, particularly for the SHA-256 transition. The series also surfaced a latent issue where Rust had been accidentally disabled in CI for six months, which will be addressed in follow-up work.

**Technical details**: Key changes include:
- Repurposing the `linux-reftable-leaks` job as `linux-reftable-sha256` to test SHA-256 + reftable
- Renaming `linux-TEST-vars` to `linux-feature-testbed`
- Updating 32-bit CI to use `i386/debian:latest` instead of EOL Ubuntu
- Removing Python 2 logic from CI scripts
- Fixing a test in `t/t5004-archive-corner-cases.sh` to work in both SHA-1 and SHA-256 repositories

**Status**: Marked for `next` by Junio C Hamano.

---

### Severe performance regression in `git log --follow`
**What changed**: Miklos Vajna proposed a fix for a performance regression in `git log --stat --follow` that slowed the command from 33 seconds to nearly 8 minutes. The regression was introduced by commit 304813ed33 ("log: improve --follow following renames for non-linear history").

**Why it matters**: The regression affects users tracing file history in repositories with complex merge structures, particularly the Linux kernel. The fix restores near-original performance while preserving the corrected rename-following behavior.

**Technical details**: The patch introduces two optimizations in `propagate_follow_pathspec_to_parent()`:
1. Only perform rename detection when the followed path is *added* relative to a parent
2. If the path is unchanged in any parent, follow that parent's history without further diffing

The implementation adds a new helper function, `get_follow_path_change()`, which classifies how the followed path differs between a commit and its parent (unchanged, added, or other).

**Status**: Under review. Junio C Hamano proposed an alternative architectural approach that would move the optimization logic into `log_tree_diff_flush()` to avoid redundant diff passes.

---

### `git repo structure` revision filtering ready for `next`
**What changed**: Mark C. Chu-Carroll's patch adding revision-based filtering to `git repo structure` was marked ready for `next` after addressing all prior feedback. The feature allows users to analyze specific subsets of the repository object graph.

**Why it matters**: The change improves diagnostic capabilities for large repositories by enabling targeted analysis without requiring full object graph traversal. This is particularly useful for performance debugging in monorepos.

**Technical details**: The patch:
- Uses Git's standard `setup_revisions()` machinery
- Adds a parameter to `count_references()` to avoid redundant ref-walks
- Touches `builtin/repo.c`, `Documentation/git-repo.adoc`, and `t/t1901-repo-structure.sh`
- Expands test coverage to 197 lines, validating edge cases like overlapping includes/excludes

**Status**: Ready for `next` after v7 addressed the final editorial nit.

---

### Incremental connectivity check for fetch/push
**What changed**: Kristofer Karlsson's four-patch series introducing an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) saw significant review progress. The feature reduces the cost of verifying object connectivity during fetches and pushes.

**Why it matters**: The existing full check scales poorly with repository size. The new mode processes incoming commits in topological order, shifting cost from repository size to incoming change size. Benchmarks show up to 190x speedup for small pushes to large repos.

**Technical details**: Key components:
- New config key: `transfer.connectivityCheck=incremental`
- New internal `rev-list` flag: `--verify-trees-incremental`
- New source file: `tree-verify.c` (316 lines) implementing the inductive trust algorithm
- Performance test suite: `t/perf/p5413-connectivity-check.sh` (92 lines)

**Status**: Under active review. Junio signaled strong interest, and Patrick Steinhardt provided detailed feedback on implementation details, including type usage, error handling, and stack overflow risk.

---

### `includeIf.hostname:<pattern>` implementation
**What changed**: Isabella Caselli posted v2 of a three-patch series implementing `includeIf.hostname:<pattern>` in Git's configuration system. The feature allows users to conditionally include configuration files based on the machine's hostname.

**Why it matters**: This addresses the use case of dotfile sharing across multiple machines, where per-host configuration is needed. The implementation aligns with hosting platform conventions and provides a debugging aid via `git var GIT_HOSTNAME`.

**Technical details**: The series:
1. Adds `git var GIT_HOSTNAME` as a debugging aid
2. Fixes a test reliability issue in `t/t6500-gc.sh`
3. Implements the core `includeIf.hostname:<pattern>` logic with glob wildcard support

**Status**: Under review. Junio described the first patch as "nicely written" and "sensible," and the third patch's implementation as "neat" and "quite nice."

---

### Detached HEAD documentation rewrite
**What changed**: Julia Evans posted v3 of a documentation rewrite for Git's detached HEAD state, addressing four specific shortcomings in the current text.

**Why it matters**: The current documentation doesn't explain *why* detached HEAD is useful, takes too long to define the concept, doesn't clarify how users might end up in this state accidentally, and includes overly complex diagrams. The rewrite moves the explanation to a new standalone guide (`gitdetached-head(7)`).

**Technical details**: The patch:
- Replaces the existing section in `git-checkout.adoc`
- Adds a new standalone guide (`gitdetached-head(7)`)
- Updates advice messages to reference the new guide
- Touches `Documentation/git-checkout.adoc`, `advice.c`, and build system files

**Status**: Under review. Junio proposed further improvements to the introductory definition of HEAD and the explanation of detached HEAD's advantages.

---

### Reflog usability improvements
**What changed**: Harald Nordgren proposed a three-patch series adding `--operations` and `--historical-branch` options to `git reflog` and `git log -g` to improve usability.

**Why it matters**: The current reflog shows every intermediate step of operations like rebase or branch switching, making it difficult to see the actual commands executed and which branch they affected. The new options address long-standing pain points in reflog usability.

**Technical details**: The series:
1. Refactors hard-coded strings used to mark branch switches, renames, and rebase steps
2. Implements `--operations` to collapse multi-step operations into single entries
3. Implements `--historical-branch` to show which branch HEAD was on when each entry was recorded

**Status**: Under review. Kristoffer Haugsbakk raised questions about scope and design trade-offs, noting that `tig reflog` already mitigates some of the pain points the series aims to address.

---

### Loose-ref iterator positioning fix
**What changed**: Venkat Ragavan posted a bugfix for a correctness issue in `cache_ref_iterator_seek()` where loose-ref iteration can misposition when a directory name shares a prefix with a ref name.

**Why it matters**: The bug can cause `git` to return incorrect ref iteration results, potentially affecting any command that iterates over refs. The fix ensures directory components are compared with their trailing slash included.

**Technical details**: The patch:
- Fixes the logic in `refs/ref-cache.c`
- Adds regression tests covering loose, packed, and mixed ref storage
- Handles edge cases like shared-prefix names and missing nested markers

**Status**: Under review. Junio clarified the test structure, explaining the conditional setup is intentional and necessary for testing mixed storage modes.

---

### `git format-rev` deprecation proposal
**What changed**: Kristoffer Haugsbakk proposed deprecating the experimental `git format-rev` command, arguing its core functionality is already provided by `git diff-tree --stdin --format=medium`.

**Why it matters**: The proposal highlights a functional gap in the replacement pipeline: `git diff-tree` does not support NUL termination for the `--format=medium` pretty-printing mode, which breaks handling of multiline commit messages or revision expressions.

**Technical details**: The discussion revealed:
- `git format-rev` offers minor conveniences like direct revision expressions and object-name annotation
- The replacement pipeline cannot handle NUL-terminated input for multiline content
- `tig` provides similar functionality, raising questions about scope

**Status**: Under discussion. The proposal remains at the conceptual stage.

---

## In brief
- **`git worktree repair` bugfix**: Yoichi NAKAYAMA's five-patch series fixing incorrect repair of unrelated worktrees saw substantive review from Junio, who probed the robustness of `get_worktree_id()` in handling non-canonical path separators.
- **`git fetch --all --depth` regression**: Junio raised concerns about Rong Bao's bugfix, questioning whether the same propagation issue exists for submodule fetches and whether the patch only propagates `--depth` but not other deepening options.
- **`git pull --rebase` and custom fetch refspecs**: Jyotish Kumar confirmed the issue is not yet being addressed by any in-flight topic and is still in the investigation phase.
- **`core.indexLockTimeout`**: Jun Wu proposed a configurable timeout for Git's index lock to reduce contention between background IDE calls and user-initiated commands. The discussion revealed a pre-existing race condition in `git commit` that the patch also fixes.
- **Coverity-inspired integer-overflow fixes**: Johannes Schindelin posted a five-patch series hardening Git against integer overflows. Phillip Wood raised concerns about the portability of the new `mult_overflows` macro.
- **`git help user-manual` bug**: John Wilson reported a bug on Git for Windows where `git help user-manual` attempts to open `gituser-manual.html` instead of the correct filename `user-manual.html`. Junio confirmed the issue is upstream and identified the necessary fixes.
- **`git describe` l10n update**: Raj Ojha posted a patch updating two debug messages in `builtin/describe.c` to use the `Q_()` pluralization macro.
- **`the_repository` removal**: Okhuomon Ajayi posted a refactoring patch advancing the ongoing `the_repository` removal effort by updating two functions in `merge.c` to use an explicitly passed `struct repository *`.
- **Outreachy introductions**: Keryle Fondji introduced themselves as an Outreachy applicant for the December 2026 cohort, announcing their project to improve Git command argument parsing.
- **Cache-tree crash fix**: Szymon Wilczek posted a bugfix preventing `git rebase` from crashing when another process holds the index lock during an in-process commit attempt.