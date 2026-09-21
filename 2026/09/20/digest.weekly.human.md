# The Git Project -- Weekly Digest for 2026/09/14 -- 2026/09/20

## The period in brief

The week of 2026/09/14--2026/09/20 was unusually eventful, with six active days of high-volume traffic. The mailing list saw the **finalization of two major security features** (`http.sslVerifyStatus` and `--force-if-includes` fixes), the **resolution of a critical data-loss bug in `git repack`**, and the **graduation of Rust infrastructure reorganization** to `next`. Outreachy mentor assignments were completed, and Git v2.56.0-rc1 was announced, marking the start of the stabilization period for the next release. Two long-running architectural efforts—the `git var` extension and ODB transaction safety—advanced significantly, while a new "safe" `strbuf` API RFC sparked architectural discussion.

A reader who followed the daily digests closely will find little new here; a reader who skipped them will learn what mattered: the repack data-loss fix, the `--force-if-includes` logic overhaul, the OCSP staple validation feature, and the Rust reorganization are the developments that will shape Git 2.56 and beyond.

---

## Key developments

### **Repack data-loss race fixed after production exposure**
A six-patch series from Qin ShiCheng (qeesung) fixed a race condition in `git repack -d` where concurrent pushes could cause the repack machinery to delete packs whose objects were never copied elsewhere, resulting in data loss. The series replaces the `--honor-pack-keep` mechanism with a consistent snapshot of kept packs observed at repack startup, ensuring `.keep` files are only removed by the process that created them. The fix is production-tested and includes optimizations for repositories with many kept packs, yielding an 11× speed-up in `--keep-pack` lookups. The series landed in `next` and is poised for `master`, addressing a subtle but serious bug that affects repositories with concurrent push operations.

### **`--force-if-includes` logic overhauled to check the correct reflog**
Tyler Cipriani posted a six-iteration series fixing two critical flaws in the `--force-if-includes` safety mechanism in `git push`. The feature previously checked the reflog of the *local* branch matching the *remote* destination name (e.g., `refs/heads/main`) instead of the branch actually being pushed (e.g., `refs/heads/src` in `src:main`), and unnecessarily rejected fast-forward pushes of non-branch refs (tags, detached HEADs). The final v6 patch renames the `deferred_reject_reason` variable to `needs_force_reject_reason` for clarity, consults the reflog of the pushed ref (`ref->peer_ref->name`), and defers reflog checks until after fast-forward determination. The series also introduces `advice.forceIfIncludesDetachedHead` to provide actionable guidance for rejected cases and includes 108 lines of new test coverage. All prior review feedback from Patrick Steinhardt, Junio C Hamano, and D. Ben Knoble is resolved, and the series is ready for final scrutiny.

### **OCSP staple validation graduates to `master`**
The `http.sslVerifyStatus` feature, which enables OCSP staple validation for HTTPS connections, graduated to `master`. The feature adds a boolean configuration option (default `false`) that causes Git to fail connections if the server does not provide a valid OCSP staple. This addresses a security gap for government and FIPS-compliant users (e.g., US DoD PKI deployments) where OCSP stapling is mandated. The final combined patch merges the author's original tests with Patrick Steinhardt's OCSP test harness, providing comprehensive coverage of fail-closed behavior, per-URL scoping, and staple validation. The feature is now technically complete, with all review feedback addressed and test coverage confirmed.

### **Rust infrastructure reorganization queued for `next`**
The effort to reorganize Git’s Rust code into a dedicated `rust/` subdirectory reached a milestone. Mike Hommey’s v5 patch moves Rust sources from `src/` to `rust/src/` while keeping build artifacts in their original locations, addressing project hygiene and downstream vendoring concerns. The patch updates the build system (Makefile, meson.build, CI scripts) to reference the new locations and is now queued for `next`. Junio C Hamano confirmed the patch applies cleanly on Git 2.56-rc1 and correctly updates the build system, marking a key step toward Git 3.0’s mandatory Rust components.

