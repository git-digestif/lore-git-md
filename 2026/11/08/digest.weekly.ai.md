# The Git Project -- Weekly Digest for 2026/11/02 -- 2026/11/08

## The period in brief

The week of 2026/11/02--2026/11/08 saw **heavy traffic** (six active days) with a **mixed tone**: routine patch flow was punctuated by **critical regressions**, **stalled series warnings**, and **design debates** over new features. The two things a reader absolutely should not miss are **Junio C Hamano’s new `seen` ejection policy**, which formalizes the removal of unreviewed topics after four weeks, and **the `git repo info` path keys series**, which reached design consensus but faces abandonment due to persistent lack of reviewer engagement. Other high-signal developments include a **promisor remote lazy-fetch regression fix**, a **commit-graph corruption bug in Git 2.54’s geometric maintenance strategy**, and **Phillip Wood’s design objections** to the `git repo info` path keys naming and necessity.

---

## Key developments

### Integration pipeline policy: `seen` ejection after four weeks
Junio C Hamano adopted a **mechanical policy** to eject unreviewed topics from the `seen` integration branch after four weeks. The policy, announced in the "What's cooking" report, aims to balance reviewer productivity with quality control by removing topics that have not attracted sufficient review attention. Phillip Wood suggested framing ejection as a **reversible, routine event** rather than a final rejection, and Junio updated the "Discarded" section wording to emphasize that topics can be resubmitted if they gain enough interest. The policy reflects systemic challenges in Git's review process, where contributor volume often outstrips reviewer bandwidth. By the end of the week, the policy was in effect, with several topics (e.g., `git repo info` path keys, `git-p4` shell injection fix) explicitly flagged for imminent removal unless reviewers step up.

---

### `git repo info` path keys: design consensus reached, but series at risk
The **v8 series** adding seven new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, `path.cdup`) to `git repo info` saw **key design resolutions** but remains at **critical risk of abandonment** due to lack of reviewer engagement. K Jayatheerth and Justin Tobler agreed to special-case `/dev/null` handling for `path.hooks` keys, returning `/dev/null` verbatim when `core.hooksPath` is set to that value. However, Phillip Wood raised **fundamental objections** to the series: he questioned the necessity of exposing hooks and grafts paths, argued for renaming `path.toplevel` to `path.working-tree` and `path.superproject-root` to `path.superproject-working-tree`, and identified an edge case in `path.cdup` behavior at the repository root. The objections are substantive and must be addressed before merging, but the series has languished for weeks without sufficient review. Junio C Hamano warned that the series will be dropped from `seen` unless reviewers engage, highlighting the tension between contributor effort and reviewer bandwidth.

---

### Promisor remote lazy-fetch regression fixed
Patrick Steinhardt posted a **two-patch series** fixing a **critical regression** in promisor remote lazy-fetch that was causing `git merge` to fail with "not our ref" errors. The issue, introduced during ODB abstraction work, stemmed from stale loose object cache entries not being updated when new objects were written during merges. Kristofer Karlsson identified the root cause: the loose object cache's oidtree was not being updated after writing objects to listed subdirectories, causing Git to incorrectly assume objects were missing locally and trigger unnecessary, failing backfill fetches from promisor remotes. The fix updates the oidtree after writing objects, addressing a blocker for CI workflows in projects like Kubernetes that rely on blobless partial clones. The series was endorsed for `next` and is explicitly intended for backporting to stable branches.

---

### Commit-graph corruption in Git 2.54’s geometric maintenance strategy
Christoph Weiss reported a **critical bug** in Git 2.54’s geometric maintenance strategy where `git maintenance run --auto` prunes unreachable commits but fails to update the commit-graph, leaving it in an inconsistent state. The bug affects operations that rely on the commit-graph, such as `git log --graph` and `git fsck`. The reproduction script is self-contained and demonstrates the corruption via `git fsck`. The issue was introduced when the geometric strategy became the default in Git 2.54, and it has not yet been fixed. The bug is high-impact for users upgrading to Git 2.54, as it can cause silent data corruption and performance degradation.

---

