# Git mailing list daily digest for 2026/11/16

## The day in brief
Today's Git mailing list activity focused on finalizing several long-running patch series, addressing performance regressions, and fixing bugs across various subsystems. Key developments include the final approval of the `git repo info` path-related keys series, progress on the `git branch --delete-integrated` feature, and resolution of several crash bugs in the ODB layer and submodule handling. Performance optimizations for `git repo structure` and `git log --follow` were also prominent.

## Notable threads

### [PATCH v9] series adding path-related keys to `git repo info`
**What changed**: The series adding seven new path-related keys to `git repo info` (`path.working_tree`, `path.superproject_working_tree`, `path.index`, and `path.prefix`) received final confirmation and is now fully approved for merging.

**Problem/goal**: Expose filesystem locations of repository components in a user-facing, scriptable format via `git repo info`.

**Subsystem**: `git repo` built-in command

**Impact**: The series is now in its final form (v9), with all design objections resolved and explicit approval from Phillip Wood. Junio can queue it without further changes.

**Today's development**: [2026/11/16/13-00-40 by K Jayatheerth] The author acknowledges Phillip Wood's feedback about test efficiency but defers optimization to a follow-up series. Phillip explicitly accepts this plan and reiterates approval for merging the series as-is.

### [PATCH v5 0/4] branch --delete-merged: detect squash-merged and rebase-merged branches
**What changed**: The fourth patch in Harald Nordgren's series adding `--delete-integrated` to `git branch` received final review confirmation.

**Problem/goal**: Extend `git branch --delete-merged` to recognize branches whose changes (but not their tips) have been integrated via squash merges, rebases, or patch-based workflows.

**Subsystem**: Branch management

**Impact**: The patch is now technically complete and ready for integration, with Jasper Mayone confirming all substantive issues from v6 are resolved in v7.

**Today's development**: [2026/11/16/16-16-03 by Jasper Mayone] The reviewer explicitly states that all three substantive issues raised in v6—performance inefficiency, correctness ambiguity, and edge-case robustness—are now resolved in v7. He confirms the patch builds cleanly, passes all tests, and adds his `Tested-by` trailer.

### Regression in partial clones (blob:none) after ODB abstraction refactor
**What changed**: A regression in Git 2.54.0 causing partial clones to attempt lazy-fetch of blobs just written locally was confirmed and a fix validated.

**Problem/goal**: Fix a bug where newly written merged blobs in `git merge` cannot be read back due to stale loose object cache entries.

**Subsystem**: Object database (ODB) abstraction layer

**Impact**: The regression affects production workflows and has been confirmed to exist in `master`, `next`, and `seen`. The fix is ready for review/merge.

**Today's development**:
- [2026/11/16/11-45-52 by Google — Mergify] The author reports the regression with a clear reproduction case and bisect.
- [2026/11/16/12-46-00 by Patrick Steinhardt] The reviewer confirms the issue is known and points to an in-flight fix series.
- [2026/11/16/20-16-05 by Patrick Steinhardt] The fix is validated and ready for review/merge.

### [PATCH 0/12] Introduce pluggable commit graphs for ODB abstraction
**What changed**: Junio C Hamano provided substantive review of the first patch in Patrick Steinhardt's series introducing pluggable commit graphs.

**Problem/goal**: Move commit-graph storage logic into the ODB backend layer, enabling alternative ODB implementations to control where and how commit graphs are read and written.

**Subsystem**: Object database (ODB) abstraction

**Impact**: This is a foundational step in the ODB abstraction effort. Junio's feedback may prompt a v2 update to address validation gaps.

**Today's development**: [2026/11/16/20-53-56 by Junio C Hamano] The reviewer raises questions about error reporting conventions, the role of the `oid` field in `struct commit_graph`, and potential validation gaps that may need addressing in v2.

### `index-pack` regression in Git 2.56.0
**What changed**: A new regression was reported in `git index-pack` causing redundant lazy fetches for in-pack `REF_DELTA` bases.

**Problem/goal**: Fix a bug where `git index-pack` triggers unnecessary nested lazy fetches for `REF_DELTA` bases already present in the thin pack.

**Subsystem**: `index-pack`, promisor-remote lazy fetching

**Impact**: The bug affects blobless partial clones when the server does not support `OFS_DELTA`, leading to redundant network round-trips.

**Today's development**: [2026/11/16/17-38-38 by לירון עדן] The author reports the regression with a reproduction script and identifies the root cause in `fix_unresolved_deltas()`.

