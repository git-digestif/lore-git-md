# Git mailing list daily digest for 2026/10/23

## The day in brief
The incremental connectivity check series was merged to `next`, completing a major performance feature for fetches and pushes. Junio C Hamano signed off on the `gitmergeconflicts(7)` documentation series after a final mechanical fix. Two CI/build system patches sparked debate about upstream policy versus downstream constraints. Several Outreachy contributions advanced, with one merged to `master` and others nearing completion.

## Notable threads

### Incremental connectivity check merged to `next`
**Thread root**: 2026/09/14/09-47-54
**What changed**: Junio C Hamano signed off on v5 of Kristofer Karlsson’s six-patch series introducing an opt-in incremental connectivity check for fetches and pushes. The series is now merged to `next`.
**Problem/goal**: Reduce the cost of verifying object connectivity during transfers by shifting the cost from repository size to incoming change size.
**Subsystem**: fetch/push, connectivity check
**Impact**: Dramatic speedups for small pushes to large repositories (up to 190x faster, 14 MB vs. 3.4 GB memory usage) with a modest regression for very long incoming histories (1.5x slower), mitigated by the auto mode fallback.
**Key technical details**: New config keys (`transfer.connectivityCheck=full|incremental|auto[=<n>]`, `transfer.connectivityCheckIncrementalMaxCommits=<n>`), new internal `rev-list` flag (`--connectivity-check=`), and a new source file (`tree-verify.c`) implementing an inductive trust algorithm. The auto mode (patch 6/6) dynamically switches between incremental and full checks at a configurable 1,000-commit cutoff.
**Today’s delta**: [2026/10/23/18-52-09] Junio signed off on v5, confirming the series will be merged as-is. The two key improvements in v5—eliminating reliance on `repo->fetch_if_missing` and reporting corrupt trees as errors—directly addressed prior maintainer concerns.

---

### `gitmergeconflicts(7)` documentation series finalized
**Thread root**: 2026/09/24/14-44-15
**What changed**: Junio C Hamano queued the six-patch series introducing `gitmergeconflicts(7)` after addressing a mechanical blocker (adding an empty `SYNOPSIS` section to pass `make check-docs`).
**Problem/goal**: Consolidate scattered merge-conflict advice across multiple man pages into a single, example-driven guide.
**Subsystem**: documentation
**Impact**: Replaces duplicated conflict-resolution sections in existing man pages with cross-references to the new guide, improving clarity and reducing maintenance burden.
**Key technical details**: The new guide covers conflict markers, "ours/theirs" terminology, `diff3` vs. default formats, resolution tools, and an exhaustive list of conflict-causing commands. The series touches 10 files, including the new `Documentation/gitmergeconflicts.adoc` (339 lines) and updates to `git-merge.adoc`, `git-rebase.adoc`, and others.
**Today’s delta**: [2026/10/23/16-18-31] Junio confirmed the `SYNOPSIS` issue was resolved via the merge to `seen`, unblocking graduation to `next`.

---

### CI: Pin GitHub Actions to full commit IDs
**Thread root**: 2026/10/23/16-38-23
**What changed**: Taylor Blau proposed pinning all GitHub Actions in the main workflow file to full commit SHAs instead of version tags or branches, motivated by organizational compliance requirements. Junio C Hamano raised a policy objection, questioning whether downstream constraints should dictate upstream workflows.
**Problem/goal**: Prevent CI compromise via malicious action updates by using explicit commit hashes, as required by some organizational policies.
**Subsystem**: CI/build system
**Impact**: No functional changes to Git; purely affects CI execution on GitHub. The patch replaces 32 action references in `.github/workflows/main.yml` with their corresponding commit SHAs.
**Key technical details**: Actions affected include `actions/checkout`, `actions/github-script`, `actions/upload-artifact`, and others. Each replacement retains the original version tag in a trailing comment.
**Today’s delta**: [2026/10/23/21-45-01] Junio suggested maintaining such modifications in downstream forks to avoid precedent risks, while Ben Knoble [2026/10/23/18-05-00] proposed clarifying the security rationale in the commit message.

