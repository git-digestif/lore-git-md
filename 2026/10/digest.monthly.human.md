# The Git Project -- Monthly Digest for October 2026

## The period in brief

October 2026 was an **exceptionally consequential month** for the Git project, marked by **high-stakes architectural decisions, performance breakthroughs, and governance debates**. The period saw **four major features land** (incremental connectivity checks, pack index v3, `git blame` default ignore-revs, and `gitmergeconflicts(7)`), **two critical bugfix series finalized** (`--force-if-includes` and `rerere` race condition), and **three governance discussions reach critical junctures** (SHA-256 transition, AI-assisted contributions, and community voting). Traffic was **heavy and technically dense**, with **performance optimizations, interoperability work, and usability refinements** dominating the agenda. Three developments stand out as must-know: the **incremental connectivity check** (190× speedup for small pushes), the **pack index v3 series** (enabling SHA-1/SHA-256 interoperability), and the **escalating governance debate** about AI contributions and voting processes.

---

## Key developments

### Incremental connectivity check lands
Kristofer Karlsson’s **six-patch series** introducing an opt-in incremental connectivity check for fetches and pushes was **merged to `next`** and later **promoted to `master`**. The feature reduces the cost of verifying object connectivity by shifting the cost from repository size to incoming change size, offering up to **190× speedup** for small pushes to large repositories. The implementation introduces new config keys (`transfer.connectivityCheck=full|incremental|auto[=<n>]`, `transfer.connectivityCheckIncrementalMaxCommits=<n>`) and a new internal `rev-list` flag (`--connectivity-check=`). The auto mode dynamically switches between incremental and full checks at a configurable 1,000-commit cutoff. **Key participants**: Kristofer Karlsson, Junio C Hamano, Patrick Steinhardt. **Status**: Merged to `master`.

### Pack index v3 enables SHA-1/SHA-256 interoperability
Christian Couder’s **13-patch v2 series** implementing pack index v3 was **merged to `next`** and later **promoted to `master`**. The new index format stores object IDs in both primary and compatibility hash algorithms, enabling bidirectional mapping between SHA-1 and SHA-256 repositories. The series includes strict validation logic to prevent corruption from malformed indices and is a **critical step toward full SHA-1/SHA-256 interoperability**. The implementation builds on parts 1 and 2 (already merged to `master`). **Key participants**: Christian Couder, Junio C Hamano. **Status**: Merged to `master`.

### Governance: Community voting process proposed
Emily Shaffer proposed a **formal voting process** to resolve contentious decisions (e.g., AI contribution policy) when consensus cannot be reached. The patch defines eligibility criteria (50 merged patches or multiple code reviews) and a two-year activity window, sparking debate about **inclusivity, corporate influence, and the role of voting in Git’s consensus-driven culture**. Junio C Hamano endorsed secret ballots to mitigate corporate influence, while Kristoffer Haugsbakk criticized the proposal as favoring corporate interests. The thread revealed **deep divisions about Git’s governance model** and the role of corporate contributors. **Key participants**: Emily Shaffer, Junio C Hamano, Kristoffer Haugsbakk, Patrick Steinhardt, brian m. carlson, Phillip Wood. **Status**: Under active discussion; no consensus reached.

### `--force-if-includes` bugfix series finalized
Tyler Cipriani’s **three-patch v7 series** fixing critical bugs in the `--force-if-includes` safety mechanism for `git push` was **merged to `master`**. The series addresses two long-standing issues: (1) the feature incorrectly checks the reflog of the *local* branch whose name matches the *remote* destination, causing both false rejections and data loss risks, and (2) it unnecessarily rejects fast-forward pushes of non-branch refs (tags, detached HEADs). The series introduces new test coverage for edge cases like differently named branches, HEAD-based pushes, orphan branches, and tags. **Key participants**: Tyler Cipriani, Junio C Hamano. **Status**: Merged to `master`.

### `git blame` default ignore-revs feature lands
Ravi Mistry’s **feature series** making `git blame` automatically use `.git-blame-ignore-revs` (or `HEAD:.git-blame-ignore-revs` in bare repositories) when no explicit ignore file is configured was **merged to `master`**. The series addresses a long-standing usability gap by making ignore-revs files **discoverable and automatic**, while hardening the parser against malformed input (NUL bytes, promisor fetches in partial clones, 100MB size cap). The feature aligns local behavior with hosting platforms like GitHub and GitLab. **Key participants**: Ravi Mistry, Junio C Hamano. **Status**: Merged to `master`.

