# Git mailing list daily digest for 2026/11/10

## The day in brief
The Git project addressed a critical security release (Git 2.56.1 and Git for Windows 2.56.1) fixing six CVEs, including remote code execution and path traversal vulnerabilities. Major refactoring efforts continued with the `the_repository` removal initiative making progress in multiple subsystems. Performance optimizations for large repositories and CI systems were prominent, with new configuration options to disable expensive checks. Documentation improvements and test modernization efforts advanced, particularly for beginner tutorials and the `fast-import` test suite.

## Notable threads

### Git 2.56.1 Security Release
**What changed**: Junio C Hamano announced Git 2.56.1 and other maintenance releases addressing six security vulnerabilities (CVE-2026-111760, CVE-2026-113408, CVE-2026-114467, CVE-2026-114469, CVE-2026-105437, CVE-2026-114466) discovered by Luke Martin and pwn2addr. Johannes Schindelin simultaneously announced Git for Windows 2.56.1 with the same fixes.

**Problem/goal**: The vulnerabilities included remote code execution via malicious submodule gitfiles, path traversal in the files ref backend, stack corruption from multi-signature commits, out-of-bounds reads from oversized binary patches, heap corruption during large merges, and unsafe `fast-import` features.

**Impact**: These vulnerabilities affect all users of Git, with potential impacts ranging from denial-of-service to arbitrary code execution. The fixes are backported to all supported maintenance tracks (2.56.1 down to maint-2.43).

**Technical details**: The fixes span multiple subsystems:
- xdiff: Added size_t overflow checks
- `remove_signature()`: Improved multi-signature handling
- Files ref backend: Fixed path traversal issues
- Protocol v2: Added symref validation
- `fast-import`: Hardened unsafe features
- Submodule gitfile parsing: Added validation
- Merge machinery: Fixed heap corruption issues

**Status**: Released. All users should upgrade immediately.

---

### `git repo info` Path Keys Series
**What changed**: Phillip Wood and K Jayatheerth continued their discussion about the `[PATCH v8]` series adding seven new path-related keys to `git repo info`. The naming convention with underscores (`_`) is now settled, but the necessity of `path.hooks` and behavioral clarity of `path.cdup` remain unresolved.

**Problem/goal**: The series aims to expose filesystem locations of repository components (hooks directory, index file, superproject working tree, etc.) in a user-facing, scriptable format via `git repo info`. The goal is to avoid requiring direct access to Git's internals while providing complete path information.

**Impact**: This feature would benefit script authors and tool developers who need to access repository component paths without parsing Git's internal configuration. The unresolved design questions about `path.hooks` and `path.cdup` could affect the usability and safety of the feature.

**Technical details**:
- New keys: `path.working_tree`, `path.superproject_working_tree`, `path.hooks`, `path.index`, `path.prefix`, `path.cdup`
- Implementation: Uses `repo_get_work_tree()`, `format_path()`, and other repository access functions
- Controversy: Phillip Wood argues `path.hooks` is unnecessary and potentially harmful, while Jayatheerth defends it as a migration path for scripts using `git rev-parse --git-path hooks`
- Edge case: `path.cdup` behavior when outside the working tree remains a usability concern

**Status**: The series is mechanically complete and design-resolved for most keys, but the remaining objections risk abandonment unless resolved soon. The next version should include the agreed-upon naming changes and test optimizations.

---

### Git 3.0 Planning and Ecosystem Coordination
**What changed**: Daniele Sassoli clarified that Google's SHA-256 talk was removed from the Gerrit User Summit due to a last-minute family emergency for the speaker, not a change in technical plans. Luca Milanesio confirmed his Cassandra-based ODB backend prototype talk remains scheduled.

**Problem/goal**: The Git project is planning for Git 3.0, with goals including Rust support, SHA-256 interoperability, and pluggable ODBs. The Gerrit User Summit was intended as a coordination milestone for Git and JGit maintainers.

