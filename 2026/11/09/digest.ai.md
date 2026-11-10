Here is today's Git mailing list digest:

# Git mailing list daily digest for 2026/11/09

## The day in brief
The Git project saw significant progress on several fronts today. The `git repo info` path keys series reached a design consensus on naming conventions, with maintainer arbitration settling on underscore-separated keys like `path.working_tree`. The fsmonitor untracked cache fix was marked ready for integration after final review. A new voting platform option (Belenios) was proposed for the community voting process, expanding the tooling choices. Several long-standing refactoring efforts advanced, including the `the_repository` removal work and ODB abstraction improvements.

## Notable threads

### Path keys for `git repo info` reach naming consensus
The `[PATCH v8]` series adding path-related keys to `git repo info` saw productive design discussion today. After Phillip Wood raised objections to the original naming (`path.toplevel`, `path.superproject-root`, `path.git-prefix`), K Jayatheerth agreed to rename them to `path.working-tree`, `path.superproject-working-tree`, and `path.prefix`. However, Junio C Hamano intervened to enforce Git's configuration naming conventions, requiring underscores instead of hyphens (`path.working_tree`, `path.superproject_working_tree`).

The discussion also addressed the necessity of `path.hooks`, with Phillip arguing that scripts should use `git hook run` instead of querying the hooks directory directly. Jayatheerth countered that `git repo info` should serve as a replacement for `git rev-parse --git-path hooks`, maintaining backward compatibility. The behavioral consistency of `path.cdup` was also debated, with Jayatheerth clarifying its implementation details.

**What's next**: The author will implement the agreed-upon naming changes in the next version of the series, while the `path.hooks` debate may require maintainer arbitration.

### fsmonitor untracked cache fix ready for integration
The bugfix for stale untracked cache entries after trivial fsmonitor responses was marked ready for integration today. The patch, which introduces `untracked_cache_invalidate_stale_dirs()` to recursively check cached directories against their on-disk stat data, received final confirmation from Ilia K that no commands trust stale entries between `refresh_fsmonitor()` and index writing.

Junio C Hamano noted that the topic is already in `next` but suggested avoiding a wholesale replacement to preserve rerere history. The patch addresses a subtle correctness issue where Git would emit spurious warnings about missing directories after fsmonitor daemon restarts.

**What's next**: The patch will graduate to `master` in the next integration cycle.

### New voting platform option proposed for community process
Stéphane Glondu proposed Belenios as a third voting platform option for Git's community voting process. Belenios is already used by Debian (`vote.debian.net`), supports Condorcet voting, and can be self-hosted, addressing concerns about third-party dependencies. This expands the current options (CIVS and devotee) while maintaining the project's commitment to tooling neutrality.

Jonathan Corbet's earlier review of CIVS highlighted a usability hurdle: voters must proactively sign up on the platform, which could create friction for Git's email-centric workflow. Belenios addresses this by offering both hosted and self-hosted options.

**What's next**: The community will evaluate Belenios alongside CIVS and devotee, focusing on usability, auditability, and neutrality.

### `the_repository` removal work advances
Several threads today advanced the ongoing effort to eliminate Git's global `the_repository` variable:

1. **`unpack-trees.c` refactoring**: Junio C Hamano posted v2 of a series removing `USE_THE_REPOSITORY` from `unpack-trees.c`, introducing a new `unpack_repo()` helper. Patrick Steinhardt and Derrick Stolee provided positive reviews, with Stolee calling the helper a "valuable abstraction."

2. **Archive subsystem**: Srujith A Poojary posted a two-patch series removing `USE_THE_REPOSITORY_VARIABLE` from Git's archival modules. Junio raised a concern about an inconsistency in `builtin/archive.c` where the patch adds `#define USE_THE_REPOSITORY_VARIABLE` while simultaneously updating `init_archivers()` to take an explicit `the_repository` parameter.

3. **Notes subsystem**: Patrick Steinhardt suggested interface improvements for Okhuomon Ajayi's notes refactoring series, proposing renames for `init_notes()` and `load_notes_trees()` to better reflect their subsystem ownership.

**What's next**: The `unpack-trees.c` series is likely to proceed to `next`, while the archive and notes refactoring efforts will address the remaining feedback.

### ODB abstraction improvements
Patrick Steinhardt's work on object storage abstraction saw progress today:

1. **Test suite preparation**: The series preparing Git's test suite for non-default object storage backends was marked ready for integration after addressing Junio's surface-level feedback.

2. **Loose object cache invalidation**: Jeff King provided data on the performance impact of the `QUICK` flag, showing a 1320% slowdown in a pathological case when omitted. This quantifies the trade-off between correctness and performance in the ongoing debate about the flag's future.

**What's next**: The test suite preparation series will proceed to `next`, while the `QUICK` flag discussion continues.

## In brief
- **`git apply --check` crash fix**: The series fixing assertion failures in `git apply --check` when processing malformed patch headers reached consensus on stricter validation, with Junio endorsing the approach.
- **`git history reword --verbose`**: The debate about whether diff display should be opt-in or opt-out continued, with Phillip Wood now preferring to make it the default behavior.
- **`git reflog expire` refactoring**: Junio endorsed Patrick Steinhardt's suggestion to deduplicate refs in the unified loop, calling the current behavior a bug.
- **`git diff` documentation**: Junio queued squashable commits to address build toolchain compatibility issues in Julia Evans's EXAMPLES rewrite.
- **`git check-ref-format` conversion**: A new contributor posted a patch converting the command to use the `parse-options` API.
- **Platform compatibility**: Sven Strickroth raised a question about zlib 1.4's behavioral change and its impact on Git's zlib usage.
- **Documentation**: Julia Evans posted a new beginner-friendly tutorial on Git branches, while Junio raised a meta-concern about review bandwidth.

The Git project continues to make steady progress on both user-facing features and internal refactoring efforts, with several series now poised for integration.