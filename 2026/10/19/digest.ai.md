Here is the 800–1400 word AI-mode digest for 2026-10-19, preserving all technically consequential deltas:

---

# Git mailing list daily digest for 2026/10/19

## The day in brief
Junio C Hamano’s review of the CRLF conversion size-trusting bugfix series sharpened the focus on performance trade-offs, while the macOS character encoding effort saw no new deltas. A ZIP archive date-handling patch reached integration readiness, and the pluggable object storage RFC series advanced with substantive review feedback. Outreachy applicants proposed microprojects, and a performance regression in `git log --follow` received a targeted fix.

## Notable threads

### macOS CRLF conversion size-trusting bug (read-cache.c)
**What changed?**
Junio C Hamano responded to Torsten Bögershausen’s proposal to move the `would_convert_to_git()` call from `ie_modified()` to `ie_match_stat()` to avoid racy Git interference. Junio agreed that `ie_match_stat()` might be a better fit but emphasized that correctness alone is insufficient—the patch must also preserve the lstat()-based optimization that avoids unnecessary content comparisons.

**Why it matters**
The lstat() optimization is a long-standing performance trade-off that skips content comparisons for files whose sizes haven’t changed. Junio’s framing—"Having the code behave the way we want is a mere baseline requirement"—implies the current design may not yet meet Git’s efficiency standards. The architectural debate remains unresolved among three competing approaches: (1) Curtis Allen Smith’s original placement in `ie_modified()`, (2) Johannes Sixt’s preference for fixing size recording during `git add`, and (3) Torsten’s proposal. Junio’s latest review introduces a fourth concern: the performance cost of unconditionally triggering content comparisons.

**Technical details**
- Subsystem: `read-cache.c` (index refresh logic)
- Core problem: Git blindly trusts the reported post-conversion file size without verifying conversion success, causing false positives in commands like `git status` when clean filters or CRLF conversion alter file size.
- Performance data: Curtis’s benchmarks show negligible overhead in realistic scenarios (12 microseconds per modified 2.6 KB file) but measurable costs in extreme cases (138ms for 10,000 modified files).
- Open question: How to reconcile correctness, racy Git concerns, and the lstat() optimization.

---

