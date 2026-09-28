# Git Mailing List Digest: 2026/09/21 -- 2026/09/27

## The period in brief
This week (2026/09/21--2026/09/27) saw **high-volume, technically dense traffic** across Git's core subsystems. The list was unusually active for a non-release week, with **six days of substantive discussion** and no quiet periods. Key developments included a **critical regression fix** for the reference-transaction hook, **major architectural progress** on the `strbuf` safety API, and **significant CI/build system improvements**. The Git 3.0 timeline discussion continued, with new data on Rust portability and JGit's production readiness. Readers who missed the daily digests should focus on the **reference-transaction hook fix**, the **`strbuf` safety API RFC**, and the **`git stash` autostash bugfix** -- all of which reached critical milestones this week.

---

## Key developments

### Reference-transaction hook regression fix reaches v6
The six-iteration series fixing a **long-standing regression** in the reference-transaction hook reached a major milestone with v6. The bug, introduced in Git 2.31, caused the hook to receive all-zero OIDs when branches or tags were deleted, breaking external tools like Gerrit and GitLab that rely on the hook for ref monitoring. The v6 series abandons the v5 approach of modifying `refs_delete_refs()` and instead moves the fix into the **transaction layer itself**, recording a separate "observed old value" for updates where callers didn't supply one. This ensures the hook always receives meaningful old OIDs while preserving the performance gains of the original optimization. The series includes updated regression tests and is now **architecturally aligned with Patrick Steinhardt's feedback**, though the on-demand resolution of old OIDs may prompt further discussion about latency in large batches.

### `strbuf` safety API RFC receives substantive feedback
Derrick Stolee's RFC series introducing a "safe" `strbuf` API subset that cannot call `die()` received **detailed architectural feedback** from Junio C Hamano and Phillip Wood. The review identified **two critical bugs** (a memory leak in `srealloc()` and a misleading error message in `safe_memory_limit_check()`) and questioned the framing of the API as a "safe" subset rather than a hardened version of the entire `strbuf` API. Junio proposed that higher-level utilities like `strbuf_realpath()` should avoid the `strbuf_` prefix to prevent confusion, while Jeff King suggested an alternative design using **fixed-size, stack-allocated buffers** to avoid `malloc()` entirely. The discussion has shifted from implementation details to **fundamental design questions** about how to best serve libification and trace2 use cases, with no clear resolution yet.

### `git stash` autostash bugfix series finalized
A **critical bug** in `git stash` autostash handling of staged index entries was fixed in a five-patch series that reached v3. The bug, which could corrupt staged entries during autostash operations when `stash.index=true` was set, was caused by a race condition in the subprocess-based index merging logic. The fix replaces this with **in-core merging via `merge-ort.c`**, eliminating the race condition and ensuring staged entries are preserved. The series addresses all prior feedback, including a **tree argument order fix** in `merge_incore_nonrecursive()` (flagged by Junio) and a **memory leak fix** (flagged by Phillip Wood). The patches are now **technically complete and ready for integration**, though a **verbosity regression** in the final patch (identified by Junio) may require a minor follow-up.

### `gitmergeconflicts(7)`: A new man page for conflict resolution
Julia Evans posted a **seven-patch series** introducing `gitmergeconflicts(7)`, a new man page consolidating merge conflict resolution guidance. The current documentation scatters advice across `git merge`, `git rebase`, `git revert`, and other command man pages, leading to duplication and user confusion. The new guide centralizes this information, clarifies terminology like "ours" vs. "theirs," and provides **practical, example-driven workflows**. The series updates existing man pages to link to the new guide and includes build system integration. Junio requested **mechanical adjustments** (e.g., updating `command-list.txt`) but has already merged the series to `seen` for integration testing. The only open question is whether **cherry-pick-specific details** should be retained in the cherry-pick man page alongside the cross-reference.

### CI and build system improvements land
Several **CI and build system improvements** landed this week, addressing long-standing pain points:
- **Windows unit tests**: Karthik Nayak fixed a regression where unit tests were being skipped on Windows CI due to a mismatch in test-slice indexing.
- **Meson build optimizations**: Patrick Steinhardt posted a seven-patch series reducing clean build times by ~35% (from ~6.8s to ~5.0s) by avoiding redundant HTTP source compilation and using precompiled headers for test helpers.
- **Rust compatibility**: Ramsay Jones reported that Git's experimental Rust support compiles successfully on Cygwin using an unofficial toolchain, with no new test failures. This provides the first public data point on Rust's viability outside mainstream platforms.
- **Leak sanitizer reporting**: Harald Nordgren improved leak sanitizer failure reporting in GitHub Actions by stopping tests at the first failure and emitting detailed annotations.