## In brief

- **[PATCH v3] rerere: fix race condition between rebase and background maintenance**: [2026/11/16/23-39-08 by Junio C Hamano] The follow-up patch for timeout naming consistency is marked ready for 'next'.
- **[PATCH v5] git imap-send: handle non-canonical email header order**: [2026/11/16/07-44-07 by Patrick Steinhardt] The reviewer suggests a minor readability improvement to the CRLF-handling logic in the new `has_required_headers()` helper function.
- **[RFC PATCH 0/2] git history: add --verbose support for reword**: [2026/11/16/19-23-27 by Jelle van der Waa] The author announces a pivot to make diff display the default behavior for `git history reword`, `split`, and `fixup`, controlled by `commit.verbose`.
- **[PATCH 0/3] Fixes for compatObjectFormat interoperability layer**: [2026/11/16/07-36-53, 07-37-02, 07-37-09 by Patrick Steinhardt] The reviewer provides feedback on all three patches, including cosmetic adjustments, test design concerns, and substantive questions about header validation.
- **[PATCH] promisor-remote: fix crash in `git ls-remote` outside repository**: [2026/11/16/06-35-26, 15-29-46 by Patrick Steinhardt, Junio C Hamano] The maintainer and reviewer discuss design approaches, with consensus forming around a minimal fix.
- **[PATCH] http-backend: remove unnecessary pre-sizing in get_info_packs()**: [2026/11/16/06-29-44, 15-24-35 by Patrick Steinhardt, Junio C Hamano] The patch is approved and marked ready for 'next'.
- **[PATCH 0/2] repo structure: performance and memory-usage improvements**: [2026/11/16/17-32-00 by Junio C Hamano] The series is marked ready for 'next' after addressing all feedback.
- **[PATCH] for-each-ref: fix segfault with --no-format**: [2026/11/16/06-45-45 by Patrick Steinhardt] The patch is reviewed and approved.
- **test modernization: replace `!` with `test_must_fail`**: [2026/11/16/16-07-11 by Junio C Hamano] The patch is approved and planned for integration.
- **[PATCH] log: add --follow=merges option to restore pre-304812ed33 performance**: [2026/11/16/07-35-50 by Miklos Vajna] The author posts v2 incorporating feedback on error handling and documentation.
- **[PATCH] read-cache: remove `the_repository` use in `overlay_tree_on_index`**: [2026/11/16/13-11-33 by Derrick Stolee] The patch receives surface-level review approval.
- **[PATCH] setup: fix hash algorithm detection**: [2026/11/16/06-22-53 by Patrick Steinhardt] The reviewer suggests minor documentation and test improvements.
- **[PATCH 0/12] Modernize git-rerere's subcommand handling**: [2026/11/16/07-46-56, 15-43-53 by Patrick Steinhardt, Junio C Hamano] The maintainer accepts the UI regression as a deliberate trade-off.
- **[PATCH v4 0/5] glossary & docs: align "pseudoref" vs "root ref" terminology**: [2026/11/16/08-09-07 by Patrick Steinhardt] The series is confirmed ready for integration.
- **[PATCH 0/2] worktree list --merged / --no-merged**: [2026/11/16/16-15-40, 16-15-53, 16-42-29 by Jasper Mayone, Phillip Wood] The author drops controversial patches and the reviewer questions the necessity of `%(currentworktree)`.
- **[PATCH] doc: rewrite explanation of detached HEAD state**: [2026/11/16/22-08-39 by Junio C Hamano] The maintainer requests a final review pass before merging.
- **[PATCH] git apply: fix crash with malformed patch headers**: [2026/11/16/17-54-21, 18-06-21 by Tanga Joris] The author confirms error message improvements for v3.
- **[PATCH 0/12] log --follow: fix bugs and optimize for merges**: [2026/11/16/08-01-05, 17-39-12 by Patrick Steinhardt, Junio C Hamano] The toggle patch is marked ready for 'next'.
- **[PATCH] utf8.c: fix bounds handling and documentation in utf8_strnwidth()**: [2026/11/16/18-56-59, 18-57-00, 18-57-01 by Dirk Gouders] The author posts a two-patch series fixing boundary checking and documentation in UTF-8 handling.
- **Submodule crash in validate_submodule_legacy_git_dir()**: [2026/11/16/22-51-48 by Andreas Wendleder] The author reports a crash when a submodule is itself a linked worktree.