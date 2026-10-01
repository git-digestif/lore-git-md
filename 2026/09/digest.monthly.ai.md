# Git Mailing List Monthly Digest: September 2026

**Period: 2026/09/01 -- 2026/09/30**

## The period in brief

September 2026 was an exceptionally active month for the Git project, with **24 active days** of development and **over 100 distinct topics** receiving attention. The period was marked by **major architectural progress**, **critical bugfixes**, and **significant community initiatives**. Three developments stand out: the **finalization of the `--force-if-includes` safety mechanism fix**, which addresses a long-standing data-loss risk in `git push`; the **graduation of Rust infrastructure reorganization** to `next`, a key milestone toward Git 3.0; and the **resolution of a critical data-loss bug in `git repack`**, which had affected production repositories with concurrent push operations. The month also saw the **start of the Git 2.56 stabilization period**, the **completion of Outreachy mentor assignments**, and the **introduction of a new `gitmergeconflicts(7)` man page** to centralize conflict resolution guidance.

---

## Key developments

### `--force-if-includes` safety mechanism overhauled
The `--force-if-includes` safety mechanism in `git push` received a **comprehensive overhaul** after a critical flaw was identified: the feature was checking the reflog of the *local* branch matching the *remote* destination name (e.g., `refs/heads/main`) instead of the branch actually being pushed (e.g., `refs/heads/src` in `src:main`). Tyler Cipriani's six-iteration series fixed this logic, ensuring the mechanism consults the reflog of the pushed ref and defers reflog checks until after fast-forward determination. The series also introduced `advice.forceIfIncludesDetachedHead` to provide actionable guidance for rejected cases and added 108 lines of new test coverage. The fix addresses a **long-standing safety concern** and is now ready for integration, with all prior review feedback from Patrick Steinhardt, Junio C Hamano, and D. Ben Knoble resolved.

### Rust infrastructure reorganization graduates to `next`
The effort to reorganize Git’s Rust code into a dedicated `rust/` subdirectory reached a **major milestone**, with Mike Hommey’s v5 patch moving Rust sources from `src/` to `rust/src/` while keeping build artifacts in their original locations. The patch updates the build system (Makefile, meson.build, CI scripts) to reference the new locations and is now queued for `next`. Junio C Hamano confirmed the patch applies cleanly on Git 2.56-rc1 and correctly updates the build system, marking a **key step toward Git 3.0’s mandatory Rust components**. The reorganization addresses project hygiene and downstream vendoring concerns, unblocking further Rustification work.

### Repack data-loss race fixed after production exposure
A **critical race condition** in `git repack -d` was fixed in a six-patch series from Qin ShiCheng (qeesung). The bug, which could cause concurrent pushes to trigger the repack machinery to delete packs whose objects were never copied elsewhere, resulted in **data loss in production repositories**. The series replaces the `--honor-pack-keep` mechanism with a consistent snapshot of kept packs observed at repack startup, ensuring `.keep` files are only removed by the process that created them. The fix includes optimizations for repositories with many kept packs, yielding an **11× speed-up in `--keep-pack` lookups**. The series landed in `next` and is poised for `master`, addressing a subtle but serious bug that affects repositories with concurrent push operations.

### `git history` signing series nears integration
Souma’s series teaching `git history` to sign rewritten commits (`drop`, `fixup`, `reword`, `split`) using GPG reached v3, addressing all prior review feedback from Patrick Steinhardt. The series now consists of two patches: a preparatory refactoring of the replay API and the main feature implementation. Rewritten commits now respect the same signing configuration (`commit.gpgsign`) and command-line options (`-S/--gpg-sign`, `--no-gpg-sign`) as `git commit`, with identical precedence rules. The series touches `replay.c`, `replay.h`, `builtin/history.c`, the `git-history` documentation, and four test scripts (`t3451`–`t3454`). Junio C Hamano has not yet weighed in, but the series appears **ready for integration**.

### `receive-report` hook series overcomes critical design flaw
Karthik Nayak’s `receive-report` hook series, which enables server administrators to intercept and modify the status report sent to clients after ref updates, reached v10 after addressing a **critical design flaw** identified by Junio C Hamano. The flaw—a `BUG()` call in the `switch` statement that assumed the client’s requested protocol version was always valid—was fixed by replacing the `default: BUG(...)` case with an explicit `REPORT_STATUS_UNKNOWN` case that `break`s, ensuring the code silently skips the status report when the client does not request one. The series also fixed a memory leak in the `override_cmds_error()` helper. The hook is motivated by GitLab’s need to implement multi-version concurrency control (MVCC), where the final status must reflect operations occurring after `ref_transaction_commit()`. The series is now **feature-complete** and ready for graduation to `master`.

### `git history squash` reaches technical completion
Harald Nordgren’s v15 reroll of the `git history squash` feature enforced strict case-sensitive matching for autosquash markers (e.g., rejecting `fixup! ABCDEF` while accepting `fixup! abcdef`), aligning with Git’s historical convention of emitting only lowercase hexadecimal OIDs. The series is now **technically complete**, addressing all prior feedback on autosquash marker resolution, shape-based validation, ref protection for local branches, and the `--no-edit` workflow. Junio C Hamano’s "Will replace" sign-off from v7 signals intent to queue it for the next release. The feature collapses a commit range into its oldest ancestor while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach.

