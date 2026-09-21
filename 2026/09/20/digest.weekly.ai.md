# The Git Project -- Weekly Digest for 2026/09/14 -- 2026/09/20

## The period in brief

This week (2026/09/14--2026/09/20) saw **high-volume, high-impact activity** on the Git mailing list, with **six days of sustained traffic** and **no weekend lull**. The period was **exceptionally eventful**, featuring **critical bugfixes** (repack data-loss race, `--force-if-includes` logic flaw, `rerere` conflict resolution), **major feature graduations** (`http.sslVerifyStatus`, Rust infrastructure reorganization), and **architectural discussions** (ODB transactions, "safe" strbuf API). **Git v2.56.0-rc1** was announced, marking the start of the stabilization period for the next release. **Three things a reader absolutely should not miss**: the **repack data-loss race fix**, the **Rust infrastructure reorganization**, and the **`reference-transaction` hook regression fix**.

---

## Key developments

### Repack data-loss race fix lands
A **critical data-loss bug** in `git repack -d`—where concurrent pushes could cause objects to vanish despite `.keep` files—was **fixed in a six-patch series** by Qin ShiCheng (qeesung). The bug manifested when `git repack -d` deleted packs whose objects were never copied elsewhere, leaving dangling refs. The fix replaces the `--honor-pack-keep` mechanism with a **snapshot-based approach**, ensuring `.keep` files are managed consistently. Key patches include:
- **Patch 1/6**: Prevents `receive-pack` from removing `.keep` files it didn’t create.
- **Patch 2/6**: Fixes traversal logic in `pack-objects` to treat `--keep-pack` packs as "kept-open."
- **Patch 5/6**: Adds `--keep-pack-from-file` to handle repositories with many kept packs.
- **Patch 6/6**: Passes `repack`’s snapshot of `.keep` packs to `pack-objects`.

The series is **production-tested** and includes **comprehensive test coverage**. Justin Tobler’s review proposed an alternative ODB-focused refactoring, but the incremental fix was deemed safer for immediate integration. The series **landed in `next`** and is poised for `master`.

---

### Rust infrastructure reorganization queued for `next`
The **long-running effort to reorganize Git’s Rust code** into a dedicated `rust/` subdirectory reached a milestone. Mike Hommey’s **v5 patch** moves Rust sources from `src/` to `rust/src/` while preserving build artifact locations, addressing project hygiene and downstream vendoring concerns. The patch updates the **Makefile, meson.build, and CI scripts** to reference the new paths and was **queued for `next`** after Junio C Hamano confirmed it applies cleanly on Git 2.56-rc1.

The reorganization is a **prerequisite for Git 3.0’s mandatory Rust components** and resolves a blocker for downstream packagers. Minor stylistic questions (e.g., `CARGO_MANIFEST_DIR` usage) remain open but are not blockers. The patch is **uncontroversial** and marks a key step toward Rust integration.

---

### `reference-transaction` hook regression fix advances
A **regression in the `reference-transaction` hook**—where branch renames (`git branch -m`) omitted the creation of the new ref—was **diagnosed and fixed** in a **two-patch series** by Maciej Ciemborowicz. The fix introduces a new transaction type (`REF_TRANSACTION_TYPE_RENAME`/`COPY`) and a `REF_TRANSACTION_FLAG_SKIP_HOOK` flag to suppress hooks for nested operations. It ensures both the deletion of the old ref and the creation of the new ref are reported as a **single transaction**, addressing a long-standing inconsistency between `git branch -m` and `git update-ref --stdin`.

The series includes **thorough test coverage** for rename, copy, forced updates, D/F conflicts, and hook-triggered rollback. Karthik Nayak identified a **race condition** in a related series restoring the old OID in the hook for deletions, but the fix for branch renames is **technically complete** and ready for review.

---

### `--force-if-includes` logic flaw fixed
A **critical flaw in `--force-if-includes`**—where the feature incorrectly checked the reflog of the *local* branch matching the *remote* destination—was **fixed in a six-patch series** by Tyler Cipriani. The series addresses two issues:
1. The feature checked the wrong reflog (e.g., `refs/heads/main` instead of `refs/heads/src` in `src:main`).
2. It unnecessarily rejected fast-forward pushes of non-branch refs (tags, detached HEADs).

The v6 reroll renames the `deferred_reject_reason` variable to `needs_force_reject_reason` for clarity and adds a new `ref->unverifiable` flag to provide actionable advice for rejected non-branch pushes. The series includes **108 lines of new test coverage** and is **ready for final review**.

---

### `http.sslVerifyStatus` graduates to `master`
The **`http.sslVerifyStatus` feature**, which enables OCSP staple validation for HTTPS connections, **graduated to `master`**. The feature adds a boolean configuration option (default `false`) that causes Git to fail connections if the server does not provide a valid OCSP staple. This addresses a **security gap for government and FIPS-compliant users** (e.g., US DoD PKI deployments) where OCSP stapling is mandated.