**Impact**: The deferral of Google's SHA-256 talk delays clarity on JGit's SHA-256 interoperability timelines, which is critical for Git 3.0's ecosystem compatibility. However, Luca Milanesio's Cassandra prototype demonstrates JGit's production-ready pluggable backend architecture.

**Technical details**:
- Git 3.0 timeline: April 2027 or later, with Git 2.99 as a transitional release
- Rust support: Compiles successfully on Cygwin with an unofficial toolchain
- JGit readiness: Pluggable backend infrastructure is production-ready (Google's internal Git servers, GerritForge's global refdb backend)
- SHA-256: Technically feasible in JGit but not actively developed due to resource constraints

**Status**: The Git 3.0 timeline remains tentative, with ecosystem coordination efforts continuing. The Gerrit User Summit (November 9-10) will still provide a venue for discussion on pluggable ODBs.

---

### `git log --follow` Performance Fix
**What changed**: Justin Tobler sent the test patch for Miklos Vajna's performance fix for `git log --follow` on non-linear history. Toon Claes proposed a readability-focused refactoring of the merge-diff decision tree in `log_tree_diff()`.

**Problem/goal**: A severe performance regression in `git log --follow` (28s → ~7min) was introduced in Git v4.30.0 (commit 304812ed33). The regression occurs because the implementation diffs each merge commit against all parents with rename detection enabled, even when a parent lacks the tracked path.

**Impact**: This performance regression makes `git log --follow` unusable on large repositories with complex histories. The fix aims to restore performance to near pre-regression levels.

**Technical details**:
- Root cause: Full rename-detection diffs between every merge and each parent
- Solution: Two-phase strategy: (1) fast diff without rename detection to identify a parent where the tracked path is unchanged, (2) full rename detection only against other parents if necessary
- Performance: linux.git test case reduced from 6m43s → 42s
- Test coverage: Expanded for subtree merges, copy detection, and merges where one side edits the old path while the other renames it

**Status**: The series is in its fifth iteration (v5) with all prior feedback addressed. Toon Claes's proposed refactoring could improve code readability but may be better suited for a follow-up patch.

---

### Test Modernization: `t9300-fast-import.sh`
**What changed**: Karthik Nayak sent a 20-patch series refactoring the monolithic `t9300-fast-import.sh` test file into smaller, self-contained test files. Junio C Hamano praised the effort and requested a rebase on a synthetic base merging two in-flight topics.

**Problem/goal**: The `t9300-fast-import.sh` test file is ~4000 lines long and contains tests that depend on side effects from earlier tests, making it difficult to maintain and debug.

**Impact**: This refactoring will make the `fast-import` test suite more maintainable and easier to debug by eliminating shared state dependencies. It will also enable safer parallel test execution.

**Technical details**:
- New test files: `t9301-fast-import-notes.sh`, `t9304-fast-import-marks.sh`, `t9307-fast-import-validation.sh`, `t9308-fast-import-stream.sh`, `t9309-fast-import-checkpoint.sh`, `t9310-fast-import-submodule.sh`, `t9311-fast-import-refs.sh`
- Strategy: Each patch makes a subset of tests self-contained by removing dependencies on prior test state, then moves them to dedicated files
- Helpers: New setup functions (`setup_notes_repo()`, `setup_gitlinks_repo()`, etc.) replace shared state with fresh repositories
- Final patch: Removes legacy "series letter" prefixes from test descriptions

**Status**: The series is under review. Junio C Hamano has requested a rebase on a synthetic base merging `cc/early-scan-options` and `tj/drop-no-parseopt` on top of the latest `master`.

---

### `checkout.orphanedCommitWarning` Configuration
**What changed**: Wyatt Holden sent a three-patch series introducing `checkout.orphanedCommitWarning` to disable the expensive orphaned commit check during `git checkout`/`git switch`. Junio C Hamano raised substantive questions about the design, while Derrick Stolee endorsed the series.

