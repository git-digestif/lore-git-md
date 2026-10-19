# The Git Project -- Weekly Digest for 2026/10/12 -- 2026/10/18

## The period in brief

The week of 2026/10/12--2026/10/18 saw **moderate to heavy traffic** (six active days) with a **mix of performance fixes, architectural improvements, and usability refinements**. The most consequential developments were the finalization of the `--force-if-includes` bugfix series, a severe performance regression identified in `git log --follow`, and significant progress on the incremental connectivity check for fetch/push. Other notable threads included the `git blame` default ignore-revs feature landing in `next`, architectural debates about CRLF conversion handling, and ongoing design discussions for `git branch --delete-merged`.

## Key developments

### `--force-if-includes` bugfix series finalized
Tyler Cipriani posted **v7 of a three-patch series** fixing critical bugs in the `--force-if-includes` safety mechanism for `git push`. The series addresses two long-standing issues: (1) the feature incorrectly checks the reflog of the *local* branch whose name matches the *remote* destination (e.g., `refs/heads/main` instead of `refs/heads/src` in `src:main`), causing both false rejections and data loss risks, and (2) it unnecessarily rejects fast-forward pushes of non-branch refs (tags, detached HEADs). The v7 series is technically complete, with all prior feedback resolved, and introduces new test coverage for edge cases like differently named branches, HEAD-based pushes, orphan branches, and tags. The series touches 12 files and introduces new symbols (`advice.pushRefUnverifiable`, `ref->unverifiable`), but the behavior changes are well-justified: force-pushes of non-branch refs are now rejected by default (with actionable advice), while fast-forward pushes of non-branch refs are allowed, fixing a long-standing bug. **Key participants**: Tyler Cipriani, Junio C Hamano. **Status**: Ready for merge; no unresolved technical concerns.

### Severe performance regression in `git log --follow`
A **14× slowdown** in `git log --stat --follow` was bisected to commit 304813ed33 ("log: improve --follow following renames for non-linear history"). Miklos Vajna proposed a fix that restores near-original performance (33 seconds → 8 minutes → 33 seconds) by optimizing `propagate_follow_pathspec_to_parent()` to skip redundant diffs. However, Junio C Hamano identified a **correctness issue**: the patch's `has_same` flag may incorrectly skip rename detection in merge commits where a file is semantically renamed in one parent but syntactically unchanged in the merge. The discussion shifted from performance to correctness, with no consensus on whether to prioritize speed or accuracy. The regression affects users tracing file history in large repositories (e.g., the Linux kernel). **Key participants**: Miklos Vajna, Junio C Hamano, SZEDER Gábor. **Status**: Under review; awaiting revised patch.

### Incremental connectivity check for fetch/push
Kristofer Karlsson's **four-patch series** introducing an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) saw significant review progress. The feature reduces the cost of verifying object connectivity during fetches and pushes by processing incoming commits in topological order and tracking trusted objects from parent commits. Benchmarks show up to **190× speedup** for small pushes to large repositories. Junio C Hamano signaled strong interest, while Patrick Steinhardt provided detailed feedback on implementation details (type usage, error handling, stack overflow risk). The series includes a new source file (`tree-verify.c`, 316 lines) and performance test suite (`t/perf/p5413-connectivity-check.sh`, 92 lines). **Key participants**: Kristofer Karlsson, Junio C Hamano, Patrick Steinhardt. **Status**: Under active review; no major objections.

### `git blame` default ignore-revs feature lands in `next`
Ravi Mistry's **feature series** making `git blame` automatically use `.git-blame-ignore-revs` (or `HEAD:.git-blame-ignore-revs` in bare repositories) when no explicit ignore file is configured was approved for integration. The v6 revision addresses Junio C Hamano's polish-only feedback and includes security hardening (NUL-byte rejection, promisor fetch prevention, 100MB size cap). The feature aligns local behavior with hosting platforms like GitHub and GitLab, reducing friction for users who rely on ignore-revs files. **Key participants**: Ravi Mistry, Junio C Hamano. **Status**: Queued in `next`; ready for `master`.

### CRLF conversion handling: architectural debate
The ongoing thread about Curtis Allen Smith's **v5 patch** for a CRLF conversion bug in `read-cache.c` saw renewed debate. The bug causes Git to incorrectly report modifications when external tools rewrite files with CRLF line endings after checkout. Torsten Bögershausen proposed relocating the `would_convert_to_git()` call from `ie_modified()` to `ie_match_stat()` to avoid interfering with Git's racy Git logic, but this reintroduces performance overhead in a broader set of commands. Johannes Sixt argued the fix belongs in `git add` rather than refresh operations, while Junio C Hamano raised concerns about performance trade-offs. The thread remains unresolved, with no clear path forward. **Key participants**: Curtis Allen Smith, Torsten Bögershausen, Johannes Sixt, Junio C Hamano. **Status**: Architectural debate ongoing; no consensus.

### `git branch --delete-merged`: design debate
Harald Nordgren's **proposal** to enhance `git branch --delete-merged` to detect squash merges and rebases sparked a design debate. Junio C Hamano proposed splitting the feature into two distinct options: `--delete-merged` (strict tip-reachability) and `--delete-irrelevant` (change-based detection). Harald argued for a unified approach, prioritizing workflow efficiency over safety concerns. A local experiment revealed that `git range-diff` is **20× slower** than tree-based diffing, complicating the design. Phillip Wood described workflows where the current behavior (doing nothing for non-tracking starting points) is preferable, undermining the patch's proposed behavior. **Key participants**: Harald Nordgren, Junio C Hamano, Phillip Wood, D. Ben Knoble. **Status**: Design debate ongoing; no consensus.