---

### Untracked cache performance improvements
**Thread root**: 2026/09/23/17-10-07
**What changed**: Tamir Duberstein posted v3 of a three-patch series teaching `git ls-files` to reuse and update the untracked cache populated by `git status`. The series is under active review with no objections recorded yet.
**Problem/goal**: Eliminate redundant directory traversals in common workflows (e.g., `git status` followed by `git ls-files`) by reusing the untracked cache.
**Subsystem**: untracked cache, `dir.c`
**Impact**: Dramatic speedups for wildcard queries (7.3× faster for `**/pyproject.toml` on a synthetic tree with 100k files) and combined `git status` + `git ls-files` time (293 ms → 93 ms).
**Key technical details**: Patch 1 fixes an ignore-file hash inconsistency, patch 2 enables cache sharing between `git status -unormal` and `git status -uall`, and patch 3 implements cache reuse in `git ls-files` with optional index writes. The v3 update releases the index lock after reading the directory but before printing paths, preventing slow readers from blocking index writers.
**Today’s delta**: [2026/10/23/20-03-03] The v3 cover letter confirms the lock release timing and `--debug` behavior changes, with no new objections.

---

### Shallow fetch regression fix
**Thread root**: 2026/10/08/22-10-37
**What changed**: Karthik Nayak and Patrick Steinhardt discussed a blocker-level memory management issue in the v4/v5 patch restoring dynamic reloading of alternate object databases. The issue stems from shared ownership of the `negotiation_include_tips` `oid_array`, which could leak memory or corrupt negotiation state if reused by other callers.
**Problem/goal**: Fix a performance regression in shallow fetches introduced by commit 0e358de64a ("fetch: use batched reference updates", 2025-05-19), where batched updates delay committing references to disk until after tag backfilling begins, causing the client to report no 'have' references during negotiation.
**Subsystem**: fetch, alternates, ODB
**Impact**: Restores correct 'have' reference reporting during negotiation for shallow fetches with tags, eliminating multi-minute delays.
**Key technical details**: The patch exposes pending transaction state via `negotiation_include_tips` and introduces a `tail` pointer to track newly added directories. The memory management issue requires redesign to avoid shared-state risks.
**Today’s delta**: [2026/10/23/22-12-25] Karthik acknowledged the issue and noted that remote helpers (e.g., those used in `t5509`) support `smart_options` but fail to clean up `negotiation_include_tips`, exposing an existing bug in the transport layer.

---

### `git stash` shell completion improvements
**Thread root**: 2026/10/12/17-48-26
**What changed**: Junio C Hamano enthusiastically endorsed the second patch in D. Ben Knoble’s five-patch series, calling it a clear improvement. The author declined to implement Junio’s proposed enhancement for the fifth patch (automatically rewriting `git stash save --<TAB>` to `git stash push --<TAB>`).
**Problem/goal**: Improve the usability and correctness of `git stash` shell completion by refactoring existing logic, fixing a Zsh-specific bug, and removing completion support for the deprecated `git stash save` subcommand.
**Subsystem**: shell completion
**Impact**: Zsh users can now correctly complete stash names (e.g., `stash@{0}`), and the deprecated `save` subcommand is removed from completion candidates unless `GIT_COMPLETION_SHOW_ALL` is set.
**Key technical details**: Patch 2 extracts common stash-listing code into a helper function (`__git_complete_stashes`), and patch 3 fixes the Zsh bug by removing the `-Q` flag in `compadd`. Patch 5 removes `save` from Bash completion candidates.
**Today’s delta**: [2026/10/23/17-06-32] Junio endorsed patch 2/5 ("Nice to see two of these unsightly constructs go"). [2026/10/23/18-07-36] The author declined Junio’s enhancement for patch 5/5, citing increased complexity.

---

