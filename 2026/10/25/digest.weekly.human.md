# The Git Project -- Weekly Digest for 2026/10/19 -- 2026/10/25

## The period in brief

This week (2026/10/19--2026/10/25) was **exceptionally active**, with **high-volume, high-stakes discussions** across performance, interoperability, governance, and usability. Traffic was **heavy and eventful**, with **two major features merged** (incremental connectivity checks, pack index v3), **one governance proposal gaining momentum** (community voting process), and **multiple long-running debates reaching critical junctures** (SHA-256 interoperability, `git status` defaults, worktree reflink). The most consequential developments were the **incremental connectivity check series landing in `next`**, the **pack index v3 series enabling SHA-1/SHA-256 interoperability**, and the **escalating governance debate** about AI contributions and voting processes. Readers should not miss the **incremental connectivity check feature**, the **pack index v3 interoperability work**, and the **governance discussions** shaping Git’s future.

---

## Key developments

### Incremental connectivity check merged to `next`

**What happened**: Kristofer Karlsson’s six-patch series introducing an opt-in incremental connectivity check for fetches and pushes was merged to `next`. The feature reduces the cost of verifying object connectivity by shifting the cost from repository size to incoming change size, offering up to **190x speedup** for small pushes to large repositories.

**Who was involved**: Kristofer Karlsson (author), Junio C Hamano (maintainer), Patrick Steinhardt (reviewer).

**Current status**: Merged to `next` (2026/10/23). The series introduces new config keys (`transfer.connectivityCheck=full|incremental|auto[=<n>]`, `transfer.connectivityCheckIncrementalMaxCommits=<n>`) and a new internal `rev-list` flag (`--connectivity-check=`). The auto mode dynamically switches between incremental and full checks at a configurable 1,000-commit cutoff.

---

### Pack index v3 enables SHA-1/SHA-256 interoperability

**What happened**: Christian Couder’s 13-patch v2 series implementing pack index v3 was merged to `next`. The new index format stores object IDs in both primary and compatibility hash algorithms, enabling bidirectional mapping between SHA-1 and SHA-256 repositories.

**Who was involved**: Christian Couder (author), Junio C Hamano (maintainer).

**Current status**: Merged to `next` (2026/10/23). The series includes strict validation logic to prevent corruption from malformed indices and is a critical step toward full SHA-1/SHA-256 interoperability. The implementation builds on parts 1 and 2 (already merged to `master`).

---

### Governance: Community voting process proposed

**What happened**: Emily Shaffer proposed a formal voting process to resolve contentious decisions (e.g., AI contribution policy) when consensus cannot be reached. The patch defines eligibility criteria (50 merged patches or multiple code reviews) and a two-year activity window, sparking debate about inclusivity, corporate influence, and the role of voting in Git’s consensus-driven culture.

**Who was involved**: Emily Shaffer (author), Junio C Hamano (maintainer), Kristoffer Haugsbakk, Patrick Steinhardt, brian m. carlson, Phillip Wood (reviewers).

**Current status**: Under active discussion. Junio endorsed secret ballots to mitigate corporate influence, while Kristoffer criticized the proposal as favoring corporate interests. The thread revealed deep divisions about Git’s governance model and the role of corporate contributors.

---

### Worktree reflink: Windows ReFS silent corruption risk