### **`git var` extension series nears completion**
Andrew Pleeter’s series extending `git var` to provide a unified, scriptable interface to Git’s identity and signing configuration advanced to v8. The series adds support for identity components (name/email/date for author and committer), signing keys (`GIT_SIGNING_KEY`), multiple variable arguments, and NUL-terminated output with `-z`. Junio C Hamano requested the series be split into three smaller patches—(1) adding `-z` output mode, (2) enabling multi-variable queries, and (3) introducing the new variables—for clarity and maintainability. The v8 patch is technically complete and ready for integration, with Andrew agreeing to the reorganization. The series remains in `seen` pending the v9 reroll.

### **Sequencer auto-maintenance deferral series reaches v5**
Thomas Bachem posted the fifth iteration of a three-patch series adjusting when auto maintenance runs during sequencer-driven operations (`git rebase`, `git cherry-pick`, `git revert`). The series defers auto maintenance until sequence completion, matching the apply backend’s behavior and preventing maintenance from interfering with intermediate steps. The v5 iteration is code-identical to v4; only commit messages, header comments, and test annotations have been updated to address reviewer feedback. The series is now in `seen` and likely to proceed to `next` unless new objections arise.

### **Stash/merge autostash race fixed with in-core merging**
D. Ben Knoble submitted a two-patch series fixing a bug in `git stash` where autostashing failed to correctly handle staged index entries when `stash.index=true` is set. The bug, reported by Eli Barzilay, manifested during merge operations that trigger autostash, causing incorrect index state preservation due to a race condition in the subprocess-based logic (`git diff-tree` and `git apply`). The fix replaces the external process calls with in-core merging via `merge-ort.c`, ensuring staged entries are properly preserved. The series also adds a test case to `t/t7600-merge.sh` covering the bug scenario. Phillip Wood’s earlier feedback has been incorporated, and the series is ready for review.

### **Reference-transaction hook regression and branch rename visibility fixed**
Maciej Ciemborowicz advanced two critical ref backend fixes. The first, a v2 series, restores the correct behavior of the `reference-transaction` hook when branches, tags, or remote refs are deleted, addressing a regression introduced in Git 2.31 where the hook received all-zero OIDs instead of the resolved old OID. The series makes deletions conditional if an OID is provided, restoring the pre-regression safety check. The second patch fixes the long-standing issue where the `reference-transaction` hook omitted the new ref when a branch was renamed (`git branch -m`). The patch introduces a new transaction type (`REF_TRANSACTION_TYPE_RENAME`/`COPY`) and ensures both the deletion of the old ref and the creation of the new ref are reported as a single transaction. Both series include thorough test coverage for files and reftable backends, SHA-1/SHA-256, and edge cases.

---

## In brief

**Advice system** -- Junio C Hamano posted v5 of a patch adding a scope hint to Git’s advice settings, implementing the now-consensus design: always recommending `--global` in the hint for *all* `advice.*` settings. The patch touches `advice.c` and updates twelve test scripts to expect the new `--global` wording.

**Autostash bug** -- D. Ben Knoble’s two-patch series fixing a bug in `git stash` where autostashing failed to correctly handle staged index entries when `stash.index=true` is set was submitted. The fix replaces subprocess-based logic with in-core merging via `merge-ort.c`.

**Build system** -- SZEDER Gábor posted v2 of a build system series introducing precompiled headers for `git-compat-util.h`, updating only the commit message of patch 3/4 to address Junio’s editorial feedback. The series aims to reduce build times by ~35% and is uncontroversial.

**Coverity fixes** -- Johannes Schindelin posted a seven-patch series addressing Coverity-reported static-analysis warnings in Git for Windows after merging v2.56.0-rc0. The issues span potential signed-overflow in writev, missing input validation in GPG signature parsing, MIDX pack-ID checks, rerere conflict resolution edge cases, reftable iterator initialization, fuzz-testing harnesses, and a latent crash in the `test-read-midx` helper.

**Documentation** -- Todd Zullinger’s v3 series fixing inconsistent backtick-quoting in `git-refs` and `git-pack-refs` man pages was accepted by Junio C Hamano and merged to `master`. The changes ensure visual consistency in rendered man pages and HTML output.

**Git v2.56.0-rc1** -- Junio C Hamano announced the first release candidate of Git v2.56.0, marking the start of the stabilization period for the upcoming release. The release candidate includes 721 non-merge commits since v2.55.0, contributed by 90 people, 37 of whom are first-time contributors.

