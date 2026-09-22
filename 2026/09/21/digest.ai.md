# Git mailing list daily digest for 2026/09/21

## The day in brief
The Git mailing list saw active discussion on several fronts today. A critical design flaw was identified in a `reference-transaction` hook bugfix, requiring a rework of its transaction handling. The "safe" `strbuf` API RFC received substantive feedback, including bug reports and architectural suggestions. Windows platform compatibility fixes advanced, with one patch now ready for `next`. OpenBSD `RUNTIME_PREFIX` support and shallow-clone advice improvements also saw progress, while a governance documentation proposal sparked cautious discussion.

## Notable threads

### `reference-transaction` hook: critical design flaw in deletion fix
The v2 series addressing a regression where branch/tag deletions report all-zero OIDs instead of the resolved old OID hit a major roadblock. Junio C Hamano identified that the patch's use of a single transaction for all deletions turns the operation into an all-or-nothing affair, contradicting the documented "best-effort" behavior of `refs_delete_refs()`. This would break user expectations for commands like `git fetch --prune` or `git remote prune`, where users expect unconditional deletion of stale refs even if some refs are concurrently updated. The author has accepted the feedback and will adopt `REF_TRANSACTION_ALLOW_FAILURE` in v3 to preserve unconditional deletion while still passing old OIDs to the hook.

The thread also surfaced a secondary concern about implicit conversion of all-zero OIDs to `NULL` in `refs_delete_refs()`, which Karthik Nayak argued could mask bugs in callers. The v3 series will either remove this implicit conversion or explicitly document it.

### Introducing a "safe" strbuf API: bugs and architectural feedback
Derrick Stolee's RFC series introducing a "safe" `strbuf` API that cannot `die()` received substantive feedback from Junio C Hamano, identifying several critical issues:

1. A **user-facing bug** in `safe_memory_limit_check()` where the error message reports the original `git_alloc_limit` value (e.g., zero) instead of the effective limit (`SIZE_MAX`), creating a misleading mismatch.
2. A **memory leak bug** in `srealloc()`: when `realloc()` fails, the function overwrites `*ptr` with `NULL` before checking for failure, clobbering the caller's pointer and leaking the original memory.
3. A **design flaw** in `jw_release()`'s error aggregation logic, where Junio questioned whether `enum safe_result` should represent distinct error types or bitmasked flags, and suggested alternatives to the current `||` coalescing.

Junio also endorsed the direction of making core service routines like `strbuf` "safe" as part of libification, but questioned whether the current approach of carving out a niche "safe" subset is the right long-term strategy. He suggested the entire `strbuf` API should eventually be hardened, not duplicated, and that the series would have greater value if it aimed for that goal. Phillip Wood's earlier review had raised similar concerns about the "safe" framing and usability, proposing a sticky error bit to simplify error checking.

### Windows ANSI emulation: `die_lasterr()` formatting bugfix ready for `next`
Yongqiang Tian's bugfix for a long-standing formatting bug in `compat/winansi.c:die_lasterr()` is now effectively ready for integration. The patch removes the problematic `die_lasterr()` helper and replaces its four call sites with direct `die()` calls that report the raw `GetLastError()` value and the handle (formatted as `%li`). This preserves the exact Windows error code, avoids the `va_list` forwarding bug, and aligns with existing Windows-specific error reporting in Git.

Junio C Hamano's review of v2 requested only minor commit message improvements, and the v3 patch incorporated this feedback by adding `Helped-by:` trailers and moving build validation details below the separator. The only remaining discussion point is cosmetic (error message phrasing), but Junio does not consider it a blocker. The patch has resolved all prior technical objections and is a strong candidate for `next`.

### OpenBSD `RUNTIME_PREFIX` support: `getexecpath()` clarification
Brad Smith's patch to enable `RUNTIME_PREFIX` support on OpenBSD via `getexecpath()` saw follow-up discussion clarifying the motivation and scope. Brad explained that `getexecpath()` is a new, more accurate API for resolving executable paths on OpenBSD, replacing a "pile of hacks" that produced inconsistent results. The API is available starting with OpenBSD 8.0, which the patch already enforces via `config.mak.uname`.

Junio C Hamano acknowledged the rationale but noted that OpenBSD 8.0 is not yet released, raising questions about the API's availability timeline and whether the old BSD sysctl method is being deprecated. The patch itself remains technically unchanged, targeting OpenBSD 8.0+ with a new `git_get_exec_path_getexecpath()` wrapper in `exec_cmd.c` and corresponding `Makefile`/`config.mak.uname` updates.

### Shallow-clone advice: design pivot toward `git remote add` fix
The discussion around Harald Nordgren's `fetch.shallow` config proposal pivoted toward exploring an alternative solution: modifying `git remote add` to avoid wildcard refspecs in shallow repositories. Harald acknowledged that the performance issue he encountered is not tied to sparse checkouts but to shallow repositories with many remote-tracking branches, and agreed that improving `git remote add` behavior could be a more targeted solution.

The thread is now focused on whether the original `fetch.shallow` config is the best solution, or whether improving `git remote add` behavior in shallow repositories (e.g., avoiding wildcard refspecs) would be more maintainable. No new patch has been proposed yet, but the design direction has shifted.

### Governance documentation: cautious openness to descriptive write-up
Antonin Delpeuch's proposal to formally document Git's governance structure received a cautiously open response from Junio C Hamano. Junio emphasized that Git's governance is informal and emergent, with clout earned through sustained contribution rather than formal roles. He does not oppose a *descriptive* document outlining current practice but warns against prescriptive rules that could spark unproductive debate.

The discussion remains exploratory, with no decisions or next steps established. Antonin offered to draft a `GOVERNANCE.md` file, either solo or collaboratively, but Junio's response suggests the project is unlikely to adopt a formal, rule-based governance document.

## In brief
- **`git pull` segfault fix**: Jiri Kuncar posted v2 of a patch preventing a segmentation fault when an invalid merge head is encountered, addressing Junio C Hamano's feedback by simplifying the test setup and adding detailed comments.
- **Rust Windows CI**: Johannes Schindelin and Junio C Hamano confirmed that GitLab's Windows runners do not have Rust preinstalled, closing the last loose end from the merged Rust Windows CI series.
- **`git diff --no-index -R` fix**: Junio C Hamano queued Haokai Ding's patch fixing a regression where file/directory conflicts were incorrectly reported as deletions or additions when the `-R` flag was used.
- **`git stash` autostash fix**: Phillip Wood identified correctness risks in D. Ben Knoble's patch replacing subprocess-based index merging with in-core logic, including a merge algorithm mismatch and the removal of assertions guarding against NULL pointers.
- **`git log -L` trailing blank lines**: Kristofer Karlsson posted a patch to trim trailing empty lines from function ranges in `git log -L`, aligning its behavior with `git grep -W`, but Junio C Hamano identified a behavioral discrepancy with whitespace-only lines.
- **OpenBSD `RUNTIME_PREFIX`**: Chris Torek and Johannes Sixt discussed a minor structural issue in Brad Smith's patch, with Sixt clarifying that the `&&` chaining in `git_get_exec_path()` enforces mutual exclusivity at runtime.