---

## In brief

**`reference-transaction` hook backward compatibility** -- The v2 series fixing a regression where branch/tag deletions reported all-zero OIDs instead of the resolved old OID hit a **critical design flaw**: the patch used a single transaction for all deletions, turning the operation into an all-or-nothing affair that contradicts `refs_delete_refs()`’s documented "best-effort" behavior. The v3 series will require significant rework to handle partial failures gracefully.

**Windows ANSI emulation bugfix** -- Yongqiang Tian’s four-iteration bugfix for `compat/winansi.c:die_lasterr()` is ready for `next`. The patch removes a problematic variadic helper and replaces its call sites with direct `die()` calls that report the raw `GetLastError()` value, addressing a long-standing bug where Windows API failure messages displayed a `va_list` address instead of the intended integer handle.

**`git log -L` and regex-based function detection** -- A bug report about `git log -L` incorrectly including commits that modify trailing blank lines after Python functions sparked a **design discussion** about Git’s regex-based function boundary detection. Johannes Sixt clarified that the behavior is not a bug but a deliberate trade-off: Git’s line-log parser relies on the same heuristic patterns as diff hunk headers, which cannot distinguish between code, whitespace, or comments.

**`git pull` segfault fix** -- Jiri Kuncar posted v2 of a patch preventing a segmentation fault when `lookup_commit_reference()` returns NULL due to corrupted merge heads. The patch adds NULL guards and includes a test simulating corruption via garbage-written object files.

**OpenBSD `RUNTIME_PREFIX` support** -- Brad Smith clarified that `getexecpath()` is a new, more accurate API for resolving executable paths on OpenBSD, replacing a "pile of hacks." Junio requested the commit message incorporate this rationale and clarify the OpenBSD 8.0+ version gating.

**`git reflog expire` regression fix** -- A patch restoring the correct default expiry periods for `git reflog expire` (reachable=90d, unreachable=30d) was approved and is ready for `next`. The regression, introduced in Git 2.50, broke external tools that rely on the documented expiry behavior.

**`git-p4` security fix** -- Anupam Mediratta posted a patch replacing a shell-based pipeline with direct subprocess calls to eliminate a shell injection vulnerability in the `--commit` option. The v2 patch restores the original error-handling behavior while preserving the shell injection mitigation.

**`git repo structure` filtering options** -- Mark C. Chu-Carroll posted a patch adding `--no-<reftype>` filtering options to `git repo structure`, allowing users to exclude specific reference types (e.g., `--no-tags`, `--no-branches`) and their transitive dependencies from the report.

**`git var` extension series** -- Andrew Pleeter posted v9 of the series implementing a unified, scriptable interface for Git’s identity and signing configuration. The controversial `GIT_SIGNING_KEY` variable was removed due to ambiguity across signing mechanisms, and the series now focuses on six identity variables (`GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL`, `GIT_AUTHOR_DATE`, `GIT_COMMITTER_NAME`, `GIT_COMMITTER_EMAIL`, `GIT_COMMITTER_DATE`).

**`git fetch-pack` design clarification** -- Junio C Hamano clarified that `git fetch-pack` should remain a low-level command without DWIM heuristics, preserving the ability to send `want-ref` requests for arbitrary ref names.

**`git shortlog` regression fix** -- Jeff King (Peff) posted a two-patch series fixing a regression where unknown options were incorrectly reported as `(null)` instead of their actual name.

**`git-interpret-trailers` documentation** -- Kristoffer Haugsbakk posted a patch improving clarity of `trailer.<key-alias>.cmd` examples in the `git-interpret-trailers` man page.

**Localization updates** -- Jiang Xin submitted l10n updates for Git 2.56.0, adding Afrikaans and Brazilian Portuguese translations and refreshing eight existing languages.

---

## Looking ahead
The next week is likely to see **continued progress on the `strbuf` safety API**, with Junio and Jeff King’s architectural feedback prompting a potential redesign. The **reference-transaction hook regression fix** may see further iterations if the on-demand resolution of old OIDs introduces unacceptable latency. The **Git 3.0 timeline discussion** will continue, with downstream projects (e.g., Git for Windows) likely to weigh in on the proposed version numbering and default changes. The **`git stash` autostash bugfix** is expected to land in `next`, though the verbosity regression may require a follow-up patch. Finally, the **`gitmergeconflicts(7)` man page** will likely see integration, with any remaining mechanical adjustments (e.g., `command-list.txt` updates) addressed in the coming days.