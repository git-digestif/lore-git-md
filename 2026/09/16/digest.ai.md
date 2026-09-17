# Git mailing list daily digest for 2026/09/16

## The day in brief
The Git project tagged **v2.56.0-rc1**, marking the start of the stabilization period for the next release, while a critical regression fix for `git rebase` involving commit graphs and submodules was queued for integration. Reviewers debated implementation approaches for a Windows compatibility bug in `compat/winansi.c`, and the Outreachy December 2026 cohort expanded its project list with a new promisor-remote fetch ordering proposal.

## Notable threads

### Regression fix for `git rebase` with commit graphs and submodules
A regression in `git rebase` (and related operations like merge and fetch) that caused fatal errors when commit graphs were enabled and submodule pointer changes were involved has been resolved. The fix, authored by Orestis Floros, modifies `commit-reach.c` to propagate the `struct repository *` parameter through reachability functions, ensuring commits are parsed in the correct repository. This prevents cross-repository commit-graph lookups that previously triggered the error `fatal: invalid commit position. commit-graph is likely corrupt`.

[2026/09/16/14-55-33 by Kristofer Karlsson] Kristofer Karlsson confirmed the new test case in `t/t6437-submodule-merge.sh` reliably reproduces the bug and noted the patch reduces `commit-reach.c`'s reliance on `the_repository` from 17 to 11 instances, aligning with the broader effort to eliminate implicit global state. The patch is now queued for integration, with Junio C Hamano calling it "reasonable" and ready for the `next` branch.

The fix is narrowly scoped to the reachability functions and includes a deterministic test case that verifies the regression is resolved. No further changes are expected, and the patch should graduate to `master` in the near term.

---

### Outreachy December 2026 cohort expands project list
Git’s participation in the Outreachy December 2026 cohort is underway, with the project list expanding to include a new proposal: **"Implement promisor remote fetch ordering."** The project aims to improve partial-clone functionality by allowing configurable fetch ordering for promisor remotes, optimizing performance for large repositories.

[2026/09/16/20-58-11 by Christian Couder] Christian Couder agreed to include the new project in the Outreachy application, calling it a good fit despite its perceived difficulty. However, he expressed reservations about a second proposed project, **"Enhance promisor-remote protocol for better-connected remotes,"** describing it as riskier and more challenging. The final decision on the second project was deferred to Kaartic Sivaraam, who would be mentoring it. Christian’s preference is to limit the expansion to the first project, and no mentor assignments for the new project have been finalized yet.

The Outreachy application deadline is September 11, 2026, and sponsorship discussions are still pending. The thread remains administrative, with no technical implementation work yet underway.

---

### Fix for `--force-if-includes` checking the wrong reflog
A bugfix series addressing flaws in the `--force-if-includes` safety mechanism in `git push` continues to advance, with reviewers now focusing on clarity and maintainability. The series fixes two critical issues: (1) the mechanism incorrectly checked the reflog of the *local* branch matching the *remote* destination branch, and (2) it unnecessarily rejected fast-forward pushes of non-branch refs (e.g., tags), breaking workflows.

[2026/09/16/12-29-28 by D. Ben Knoble] D. Ben Knoble raised a low-weight concern about the `deferred_reject_reason` variable name in `set_ref_status_for_push`, suggesting it could be clearer for future maintainability. The variable defers reflog checks until after fast-forward determination, and Knoble proposed a rename (e.g., `deferred_non_ff_reject_reason`) or additional comments to clarify its purpose. The author, Tyler Cipriani, acknowledged the name is "a little broad" and invited further feedback before rerolling.

[2026/09/16/15-52-34 by Tyler Cipriani] The author confirmed the logic is correct and signaled openness to a v6 reroll addressing the variable name and a commit message typo ("push force" → "force push"). The series is technically complete, with 108 lines of new test coverage in `t/t5533-push-cas.sh` and no external dependencies. The fast-forward exemption logic remains uncontroversial, and the series is poised for final approval once the minor readability improvements are addressed.

---

### Windows compatibility bug in `compat/winansi.c`
A long-standing formatting bug in `compat/winansi.c:die_lasterr()` that caused Windows API failure messages to display the address of a `va_list` instead of the intended integer handle sparked a substantive discussion about implementation approaches. The bug affects only the `DuplicateHandle()` failure path, as other callers pass fixed strings.

[2026/09/16/06-13-15 by Johannes Sixt] Johannes Sixt proposed removing the variadic argument from `die_lasterr()` entirely, arguing that the handle value is opaque and unhelpful for debugging. This would simplify the code but remove potentially useful diagnostic information. René Scharfe countered with a macro-based alternative that forwards arguments directly to `die_errno()`, avoiding heap operations and `va_list` manipulation.

