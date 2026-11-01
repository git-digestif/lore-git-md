# The Git Project -- Monthly Digest for October 2026

## The period in brief

October 2026 was an **exceptionally consequential month** for the Git project, marked by **high-stakes architectural decisions, performance breakthroughs, and governance debates**. The period saw **4,200+ emails** across **22 active days**, with traffic **heavier and more technically dense** than any month since the Git 3.0 planning phase began. Three developments stand out: the **landing of incremental connectivity checks** (190× speedup for small pushes), the **pack index v3 series enabling SHA-1/SHA-256 interoperability**, and the **escalating governance debate** about AI contributions and community voting processes. Other major themes included **SHA-256 transition strategy**, **Windows ReFS silent corruption risks**, and **usability refinements** for `git blame`, `git status`, and `git branch --delete-merged`.

---

## Key developments

### Incremental connectivity check lands in `next`

Kristofer Karlsson’s **six-patch series** introducing an opt-in incremental connectivity check for fetches and pushes was **merged to `next`** on 2026/10/23. The feature reduces the cost of verifying object connectivity by shifting the cost from repository size to incoming change size, offering up to **190× speedup** for small pushes to large repositories. The implementation introduces new config keys (`transfer.connectivityCheck=full|incremental|auto[=<n>]`, `transfer.connectivityCheckIncrementalMaxCommits=<n>`) and a new internal `rev-list` flag (`--connectivity-check=`). The auto mode dynamically switches between incremental and full checks at a configurable 1,000-commit cutoff.

The series addresses a long-standing scalability bottleneck in CI/CD pipelines and large monorepos, where full connectivity checks could take minutes for even trivial pushes. **Key participants**: Kristofer Karlsson (author), Junio C Hamano (maintainer), Patrick Steinhardt (reviewer). **Status**: Merged to `next`; expected to graduate to `master` in November.

---

### Pack index v3 enables SHA-1/SHA-256 interoperability

Christian Couder’s **13-patch v2 series** implementing pack index v3 was **merged to `next`** on 2026/10/23. The new index format stores object IDs in both primary and compatibility hash algorithms, enabling bidirectional mapping between SHA-1 and SHA-256 repositories. The series includes strict validation logic to prevent corruption from malformed indices and is a **critical step toward full SHA-1/SHA-256 interoperability**. The implementation builds on parts 1 and 2 (already merged to `master`).

This work is foundational for Git 3.0’s SHA-256 transition, allowing repositories to migrate incrementally while maintaining compatibility with existing tooling. **Key participants**: Christian Couder (author), Junio C Hamano (maintainer). **Status**: Merged to `next`; expected to graduate to `master` in November.

---

### Governance: Community voting process proposed

Emily Shaffer’s **proposal for a formal voting process** to resolve contentious decisions (e.g., AI contribution policy) sparked a **multi-week debate** about Git’s governance model. The patch defines eligibility criteria (50 merged patches or multiple code reviews) and a two-year activity window, but the discussion revealed deep divisions about **inclusivity, corporate influence, and the role of voting in Git’s consensus-driven culture**. Junio C Hamano endorsed secret ballots to mitigate corporate influence, while Kristoffer Haugsbakk criticized the proposal as favoring corporate interests.

The thread highlighted tensions between Git’s traditional meritocratic model and the growing influence of corporate contributors. **Key participants**: Emily Shaffer (author), Junio C Hamano (maintainer), Kristoffer Haugsbakk, Patrick Steinhardt, brian m. carlson, Phillip Wood. **Status**: Under active discussion; no consensus reached. A vote on the proposal is possible in November.

---

### Worktree reflink: Windows ReFS silent corruption risk