### ZIP archive date-handling bugs (archive-zip.c)
**What changed?**
René Scharfe’s patch to omit the ZIP Extended Timestamp (UT) extra field for out-of-range commit times (bug #2) reached integration readiness after Matthew E. Luallen’s test infrastructure fix. The patch now uses `TIME_T_IS_64BIT` instead of `TIME_IS_64BIT` to handle 32-bit platforms correctly, resolving CI failures on Ubuntu 20.02 i386 and Debian i386.

**Why it matters**
The fix prevents silent timestamp wrap (e.g., 2106-02-07 → 1970) in UnZip’s update mode, which could cause data inconsistencies. The patch is narrowly scoped to the UT field omission and does not address the separate ZIP legacy DOS date overflow bug (bug #1), which remains open.

**Technical details**
- Files touched: `archive-zip.c`, `t/t5004-archive-corner-cases.sh`
- Behavior change: UT field omitted when commit time exceeds 32-bit signed integer range (`> 0x7fffffff`).
- Test coverage: New test verifies UT field omission; tests skip on 32-bit platforms where `zipinfo(1)` lacks UT field support.
- Unresolved issue: The `dos_time()` function produces invalid timestamps for values `> 0x7fffffff` (e.g., `@2147483648 +0000` → `18446744071562067968`), but this is unrelated to the UT field patch.

---

### Pluggable object storage RFC (extensions.objectStorage)
**What changed?**
Patrick Steinhardt posted a seven-patch RFC series introducing the `extensions.objectStorage` mechanism, modeled after the existing ref storage extension. The series implements the baseline `files` backend, which supports relocating the `.git/objects` directory via a payload (e.g., `files://objs` moves objects to `.git/objs`). Karthik Nayak provided substantive review feedback on patches 2/7–4/7, focusing on design consistency, test coverage gaps, and architectural concerns.

**Why it matters**
This is the foundational step toward pluggable object storage backends (e.g., cloud storage, databases). The series aligns with Git’s long-term modularity goals but raises design questions about path normalization, default encapsulation, and test thoroughness.

**Technical details**
- Files touched: `odb/` (including `loose.c`/`source.c`), `setup.c`, `repository.c`, documentation, and tests.
- New config keys: `extensions.objectStorage`, `init.defaultObjectStorageFormat`.
- Environment variables: `GIT_OBJECT_STORAGE_FORMAT`, `GIT_DEFAULT_OBJECT_STORAGE_FORMAT`.
- CLI options: `--object-storage-format=` (for `git clone`, `git init`, `git submodule`).
- Backend: `files` (default, with payload support for directory relocation).
- Test coverage: Covers initialization, cloning, submodule propagation, schema validation, and environment variable overrides.
- Open questions:
  - Should the `files` backend normalize relative paths to absolute paths using `strbuf_realpath()` (Karthik’s suggestion)?
  - Why expose the hard-coded default (`files`) in inspection tools like `git repo` (Karthik’s concern)?
  - Are the tests exhaustive enough for payload handling and path resolution?

---

### `git log --follow` performance regression fix
**What changed?**
Miklos Vajna posted a patch to fix a severe performance regression in `git log --follow` introduced by commit 304812ed33. The regression caused a test case on linux.git to balloon from ~28s to ~6m43s; the patch reduces this to ~37s.

**Why it matters**
The fix addresses a real-world performance pain point for users working with large repositories. The original improvement introduced a function (`propagate_follow_pathspec_to_parent()`) that diffed each commit against *all* parents with rename detection enabled, which is computationally expensive. The new approach reuses diffs already computed in `log_tree_diff()`’s parent loop and avoids redundant work.

**Technical details**
- Files touched: `log-tree.c`, `t/t4219-log-follow-merge.sh`
- New functions: `commit_has_path()`, `follow_needs_renames()`, `log_tree_merge_diff()` (refactored from existing code).
- Edge case handling: Correctly follows renames on the side of a merge that wasn’t taken.
- Performance: Test case improves from 6m43s to 37s.
- Test coverage: New test verifies the fix for merge commits with `-c`, `--cc`, or `--remerge-diff`.

---

### `git imap-send` OpenSSL compatibility and correctness fixes
**What changed?**
Patrick Steinhardt raised three critical concerns blocking the OpenSSL 4.0 compatibility macros in Beat Bolli’s v3 series:
1. **Procedural issue**: The macros should have been introduced in the same commit as the `SSL_set1_host()` call (patch 2/3), not as a separate follow-up.
2. **Correctness issue**: The patch assumes `cfg->host` is always a DNS name, but OpenSSL 4.0 splits hostname verification into `SSL_set1_dnsname()` (DNS names) and `SSL_set1_ipaddr()` (IP addresses). If `cfg->host` could ever be an IP address, the current patch would silently misbehave.
3. **Clarity issue**: The error message conditionally changes the function name, which adds no value and could confuse debugging.

**Why it matters**
The v3 series is blocked on these issues, which must be resolved before the macros can be merged. The broader series remains stalled on a **security regression** (certificate stripping) and a **behavioral regression** (redundant `verify_peer()` call), but the OpenSSL 4.0 compatibility macros are now a secondary blocker.

**Technical details**
- Files touched: `imap-send.c`
- New macros: `#define SSL_set1_dnsname SSL_set1_host` for OpenSSL < 4.0.
- Unresolved regressions:
  - **Security regression**: Removal of the `SSL_get_peer_certificate()` check allows certificate stripping.
  - **Behavioral regression**: The redundant `verify_peer()` call after `SSL_connect()` fails under `SSL_VERIFY_PEER` is dead code.

---

### Incremental connectivity check for fetch/push
**What changed?**
Patrick Steinhardt accepted the **1,000-commit cutoff** as a reasonable heuristic after seeing benchmark data across small (`git.git`), medium (`linux.git`), and large (proprietary) repositories. The data showed incremental mode is **2.6–2.8x faster** at 1,000 commits for medium/large repos, while small repos saw a **~30% slowdown** (but absolute times remained negligible). Patrick suggested making the cutoff configurable via `auto[=<cutoff>]`, defaulting to 1,000, to allow future tuning without breaking changes.

**Why it matters**
The series introduces an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental` or `auto`) to reduce the cost of verifying object connectivity during fetches and pushes. The configurable cutoff balances performance and safety, enabling gradual adoption and potential future default changes.

**Technical details**
- New config keys: `transfer.connectivityCheck=full|incremental|auto[=<n>]` (default: `full`), `transfer.connectivityCheckIncrementalMaxCommits=<n>` (default: 1,000).
- New internal `rev-list` flag: `--verify-trees-incremental`.
- Performance: Up to 190x faster for small pushes to large repos (14 MB vs. 3.4 GB memory), 2.6–2.8x faster for large/medium repos at 1,000 commits.
- Open question: Whether the stack overflow risk in recursive tree verification should be addressed in a follow-up.

---

### `git bundle create` memory safety bugfix
**What changed?**
Junio C Hamano queued Olof Fairbrook’s patch to fix a memory safety issue in `git bundle create` where uninitialized stack memory was freed on unrecognized arguments. The fix initializes the `revs_copy` struct at declaration with `struct rev_info revs, revs_copy = { 0 }`.

**Why it matters**
The bug was introduced in commit 11ee9a75e7 (2024-06-11) and could cause crashes or undefined behavior. The fix is minimal and uncontroversial, addressing a latent correctness issue.

**Technical details**
- Files touched: `bundle.c`, `t/t5607-bundle.sh`
- Behavior change: None for valid invocations; fixes memory safety in error paths.
- Test coverage: New test verifies error handling without leaving cruft.

---

### Outreachy microprojects
**What changed?**
Two Outreachy applicants proposed microprojects:
1. **Cynthia (Priestess)**: Fix suppressed exit codes in `t/t2104-update-index-skip-worktree.sh` by writing `git ls-files -t` output to a temporary file before comparing.
2. **Joy**: Requested a scoped task for migrating a global variable into `struct repo_config_values` as part of the `the_repository` removal effort. Usman Akinyemi directed Joy to the official Git microprojects page.

**Why it matters**
These microprojects provide beginner-friendly entry points for new contributors, aligning with Git’s mentorship goals. Cynthia’s proposal is actionable and addresses a real test-suite issue, while Joy’s request highlights the need for well-scoped tasks in the `the_repository` removal effort.

---

## In brief
- **`git stash` shell completion**: D. Ben Knoble confirmed that Bash is unaffected by the Zsh stash name quoting bug and that no portable solution exists, leaving the Zsh-specific override as the only viable fix.
- **`git rebase` documentation**: Omri Sarig posted v2 of a patch to correct outdated and missing option links in the "INCOMPATIBLE OPTIONS" section, incorporating Phillip Wood’s feedback on commit message phrasing and the `--onto` compatibility note.
- **`git fsck` i18n**: Aniket Gupta’s patch to use `Q_()` for the verbose "Checking connectivity" message was approved by Junio C Hamano, who noted that while the displayed count is not an accurate object count, using `Q_()` is still the correct approach for i18n support.
- **`git imap-send` header parsing**: Junio C Hamano identified a protocol-compliance issue in Harald Nordgren’s patch: the new `has_required_headers()` helper uses case-sensitive `starts_with()` instead of case-insensitive `istarts_with()`, violating RFC 5322.
- **`git status` default comparison**: Junio C Hamano suggested the commit message for Harald Nordgren’s patch to compare against both `@{upstream}` and `@{push}` by default should first establish the historical context of `status.compareBranches` and questioned whether the test coverage adequately demonstrates both the triangular case and the common case where upstream and push are the same.
- **`git fetch` skipping negotiator**: Taylor Blau posted a patch to fix a correctness issue in the skipping negotiator’s queue management, ensuring parent commits are parsed before being added to the queue to prioritize recent common ancestors.
- **`MyFirstContribution` tutorial**: Junio C Hamano posted a patch to reorder the tutorial to emphasize review discussion before sending v2, addressing an uptick in new contributors sending updated patches without first addressing review feedback.
- **CI updates**: Todd Zullinger asked whether `macos-latest` would be a more maintainable choice than `macos-26` for jobs where version pinning is not critical. Junio C Hamano posted v2 of the macOS CI update, splitting the jobs between `macos-15` and `macos-26` as Patrick Steinhardt suggested.

---