**Problem/goal**: The reachability check that warns about orphaned commits can consume >90% of checkout time in large mono-repos (e.g., 193s out of 205s in Microsoft's internal repo). The goal is to provide a configurable opt-out for performance-critical workflows while preserving default safety.

**Impact**: This feature would benefit users of large repositories who frequently switch branches and can tolerate the risk of losing orphaned commits (which remain recoverable via the HEAD reflog).

**Technical details**:
- New config key: `checkout.orphanedCommitWarning` (boolean, default `true`)
- New Trace2 region: `orphaned_commit_warning` exposes check duration
- New advice: `advice.checkoutOrphanedCommitWarning` triggers when check exceeds 2s
- Scalar integration: Disables warning by default via `scalar_config_set()`
- Performance: Trace2 data from Microsoft's internal repo showed 193s out of 205s spent in the check

**Status**: Under review. Junio C Hamano has raised substantive questions about whether the check could be optimized rather than made optional, what factors make it expensive, and why a command-line option was omitted. The author has not yet responded to these points.

---

### `the_repository` Removal Efforts
**What changed**: Multiple threads advanced the `the_repository` removal effort:
1. Srujith A Poojary's series removing `USE_THE_REPOSITORY_VARIABLE` from archival modules received feedback on edge cases and design choices.
2. Junio C Hamano's series updating the unpack-trees API to avoid `the_repository` was queued for `next`.
3. Øystein Walle's series removing `the_repository` from merge.c is cooking in `next`.

**Problem/goal**: The `the_repository` global variable makes it difficult to support multiple repositories in the same process and complicates testing. The goal is to eliminate its use by passing repository instances explicitly.

**Impact**: This refactoring is foundational for future features like pluggable backends and better multi-repository support. It also improves code clarity and testability.

**Technical details**:
- Files touched: `unpack-trees.c`, `archive-tar.c`, `archive-zip.c`, `merge.c`, and related files
- New helpers: `unpack_repo()`, `ref_transaction_resolve_new_oid()`
- Edge cases: Handling uninitialized submodules, repository initialization safety
- Performance: No impact; purely refactoring

**Status**: Progressing well, with multiple topics in various stages of review and integration.

## In brief
- **Documentation**: Julia Evans and Junio C Hamano exchanged views on the best way to explain `HEAD`'s role in detached HEAD state documentation, with Junio favoring technical precision and Julia favoring user-centric language.
- **`git mktree --batch`**: Roger Koranteng sent v2 of the test coverage patch, addressing feedback by verifying error messages on `stderr`.
- **`git reflog expire`**: Laveesh Bansal sent v2 of the refactoring series, consolidating two loops into one and adding deduplication.
- **`git rebase` documentation**: Omri Sarig's patch updating the "INCOMPATIBLE OPTIONS" section was marked ready for `next`.
- **HTTP transport**: Ravi Mistry offered two options to resolve a procedural concern about the two-phase introduction of the default `http.retryAfter` value.
- **ODB refactoring**: Patrick Steinhardt's series isolating the loose-object backend received surface-level reviews confirming mechanical correctness.
- **`git worktree` safety**: Jasper Mayone's four-patch series improving worktree safety and consistency received substantive reviews, with a performance regression identified in patch 3.
- **Bugfix series**: Thomas Bachem's 15-patch series fixing segfaults from unvalued config keys received substantive reviews, with consistency concerns raised about error handling.
- **`test-tool genbytes`**: Kristofer Karlsson's patch replacing slow `tr` invocations with a native helper was queued for `next`.
- **Early option scanning**: Azuama Odinaka Maximus sent an RFC series introducing a reusable early option-scanning API for Git's parse-options subsystem.
- **L10n**: aeron-gh sent a patch using `Q_()` for the "Split into %d hunks" message to enable correct Polish pluralization.
- **What's cooking**: Junio C Hamano's report showed active development across multiple fronts, with notable progress in object storage flexibility, CI optimizations, and `the_repository` removal.