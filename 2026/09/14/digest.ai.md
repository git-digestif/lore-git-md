# Git mailing list daily digest for 2026/09/14

## The day in brief
Git 2.56-rc0 was tagged, with several topics graduating to `master`. Key developments include a new incremental connectivity check for fetches/pushes, fixes for a repack race condition causing data loss, and a resolution on advice-scope hints. Outreachy mentor assignments were completed, and two additional projects were proposed.

## Notable threads

### Incremental connectivity check for fetch/push
**[PATCH 0/2] Add incremental connectivity check for fetch/push** by Kristofer Karlsson introduces an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) to reduce the cost of verifying object connectivity during fetches and pushes. The new mode processes incoming commits in topological order, tracking trusted objects from parent commits to skip unchanged subtrees.

- **What it changes**: Adds a new config option and internal `rev-list` flag (`--verify-trees-incremental`) that shifts the cost from repository size to incoming change size.
- **Motivation**: The existing full check scales poorly with repository size, making small pushes to large repositories expensive.
- **Impact**: Dramatic speedups (up to 190x) for small pushes to large repositories, with a modest regression (1.5x slower) for very long incoming histories (10,000+ commits).
- **Status**: Under review. Junio called the benchmarks "exciting," but raised technical concerns about promisor object handling and NULL-dereference risks in the implementation.

### Repack race condition fix
**[PATCH 0/6] repack: fix concurrent-push race leading to data loss** by qeesung (Qin ShiCheng) addresses a race condition in `git repack -d` where concurrent pushes can cause the repack machinery to delete packs whose objects were never copied elsewhere, resulting in data loss.

- **What it changes**: Ensures `.keep` files are only removed by the process that created them and replaces the `--honor-pack-keep` mechanism with a consistent snapshot of kept packs observed at repack startup.
- **Motivation**: The race was observed in production on Git 2.43, causing dangling refs pointing to nonexistent commits.
- **Impact**: Fixes a subtle but serious data-loss bug with no user-visible behavior changes.
- **Status**: Under review. The series is production-tested and includes comprehensive tests reproducing the race.

### Advice scope hint resolution
**[PATCH v5] advice: add scope hint for disabling advice messages** by Junio C Hamano implements the now-consensus design for advice scope hints: always recommending `--global` in the hint for all `advice.*` settings.

- **What it changes**: Updates the translatable string to consistently recommend `--global` (e.g., `git config --global advice.detachedHead false`).
- **Motivation**: Advice messages are about silencing chatter for a user, not a specific repository, so `--global` is the appropriate scope.
- **Impact**: Simplifies the design by eliminating per-setting scope tracking and resolves build-failure concerns from earlier iterations.
- **Status**: Ready for review. The central design debate is resolved, and the implementation is minimal and mechanical.

### Outreachy December 2026 cohort
Git's application for the Outreachy December 2026 cohort was submitted with two approved projects: "Improve how command arguments and options are scanned and parsed" and "Reduce Git’s global state to enable Git's libification." Mentor assignments are now complete for both projects.

- **New development**: Kaartic Sivaraam proposed expanding the application to include two additional projects originally prepared for GSoC 2026: "Implement promisor remote fetch ordering" and "Enhance promisor-remote protocol for better-connected remotes."
- **Status**: Mentor assignments are complete, and the thread is now focused on finalizing project scopes and pursuing sponsorship.

## In brief

- **[PATCH v8 0/4] receive-pack: add new `receive-report` hook**: Junio proposed a mechanical cleanup patch to remove a redundant null-check in the `receive-report` hook series, now in its v10 iteration and queued in `next`.
- **Refactoring ODB alternates handling**: Karthik Nayak confirmed the 9-patch series is fully reviewed and ready for integration into `next`.
- **`git var` extension series**: Andrew Pleeter posted v8, addressing Junio’s feedback by adopting `VARIABLE=value` output format and aligning exit code behavior with the implementation.
- **Rerere lock race fix**: Thomas Bachem posted v4 of the two-patch series, splitting the series as requested by reviewers and adding deterministic test cases.
- **Pathspec exclusion fix**: Yannik Tausch posted v4 of the two-patch series, reverting to a two-patch structure and adding deterministic test cases for the heap-buffer-overflow.
- **`--force-if-includes` fix**: Tyler Cipriani posted v4 of the two-patch series, fixing the reflog lookup logic and advice messages, but a backwards-compatibility concern was raised about rejecting non-branch pushes.
- **Imap-send OpenSSL compatibility**: Beat Bolli clarified that the OpenSSL 4.1 compatibility macro is a forward-compatibility fix to avoid build failures under `DEVELOPER=1`.
- **Merge-ll tempfile cleanup**: Jeff King accepted Elijah Newren’s proposed fix for a latent type-safety bug in the `strbuf`-based refactoring and proposed a stylistic improvement to split a combined error check.
- **`--matched-only` for `git range-diff`**: Junio rejected the use of `die_for_incompatible_opt3()` on architectural grounds, requiring the error handling to revert to the original `error()` call.
- **`git worktree repair` completion**: Yoichi Nakayama posted v2 of the completion patch with an adjusted commit message.
- **MacOS CI dependency script**: Harald Nordgren posted v2 explaining the Homebrew policy change that made the `brew link --force gettext` command redundant.
- **Documentation backtick-quoting**: Todd Zullinger posted v2 of the series, dropping the lint-script change and addressing Junio’s correction to restrict backtick-quoting to configuration keys.
- **Git for Windows 2.56.0-rc0**: Johannes Schindelin announced the release, noting dropped Windows 8.1 support and installer fixes.
- **`git status --ignored` bug**: Sean Whitton reported a bug where a pathspec (e.g., `ba`) incorrectly matches ignored directories whose names contain the pathspec as a substring (e.g., `bar/baz/`).
- **Test modernization**: Junio identified a logical error in a test modernization patch, requiring a v2 to use the correct helper function.
- **"What's cooking" report**: Junio posted the integration report for September 2026, listing topics cooking in `next` and `seen`, those graduated to `master`, and new topics under review.