brian m. carlson raised **critical correctness concerns** about the proposed `FSCTL_DUPLICATE_EXTENTS_TO_FILE` implementation for Windows ReFS in Jason Newton’s worktree reflink series. The underlying bug (Git LFS #6312) can cause **silent corruption** when cloning extents, making the feature unsafe for production use until Microsoft provides a fix. The macOS and Linux implementations remain unaffected, but the Windows implementation is **permanently disabled** due to the risk.

The series faces **substantive objections** on both the reflink design (opt-in vs. automatic) and the ephemeral branch reaping feature. **Key participants**: brian m. carlson (reviewer), Jason Newton (author), Eric Sunshine, Phillip Wood. **Status**: Blocked; awaiting Microsoft fix or design revisions.

---

### `git log --follow` performance regression fixed

Miklos Vajna posted a **four-patch series** fixing a severe performance regression in `git log --follow` when tracking renames across merges. The regression, introduced in June 2026, made `--follow` unusably slow (6m43s vs. 28s on the Linux kernel repository). The fix splits diffing into two passes: a fast pass without rename detection to identify unchanged parents, and a fallback to expensive rename detection only when necessary. The series includes comprehensive test coverage and benchmarks showing a speedup to 36s.

The implementation avoids the expensive `propagate_follow_pathspec_to_parent()` function and reuses diffs already computed in `log_tree_diff()`’s parent loop. **Key participants**: Miklos Vajna (author), Junio C Hamano (maintainer). **Status**: Under review; expected to land in November.

---

### SSH signing verification security fixes

Christian Noé Ramos López posted a **v2 security bugfix** for Git’s SSH signing verification, addressing two critical flaws: (1) a misleading hint message that included full ref names instead of short names, and (2) a revocation file that failed open when missing or unreadable. The patch ensures signature information is displayed consistently and makes revocation file errors fatal by default. The patch uses Git’s `:(optional)` pathname syntax to provide an escape hatch for revocation file errors.

**Key participants**: Christian Noé Ramos López (author), Junio C Hamano (maintainer), Patrick Steinhardt (reviewer). **Status**: Under review; expected to land in November.

---

### `git status` defaults for triangular workflows

Harald Nordgren’s patch to make `git status` compare against both `@{upstream}` and `@{push}` by default sparked a **philosophical debate** about Git’s defaults for triangular workflows. Junio C Hamano argued that `@{push}` divergence is less critical to monitor by default because it reflects the user’s own work, while D. Ben Knoble provided anecdotal evidence of workflows where monitoring `@{push}` divergence is valuable. The discussion centers on whether the benefits for triangular workflow users outweigh the potential noise for others.

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), D. Ben Knoble (reviewer). **Status**: Under review (v3); no consensus reached. The debate remains at an impasse.

---

### `git branch --delete-merged` enhancements

Harald Nordgren’s **v3 series** extends `git branch --delete-merged` to detect branches whose changes (but not their tips) have been integrated via squash or rebase merges. Junio C Hamano proposed a fundamental CLI redesign—decoupling strict `--delete-merged` (tip-reachability only) from a new `--delete-irrelevant` (change-based detection)—while Harald rejects the split. The first patch (refactoring) is uncontroversial and ready for integration, but the design debate remains unresolved.

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), Jasper Mayone (reviewer). **Status**: Blocked on CLI design debate. A compromise may emerge in November.

---

### `gitmergeconflicts(7)` documentation series finalized

Junio C Hamano queued a **six-patch series** introducing `gitmergeconflicts(7)`, a new man page consolidating scattered merge-conflict advice across multiple man pages. The guide covers conflict markers, "ours/theirs" terminology, `diff3` vs. default formats, resolution tools, and an exhaustive list of conflict-causing commands. The series touches 10 files, including the new `Documentation/gitmergeconflicts.adoc` (339 lines) and updates to `git-merge.adoc`, `git-rebase.adoc`, and others.

**Key participants**: Junio C Hamano (maintainer), Phillip Wood (reviewer). **Status**: Merged to `next` (2026/10/23); expected to graduate to `master` in November.

---

### `--force-if-includes` bugfix series finalized

Tyler Cipriani’s **v7 of a three-patch series** fixing critical bugs in the `--force-if-includes` safety mechanism for `git push` was **ready for merge** by the end of October. The series addresses two long-standing issues: (1) the feature incorrectly checks the reflog of the *local* branch whose name matches the *remote* destination, causing both false rejections and data loss risks, and (2) it unnecessarily rejects fast-forward pushes of non-branch refs (tags, detached HEADs). The v7 series is technically complete, with all prior feedback resolved, and introduces new test coverage for edge cases.

**Key participants**: Tyler Cipriani (author), Junio C Hamano (maintainer). **Status**: Ready for merge; expected to land in `master` in November.

---

### Git 3.0 SHA-256 transition: Strategic debate escalates

