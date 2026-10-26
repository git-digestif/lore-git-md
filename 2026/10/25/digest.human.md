# Git mailing list daily digest for 2026/10/25

## The day in brief
The Git mailing list saw active discussion on several fronts today. Harald Nordgren clarified the fallback logic in the `remote.<name>.refmap` series, now cooking in `next`, while Tyler Cipriani posted a performance optimization for `test-lint` that consolidates parsing effort between `chainlint` and `greplint`. A philosophical debate about `git status` defaults continued, and several documentation and bugfix patches advanced toward integration.

## Notable threads

### `remote.<name>.refmap` fallback logic clarified
Harald Nordgren addressed lingering questions about the fallback behavior in `remote_find_tracking()`, confirming that the fallback to `remote->refmap` is necessary for compatibility with `@{upstream}`, `--delete-merged`, and `git branch --track` operations. The `--limited-fetch` flag does not require this fallback, resolving an ambiguity raised by D. Ben Knoble. Harald committed to adding tests to verify refmap-tracked refs and negative refspec exclusion, locking down the behavior before the series graduates from `next`.

The series introduces a "matching-inspired" fetch mode for shallow repositories, dynamically fetching only branches tracked by local branches at the remote. The fallback logic ensures that refs fetched via the refmap are correctly tracked for operations like `@{upstream}`, while negative refspecs in `remote.<name>.fetch` continue to exclude refs as expected. The clarification removes the last open design question before the series can proceed to `master`.

### `test-lint` performance optimization posted
Tyler Cipriani posted a four-patch series addressing the slowdown in `test-lint` caused by the `test-grep-lint` check. The series consolidates parsing effort between `chainlint.pl` and `greplint.pl`, reducing redundant file I/O and leveraging parallel execution. The first patch incorporates Jeff King’s optimization (an early return in `greplint.pl` to skip files without "grep"), while the remaining patches refactor the core logic into a shared library (`lib-greplint.pl`) and teach `chainlint.pl` to run greplint checks during its tokenization pass.

The series reduces `test-lint` runtime from ~25 seconds to ~1 second for five consecutive runs on a 16-core machine, addressing a real developer workflow bottleneck reported by Junio. The implementation is pragmatic, preserving standalone `greplint.pl` functionality while eliminating duplicate work. The patches are uncontroversial and likely to land quickly if no edge cases are uncovered.

### `git status` defaults debate continues
The philosophical debate about whether `git status` should default to comparing against both `@{upstream}` and `@{push}` remained unresolved. Harald Nordgren posted v3 of the patch, refining the justification to emphasize practical benefits for users in triangular workflows: reminding them of divergence from the last push, indicating local rebases, and validating what is about to be pushed. Junio C Hamano’s latest critique focuses on whether `@{push}` divergence is inherently less important to monitor by default than `@{upstream}` divergence, arguing that `@{push}` reflects the user’s own work rather than external changes.

D. Ben Knoble countered with a pragmatic, user-centric argument, describing his own workflow where he frequently checks divergence between local and `@{push}` branches to answer questions like *"Have I rebased lately and need to force-push?"* The discussion now centers on whether the benefits for triangular workflow users outweigh the potential noise for others, and whether the change aligns with Git’s long-term design principles. No other contributors have weighed in since Junio’s latest reply, leaving the debate at an impasse.

### `git worktree list --branch` reviewed
Phillip Wood reviewed Jasper Mayone’s `--branch=<branch>` filtering patch for `git worktree list`, identifying code duplication in the branch-checking logic and proposing a refactoring of `branch.c:prepare_checked_out_branches()` to expose a reusable `branch_find_checkout()` helper. He also suggested adding an `is-main-worktree` annotation to the porcelain output to help scripts avoid attempting to delete the main worktree. The review is constructive and focuses on maintainability and scriptability, though it does not block the patch.

The patch introduces a general filter primitive for `git worktree list`, enabling users to identify worktrees associated with specific branches. The implementation is designed to compose with existing branch-filtering commands (e.g., `git branch --merged`) to achieve merged/unmerged filtering. The discussion now turns to whether the minimal fix will be merged first, followed by a refactoring patch, or if the refactoring will be adopted in a v2.

## In brief
- **`git apply --check` crash**: Tanga Joris confirmed the root cause of the crash in `git apply --check` when processing a patch that creates a new file. The issue is triggered by a blank line after the `diff --git` header, not the patch content or header ordering. A fix is in progress.
- **`diff-delta.c` safe integer helpers**: Junio expressed dissatisfaction with the v4 commit message for Srujith A. Poojary’s patch, which replaces manual integer arithmetic with safe helpers. The v5 iteration now includes a detailed breakdown of the arithmetic, struct layout, and bounds interaction, addressing Junio’s demand for transparency.
- **Reflog pluralization fix**: Aniket Gupta posted v2 of the patch fixing a pluralization error in reflog error messages. The patch incorporates Junio’s test fix for `t/t1506-rev-parse-diagnosis.sh` and is ready to land.
- **`git-rebase` man page update**: Omri Sarig posted v4 of the patch updating the `git-rebase` man page to correct outdated references in the "INCOMPATIBLE OPTIONS" section. The patch is uncontroversial and ready for merging.
- **`the_repository` removal**: Dorine Tipo posted a patch mechanically refactoring `partial_clone_register()` and `partial_clone_get_default_filter_spec()` to accept a `struct repository *` parameter, updating callers to pass `the_repository` explicitly. The patch is part of the broader effort to eliminate global state in Git’s codebase.
- **`git reflog expire` dry-run output**: Junio suggested a refactoring to consolidate the `--all` and named-refs loops in `git reflog expire` to eliminate duplication. Laveesh Bansal proposed merging the minimal fix first, followed by a refactoring patch.
- **Completion regression fix**: René Scharfe noted an edge case in the completion behavior for `git fast-import`, where `--git-completion-helper` only prints options after receiving input. The fix may require adjustment for this command.