**What happened**: brian m. carlson raised **critical correctness concerns** about the proposed `FSCTL_DUPLICATE_EXTENTS_TO_FILE` implementation for Windows ReFS in Jason Newton’s worktree reflink series. The underlying bug (Git LFS #6312) can cause silent corruption when cloning extents, making the feature unsafe for production use until Microsoft provides a fix.

**Who was involved**: brian m. carlson (reviewer), Jason Newton (author), Eric Sunshine, Phillip Wood (reviewers).

**Current status**: The Windows reflink implementation is **permanently disabled** due to the silent corruption risk. The macOS and Linux implementations remain unaffected. The series faces **substantive objections** on both the reflink design (opt-in vs. automatic) and the ephemeral branch reaping feature.

---

### `git log --follow` performance regression fixed

**What happened**: Miklos Vajna posted a four-patch series fixing a severe performance regression in `git log --follow` when tracking renames across merges. The regression, introduced in June 2026, made `--follow` unusably slow (6m43s vs. 28s on the Linux kernel repository). The fix splits diffing into two passes: a fast pass without rename detection to identify unchanged parents, and a fallback to expensive rename detection only when necessary.

**Who was involved**: Miklos Vajna (author), Junio C Hamano (maintainer).

**Current status**: Under review. The series includes comprehensive test coverage and benchmarks showing a speedup to 36s. The implementation avoids the expensive `propagate_follow_pathspec_to_parent()` function and reuses diffs already computed in `log_tree_diff()`’s parent loop.

---

### SSH signing verification security fixes

**What happened**: Christian Noé Ramos López posted a v2 security bugfix for Git’s SSH signing verification, addressing two critical flaws: (1) a misleading hint message that included full ref names instead of short names, and (2) a revocation file that failed open when missing or unreadable. The patch ensures signature information is displayed consistently and makes revocation file errors fatal by default.

**Who was involved**: Christian Noé Ramos López (author), Junio C Hamano (maintainer), Patrick Steinhardt (reviewer).

**Current status**: Under review. The patch uses Git’s `:(optional)` pathname syntax to provide an escape hatch for revocation file errors. Junio requested a procedural correction for future rerolls, and Patrick helped shape the documentation wording.

---

### `git status` defaults for triangular workflows

**What happened**: Harald Nordgren’s patch to make `git status` compare against both `@{upstream}` and `@{push}` by default sparked a **philosophical debate** about Git’s defaults for triangular workflows. Junio C Hamano argued that `@{push}` divergence is less critical to monitor by default because it reflects the user’s own work, while D. Ben Knoble provided anecdotal evidence of workflows where monitoring `@{push}` divergence is valuable.

**Who was involved**: Harald Nordgren (author), Junio C Hamano (maintainer), D. Ben Knoble (reviewer).

**Current status**: Under review (v3). The discussion centers on whether the benefits for triangular workflow users outweigh the potential noise for others. No consensus has been reached, and the debate remains at an impasse.

---

### `git branch --delete-merged` enhancements

**What happened**: Harald Nordgren’s v3 series extends `git branch --delete-merged` to detect branches whose changes (but not their tips) have been integrated via squash or rebase merges. Junio C Hamano proposed a fundamental CLI redesign—decoupling strict `--delete-merged` (tip-reachability only) from a new `--delete-irrelevant` (change-based detection)—while Harald rejects the split.

**Who was involved**: Harald Nordgren (author), Junio C Hamano (maintainer), Jasper Mayone (reviewer).

**Current status**: Blocked on the CLI design debate. The first patch (refactoring) is uncontroversial and ready for integration. Jasper’s testing confirms persistent false positives, reinforcing Junio’s safety concerns.

---

### `gitmergeconflicts(7)` documentation series finalized

**What happened**: Junio C Hamano queued a six-patch series introducing `gitmergeconflicts(7)`, a new man page consolidating scattered merge-conflict advice across multiple man pages. The guide covers conflict markers, "ours/theirs" terminology, `diff3` vs. default formats, resolution tools, and an exhaustive list of conflict-causing commands.

**Who was involved**: Junio C Hamano (maintainer), Phillip Wood (reviewer).

**Current status**: Merged to `next` (2026/10/23). The series touches 10 files, including the new `Documentation/gitmergeconflicts.adoc` (339 lines) and updates to `git-merge.adoc`, `git-rebase.adoc`, and others.

---

### `remote.<name>.refmap` series advances

**What happened**: Harald Nordgren’s v10 series introducing `remote.<name>.refmap` for "matching-inspired" fetch mode was queued to `next`. The series improves fetch performance for shallow repositories by dynamically fetching only branches tracked by local branches. A bugfix addressed a fallback logic issue in `remote_find_tracking()`, resolving a potential edge case where the fallback could return unintended refspecs.

**Who was involved**: Harald Nordgren (author), Junio C Hamano (maintainer), D. Ben Knoble (reviewer).

**Current status**: Queued to `next`. The series introduces a new configuration mechanism (`remote.<name>.refmap`) and a `--limited-fetch` flag for `git remote add`, with shallow repositories defaulting to this mode.

---

### Test-lint performance optimization

**What happened**: Tyler Cipriani posted a four-patch series addressing the slowdown in `test-lint` caused by the `test-grep-lint` check. The series consolidates parsing effort between `chainlint.pl` and `greplint.pl`, reducing redundant file I/O and leveraging parallel execution. The implementation reduces `test-lint` runtime from ~25 seconds to ~1 second for five consecutive runs on a 16-core machine.

**Who was involved**: Tyler Cipriani (author), Junio C Hamano (maintainer), Jeff King (reviewer).

**Current status**: Under review. The series is uncontroversial and likely to land quickly if no edge cases are uncovered.

---

## In brief

**CRLF conversion size-trusting bug** -- The macOS-specific CRLF conversion series remains stalled on an architectural debate about check placement (`ie_modified()` vs. `ie_match_stat()`). Torsten Bögershausen’s proposal avoids racy Git issues but reintroduces performance overhead. Thread.

**`imap-send` OpenSSL compatibility** -- Beat Bolli’s v3 series faces **four blocker-level issues**: a security regression (certificate stripping), a behavioral regression (redundant `verify_peer()` call), process issues (lack of cover letter and range-diff), and OpenSSL 4.0 compatibility concerns. Thread.

**`git stash` shell completion** -- D. Ben Knoble’s five-patch series to improve `git stash` completion faces design questions about Zsh-specific fixes and the removal of deprecated `stash save` completion. Junio endorsed the second patch as a clear improvement. Thread.

**Index lock timeout** -- Jun Wu’s configurable timeout for index locks (`core.indexLockTimeout`) sparked debate about configurability vs. root-cause fixes. Mihai Cosma proposed an alternative approach deferring lock acquisition until write time, fixing a silent data-loss bug in the `index.skipHash` code path. Thread.

**`git worktree list --branch`** -- Jasper Mayone’s `--branch=<branch>` filtering patch for `git worktree list` received constructive feedback from Phillip Wood, who proposed a refactoring to expose a reusable `branch_find_checkout()` helper. Thread.

**`git remote set-password`** -- brian m. carlson rejected the proposal outright, citing project policy against encouraging URL-embedded credentials and advocating for credential helpers or SSH with public keys. Thread.

**`git init` absolute path output** -- Matt Hunter posted a v2 patch restoring the absolute path output in the "Initialized empty Git repository" message, incorporating all feedback and adding a regression test. Thread.

**Terminology cleanup** -- Andy Koppe’s five-patch series aligning Git’s documentation, code comments, and tests with the 2024 terminology change (narrowing "pseudoref" to FETCH_HEAD and MERGE_HEAD only) received feedback from Junio about the "pseudoref" definition. Thread.

**ODB transaction race condition** -- Justin Tobler’s v4 bugfix series targeting a race condition in Git’s ODB transaction machinery was approved by Patrick Steinhardt and is ready for integration. Thread.

**CI: Pin GitHub Actions to full commit IDs** -- Taylor Blau’s patch to pin GitHub Actions to full commit SHAs sparked a policy debate about whether downstream constraints should dictate upstream workflows. Junio conditionally accepted the change pending a revised commit message. Thread.

**`gitbreaking-changes(7)` manpage** -- Junio signaled tentative approval for the series, contingent on the post-transition awkwardness being manageable. The discussion resolved the document’s lifecycle after Git 3.0. Thread.

**`git stash` performance** -- Phillip Wood’s seven-patch refactoring of `git stash` to eliminate redundant index operations was queued for `next`. Thread.

**`git fsck` object count** -- Aniket Gupta posted a patch to correct a misleading progress message in `git fsck --verbose`. Thread.

**`git mktree --batch` tests** -- Roger Koranteng added basic test coverage for `git mktree --batch`, addressing a gap that allowed a memory leak to go unnoticed. Thread.

**`git init` tutorial** -- Julia Evans’s beginner-focused `gittutorial` replacement is one mechanical fix away from merging. Thread.

**Outreachy inquiries** -- Wasswa Edson and Tanga Joris sought clarification on project scoping for "Implement promisor remote fetch ordering" and "Improve how command arguments and options are scanned and parsed," respectively. Thread.

**`gitk` cherry-pick context menu** -- Johannes Sixt reviewed a patch adding a "Cherry-pick with -x" context-menu action, questioning the necessity of the feature and suggesting UI refinements. Thread.

**`git reflog` usability** -- Harald Nordgren’s series adding `--operations` and `--historical-branch` options to `git reflog` faced a major redesign proposal from Phillip Wood, who suggested replacing `--historical-branch` with a format specifier (`%gb`). Thread.

**Untracked cache performance** -- Tamir Duberstein’s v3 series teaching `git ls-files` to reuse the untracked cache populated by `git status` is under active review, with no objections recorded yet. Thread.

**Shallow fetch regression** -- Karthik Nayak and Patrick Steinhardt discussed a blocker-level memory management issue in the v4/v5 patch restoring dynamic reloading of alternate object databases. Thread.

**`git apply --check` crash** -- Tanga Joris confirmed the root cause of a crash in `git apply --check` when processing a patch that creates a new file. Thread.

**`diff-delta.c` safe integer helpers** -- Srujith A. Poojary’s patch replacing manual integer arithmetic with safe helpers received feedback from Junio about the commit message justification. Thread.

**`git reflog expire` dry-run output** -- Laveesh Bansal submitted a bugfix for a UI inconsistency in `git reflog expire --dry-run --verbose`, ensuring consistent output wording during dry runs. Thread.

**Completion regression fix** -- René Scharfe noted an edge case in the completion behavior for `git fast-import`, where `--git-completion-helper` only prints options after receiving input. Thread.

**`git rebase` man page update** -- Omri Sarig posted v4 of a patch updating the `git-rebase` man page to correct outdated references in the "INCOMPATIBLE OPTIONS" section. Thread.

**`the_repository` removal** -- Dorine Tipo posted a patch mechanically refactoring `partial_clone_register()` and `partial_clone_get_default_filter_spec()` to accept a `struct repository *` parameter. Thread.

**`git remote set-password`** -- Colin Hinton’s patch to defer validation of `remote.<name>.followRemoteHEAD` faces design questions about error message clarity and syntax unification. Thread.

**ZIP archive pluralization** -- Keryle Fondji’s patch to use `Q_()` for plural-form error messages in the ZIP backend received feedback on commit message clarity. Thread.

**`git fsck` pluralization** -- Aniket Gupta’s patch to use `Q_()` for the verbose "Checking connectivity" message was approved and queued for integration. Thread.

**`git status` triangular workflows** -- Harald Nordgren’s patch to make `git status` compare against both `@{upstream}` and `@{push}` by default faces questions about test coverage and justification. Thread.

**Skipping negotiator correctness** -- Taylor Blau posted a patch to fix a correctness issue in the skipping negotiator’s queue management. Thread.

**`MyFirstContribution` tutorial** -- Junio C Hamano posted a patch to reorder the tutorial to emphasize review discussion before sending v2. Thread.

---

## Looking ahead

The next week is likely to see **continued governance discussions** about the voting process and AI contribution policy, with a potential vote on the horizon. The **SHA-256 interoperability effort** will advance as the pack index v3 series stabilizes, while the **worktree reflink series** may see a reroll addressing the Windows ReFS silent corruption risk. The **`git status` defaults debate** could reach a resolution, and the **`git branch --delete-merged` enhancements** may see a CLI design compromise. Finally, the **incremental connectivity check** and **pack index v3** features will undergo further testing in `next` as they prepare for graduation to `master`.