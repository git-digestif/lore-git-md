# Git Mailing List Digest (2026/09/07 -- 2026/09/13)

## The period in brief
This week saw significant progress on several long-running efforts: the `git history squash` feature reached technical completion, ODB alternates refactoring advanced toward integration, and Rust compilation in Windows CI was enabled. A critical design flaw in the `receive-report` hook was resolved, while new bugs in worktree management and ODB transactions were reported and fixed. The week was moderately busy with seven active days, featuring both foundational refactoring and user-facing feature work. Readers should not miss the `git history squash` v15 release, the ODB transaction race fix, and the Windows CI Rust enablement.

---

## Key developments

### `git history squash` reaches technical completion
Harald Nordgren's v15 reroll of the `git history squash` feature is now feature-complete and ready for integration. The series enforces strict case-sensitive matching for autosquash markers (rejecting `fixup! ABCDEF` while accepting `fixup! abcdef`), aligning with Git's historical convention of emitting only lowercase hexadecimal OIDs. The implementation includes new helpers (`first_parent_tree_oid()`, `resolve_squash_range()`, `squash_check_autosquash_subject()`, `build_squash_message()`) and robust validation that rejects ranges that are empty, single-commit, contain root commits, have multiple tips, or include merges with external parents. Ref protection blocks operations if any local branch descended from the squashed range would be left dangling. The series touches `builtin/history.c`, `sequencer.c`/`sequencer.h`, `advice.c`/`advice.h`, documentation, and test scripts. Junio C Hamano's "Will replace" sign-off from v7 signals intent to queue it for the next release.

---

### ODB alternates refactoring advances toward integration
Patrick Steinhardt's ODB alternates refactoring series reached v5, incorporating all substantive review feedback. The series introduces `create_repository()` to split repository initialization into skeleton creation, ODB setup, and refdb setup, eliminating skip flags and consolidating alternates logic into a single code path. Key changes include deferring ODB initialization in `git clone` until after URI resolution and using Git's lockfile API to write alternates atomically. The series removes the ability to write alternates after repository creation, simplifying the ODB interface and preparing for future migration of alternate handling into the "files" backend. Karthik Nayak's review identified a potential robustness issue in the alternates-writing loop, suggesting to check `ferror()` after each `fprintf()` call rather than once at the end. The series is now well-documented and ready for integration.

---

### Rust compilation enabled in Windows CI
Johannes Schindelin's two-patch series enabling Rust compilation in Git's GitHub Actions Windows CI jobs was approved and fast-tracked to `master`. The series configures the Rust toolchain to target the GCC ABI (MinGW) instead of the default MSVC ABI, ensuring Cargo produces a static library (`libgitcore.a`) compatible with Git for Windows' linker. The patches touch `.github/workflows/main.yml`, `ci/lib.sh`, `config.mak.uname`, and the `Makefile`, but do not affect core Git functionality or user-facing features. Junio C Hamano agreed to merge the series directly into `next` and fast-track it to `master`, bypassing the usual one-week cooking period due to the lower risk of Windows users building from source. This resolves a key blocker for the broader Rustification effort.

---

### ODB transaction race fixed
Justin Tobler posted a two-patch series fixing a race in the ODB transaction layer during commit when loose objects and large blobs are present with `core.fsync` batching enabled. The bug occurs when a large blob is written to a packfile in a transaction's temporary directory, but the transaction migrates to the main ODB before the packfile is finalized, leaving it unflushed and causing a "No such file or directory" error. The fix refactors `odb_maybe_flush_packfile()` to separate ODB reprepare logic from packfile flushing (patch 1/2) and adds a call to flush pending transaction packfiles before migration (patch 2/2). The series includes a new test in `t/t1050-large.sh` that reproduces the bug with a minimal repository setup. The patches are narrowly scoped to `object-file.c` and align with the ongoing ODB abstraction effort.

---

### `receive-report` hook design flaw resolved
Karthik Nayak's v9 release of the `receive-report` hook series addressed a critical design flaw identified by Junio C Hamano: the `BUG()` call in the `switch` statement assumed the client's requested protocol version was always valid, but clients may omit the `report-status` or `report-status-v2` capability entirely, leaving `version` as `REPORT_STATUS_UNKNOWN`. The fix replaces the `default: BUG(...)` case with an explicit `REPORT_STATUS_UNKNOWN` case that `break`s, ensuring the code silently skips the status report when the client does not request one. The series is now feature-complete with all prior feedback incorporated and appears ready for graduation to `master`. The hook enables server administrators to filter or modify the status report sent to clients after ref updates, with GitLab's MVCC use case as the primary motivation.

---