### `git describe` regression fix converges on opt-out `--exact` flag
Linus Torvalds' patch fixing `git describe`'s commit distance calculation when commit timestamps are non-monotonic gained consensus on its user-facing design. The discussion settled on an **opt-out `--exact` flag** (defaulting to on) with a warning mechanism when the commit graph is unavailable. Linus endorsed Patrick Steinhardt's counterproposal, removing the last substantive objection. The patch addresses a real-world regression in the Linux kernel's version reporting during the v7.4 merge window, where skewed commit dates (caused by GitLab's Marge-bot workflow) led to incorrect version strings. The patch is now queued for integration and is a high-priority fix for users of `git describe` in large repositories with non-monotonic commit dates.

---

### `parse-options` early scanning API marked for `next`
Christian Couder's **v4 series** introducing a minimal, reusable `parse-options` sub-API (`early_scan_options()`) was marked for `next` after addressing all prior feedback. The series replaces fragile hand-rolled early argument scans with a standardized API, including validation checks, clearer documentation, and thorough test coverage. The `git fast-import` bugfix and preparatory refactorings are now uncontroversial, and the core API is ready for integration. The series is a significant step toward reducing code duplication and improving robustness in Git's argument parsing. Kaartic Sivaraam's substantive review confirmed the series' readiness, and Junio C Hamano's merge signal indicates it will graduate to `master` in the near term.

---

### HTTP retry logic: 10-second default adopted
Ravi Mistry's **five-patch series** addressing gaps in Git's HTTP transport error handling adopted a **10-second default** for `http.retryAfter`, striking a balance between server protection (brian m. carlson's priority) and interactive usability (Ravi's concern). The series adds exponential backoff (10s, 20s, 40s, ...), makes retryable status codes configurable, and extends retries to HTTP POST requests. The 10-second default was uncontested, but Junio C Hamano raised a **procedural concern** about the two-phase default value introduction (patches 1–4 assume 0, patch 5 sets 10s), questioning whether it is the right way to structure the series for maintainability. The v3 series is technically complete and awaits Junio's final review.

---

### Split-index corruption bug diagnosed
Tanga Joris diagnosed the root cause of a **critical bug** causing silent index corruption (null OIDs, missing entries) and crashes (`BUG: wt-status.c:585`) when both `core.splitIndex` and `index.skipHash` are enabled. The bug manifests in repositories using split-index or skipHash optimizations (e.g., large repos with `feature.manyFiles`), causing data loss and crashes during `git status`. The root cause is `write_shared_index()` naming the shared index file after its trailing checksum, which is written as zeros when `index.skipHash` is enabled, resulting in a null base OID and silent data loss. Tanga posted a fix and test coverage, but the patch has not yet received maintainer feedback. The bug is high-impact for users of split-index or skipHash optimizations and must be addressed before the next release.

---

## In brief

**`git annotate` completion fix** -- Tanga Joris's patch fixing command-line completion for `git annotate` was approved by Junio C Hamano and marked ready for `next`. The fix removes redundant `-c` prepending in `cmd_annotate()`, ensuring `git annotate --<TAB>` offers the correct options.

**`git apply` crash regression fix** -- Hannes Brandstätter-Müller's fix for a crash in `git apply` when `core.fileMode=false` received substantive review. The regression causes patch series containing a pure rename followed by an edit to fail with `error: renamed: wrong type`. Aniket Gupta identified a potential security edge case and proposed a refined fix.

**`git bundle-uri` URI handling fix** -- Ravi Mistry's patch fixing a crash when processing relative URIs with excessive `../` components was marked ready for `next` after addressing Patrick Steinhardt's review feedback.

**`git credential-cache` daemon race condition fix** -- Farid Zakaria's fix for a race condition in the Git credential-cache daemon acknowledged Junio's architectural concern about mixing `unix_ss_*` and `tempfile` APIs. The race causes concurrent `git credential-cache store` commands to unlink each other's sockets, leading to lost credentials and orphaned daemons.

**`git grep --recurse-submodules` fix** -- Patrick Steinhardt's three-patch series fixing `git grep --recurse-submodules` for nested submodule trees was marked for `next`. The fix targets a regression introduced in Git v2.56.0 and is explicitly intended for backporting to stable branches.

**`git log --follow` performance regression** -- A severe slowdown (600x) was reported when tracking files through non-linear history, bisected to commit 304812ed33. Patrick Steinhardt identified a potential regression in Miklos Vajna's performance series, where `follow_needs_renames()` disables rename detection for commits with unrelated histories.

**`git maintenance` lock-handling consistency** -- Patrick Steinhardt's series improving lock-handling consistency in `git maintenance` was split into two patches (behavioral change and Trace2 event) after review. The series addresses a long-standing inconsistency in how `git maintenance` handles lock files.

**`git merge-blobs` repository refactoring** -- Junio C Hamano marked a patch removing the implicit dependency on `the_repository` in `merge-blobs` ready for `next`. The patch is part of the ongoing effort to eliminate global state in Git's codebase.

**`git range-diff --matched-only` feature** -- The feature, which filters `git range-diff` output to show only matched commits, received substantive review. The discussion centered on the user-facing design and edge cases.

**`git reflog` `%gb` format placeholder regression** -- A regression was identified in branch-name detection when `GIT_REFLOG_ACTION` is set. The bug affects the `%gb` format placeholder in `git reflog`, causing incorrect branch names to be displayed.

**`git rev-list` option parsing fix** -- Azuama Odinama Maximus's fix for `--grep` and related option parsing when followed by another option-like string was expanded to cover all 18 `parse_long_opt()`-style options. The v4 patch addresses all prior feedback and ensures consistent argument parsing.

**`git unpack-objects` type mismatch fix** -- Kristofer Karlsson's fix for a type mismatch in `unpack-objects` addresses a regression where blob sizes larger than 4GB are truncated on 32-bit platforms (notably Windows). The fix changes `stream_blob()` to accept `size_t`.

**`git-p4` shell injection fix** -- Junio C Hamano warned that the patch fixing a shell injection vulnerability in `git-p4.py`'s `applyCommit()` function will be dropped from `seen` unless reviewers step up. The patch is well-motivated and addresses a real security issue.

**`reftable` backend HEAD reflog bug** -- 井上司 reported a correctness regression in the reftable backend where HEAD reflog entries are either written with the wrong OID or omitted entirely when HEAD is updated in the same transaction as its target branch. The bug affects the reftable backend's atomic transaction support and was discussed as a design flaw in how symref updates are handled.

**CI optimizations** -- Patrick Steinhardt's seven-patch series targeting GitLab CI bottlenecks received maintainer acknowledgment. Junio found all patches "interesting" and "fully made sense," and Torsten Bögershausen reported tested results confirming performance improvements in `t0027-auto-crlf.sh`.

**Documentation updates** -- Julia Evans' rewrite of the `git diff` EXAMPLES section sparked debate over presentation format and terminology consistency. Junio objected to the table format, the promotion of `--staged` over `--cached`, and the omission of `git diff AUTO_MERGE`. The discussion highlights tensions between modernization and preserving historical context in documentation.

**Global config handling inconsistency fixed** -- Delilah Ashley Wu's two-patch series fixing Git's global config file handling inconsistency was accepted. The series ensures `git config list --global` reads both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`, aligning with unscoped operations.

**Outreachy proposals** -- Several Outreachy applicants proposed projects, including:
- **Configurable fetch ordering for promisor remotes**: Shashank S Mogaveera and Yopa Nelly proposed solutions for specifying the order in which Git tries multiple promisor remotes during lazy fetches.
- **Improving Git command argument scanning**: Cynthia Uche and Tanga Joris proposed projects to improve consistency and robustness in how Git built-in commands scan and parse command-line arguments.
- **Global state reduction**: Srujith Poojary proposed reducing Git's reliance on global state in pathspec skip-worktree matching logic.

**Rustification** -- Tanga Joris identified a potential inconsistency in the Rust `hash` module, where algorithm ID 0 is treated as invalid (appending `:invalid-hash-algo-0`) while the C code silently falls back to the default hash algorithm. The discrepancy could cause divergent behavior between the two codebases.

**Test modernization** -- Kristofer Karlsson's `test-tool genbytes` series, which introduces a new command to replace slow `tr(1)` invocations in the Git test suite, was posted as v2. The series addresses all prior feedback and eliminates a measurable performance bottleneck on macOS.

---

## Looking ahead

The next week is likely to see **continued focus on regressions and stalled series**. Key topics to watch include:

- **`git repo info` path keys**: The series faces abandonment unless reviewers engage with Phillip Wood's design objections. If the objections are addressed, the series could graduate to `next`; otherwise, it may be dropped from `seen`.
- **Commit-graph corruption in Git 2.54**: The bug report has not yet received a fix, and the issue is high-impact for users upgrading to Git 2.54. A fix is likely to be prioritized in the coming week.
- **Split-index corruption bug**: Tanga Joris's fix is in progress, and maintainer feedback is expected. The bug is critical for users of split-index or skipHash optimizations.
- **HTTP retry logic**: Ravi Mistry's series is technically complete and awaits Junio's final review. The 10-second default is uncontested, but the patch organization may need adjustment.
- **Reftable backend HEAD reflog bug**: 井上司's bug report and proposed fix are likely to receive further discussion, particularly around the broader refactoring to move reflog logic into the generic `refs.c` layer.
- **Outreachy proposals**: The Outreachy application period closes soon, and final proposals are likely to be refined based on feedback. Projects targeting global state reduction, promisor remote improvements, and argument parsing consistency are strong candidates.