### `rerere` race condition fix finalized
Thomas Bachem’s **v8 series** fixing a race condition in Git’s `rerere` subsystem was **merged to `master`**. The series introduces a configurable timeout (`rerere.lockTimeout`, defaulting to 1000 ms) for the `MERGE_RR.lock` file, allowing processes to wait briefly for the lock instead of failing immediately. The series also adds a hidden `--skip-locked` flag to `git rerere gc`, enabling auto-maintenance tasks to skip silently if the lock is held. **Key participants**: Thomas Bachem, Junio C Hamano, Patrick Steinhardt. **Status**: Merged to `master`.

### `git log --follow` performance regression fixed
Miklos Vajna’s **four-patch series** fixing a severe performance regression in `git log --follow` when tracking renames across merges was **merged to `master`**. The regression, introduced in June 2026, made `--follow` unusably slow (6m43s vs. 28s on the Linux kernel repository). The fix splits diffing into two passes: a fast pass without rename detection to identify unchanged parents, and a fallback to expensive rename detection only when necessary. The implementation avoids the expensive `propagate_follow_pathspec_to_parent()` function and reuses diffs already computed in `log_tree_diff()`’s parent loop. **Key participants**: Miklos Vajna, Junio C Hamano. **Status**: Merged to `master`.

### Worktree reflink: Windows ReFS silent corruption risk
brian m. carlson raised **critical correctness concerns** about the proposed `FSCTL_DUPLICATE_EXTENTS_TO_FILE` implementation for Windows ReFS in Jason Newton’s worktree reflink series. The underlying bug (Git LFS #6312) can cause **silent corruption** when cloning extents, making the feature unsafe for production use until Microsoft provides a fix. The Windows reflink implementation is **permanently disabled** due to the silent corruption risk, while the macOS and Linux implementations remain unaffected. The series faces **substantive objections** on both the reflink design (opt-in vs. automatic) and the ephemeral branch reaping feature. **Key participants**: brian m. carlson, Jason Newton, Eric Sunshine, Phillip Wood. **Status**: Blocked; awaiting Microsoft fix.

### `git status` defaults for triangular workflows
Harald Nordgren’s patch to make `git status` compare against both `@{upstream}` and `@{push}` by default sparked a **philosophical debate** about Git’s defaults for triangular workflows. Junio C Hamano argued that `@{push}` divergence is less critical to monitor by default because it reflects the user’s own work, while D. Ben Knoble provided anecdotal evidence of workflows where monitoring `@{push}` divergence is valuable. The discussion centers on whether the benefits for triangular workflow users outweigh the potential noise for others. **Key participants**: Harald Nordgren, Junio C Hamano, D. Ben Knoble. **Status**: Under review (v3); no consensus reached.

### `git branch --delete-merged` enhancements
Harald Nordgren’s **v3 series** extends `git branch --delete-merged` to detect branches whose changes (but not their tips) have been integrated via squash or rebase merges. Junio C Hamano proposed a **fundamental CLI redesign**—decoupling strict `--delete-merged` (tip-reachability only) from a new `--delete-irrelevant` (change-based detection)—while Harald rejects the split. The first patch (refactoring) is uncontroversial and ready for integration, but the series remains **blocked on the CLI design debate**. **Key participants**: Harald Nordgren, Junio C Hamano, Jasper Mayone. **Status**: Blocked; awaiting design resolution.

### `gitmergeconflicts(7)` documentation series finalized
Junio C Hamano queued a **six-patch series** introducing `gitmergeconflicts(7)`, a new man page consolidating scattered merge-conflict advice across multiple man pages. The guide covers conflict markers, "ours/theirs" terminology, `diff3` vs. default formats, resolution tools, and an exhaustive list of conflict-causing commands. The series touches 10 files, including the new `Documentation/gitmergeconflicts.adoc` (339 lines) and updates to `git-merge.adoc`, `git-rebase.adoc`, and others. **Key participants**: Junio C Hamano, Phillip Wood. **Status**: Merged to `master`.

### `remote.<name>.refmap` series advances
Harald Nordgren’s **v10 series** introducing `remote.<name>.refmap` for "matching-inspired" fetch mode was **merged to `next`**. The series improves fetch performance for shallow repositories by dynamically fetching only branches tracked by local branches. A bugfix addressed a fallback logic issue in `remote_find_tracking()`, resolving a potential edge case where the fallback could return unintended refspecs. The series introduces a new configuration mechanism (`remote.<name>.refmap`) and a `--limited-fetch` flag for `git remote add`, with shallow repositories defaulting to this mode. **Key participants**: Harald Nordgren, Junio C Hamano, D. Ben Knoble. **Status**: Merged to `next`.

---

## In brief

**Git v2.56.0 released** -- Junio C Hamano announced the release of Git v2.56.0, featuring **748 non-merge commits** from 104 contributors. Key highlights include **performance optimizations** (ref handling, pack-objects, index scanning), **new commands** (`git history drop`, `git refs`, `git replay --linearize`), and **architectural shifts** (ODB abstraction, `the_repository` removal, Rust support enabled by default). The release also includes **security hardening** and **UX improvements**. **Status**: Released.

**Rust-based SHA-1 backend (3× performance gain)** -- Johannes Schindelin introduced a **four-patch series** adding an optional Rust-based SHA-1 backend (`sha1dc` crate) that delivers **~3× speedups** for `git index-pack` on Windows, WSL, and macOS. The series includes **FFI integration**, a **runtime escape hatch** (`core.sha1dcBackend=c`), **thread-safety shims**, and **thread-safe initialization**. The Rust backend is optional and not enabled by default. **Status**: Under review; awaiting policy resolution on Rust version requirement (1.87).

**`git stash` autostash bugfix (NULL-dereference risk)** -- D. Ben Knoble’s **five-patch series** fixing a race condition in `git stash` autostash with staged index entries was **ejected from `seen`** due to a **NULL-dereference risk** in the in-core merge logic. The series replaces subprocess-based merging with `merge-ort.c` to preserve staged entries but introduces a **SIGSEGV risk** (uninitialized `result.tree` on catastrophic merge failure). **Status**: Blocked; awaiting fix for NULL-dereference risk.

**`uploadpack.lazyFetchTrusted` (v5)** -- Christian Couder posted v5 of the `uploadpack.lazyFetchTrusted` series, addressing all feedback from Junio’s v4 review. The series replaces the earlier client-side `GIT_NO_LAZY_FETCH=fromAccepted` proposal with a **server-side protected configuration variable**, providing a mechanism for operators to explicitly declare which repositories are safe to lazy-fetch from. **Status**: Ready for maintainer review.

**`gitmergeconflicts(7)`: new man page for merge conflict resolution** -- Julia Evans’ **`gitmergeconflicts(7)`** series saw significant progress, with new sections and design discussions. The man page consolidates scattered merge conflict advice into a dedicated guide. Key additions include a **"WHAT IS A MERGE CONFLICT?"** section, improved `diff3` documentation, and clarification of terminology. **Status**: Merged to `master`.

**MIDX reachability closure fixes** -- Taylor Blau’s **eight-patch v2 series** fixing MIDX reachability closure corner cases received substantive review feedback from Jeff King and Elijah Newren. The series addresses scenarios where MIDXs can end up containing objects not closed under reachability, violating the MIDX’s invariant. **Status**: Under review; awaiting resolution of `oidset` vs. `oid_array` debate.

**CI improvements: leak sanitizer and job parallelism** -- Two CI-related series addressed resource exhaustion in GitHub Actions: (1) Harald Nordgren’s v4 series improves leak sanitizer failure reporting, and (2) Tamir Duberstein’s v3 series caps job parallelism to twice the CPU count. **Status**: Leak sanitizer series merged; parallelism series under review.

**`git history` signing series** -- Souma’s series encountered integration issues with Patrick Monette’s `git replay` signing series, requiring a rebase. The `git replay` series is blocked until the `git history` work lands. **Status**: Blocked; awaiting rebase.

**Reftable reflog timezone encoding fix** -- Josh McKinney reported a **discrepancy** in Git’s reftable backend: reflog timezone offsets are written as **signed HHMM integers**, but the **reftable specification requires signed minutes**. Patrick Steinhardt posted a three-patch series fixing the issue. **Status**: Merged to `master`.

**`gitbreaking-changes(7)`** -- Julia Evans proposed converting Git’s `BreakingChanges` document into a manpage, improving discoverability via `git help` and `man`. **Status**: Under review; awaiting resolution of post-Git 3.0 lifecycle.

**`git branch --delete-merged` squash-merge detection** -- Harald Nordgren posted a patch extending `git branch --delete-merged` to recognize branches whose changes have been squash-merged or rebase-merged into an upstream branch. **Status**: Under review (v2); awaiting design resolution.

**`git stash` shared stashes** -- Hanan Arshad proposed a **new porcelain workflow** for sharing stashes through remotes, leveraging `git stash export` and `git stash import`. **Status**: RFC; awaiting feedback on remote ref namespace (`refs/stashes/`).

**`includeIf "hostname:..."`** -- Isabella Caselli proposed adding a **hostname-based `includeIf` condition** for Git configuration, enabling multi-machine dotfile sharing. **Status**: Under review (v2).

**`git(1)` man page rewrite** -- Julia Evans rewrote the `git(1)` intro to orient users toward the help system. **Status**: Merged to `master`.

**`gittutorial-2` removal** -- Julia Evans proposed a **three-patch series** to remove the obsolete `gittutorial-2` document. **Status**: Merged to `master`.

**Git for Windows 2.56.0** -- Johannes Schindelin announced the release, dropping Windows 8.1 support and fixing platform-specific bugs. **Status**: Released.

**`git -C` alias bug** -- A user reported that `git -C other_repo` via alias in a worktree **mis-sets `GIT_DIR` and `GIT_COMMON_DIR`**, causing commands to operate on the wrong repository. **Status**: Under investigation.

**`git restore-mtimes`** -- Artem S. Tashkinov proposed a new command to preserve per-file historical modification timestamps derived from Git commit history. **Status**: RFC; awaiting feedback.

**`git config --list --show-origin`** -- Christophe Lohr reported that the command displays `.git/config` as a relative path, creating confusion about the `.git` directory’s location. **Status**: Under investigation.

**`git submodule` root-relative URLs** -- Stanislav Aleksandrov posted an RFC patch introducing `^/` syntax for submodule URLs, allowing them to be resolved relative to the superproject’s remote server root. **Status**: RFC; awaiting feedback.

**`git remote` documentation** -- Junio C Hamano suggested a **broader approach** to clarify that all `git remote` commands operate locally. **Status**: Under discussion.

**`git stash create` options** -- Phillip Wood identified backward-compatibility regressions in Kazumasa Shigeta’s patch extending `git stash create` to support `--include-untracked` and `--all`. **Status**: Under review; awaiting resolution.

**`git reflog expire` regression** -- Patrick Steinhardt withdrew a fast-track request for a fix after confirming the regression’s **low real-world impact**. **Status**: Deferred.

**`git backfill --dry-run`** -- Derrick Stolee objected to the feature on conceptual grounds, arguing the uncompressed size estimate may not be useful. **Status**: Rejected.

**`checkout -m` conflict labels** -- Johannes Sixt suggested using an index extension instead of `.git/MERGE_LABELS` to store conflict labels. **Status**: Under discussion.

**`refs/packed-backend` optimization** -- Karthik Nayak posted a performance optimization patch for the packed-refs backend. **Status**: Merged to `master`.

**`revision: add @{p} shorthand`** -- Junio C Hamano queued the patch for integration. **Status**: Merged to `master`.

**`git filter-branch` bugfix** -- Michele Locati provided a new test case for the `--state-branch` commit mapping inversion regression. **Status**: Merged to `master`.

**`parse-options` API refactoring** -- Kaartic Sivaraam identified a behavioral inconsistency with `parse_options()` where the API’s decision to ignore abbreviations could misinterpret arguments. **Status**: Under review.

**Windows memory regression with fsmonitor** -- Pierre Bruno reported unexpectedly high memory consumption (around 1 GB per process) when running `git status`-like commands in Git 2.56.0.windows.1 with fsmonitor enabled. **Status**: Under investigation.

**`git repack --dry-run --drop-filtered` regression** -- Coy Geek reported a critical regression where the command incorrectly mutates repository storage despite its documented promise to only list candidate objects. **Status**: Under investigation.

**`remote.<name>.refmap` design ambiguity** -- Junio C Hamano identified a glitch and a design ambiguity in Harald Nordgren’s v6 series introducing `remote.<name>.refmap`. **Status**: Resolved in v10.

**Zsh completion regression risk** -- SZEDER Gábor identified a regression risk in a Zsh completion patch. **Status**: Fixed in subsequent revision.

**`post-worktree` hooks** -- Domen Kožar posted v3 of the unified `post-worktree` hook, consolidating three separate hooks into one subcommand-style interface. **Status**: Under review.

**CRLF conversion size-trusting bug** -- The macOS-specific CRLF conversion series remains stalled on an architectural debate about check placement (`ie_modified()` vs. `ie_match_stat()`). **Status**: Under discussion.

**`imap-send` OpenSSL compatibility** -- Beat Bolli’s v3 series faces **four blocker-level issues**: a security regression (certificate stripping), a behavioral regression (redundant `verify_peer()` call), process issues (lack of cover letter and range-diff), and OpenSSL 4.0 compatibility concerns. **Status**: Blocked; awaiting fixes.

**`git stash` shell completion** -- D. Ben Knoble’s five-patch series to improve `git stash` completion faces design questions about Zsh-specific fixes and the removal of deprecated `stash save` completion. **Status**: Under review.

**Index lock timeout** -- Jun Wu’s configurable timeout for index locks (`core.indexLockTimeout`) sparked debate about configurability vs. root-cause fixes. **Status**: Under review.

**`git worktree list --branch`** -- Jasper Mayone’s `--branch=<branch>` filtering patch for `git worktree list` received constructive feedback. **Status**: Under review.

**`git remote set-password`** -- brian m. carlson rejected the proposal outright, citing project policy against encouraging URL-embedded credentials. **Status**: Rejected.

**`git init` absolute path output** -- Matt Hunter posted a v2 patch restoring the absolute path output in the "Initialized empty Git repository" message. **Status**: Merged to `master`.

**Terminology cleanup** -- Andy Koppe’s five-patch series aligning Git’s documentation, code comments, and tests with the 2024 terminology change received feedback. **Status**: Under review.

**ODB transaction race condition** -- Justin Tobler’s v4 bugfix series targeting a race condition in Git’s ODB transaction machinery was approved and merged. **Status**: Merged to `master`.

**CI: Pin GitHub Actions to full commit IDs** -- Taylor Blau’s patch to pin GitHub Actions to full commit SHAs sparked a policy debate. **Status**: Merged to `master`.

**`git fsck` object count** -- Aniket Gupta posted a patch to correct a misleading progress message in `git fsck --verbose`. **Status**: Merged to `master`.

**`git mktree --batch` tests** -- Roger Koranteng added basic test coverage for `git mktree --batch`. **Status**: Merged to `master`.

**`git init` tutorial** -- Julia Evans’s beginner-focused `gittutorial` replacement is one mechanical fix away from merging. **Status**: Under review.

**`gitk` cherry-pick context menu** -- Johannes Sixt reviewed a patch adding a "Cherry-pick with -x" context-menu action. **Status**: Under review.

**`git reflog` usability** -- Harald Nordgren’s series adding `--operations` and `--historical-branch` options to `git reflog` faced a major redesign proposal. **Status**: Under review.

**Untracked cache performance** -- Tamir Duberstein’s v3 series teaching `git ls-files` to reuse the untracked cache populated by `git status` is under active review. **Status**: Under review.

**Shallow fetch regression** -- Karthik Nayak and Patrick Steinhardt discussed a blocker-level memory management issue in the v4/v5 patch restoring dynamic reloading of alternate object databases. **Status**: Under investigation.

**`git apply --check` crash** -- Tanga Joris confirmed the root cause of a crash in `git apply --check` when processing a patch that creates a new file. **Status**: Under investigation.

**`diff-delta.c` safe integer helpers** -- Srujith A. Poojary’s patch replacing manual integer arithmetic with safe helpers received feedback. **Status**: Under review.

**`git reflog expire` dry-run output** -- Laveesh Bansal submitted a bugfix for a UI inconsistency in `git reflog expire --dry-run --verbose`. **Status**: Merged to `master`.

**Completion regression fix** -- René Scharfe noted an edge case in the completion behavior for `git fast-import`. **Status**: Under investigation.

**`git rebase` man page update** -- Omri Sarig posted v4 of a patch updating the `git-rebase` man page. **Status**: Merged to `master`.

**`the_repository` removal** -- Dorine Tipo posted a patch mechanically refactoring `partial_clone_register()` and `partial_clone_get_default_filter_spec()` to accept a `struct repository *` parameter. **Status**: Merged to `master`.

**`git remote set-password`** -- Colin Hinton’s patch to defer validation of `remote.<name>.followRemoteHEAD` faces design questions. **Status**: Under review.

**ZIP archive pluralization** -- Keryle Fondji’s patch to use `Q_()` for plural-form error messages in the ZIP backend received feedback. **Status**: Merged to `master`.

---

## Looking ahead

The next month is likely to see **continued governance discussions** about the voting process and AI contribution policy, with a potential vote on the horizon. The **SHA-256 interoperability effort** will advance as the pack index v3 series stabilizes, while the **worktree reflink series** may see a reroll addressing the Windows ReFS silent corruption risk. The **`git status` defaults debate** could reach a resolution, and the **`git branch --delete-merged` enhancements** may see a CLI design compromise. The **incremental connectivity check** and **pack index v3** features will undergo further testing as they prepare for broader adoption. Additionally, the **Rust-based SHA-1 backend** may face policy-level objections over the Rust version requirement (1.87), and the **AI-assisted contributions debate** could evolve into a concrete patch or a decision to maintain the status quo.