# Git mailing list daily digest for 2026/10/17

## The day in brief
The Git mailing list saw significant activity around performance fixes, refactoring efforts, and design discussions. A critical performance regression in shallow fetches over protocol v2 was identified and fixed, while the ongoing `the_repository` removal effort advanced with several patches. Design debates continued around branch management features, particularly `git branch --delete-merged` and its handling of squash merges and gone upstreams. Several Outreachy applicants introduced themselves, seeking microprojects in the parse-options subsystem.

## Notable threads

### Performance fix for shallow fetches over protocol v2
**[2026/10/17/22-11-46 by Harald Nordgren]**
Harald Nordgren posted a patch to fix a severe performance regression in `git fetch` from shallow repositories over protocol v2. The issue causes fetches to take minutes instead of seconds due to inefficient history traversal when advertising shallow commits. The fix aligns protocol v2 behavior with protocol v0 by directly advertising shallow grafts when no specific depth is requested, avoiding redundant traversal. Harald reports the fix reduces fetch time from 7 minutes to 1 second in a repository with 900k commits. The patch touches `shallow.c` and `upload-pack.c` and is marked as RFC, seeking feedback on correctness.

This fix addresses a real-world performance problem that likely affects users of large, shallow repositories. The minimal diff (4 insertions, 4 deletions) suggests a targeted solution, but the RFC status indicates the author wants to ensure the approach doesn't introduce edge cases in other shallow-fetch scenarios.

### `git branch --delete-merged`: squash merge detection debate
**[2026/10/17/22-23-11 by Harald Nordgren]**
Harald Nordgren pushed back against Junio C Hamano's proposal to split `git branch --delete-merged` into two distinct options: a strict `--delete-merged` (tip-reachability only) and a looser `--delete-irrelevant` (change-based detection for rebases, squash merges, and cherry-picks). Harald argues that once a branch's changes are upstream—regardless of integration method—the branch is no longer needed, and users can recover the work from the mainline if necessary. This pragmatic stance prioritizes workflow efficiency over the safety concerns Junio raised about ambiguous cases (e.g., independently solved problems).

The debate highlights a fundamental tension in Git's design philosophy: should commands prioritize strict correctness and predictability, or should they adapt to real-world workflows where integration methods vary? The discussion remains unresolved, with no clear consensus on whether to adopt Junio's proposed CLI split or proceed with Harald's unified approach.

### Configurable index lock timeout for IDE workflows
**[2026/10/17/07-32-27 by Jun Wu]**
Jun Wu posted v2 of a series introducing a configurable timeout (`core.indexLockTimeout`) for Git's index lock, addressing lock contention between background IDE `git status` calls and user-initiated commands. The v2 series expands the scope to fix pre-existing write-race conditions in `git commit` and `git reset`, which could silently discard concurrent index updates. The series now includes four patches: a mechanical refactoring, the core timeout logic, and race fixes for `commit` and `reset`.

The default timeout (1 second) remains open to debate, but the implementation is conservative, skipping the timeout for commands that read the index before locking to avoid amplifying race windows. The series is well-structured and addresses prior feedback, making it a strong candidate for integration. Users frustrated by lock contention in IDE workflows should find this series particularly valuable.

### `git repo structure`: SHA-256 compatibility issue in tests
**[2026/10/17/03-59-41 by Junio C Hamano]**
Junio C Hamano identified a SHA-256 compatibility issue in the new test for the `git repo structure` revision-filtering feature. The test hardcodes 40-character SHA-1 object IDs and will fail when Git defaults to SHA-256 in version 3.0. Junio suggests either guarding the test with a `SHA1` prerequisite or rewriting it to be hash-agnostic, citing a recent example (`pull.2252.v4`) as inspiration. The patch is otherwise ready for merging, with Patrick Steinhardt's sign-off confirming it is procedurally clean.

This issue underscores the importance of forward-compatible testing as Git transitions to SHA-256. The author's response will determine whether the fix is a simple prerequisite addition or a more involved hash-agnostic rewrite. The patch's readiness for integration depends on resolving this test compatibility issue.