**GitLab CI** -- Karthik Nayak validated Johannes Schindelin’s four-patch series fixing Windows CI breakage after Rust support was enabled. The series addresses missing Rust toolchain provisioning, silent data loss in `.git/info/exclude`, and linker unavailability for MinGW builds.

**Incremental connectivity check** -- Kristofer Karlsson’s opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) for fetch/push operations advanced, with the author addressing Junio’s concerns about promisor object handling and NULL-dereference risks. The feature processes incoming commits in topological order, tracking trusted objects from parent commits to skip unchanged subtrees, and shows dramatic speedups for small pushes to large repositories (up to 190× faster).

**ODB transactions** -- Justin Tobler’s review of Qin ShiCheng’s repack data-loss race fix proposed an alternative: stop relying on `git index-pack` to create `.keep` files prematurely and instead have the ODB transaction’s commit phase create them explicitly. This would eliminate the need for pre-migration tracking and simplify the fix.

**Outreachy** -- Christian Couder confirmed the “Implement promisor remote fetch ordering” project for Git’s Outreachy December 2026 cohort, with Kaartic Sivaraam tentatively confirmed as mentor. The project joins two others already approved: “Improve how command arguments and options are scanned and parsed” and “Reduce Git’s global state to enable Git's libification.”

**Range-diff** -- Harald Nordgren posted v3 of the `--matched-only` feature for `git range-diff`, reverting the error-handling layering violation from v2 and restoring architectural separation. The patch is now mechanically final and ready for merging.

**Reference-transaction hook** -- Karthik Nayak diagnosed a regression in the `reference-transaction` hook where branch renames (`git branch -m`) omit the creation of the new ref. The hook only receives the deletion of the old branch name, affecting both the files-based and reftable backends.

**Rerere bug** -- Junio C Hamano posted a patch fixing a bug in `git rerere remaining` where consecutive conflicted paths with only stage-1 entries were incorrectly skipped. The fix adds a same-path check (`ce_same_name()`) to `check_one_conflict()` in `rerere.c`.

**Safe strbuf API** -- Derrick Stolee posted an RFC series introducing a subset of the `strbuf` API that cannot call `die()` or `exit()`, motivated by trace2’s need to avoid recursive fatal errors during memory allocation failures. Phillip Wood’s review identified a critical bug in `sstrbuf_grow()` and raised usability concerns, suggesting the API be framed as error-returning and aligned to existing patterns.

**Shallow clone advice** -- Harald Nordgren submitted a patch adding a new advice message (`advice.shallowHistory`) to explain why `<ref>~N` or `<ref>^N` fails in a shallow clone. The advice suggests the exact `git fetch --deepen=<n>` command needed to resolve the issue, firing only when the walk stops at a recorded shallow boundary.

**Test modernization** -- Mark C. Chu-Carroll posted v4 of a test modernization series updating three legacy test scripts (`t4001-diff-rename.sh`, `t4009-diff-rename-4.sh`, `t4010-diff-pathspec.sh`) to use current Git test conventions. The changes are purely mechanical, improving readability and maintainability without altering behavior.

**What's cooking** -- Junio C Hamano’s “What's cooking in git.git” report provided a comprehensive overview of integration status for Git 2.56-rc0. Key highlights include Rust integration topics in `next`, ODB abstraction work, ref storage format standardization, new experimental commands like `git history squash`, and performance optimizations like Bloom filter usage in `git last-modified`.

**Worktree hooks** -- Maciej Ciemborowicz suggested adding native Git hooks for worktree operations (`add`, `remove`, `move`, `lock`, `unlock`, `prune`). Two design options were presented: separate post-operation hooks or a unified `worktree-lifecycle` hook. Kristoffer Haugsbakk linked to a prior discussion from 2024, signaling that this is not a novel idea.

---

## Looking ahead

The next week is likely to see continued stabilization work around Git 2.56.0, with particular attention to the **reference-transaction hook fixes** and **stash/merge autostash race**. The **safe strbuf API RFC** will likely see further discussion, with Phillip Wood’s usability concerns shaping the next iteration. The **ODB transaction safety** discussion may converge on a design, with Justin Tobler’s prototype informing the path forward. Outreachy applicants will begin engaging with the project, and the first patches from the new cohort may appear on the list. Finally, the **Rust infrastructure reorganization** will graduate to `master`, marking a key milestone toward Git 3.0.