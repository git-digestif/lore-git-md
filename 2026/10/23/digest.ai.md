# Git mailing list daily digest for 2026/10/23

## The day in brief
The incremental connectivity check series was merged to `next`, completing a major performance feature for fetches and pushes. Two CI/build system patches (macOS dependency cleanup and stricter tool installation) advanced toward final review, while a shallow-fetch regression fix hit a memory-management blocker. Documentation efforts included the `gitmergeconflicts(7)` guide cooking in `next` and a glossary terminology cleanup under review. Outreachy contributions and test modernizations continued to land, and a philosophical debate emerged over `git status` defaults in triangular workflows.

## Notable threads

### Incremental connectivity check merged to `next`
**Thread root**: 2026/09/14/09-47-54
**What changed**: Kristofer Karlsson’s six-patch series introducing an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental` or `auto`) was signed off by Junio C Hamano and merged to `next`. The series includes a configurable 1,000-commit cutoff for the auto mode, which dynamically switches between incremental and full checks based on incoming history size.
**Why it matters**: The feature dramatically reduces the cost of verifying object connectivity during transfers, particularly for small pushes to large repositories (up to 190× faster, with memory usage reduced from 3.4 GB to 14 MB). The auto mode makes the optimization practical for users who occasionally receive large pushes, addressing a long-standing scalability bottleneck.
**Subsystem**: fetch/push, connectivity check
**Files touched**: `tree-verify.c`, `connected.c`, `Documentation/config/transfer.adoc`, `t/t5413-connectivity-check.sh`, `t/perf/p5413-connectivity-check.sh`
**Key symbols**: `transfer.connectivityCheck`, `transfer.connectivityCheckIncrementalMaxCommits`, `--connectivity-check=incremental`
**Status**: Merged to `next` (cooking)

---

### CI/build system: macOS dependency cleanup unblocked
**Thread root**: 2026/09/12/07-55-39
**What changed**: Harald Nordgren provided new empirical evidence—a passing GitHub Actions pipeline with all macOS tests—to address Patrick Steinhardt’s earlier objection about skipped macOS jobs. The patch removes a redundant `brew link --force gettext` command from the macOS CI dependency installation script, motivated by a 2020 Homebrew policy change that made gettext no longer keg-only.
**Why it matters**: The change reduces log noise (spurious “Already linked” warnings) while ensuring gettext remains correctly exposed in the system path. The patch is now unblocked and ready for final review unless new concerns arise.
**Subsystem**: GitHub Actions CI (macOS runner)
**Files touched**: `ci/install-dependencies.sh` (one line deleted)
**Status**: Awaiting final approval

---

### CI/build system: stricter tool installation approach finalized
**Thread root**: 2026/09/12/14-38-02
**What changed**: Junio C Hamano reviewed the v2 patch implementing Patrick Steinhardt’s stricter approach: failing the CI build immediately if Perforce, Git LFS, or JGit fail to install on platforms where they are required. The patch replaces the original warning-based logic with mandatory installation checks, removing redundant presence checks and moving version output inline for clarity.
**Why it matters**: The change ensures broken tool downloads are caught early rather than silently skipped, reducing the risk of undetected test skips. Junio raised a minor consistency question about the macOS Perforce installation (the `xattr` line is allowed to fail silently, but the `tar` extraction is not), but this was left as-is to avoid complicating error handling.
**Subsystem**: GitHub Actions CI
**Files touched**: `ci/install-dependencies.sh`
**Status**: Ready for final review

---

### Shallow-fetch regression fix hits memory-management blocker
**Thread root**: 2026/10/08/22-10-37
**What changed**: Karthik Nayak acknowledged a blocker-level memory management issue in the v4 patch’s handling of `negotiation_include_tips`. The problem stems from shared ownership of the `oid_array`: the patch appends fetched OIDs to `negotiation_include_tips` without ensuring exclusive control, risking state corruption or memory leaks if the array is reused by other callers.
**Why it matters**: The fix restores dynamic reloading of alternate object databases when an object lookup fails, addressing a regression introduced in March 2026. The memory management issue must be resolved before the patch can proceed, likely requiring a redesign to avoid shared-state risks.
**Subsystem**: fetch, ODB alternates
**Files touched**: `builtin/fetch.c`, `t/t5510-fetch.sh`
**Key symbols**: `negotiation_include_tips`, `prepare_alternates()`
**Status**: Blocked on memory management redesign

---

### `gitmergeconflicts(7)` guide cooking in `next`
**Thread root**: 2026/09/24/14-44-15
**What changed**: Junio C Hamano confirmed the trivial mechanical blocker (adding an empty `SYNOPSIS` section to `gitmergeconflicts.adoc`) was addressed by the merge to `seen`, unblocking the series for graduation to `next`. The guide consolidates merge-conflict resolution guidance, replacing scattered advice across multiple man pages.
**Why it matters**: The new guide addresses long-standing user pain points, such as confusion over “ours” vs. “theirs” terminology and unclear references (e.g., “steps 2 and 3”). It includes practical examples, a side-by-side comparison of `diff3` vs. default merge conflict formats, and an exhaustive list of commands that can cause conflicts.
**Subsystem**: Documentation
**Files touched**: `Documentation/gitmergeconflicts.adoc`, `git-merge.adoc`, `git-rebase.adoc`, `git-revert.adoc`, `git-cherry-pick.adoc`, `git-pull.adoc`
**Status**: Cooking in `next`

---

### Glossary terminology cleanup under review
**Thread root**: 2026/10/22/21-28-48
**What changed**: Andy Koppe’s five-patch series aligning Git’s documentation with the 2024 terminology change (distinguishing “pseudoref” from “root ref”) received substantive feedback from Patrick Steinhardt and Junio C Hamano. The series updates the glossary, man pages, and code comments to replace outdated uses of “pseudoref” with “root ref” for refs outside the `refs/` hierarchy (e.g., HEAD, ORIG_HEAD).
**Why it matters**: The cleanup resolves inconsistencies between documentation and code, improving clarity for users and contributors. Julia Evans proposed expanding the “References” section of `gitdatamodel.adoc` to cover pseudorefs and root refs, addressing a gap in the pedagogical documentation.
**Subsystem**: Documentation, glossary
**Files touched**: `Documentation/glossary-content.txt`, `git-update-ref.1`, `git-worktree.1`, `builtin/for-each-ref.c`, `refs.h`, `t/t1400-update-ref.sh`
**Key symbols**: `root ref`, `pseudoref`
**Status**: Under review; awaiting author response to feedback

---

### `git status` defaults in triangular workflows: philosophical debate
**Thread root**: 2026/10/19/19-23-18
**What changed**: Junio C Hamano and Harald Nordgren engaged in a philosophical debate over Harald’s patch to make `git status` compare against both `@{upstream}` and `@{push}` by default in triangular workflows. Junio questioned whether the change accommodates a “sad world” where users lack foundational Git knowledge due to GitHub’s influence, while Harald argued the patch is a pragmatic response to reality.
**Why it matters**: The discussion highlights tensions between idealism (teaching users “correct” Git usage) and pragmatism (adapting to common workflows). Junio suggested external tools (e.g., GitHub’s UI) should promote the necessary configuration (`status.compareBranches = @{upstream},@{push}`) instead of changing Git’s defaults.
**Subsystem**: `git status`, triangular workflows
**Files touched**: `remote.c`, `Documentation/config/status.adoc`, `t/t6040-tracking-info.sh`
**Key symbols**: `status.compareBranches`
**Status**: Under review; justification debate unresolved

---

### Outreachy contributions and test modernizations
**Notable patches**:
1. **Test modernization**: Shashank S’s patch replacing `test -d` with `test_path_is_dir` in `t/t2403-worktree-move.sh` was queued by Junio C Hamano (Thread: 2026/10/16/04-31-51).
2. **i18n pluralization fix**: Tanga Joris’s patch converting `_()` to `Q_()` for two messages in `builtin/checkout.c` was queued (Thread: 2026/10/21/02-21-56).
3. **Test robustness**: Cynthia Uche’s patch replacing `git ls-files -t | test_cmp` with a two-step sequence in `t/t2104-update-index-skip-worktree.sh` was merged to `master` (Thread: 2026/10/21/03-37-23).

## In brief
- **`git branch --delete-merged`**: Harald Nordgren confirmed the CLI design constraint, eliminating hierarchical or config-based approaches for squash/rebase detection (Thread: 2026/09/29/07-30-30).
- **`git reflog` usability**: Harald Nordgren’s series adding `--operations` and `--historical-branch` options remains under review, with Junio C Hamano raising an architectural concern about the preparatory refactoring (Thread: 2026/10/15/13-13-55).
- **`git stash` shell completion**: D. Ben Knoble’s v2 series (refactoring, Zsh bugfix, `save` subcommand removal) received Junio’s endorsement for the refactoring patch (Thread: 2026/10/12/17-48-26).
- **Hook configuration edge cases**: Thomas Bachem’s series making `hook.<event>.enabled` disable both config-defined and hookdir hooks was queued by Junio (Thread: 2026/10/20/18-19-11).
- **SHA-1/SHA-256 interoperability**: Christian Couder’s 13-patch series implementing pack index v3 was merged to `next` (Thread: 2026/10/23/07-25-45).
- **CI security**: Taylor Blau’s patch pinning GitHub Actions to full commit IDs sparked a policy debate over upstream vs. downstream modifications (Thread: 2026/10/23/16-38-23).
- **Outreachy microproject**: Fien Dora proposed adding test coverage for `git check-ref-format --no-allow-onelevel` (Thread: 2026/10/23/15-29-12).