[2026/09/16/07-09-51 by Johannes Sixt] Sixt rejected the idea of degrading the Windows-specific `GetLastError()` value to a POSIX `errno`, implying a preference for a simpler, Windows-specific solution. The discussion remains unresolved, with three distinct approaches now on the table: Yongqiang Tian’s original patch (preserving the handle value via `strbuf`), Scharfe’s macro, and Sixt’s implied preference for a non-`errno` solution. No new implementation has been proposed, and the thread awaits further input from Windows platform maintainers.

---

### Bug in `stash.index=true` with `--autostash` during fast-forward merge
A bug in Git 2.55.0 (and current `master`) causes `git merge --ff-only --autostash` to leave behind a redundant stash entry and fail to clean up the `MERGE_AUTOSTASH` ref when `stash.index=true` is set and a staged change exists at merge time. The operation succeeds but emits an error message, and the stash list contains an extra entry.

[2026/09/16/13-35-26 by Phillip Wood] Phillip Wood proposed two fixes: a surgical fix replacing `reset_head()` with `reset_tree(&c_tree, 0, 1)` to avoid branch-state cleanup, and a more ambitious refactoring using `merge_incore_nonrecursive()` for a three-way index merge. The latter would eliminate three subprocess calls and align the index merge with the working tree merge logic, which already uses `merge_ort_nonrecursive()`. The infrastructure for this refactoring is partially in place.

[2026/09/16/14-30-30 by Ben Knoble] D. Ben Knoble committed to testing both approaches and may contribute implementation work. The thread remains in the design phase, with no patches posted yet. The choice between the surgical fix and the refactoring will likely hinge on risk tolerance and the desire to eliminate subprocess calls. Eli Barzilay, the original reporter, also offered to test patches as a non-developer observer, signaling end-user demand for a fix.

---

### Git v2.56.0-rc1 released
[2026/09/16/18-00-51 by Junio C Hamano] Junio C Hamano announced the first release candidate of Git v2.56.0, marking the start of the stabilization period. The release candidate includes 721 non-merge commits since v2.55.0, contributed by 90 people, 37 of whom are first-time contributors. Key changes span UI improvements, performance optimizations, internal refactoring, and development support, with notable updates to `git status`, `git pull`, `git log --follow`, and the experimental `git history` command.

The release also includes progress on ongoing efforts like the `the_repository` removal, ODB abstraction, and reftable backend improvements. Memory safety fixes and platform-specific updates (Windows, macOS) are also included. The final release is expected soon, pending community testing and any necessary fixes.

---

## In brief
- **[2026/09/16/16-07-27 by Junio C Hamano]** A two-patch bugfix series fixing pathspec handling during directory traversal is ready for the `next` branch. The series addresses a heap-buffer-overflow in `match_pathspec_with_flags()` and a logical error in `common_prefix_len()` that broke an optimization for avoiding unrelated directory scans. All technical feedback has been incorporated, and the series is narrowly scoped to pathspec handling and directory traversal.
- **[2026/09/16/06-01-14 by Qin ShiCheng]** The author of a six-patch series fixing a data-loss race in `git repack -d` agreed to withdraw patch 1/6 in favor of Justin Tobler’s ODB-focused series, which provides the same guarantees (no foreign `.keep` removal, signal-safe cleanup) more cleanly. Patches 2/6–6/6 remain relevant and will proceed independently, addressing the repack side of the race.
- **[2026/09/16/04-52-23 by Brigham Campbell]** A feature patch extending `git-contacts` to read patch input from stdin when no arguments are provided was updated to v2, addressing Junio C Hamano’s stylistic feedback. The patch is narrowly scoped and ready for merging unless new concerns surface.
- **[2026/09/16/20-32-21 by Royce Remer]** A bugfix for a Git protocol v2 regression in `upload-pack` that caused client crashes during shallow fetches when `uploadpack.allowRefInWanted=true` is enabled was updated to v2 with an improved commit message. The patch swaps the order of `send_shallow_info()` and `send_wanted_ref_info()` to match the client’s expectations and is validated by two new test cases.
- **[2026/09/16/21-39-49 by Junio C Hamano]** A bugfix for missing `GIT_WORK_TREE` export in worktree `post-checkout` hooks was proposed, with Junio noting a "large NEEDSWORK comment" in the surrounding code. The patch exports `GIT_WORK_TREE` alongside `GIT_DIR` when a worktree is detected, but the broader hook environment handling may need reconsideration.
- **[2026/09/16/19-07-35 by Junio C Hamano]** The "What’s cooking" report for September 2026 summarized the state of integration branches, noting 87 commits in `next` and 166 in `seen`. Graduated topics include `jc/rust-cargo-build-target` and `ps/odb-pluggable-fsck`, while stalled topics like `mh/rust-crate-subdir` and `mm/diff-process-hunks` await review or rerolls.