The patch merges the author’s original tests with Patrick Steinhardt’s OCSP test harness, providing **comprehensive coverage** of fail-closed behavior, per-URL scoping, and staple validation. The feature is **technically complete**, with all review feedback addressed.

---

### "Safe" strbuf API RFC sparks architectural discussion
Derrick Stolee posted an **RFC series introducing a "safe" strbuf API** that cannot call `die()` or `exit()`, motivated by trace2’s need to avoid recursive fatal errors during memory allocation failures. The series includes:
- **Patch 1**: Moves `strbuf` struct definitions to `strbuf-safe.h`.
- **Patch 3**: Introduces `safe_memory_limit_check()` to avoid `die()`.
- **Patch 4**: Implements `sstrbuf_grow()`, the first safe method.

Phillip Wood’s review identified a **critical bug**: `sstrbuf_grow()` uses `st_add3()`, which calls `die()` on overflow, violating the safety guarantee. He proposed replacing it with `st_add_overflows()` and critiqued the API’s framing, suggesting it be described as **error-returning** with naming aligned to existing patterns (e.g., `strbuf_*_gently`). The discussion highlights the tension between **safety guarantees** and **usability** in critical code paths.

---

### Git v2.56.0-rc1 announced
Junio C Hamano announced **Git v2.56.0-rc1**, marking the start of the stabilization period for the next release. The release candidate includes **721 non-merge commits** since v2.55.0, contributed by **90 people**, 37 of whom are first-time contributors. Key changes span:
- **UI/workflows**: New `git refs` subcommands, `git history drop`.
- **Performance**: ODB abstraction, reftable backend improvements.
- **Development support**: Build system updates, CI infrastructure.

The final release is expected soon, pending community testing.

---

## In brief

**Advice system** -- Junio C Hamano posted v5 of a patch adding a scope hint (`--global`) to Git’s advice settings, implementing the now-consensus design for uniform scope recommendations.

**Autostash bugfix** -- D. Ben Knoble submitted a two-patch series fixing a bug in `git stash` where autostashing failed to correctly handle staged index entries when `stash.index=true` is set. The fix replaces subprocess-based logic with in-core merging via `merge-ort.c`.

**Coverity-reported issues** -- Johannes Schindelin posted a seven-patch series addressing Coverity-reported static-analysis warnings in Git for Windows, including potential signed-overflow in `writev`, missing input validation in GPG signature parsing, and MIDX pack-ID checks.

**GitLab CI Windows breakage** -- Johannes Schindelin’s four-patch series fixing Windows CI breakage after Rust support was enabled passed validation. The series ensures the correct Rust toolchain is provisioned and preserves `.git/info/exclude` contents.

**Outreachy December 2026** -- Christian Couder confirmed a third project, “Implement promisor remote fetch ordering,” for Git’s Outreachy December 2026 cohort. Kaartic Sivaraam is tentatively confirmed as mentor.

**`git rerere remaining` bugfix** -- Junio C Hamano posted a patch fixing a bug where consecutive conflicted paths with only stage-1 entries were incorrectly skipped. The fix adds a same-path check (`ce_same_name()`) to `check_one_conflict()` in `rerere.c`.

**`git status --ignored` regression** -- René Scharfe posted a patch fixing a regression where ignored directories were incorrectly matched when the pathspec was a substring of the directory name.

**Sequencer auto maintenance** -- Thomas Bachem posted v5 of a three-patch series deferring auto maintenance until sequencer-driven operations (`git rebase`, `git cherry-pick`) complete. The series is code-identical to v4, with only commit messages and test annotations updated.

**Shallow clone advice** -- Harald Nordgren submitted a patch adding a new advice message (`advice.shallowHistory`) to explain why `<ref>~N` or `<ref>^N` fails in a shallow clone. The advice suggests the exact `git fetch --deepen=<n>` command needed.

**Test modernization** -- Mark C. Chu-Carroll posted v4 of a test modernization series updating three legacy test scripts to use current Git test conventions. The changes are purely mechanical and improve readability.

**Worktree lifecycle hooks** -- Maciej Ciemborowicz proposed adding native Git hooks for worktree operations (`add`, `remove`, `move`, `lock`, `unlock`, `prune`). The proposal avoids veto hooks and focuses on post-operation events.

---

## Looking ahead

The next week is likely to see **continued stabilization for Git v2.56.0**, with a focus on **regression fixes** and **test coverage**. Key topics to watch:
- **`reference-transaction` hook regression fix**: The v2 series restoring the old OID in the hook for deletions is **ready for final review**, with the race condition trade-off as the remaining open question.
- **"Safe" strbuf API**: Derrick Stolee’s RFC will likely see a v2 addressing Phillip Wood’s feedback on the bug and usability concerns.
- **ODB transactions**: Justin Tobler’s ODB-focused refactoring for the repack data-loss race may gain traction as an alternative to the incremental fix.
- **Rust integration**: The `rust/` subdirectory reorganization is **queued for `next`**, and downstream packagers will begin testing the new structure.
- **Outreachy December 2026**: Mentor assignments are finalized, and the application deadline (October 5, 2026) is approaching. Project details will be refined in the coming weeks.