Scott Chacon **escalated the debate** over Git 3.0’s SHA-256 transition plan, arguing it is a **GitHub-driven effort lacking broad community buy-in** and proposing his tree-digest signing mechanism as a less disruptive alternative. The discussion revealed deep divisions: **brian m. carlson** and **Patrick Steinhardt** countered that the transition is **necessary for regulatory compliance and ecosystem momentum**, while Chacon framed it as **corporate prioritization** that risks alienating the broader community. The debate now centers on **project governance**: whether the Git 3.0 timeline reflects genuine community consensus or corporate priorities.

Chacon’s tree-digest proposal—embedding SHA-256 digests of tree contents in signed commits/tags—remains a live alternative, though **no maintainer has endorsed it**. **Key participants**: Scott Chacon, brian m. carlson, Patrick Steinhardt, Junio C Hamano. **Status**: Under active discussion; no resolution in sight. The debate may dominate November’s governance discussions.

---

### AI-assisted contributions: Policy debate

Scott Chacon’s proposal to amend `Documentation/SubmittingPatches` to explicitly permit AI-assisted contributions sparked a **multi-week debate** about enforceability, legal risks, and reviewer bandwidth. The discussion revealed **three distinct positions**:
- **Chacon**: Proposes an `Assisted-by:` trailer for substantial AI assistance, requiring contributors to review, test, and sign off on submissions.
- **brian m. carlson**: Opposes the change on **legal and ethical grounds**, citing risks of license violations and copyright infringement from AI-generated code.
- **Junio C Hamano** and **Patrick Steinhardt**: Question **enforceability and reviewer bandwidth**, noting that the current policy ("we may reject anything that looks AI-generated") is more direct and easier to apply.

**Maciej Ciemborowicz’s disclosure** of a seven-step AI-assisted workflow further underscored the moral and practical dilemmas facing contributors. **Key participants**: Scott Chacon, brian m. carlson, Junio C Hamano, Patrick Steinhardt, Maciej Ciemborowicz. **Status**: No consensus reached; the debate may culminate in a vote under Emily Shaffer’s proposed governance process.

---

## In brief

**`git blame` default ignore-revs file** -- Ravi Mistry’s series teaching `git blame` to automatically use `HEAD:.git-blame-ignore-revs` as a default ignore-revs file was **merged to `next`**. The feature aligns local behavior with hosting platforms like GitHub and GitLab. **Status**: Merged to `next`; expected to graduate to `master` in November.

**`rerere` race condition fix** -- Thomas Bachem’s v8 series fixing a race condition in Git’s `rerere` subsystem was **queued for integration**. The series introduces a configurable timeout (`rerere.lockTimeout`) and a `--skip-locked` mode for `git rerere gc`. **Status**: Queued in `next`.

**`git history` signing** -- Souma’s v5 series teaching `git history` to sign rewritten commits (`drop`, `fixup`, `reword`, `split`, `squash`) using GPG **resolved its last blocking question** and is now **cooking in `seen`**. **Status**: Ready to graduate to `next`.

**`uploadpack.lazyFetchTrusted`** -- Christian Couder’s v5 series introducing a server-side protected configuration variable for lazy fetches in partial clones is **ready for maintainer review**. The series addresses security concerns about untrusted repositories triggering arbitrary code execution. **Status**: Under review.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll’s patch adding revision-based filtering to `git repo structure` was **marked ready for `next`** after addressing SHA-256 compatibility issues. **Status**: Ready for `next`.

**`git stash` autostash bugfix** -- D. Ben Knoble’s series fixing a race condition in `git stash` autostash with staged index entries was **ejected from `seen`** due to a **NULL-dereference risk**. The author confirmed the issue and will address it in a reroll. **Status**: Blocked; awaiting fix.

**CI improvements** -- Harald Nordgren’s v6 series improving leak sanitizer failure reporting and Tamir Duberstein’s v3 series capping job parallelism to twice the CPU count **landed in `next`**. The changes reduce CI noise and improve debuggability. **Status**: Merged to `next`.

**`git worktree repair` bugfix** -- Yoichi NAKAYAMA’s v3 series fixing incorrect repairs of unrelated worktrees **hardened the `get_worktree_id()` helper** to handle non-canonical path separators. **Status**: Under review.

**`git imap-send` OpenSSL compatibility** -- Beat Bolli’s v3 series addressing OpenSSL compatibility and correctness **faces four blocker-level issues**, including a **security regression** (certificate stripping). **Status**: Blocked; awaiting fixes.

**`git remote set-password`** -- brian m. carlson **rejected the proposal outright**, citing project policy against encouraging URL-embedded credentials. **Status**: Rejected.

