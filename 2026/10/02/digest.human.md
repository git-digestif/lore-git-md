# Git mailing list daily digest for 2026/10/02

## The day in brief

The Git mailing list saw active discussion on several key topics today. The most consequential developments include:

- **`git history` signing series** encountered integration issues with a related `git replay` signing series, requiring a rebase
- **`uploadpack.lazyFetchTrusted`** series reached v5 with all feedback addressed, now ready for maintainer review
- **`gitmergeconflicts(7)`** documentation series saw significant progress with new sections and design discussions
- **MIDX reachability closure fixes** received substantive review feedback on data structure choices
- **Windows memory regression** with fsmonitor was reported and is under investigation

## Notable threads

### `git history` signing series integration conflict
**What changed**: Junio C Hamano identified a merge conflict between Souma's `git history` signing series (now in `seen`) and Patrick Monette's `git replay` signing series. The `git replay` series must wait for the `git history` work to land first.

**Problem/goal**: Both series add GPG signing support to different commands but touch overlapping commit-creation logic in `replay.c`.

**Impact**: The `git replay` signing feature is blocked until the `git history` series stabilizes. This affects users who want to sign commits during replay operations.

**Thread**: [2026/09/28/14-38-02]

---

### `uploadpack.lazyFetchTrusted` v5 ready for review
**What changed**: Christian Couder posted v5 of the `uploadpack.lazyFetchTrusted` series, addressing all feedback from Junio's v4 review. The series replaces the earlier client-side `GIT_NO_LAZY_FETCH=fromAccepted` proposal with a server-side protected configuration variable.

**Problem/goal**: Provide a server-side mechanism for operators to explicitly declare which repositories are safe to lazy-fetch from, addressing security concerns about untrusted repositories triggering arbitrary code execution.

**Impact**: Server operators can now mark repositories as trusted for lazy fetching using `uploadpack.lazyFetchTrusted`, with recursion safeguards preventing infinite loops. This is a significant security improvement over the previous client-side approach.

### Key technical details

- New protected configuration variable: `uploadpack.lazyFetchTrusted`
- Recursion depth counter via `GIT_INTERNAL_LAZY_FETCH_DEPTH` (limit of 5)
- Files touched: `promisor-remote.c`, `setup.c`, `upload-pack.c`, and related documentation

**Thread**: [2026/07/10/08-51-34]

---

### `gitmergeconflicts(7)` documentation progress
**What changed**: Julia Evans incorporated significant feedback into the new `gitmergeconflicts(7)` man page, adding a "WHAT IS A MERGE CONFLICT?" section and improving the explanation of `diff3` conflict style.

**Problem/goal**: Consolidate scattered merge conflict advice into a dedicated guide to improve user experience and reduce documentation maintenance burden.

**Impact**: Users will have a centralized resource for understanding and resolving merge conflicts, with clearer explanations of key concepts like "ours" vs. "theirs" and the benefits of `diff3` style.

### Key additions

- New "WHAT IS A MERGE CONFLICT?" section explaining the basics
- Improved `diff3` documentation with concrete examples showing its importance
- Clarification of terminology and edge cases

**Thread**: [2026/09/23/20-20-20]

---

### MIDX reachability closure fixes
**What changed**: Jeff King (Peff) and Elijah Newren provided substantive review feedback on Taylor Blau's MIDX reachability closure series, particularly focusing on the data structure choice for `extra_roots`.

**Problem/goal**: Fix corner cases where MIDXs can end up containing objects that are not closed under reachability, causing failures in bitmap generation.

**Impact**: The series ensures reachability closure for bitmaps after incremental repacks, preventing subtle corruption issues in large repositories.

### Key technical discussion

- Peff and Newren argue for preserving pack-order correlation (root trees before subtrees) for delta compression quality
- Taylor Blau defends the `oidset` approach for memory efficiency
- The discussion highlights trade-offs between functional correctness and performance

**Thread**: [2026/09/30/15-15-29]

---

### Windows memory regression with fsmonitor
**What changed**: Pierre Bruno reported unexpectedly high memory consumption (around 1 GB per process) when running `git status`-like commands in Git 2.56.0.windows.1 with fsmonitor enabled.

**Problem/goal**: Investigate and resolve a memory usage regression that appears to be specific to Windows and fsmonitor.

**Impact**: Users on Windows may experience excessive memory consumption when using fsmonitor, particularly with coding agents that spawn multiple Git processes.

### Key details

- Regression present in 2.56.0, absent in 2.55.0
- Repository has many untracked files
- Dozens of concurrent Git processes consume several GB of RAM total
- Near-zero CPU usage with steady disk I/O

**Thread**: [2026/10/02/11-04-00]

## In brief

- **[PATCH v3 0/2] format-patch: add --[no-]range-diff-notes**: Kristoffer Haugsbakk addressed Junio's feedback on commit message phrasing and documentation wording. The series is now technically complete and ready for maintainer approval. [2026/08/24/20-35-41]

- **[PATCH 0/4] Fix MIDX reachability closure corner cases**: Taylor Blau's series received substantive review feedback on data structure choices, with Jeff King and Elijah Newren engaging in a detailed discussion about the trade-offs between `oidset` and `oid_array`. [2026/09/30/15-15-29]

- **[PATCH 0/2] remote: support multiple remotes in remote.pushDefault**: Harald Nordgren proposed extending `remote.pushDefault` to accept a space-separated list of remote names, allowing a single global configuration to work across multiple repositories. [2026/10/02/07-17-50]

- **[PATCH 0/2] packfile: fix corruption from stale delta base cache entries**: Patrick Steinhardt's bugfix series for packfile corruption received review feedback from Philippe Blain and Jeff King, with Peff suggesting the use of AddressSanitizer for more reliable testing. [2026/09/25/20-53-46]

- **[PATCH 0/13] Move alternates into the files ODB backend**: Patrick Steinhardt posted a major refactoring series to move alternates handling into the "files" backend, eliminating a conceptual mismatch that caused performance regressions. [2026/10/02/10-08-11]

- **[PATCH 0/2] fetch: optimize commit-graph write strategy for large repos**: Kristofer Karlsson clarified core design assumptions in response to Patrick Steinhardt's review, confirming that the seeds passed to `write_commit_graph()` are additive and providing benchmark numbers. [2026/10/02/08-33-36]

- **[PATCH] doc: remove empty SYNOPSIS sections from section 7 man pages**: Julia Evans and Junio C Hamano discussed implementation details for the Perl linter script, with Junio providing a solution to handle multiple input files correctly. [2026/10/02/16-07-07]

- **[PATCH] stash: extend `pop` with custom conflict-label options**: Harald Nordgren withdrew the patch after maintainer skepticism, with Junio C Hamano proposing a longer-term architectural improvement to integrate the functionality directly into `git rebase`. [2026/10/01/22-48-09]