### Community voting process discussion
**Thread root**: 2026/10/06/18-05-36
**What changed**: Junio C Hamano and Kristoffer Haugsbakk continued debating the eligibility criteria for Git’s proposed community voting process, specifically the two-year activity window. Kristoffer clarified that some users—even those who do not contribute to Git—hold strong negative views about projects that use GenAI, and these users might attempt to influence contentious votes despite lacking a long-term stake in the project.
**Problem/goal**: Establish a formal mechanism to resolve contentious decisions (e.g., AI contribution policy) when consensus cannot be reached through normal discussion.
**Subsystem**: governance
**Impact**: The voting process would serve as a template for future contentious decisions, with eligibility criteria determining who can participate.
**Key technical details**: The patch proposes contributors with at least 50 merged patches or multiple code reviews, but the two-year activity window remains contentious. Junio opposes strict time-based cutoffs, while Kristoffer supports them as a pragmatic safeguard against low-effort participation.
**Today’s delta**: [2026/10/23/18-09-19] Kristoffer clarified that some users with strong negative views about GenAI might attempt to influence votes like the AI contribution policy, reinforcing the need for a two-year activity window.

---

### Pack index v3 for SHA-1/SHA-256 interoperability
**Thread root**: 2026/10/23/07-25-45
**What changed**: Junio C Hamano merged Christian Couder’s 13-patch v2 series implementing pack index v3 to `next`. The series enables bidirectional mapping between SHA-1 and SHA-256 repositories by storing object IDs in both algorithms.
**Problem/goal**: Implement pack index v3, a new index format that stores object IDs in both the main and compatibility hash algorithms, enabling interoperability between SHA-1 and SHA-256 repositories.
**Subsystem**: ODB, pack index
**Impact**: Enables bidirectional mapping between hash algorithms, a critical step toward full SHA-1/SHA-256 interoperability. The series builds on parts 1 and 2 (already merged to `master`).
**Key technical details**: The v3 index format includes a variable-length header, per-algorithm tables (short-name, full-name, order-map, CRC, offset), and strict validation logic. The series touches `pack-write.c`, `packfile.c`, `odb/`, and related headers.
**Today’s delta**: [2026/10/23/15-00-44] Junio queued the series to `next`, requesting future revisions reply to the previous cover letter to maintain automation tooling.

---

## In brief
- **[2026/10/23/16-48-39]** Harald Nordgren provided new empirical evidence for the CI/build system patch removing a redundant `brew link --force gettext` command, resolving Patrick Steinhardt’s earlier objection.
- **[2026/10/23/23-06-11]** Junio C Hamano clarified that his review of the CI/build system patch silencing "missing tool" warnings was not a full endorsement but merely an absence of objections, noting the patch still needs a "real positive review."
- **[2026/10/23/10-44-54]** Patrick Steinhardt approved the second patch in Igor Wiedler’s two-patch bugfix series addressing a connectivity-check performance regression in deepening fetches, stating no further reroll is needed to fix email threading.
- **[2026/10/23/06-21-10]** Patrick Steinhardt confirmed the current behavior of the `git stash` shell completion patch removing `save` completion is acceptable, resolving the last open design question.
- **[2026/10/23/17-55-44]** Junio C Hamano queued Tanga Joris’s i18n pluralization fix for `git checkout` to `seen`, marking it ready for merging.
- **[2026/10/23/18-30-49]** Junio C Hamano queued tsukasa-ino’s two-patch series fixing inconsistent branch switch behavior on tracking config failure, noting it is now in `seen`.
- **[2026/10/23/15-53-48]** Harald Nordgren confirmed the behavior of `git check-ref-format --no-allow-onelevel`, validating Fien Dora’s Outreachy microproject inquiry about test coverage.
- **[2026/10/23/17-56-15]** Junio C Hamano queued Cynthia Uche’s test modernization patch replacing an unreliable `git ls-files -t | test_cmp` pipeline, marking it merged to `master`.
- **[2026/10/23/04-34-48]** Srujith Poojary addressed Junio C Hamano’s feedback on the test modernization patch, confirming the `sed` line now aligns properly with surrounding commands.
- **[2026/10/23/12-21-11]** Matt Hunter proposed a refined test assertion for the `git init` regression fix, addressing Patrick Steinhardt’s feedback on testing the full output message.