**`git init` absolute path output** -- Matt Hunter’s v2 patch restoring the absolute path output in the "Initialized empty Git repository" message **incorporated all feedback** and added a regression test. **Status**: Under review.

**Terminology cleanup** -- Andy Koppe’s five-patch series aligning Git’s documentation with the 2024 terminology change (narrowing "pseudoref" to FETCH_HEAD and MERGE_HEAD only) **received feedback** on the "pseudoref" definition. **Status**: Under review.

**ODB transaction race condition** -- Justin Tobler’s v4 bugfix series targeting a race condition in Git’s ODB transaction machinery was **approved by Patrick Steinhardt** and is ready for integration. **Status**: Ready for `next`.

**`git stash` shell completion** -- D. Ben Knoble’s five-patch series improving `git stash` shell completion **faces design questions** about Zsh-specific fixes and the removal of deprecated `stash save` completion. **Status**: Under review.

**`git push` refspec matching optimization** -- Jon Simons’s 12-patch v2 series replacing linear refspec traversals with `strmap` lookups **achieved 10–100× speedups** for large-scale branch deletions. **Status**: Under review.

**`git detached-head(7)` documentation** -- Julia Evans’s rewrite of Git’s explanation of detached HEAD state was **merged to `master`**. **Status**: Merged.

**`git reflog` usability improvements** -- Harald Nordgren’s v2 series adding `--operations` and `--historical-branch` options to `git reflog` **faces a major redesign proposal** from Phillip Wood. **Status**: Under review.

**Untracked cache performance** -- Tamir Duberstein’s v3 series teaching `git ls-files` to reuse the untracked cache populated by `git status` is **under active review**. **Status**: Under review.

**Shallow fetch regression** -- Karthik Nayak and Patrick Steinhardt discussed a **blocker-level memory management issue** in the v4/v5 patch restoring dynamic reloading of alternate object databases. **Status**: Blocked; awaiting fix.

**`git apply --check` crash** -- Tanga Joris confirmed the root cause of a crash in `git apply --check` when processing a patch that creates a new file. **Status**: Under review.

**`diff-delta.c` safe integer helpers** -- Srujith A. Poojary’s patch replacing manual integer arithmetic with safe helpers **received feedback** on the commit message justification. **Status**: Under review.

**`git reflog expire` dry-run output** -- Laveesh Bansal submitted a bugfix for a UI inconsistency in `git reflog expire --dry-run --verbose`. **Status**: Under review.

**`git rebase` man page update** -- Omri Sarig posted v4 of a patch updating the `git-rebase` man page to correct outdated references. **Status**: Under review.

**`the_repository` removal** -- Dorine Tipo and Okhuomon Ajayi posted patches mechanically refactoring functions to accept explicit `struct repository *` parameters. **Status**: Under review.

**ZIP archive pluralization** -- Keryle Fondji’s patch to use `Q_()` for plural-form error messages in the ZIP backend **received feedback** on commit message clarity. **Status**: Under review.

**`git fsck` pluralization** -- Aniket Gupta’s patch to use `Q_()` for the verbose "Checking connectivity" message was **approved and queued for integration**. **Status**: Merged to `next`.

**Test-lint performance optimization** -- Tyler Cipriani’s four-patch series addressing the slowdown in `test-lint` **reduced runtime from ~25 seconds to ~1 second** for five consecutive runs. **Status**: Under review.

**`MyFirstContribution` tutorial** -- Junio C Hamano posted a patch to reorder the tutorial to emphasize review discussion before sending v2. **Status**: Under review.

---

## Looking ahead

November is likely to be dominated by **three major themes**:

1. **Governance**: The community voting process and AI contribution policy will likely culminate in a vote, with potential implications for Git’s long-term governance model.
2. **SHA-256 transition**: The pack index v3 series will stabilize, and the strategic debate about Git 3.0’s transition plan may reach a resolution.
3. **Performance and interoperability**: The incremental connectivity check and pack index v3 features will graduate to `master`, while the `git log --follow` performance regression fix and `--force-if-includes` bugfix series are expected to land.

Other topics to watch include the **worktree reflink series** (if the Windows ReFS silent corruption risk is addressed), the **`git status` defaults debate**, and the **`git branch --delete-merged` enhancements**. The month may also see progress on **CRLF conversion handling**, **index lock timeout**, and **SSH signing verification security fixes**.