### Configurable index lock timeout for IDE workflows
Jun Wu posted **v2 of a four-patch series** introducing a configurable timeout (`core.indexLockTimeout`) for Git's index lock, addressing lock contention between background IDE `git status` calls and user-initiated commands. The v2 series expands the scope to fix pre-existing write-race conditions in `git commit` and `git reset`, which could silently discard concurrent index updates. The default timeout (1 second) remains open to debate, but the implementation is conservative, skipping the timeout for commands that read the index before locking. **Key participants**: Jun Wu, Jeff King, Junio C Hamano. **Status**: Under review; awaiting feedback on default timeout value.

## In brief

**`git push` refspec matching optimization** -- Jon Simons posted v2 of a 12-patch series replacing linear refspec traversals with `strmap` lookups, achieving 10–100× speedups for large-scale branch deletions (e.g., 100 delete refspecs to a server with 100k refs). The series addresses a real-world scalability bottleneck in CI/CD pipelines and large monorepos. **Status**: Under review.

**`gitmergeconflicts(7)` documentation** -- Julia Evans's series introducing a dedicated man page consolidating merge conflict resolution guidance was finalized and queued for integration. The guide centralizes advice previously scattered across multiple command man pages and improves clarity for new users. **Status**: Merged to `master`.

**CI infrastructure modernization** -- Patrick Steinhardt's eight-patch series modernizing Git's CI configurations landed in `next`. The series removes outdated dependencies (Python 2, EOL Docker images), eliminates redundant jobs, and improves test coverage for reftable and SHA-256 combinations. **Status**: Queued in `next`.

**`git worktree repair` bugfix** -- Yoichi NAKAYAMA posted v3 of a five-patch series fixing incorrect repairs of unrelated worktrees. The v3 update hardens the `get_worktree_id()` helper to handle non-canonical path separators (backslashes, duplicate slashes). **Status**: Under review.

**`git imap-send` OpenSSL compatibility** -- Beat Bolli posted v3 of a three-patch series addressing OpenSSL compatibility and correctness. The series introduces two critical regressions: a **security regression** (certificate stripping) and an **error reporting regression** (loss of hostname-specific error messages). **Status**: Blocked; awaiting fixes.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll's patch adding revision-based filtering to `git repo structure` was marked ready for `next` after addressing SHA-256 compatibility issues in tests. The feature improves diagnostic capabilities for large repositories. **Status**: Ready for `next`.

**`git detached-head(7)` documentation** -- Julia Evans's rewrite of Git's explanation of detached HEAD state was merged. The new guide (`gitdetached-head(7)`) improves clarity and moves the explanation to a standalone man page. **Status**: Merged to `master`.

**`git reflog` usability improvements** -- Harald Nordgren posted v2 of a three-patch series adding `--operations` and `--historical-branch` options to `git reflog` and `git log -g`. The series addresses long-standing pain points in reflog usability by collapsing multi-step operations into single entries and showing which branch HEAD was on when each entry was recorded. **Status**: Under review.

**`includeIf.hostname:<pattern>` implementation** -- Isabella Caselli posted v2 of a three-patch series implementing hostname-based conditional includes in Git's configuration system. The feature enables users to conditionally include configuration files based on the machine's hostname. **Status**: Under review.

**`git stash` shell completion** -- D. Ben Knoble posted a five-patch series improving `git stash` shell completion, including a Zsh-specific bug fix for stash name completion (e.g., `stash@{0}`) and the removal of completion support for the deprecated `git stash save` subcommand. **Status**: Under review.

**`fast-import` crash fix** -- Marc Becker posted v4 of a bugfix for `git fast-import` that addresses crashes when resolving commits created earlier in the same session but not yet written to the object database. The fix remains minimal, modifying only the `parse_from_existing` function. **Status**: Queued for `master`.

**Windows credential helper parsing fix** -- Marc Becker posted v3 of a bugfix for the Windows credential helper (`contrib/credential/wincred`) that fixes multi-line secret parsing. The patch addresses unsafe string operations that could cause buffer overreads and memory corruption. **Status**: Under review.

**`rerere` race condition fix** -- Thomas Bachem's v8 series fixing a race condition in the `rerere` subsystem is queued for integration. The series introduces a configurable timeout (`rerere.lockTimeout`) and a `--skip-locked` mode for `git rerere gc`. **Status**: Queued in `next`.

**Outreachy contributions** -- Several Outreachy applicants submitted patches for test modernization, i18n improvements, and completion updates. Contributions included pluralization fixes for `git describe`, test idiom updates (`test -f` → `test_path_is_file`), and parse-options subsystem refactoring. **Status**: Under review.

**`the_repository` removal** -- Okhuomon Ajayi and Shashank S posted patches advancing the ongoing `the_repository` removal effort by updating functions in `merge.c`, `checkout.c`, and `pack-objects.c` to accept explicit `struct repository *` parameters. **Status**: Under review.

## Looking ahead

The next week is likely to see continued progress on the following topics:

- **`git log --follow` performance regression**: Miklos Vajna's fix will need to address Junio C Hamano's correctness concerns, potentially requiring a revised approach that balances performance and accuracy.
- **`git branch --delete-merged` design**: The debate between Harald Nordgren and Junio C Hamano may lead to a CLI redesign or a hybrid approach that incorporates `git range-diff` logic as a fallback.
- **Incremental connectivity check**: Kristofer Karlsson's series is expected to receive further feedback on implementation details, with a possible v4 addressing stack overflow risks and error handling.
- **CRLF conversion handling**: The architectural debate may reach a resolution, with a possible compromise that relocates the conversion check while preserving performance.
- **`--force-if-includes` bugfix**: Tyler Cipriani's series is ready for merge and may land in `master` early in the next period.
- **Configurable index lock timeout**: Jun Wu's series may see further discussion on the default timeout value and additional race fixes for other commands.