### `git worktree repair`: v3 series with robust path handling
**[2026/10/17/12-54-57 by Yoichi NAKAYAMA via GitGitGadget]**
Yoichi NAKAYAMA posted v3 of a five-patch series fixing a bug in `git worktree repair` where the command could incorrectly modify `.git` files or gitdirs belonging to unrelated worktrees. The v3 update addresses Junio C Hamano's review feedback by hardening the `get_worktree_id()` helper to handle non-canonical path separators (backslashes, duplicate slashes) in `.git` file contents. The series now includes test cases for `.git` files with extra separators and Windows-style backslashes, ensuring robust validation across platforms.

The series is narrowly scoped to `worktree.c` and the associated test script, with no changes to on-disk formats or user-visible behavior beyond the intended bugfix. The refactoring in patches 1–4 improves code clarity and reduces redundant file I/O, while patch 5/5 introduces the validation logic that prevents the original issue. The series is now ready for broader review or integration, though it has not yet been queued in `seen` or `next`.

### `git imap-send`: OpenSSL compatibility and correctness fixes
**[2026/10/17/10-09-54 by Beat Bolli]**
Beat Bolli posted v3 of a three-patch series addressing OpenSSL compatibility and correctness in `git imap-send`. The series replaces deprecated APIs (`SSLv23_method()` → `TLS_method()`) and rewrites manual hostname verification to use OpenSSL's built-in `SSL_set1_host()` API. However, patch 2/3 introduces two critical regressions: a **security regression** (certificate stripping) and an **error reporting regression** (loss of hostname-specific error messages). The series remains blocked until these issues are resolved.

The v3 series includes a new patch (3/3) implementing Xiangfei Ding's proposed verification callback to restore lost error context, but it does not address the security regression. Patrick Steinhardt's review identified both regressions and highlighted the lack of test coverage for this security-sensitive code path. The series is not ready for merging in its current form, and the next iteration will need to restore the `SSL_get_peer_certificate()` check and ensure error messages remain informative.

### Outreachy applicants introduce themselves
**[2026/10/17/09-34-59 by Muritala Mudi]**
Several Outreachy applicants introduced themselves, seeking microprojects in the parse-options subsystem. Muritala Mudi completed initial onboarding steps (building Git, running `parse-options` tests, following *MyFirstContribution*) and is now seeking guidance on a microproject. The parse-options subsystem is a common target for Outreachy contributions, as it offers well-scoped, mechanical refactoring opportunities that help new contributors become familiar with Git's codebase and contribution process.

These introductions signal the start of a new contributor engagement cycle, with established contributors likely to follow up with microproject suggestions and mentorship.

## In brief
- **[2026/10/17/16-43-05 by Kristoffer Haugsbakk]** Confirmed his `git symbolic-ref refs/H HEAD` workaround meets his needs and does not oppose stricter validation of `@` as a branch name.
- **[2026/10/17/04-10-04 by Junio C Hamano]** Rejected Simon Richter's `off_t` suggestion for `http.c`'s `posn` variable, reinforcing `size_t` on memory-safety grounds.
- **[2026/10/17/16-34-13 by Kristoffer Haugsbakk]** Clarified that `git diff-tree -z` uses separators, not terminators, making it unsuitable for incremental parsing in scripts.
- **[2026/10/17/00-35-25 by DonQuinleone]** Proposed `--into <ref>` modifier for `git branch --delete-merged` to handle branches with gone upstreams.
- **[2026/10/17/06-40-39 by Ammaar Bakshi via GitGitGadget]** Fixed missing `_()` wrapper for `git difftool --dir-diff` error message, enabling translation.
- **[2026/10/17/10-52-43 by Okhuomon Ajayi]** Refactored `merge-blobs.c` to stop using `the_repository`, advancing the ongoing removal effort.
- **[2026/10/17/15-17-02 by Gingrspacecadet]** Reported a crash in `git apply --check` with assertion `patch->is_new <= 0` when processing a new-file patch.
- **[2026/10/17/22-53-26 by Okhuomon Ajayi]** Refactored `checkout.c` to pass repository parameter in `unique_tracking_name()`, advancing the `the_repository` removal effort.