### Reference-transaction hook regression fixed
A **long-standing regression** in the reference-transaction hook, introduced in Git 2.31, was fixed in a six-iteration series. The bug caused the hook to receive all-zero OIDs when branches or tags were deleted, breaking external tools like Gerrit and GitLab that rely on the hook for ref monitoring. The v6 series abandons the v5 approach of modifying `refs_delete_refs()` and instead moves the fix into the **transaction layer itself**, recording a separate "observed old value" for updates where callers didn’t supply one. This ensures the hook always receives meaningful old OIDs while preserving the performance gains of the original optimization. The series includes updated regression tests and is now **architecturally aligned with Patrick Steinhardt’s feedback**.

### `strbuf` safety API RFC sparks architectural discussion
Derrick Stolee’s RFC series introducing a "safe" `strbuf` API subset that cannot call `die()` received **detailed architectural feedback** from Junio C Hamano and Phillip Wood. The review identified **two critical bugs** (a memory leak in `srealloc()` and a misleading error message in `safe_memory_limit_check()`) and questioned the framing of the API as a "safe" subset rather than a hardened version of the entire `strbuf` API. Junio proposed that higher-level utilities like `strbuf_realpath()` should avoid the `strbuf_` prefix to prevent confusion, while Jeff King suggested an alternative design using **fixed-size, stack-allocated buffers** to avoid `malloc()` entirely. The discussion has shifted from implementation details to **fundamental design questions** about how to best serve libification and trace2 use cases, with no clear resolution yet.

### `gitmergeconflicts(7)`: A new man page for conflict resolution
Julia Evans posted a **seven-patch series** introducing `gitmergeconflicts(7)`, a new man page consolidating merge conflict resolution guidance. The current documentation scatters advice across `git merge`, `git rebase`, `git revert`, and other command man pages, leading to duplication and user confusion. The new guide centralizes this information, clarifies terminology like "ours" vs. "theirs," and provides **practical, example-driven workflows**. The series updates existing man pages to link to the new guide and includes build system integration. Junio requested **mechanical adjustments** (e.g., updating `command-list.txt`) but has already merged the series to `seen` for integration testing.

---

## In brief

**`--missing-only` for `git rev-list`** -- Siddharth Asthana’s `--missing-only` option for `git rev-list` received final maintainer sign-off and was queued for `next`. The feature directly supports GitLab’s Gitaly partial clone workflows by enabling efficient identification of missing objects in a single-pass transaction packing operation.

**OCSP staple validation** -- The `http.sslVerifyStatus` feature, which enables OCSP staple validation for HTTPS connections, graduated to `master`. The feature adds a boolean configuration option (default `false`) that causes Git to fail connections if the server does not provide a valid OCSP staple, addressing a security gap for government and FIPS-compliant users.

**Outreachy December 2026 cohort** -- The Outreachy December 2026 cohort saw increased participation, with four confirmed or potential mentors (Christian Couder, Usman Akinyemi, Kaartic Sivaraam, Pablo Sabater) and two org admin candidates. Git’s application proposed two project ideas: continuing the removal of global state (libifying code) and improving command argument and option parsing.

**`git var` extension series** -- Andrew Pleeter’s series extending `git var` to provide a unified, scriptable interface to Git’s identity and signing configuration advanced to v9. The controversial `GIT_SIGNING_KEY` variable was removed due to ambiguity across signing mechanisms, and the series now focuses on six identity variables.

**`git stash` autostash bugfix** -- A **critical bug** in `git stash` autostash handling of staged index entries was fixed in a five-patch series that reached v3. The bug, which could corrupt staged entries during autostash operations when `stash.index=true` was set, was caused by a race condition in the subprocess-based index merging logic. The fix replaces this with **in-core merging via `merge-ort.c`**, eliminating the race condition.

**CI and build system improvements** -- Several **CI and build system improvements** landed this month, including fixes for Windows unit tests, Meson build optimizations, Rust compatibility on Cygwin, and leak sanitizer reporting in GitHub Actions.

**Git v2.56.0-rc1** -- Junio C Hamano announced the first release candidate of Git v2.56.0, marking the start of the stabilization period for the upcoming release. The release candidate includes 721 non-merge commits since v2.55.0, contributed by 90 people, 37 of whom are first-time contributors.

**Localization updates** -- Jiang Xin submitted l10n updates for Git 2.56.0, adding Afrikaans and Brazilian Portuguese translations and refreshing eight existing languages.

**Test modernization** -- Mark C. Chu-Carroll posted v4 of a test modernization series updating three legacy test scripts to use current Git test conventions, improving readability and maintainability without altering behavior.

**What's cooking** -- Junio C Hamano’s “What's cooking in git.git” reports provided comprehensive overviews of integration status for Git 2.56-rc0 and -rc1. Key highlights included Rust integration topics in `next`, ODB abstraction work, ref storage format standardization, and performance optimizations.

---

## Looking ahead

The next month is likely to see **continued stabilization work around Git 2.56.0**, with particular attention to **integration of the `--force-if-includes` fix**, the **`reference-transaction` hook regression fix**, and the **`git stash` autostash bugfix**. The **`strbuf` safety API RFC** will likely see further discussion, with Junio and Jeff King’s architectural feedback prompting a potential redesign. The **Git 3.0 timeline discussion** will continue, with downstream projects (e.g., Git for Windows) likely to weigh in on the proposed version numbering and default changes. The **ODB abstraction effort** will advance with pluggable fsck checks and ongoing alternates handling work. Finally, the **Outreachy December 2026 cohort** will begin engaging with the project, and the first patches from the new contributors may appear on the list.