### Worktree repair validation added
Yoichi NAKAYAMA posted a two-patch series preventing `git worktree repair` from incorrectly modifying unrelated worktrees. The bug manifests when cross-references between a linked worktree and its administrative data are not validated before repair, risking corruption of unrelated worktrees in edge cases like manually swapped directories. The series first refactors code to reduce redundant `.git` file reads and extract the worktree ID (patch 1/2), then adds validation logic using that ID to prevent repairs on unrelated worktrees (patch 2/2). The test script `t/t2406-worktree-repair.sh` is updated with two new test cases covering the validation. The changes are confined to `worktree.c` and the test script, with no broader behavioral changes.

---

## In brief

**`git history` signing** -- Souma posted v3 of the series teaching `git history` to sign rewritten commits, addressing prior review feedback by trimming commit messages and fixing the `OPT_HISTORY_GPG_SIGN` macro formatting. The series enables signing for operations like `drop`, `fixup`, `reword`, and `split` using the same configuration and command-line options as `git commit`.

**Advice scope hint simplification** -- Junio C Hamano proposed an even simpler design for Vsevolod Myalitsin's advice scope hint series: the hint could *always* recommend `--global` without needing to mark individual advice settings for their intended scope. This would eliminate the need for the `is_global_hint` field entirely.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder proposed a cast from `unsigned long` to `int` for the recursion depth counter in patch 4/5, citing precedent in `builtin/pack-objects.c`. Junio confirmed that casting the result of `git_env_ulong()` to `int` is acceptable, following existing precedent in Git.

**MinGW/Windows build adjustments** -- Johannes Schindelin posted v3 of the 12-patch MinGW/Windows build adjustments series, addressing Johannes Sixt's feedback by moving compiler-definition hunks and dropping a fly-by style cleanup. The series upstream Git for Windows-specific build and runtime adjustments into the main Git codebase.

**Coccinelle rule discussion** -- The discussion about removing a risky Coccinelle rule (`if (!E) free(E)`) shifted toward accepting compiler warnings for correctness. René Scharfe argued that the project should tolerate compiler warnings for the sake of correctness, noting that GCC accepts `if (!E);` without warning, while Clang warns.

**`--force-if-includes` bugfix** -- Tyler Cipriani posted v3 of the series fixing `--force-if-includes` to check the correct reflog, addressing all of Patrick Steinhardt's technical concerns. The series clarifies that `--force-if-includes` already rejects detached HEAD pushes today and improves detached HEAD advice.

**`git bundle` regression fixed** -- Taylor Blau fixed a regression where `git bundle create` with bitmaps enabled omitted tree objects required by advertised refs. The fix restricts pack "haves" to only those UNINTERESTING commits marked as BOUNDARY (i.e., prerequisites recorded in the bundle header).

**Windows test fixes** -- Johannes Schindelin posted a two-patch series fixing Windows-specific test failures that surfaced during full Windows/ARM64 test runs. The patches adjust Perl environment detection and unconditionally skip UTF-8 tests on Windows due to fundamental encoding incompatibilities.

**Documentation improvements** -- Todd Zullinger posted a three-patch series extending the AsciiDoc linting script to enforce backtick-quoting for commands in synopsis-style man pages and applying the fix to `git-pack-refs` and `git-refs` documentation.

**Shell completion for `git worktree repair`** -- Yoichi NAKAYAMA added shell completion support for the `git worktree repair` subcommand, updating `contrib/completion/git-completion.bash` to include the new subcommand and extend path completion for linked worktrees.

**Prevent `git pull` segmentation fault** -- Jiri Kuncar posted a bugfix patch preventing `git pull` from crashing when encountering an invalid merge head. The patch adds NULL guards around calls to `lookup_commit_reference()` in `builtin/pull.c`, treating failed lookups as "not up to date" rather than segfaulting.

**Avoid unnecessary packed-refs lock** -- Ariel Keselman posted v3 of a patch avoiding unnecessary packed-refs lock acquisition when deleting root refs (like `AUTO_MERGE`, `CHERRY_PICK_HEAD`, etc.), which are never packed. The change prevents `git update-ref --no-deref -d AUTO_MERGE` from failing in linked worktrees with read-only shared metadata.

**`--matched-only` option for `git range-diff`** -- Harald Nordgren posted the final version of a feature patch adding the `--matched-only` option to `git range-diff`. The new option filters output to show only commits present in both input ranges, addressing the workflow pain point of reviewing rebases or range-diffs where users want to focus on how surviving commits changed.

**Outreachy December 2026 cohort** -- Christian Couder submitted Git's application for the Outreachy December 2026 cohort, listing two approved project ideas for interns.

**Git v2.56.0-rc0** -- Junio C Hamano announced the first release candidate for Git v2.56.0, summarizing 635 non-merge commits from 82 contributors.

---

## Looking ahead
The next week is likely to see integration of the `git history squash` feature, the ODB alternates refactoring series, and the `receive-report` hook. The ODB transaction race fix and worktree repair validation series are also strong candidates for integration. The advice scope hint discussion may converge on a minimal design, while the Coccinelle rule discussion could see further debate about compiler warnings and build hygiene. The Rustification effort will continue to progress, with downstream projects like git-cinnabar likely to adapt to the new project structure.