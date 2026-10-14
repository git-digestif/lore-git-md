# Git mailing list daily digest for 2026/10/13

## The day in brief
The Git mailing list saw significant activity around platform compatibility fixes for macOS, performance optimizations in the object database, and usability improvements in commands like `git blame` and `git checkout`. Key developments include a security regression identified in OpenSSL integration, progress on the `includeIf.hostname` feature, and a performance regression in `git log --follow` that sparked discussion about algorithmic complexity. Outreachy applicants continued to submit microproject patches, and several feature series moved closer to integration.

## Notable threads

### macOS CRLF conversion bugfix reaches v5
**Thread root**: 2026/01/11/19-51-49

Curtis Allen Smith's patch series addressing a CRLF conversion bug in `read-cache.c` reached its fifth iteration. The bug causes Git to incorrectly report modifications when external tools rewrite files with CRLF line endings after checkout. The v5 patch adds four lines to `ie_modified()` to call `would_convert_to_git()` when size changes occur, with thorough test coverage in `t/t0020-crlf.sh`.

Junio C Hamano raised concerns about the patch's interaction with Git's racy Git logic and potential disruption of other performance trade-offs. The architectural debate about whether the fix belongs in `git status` or `git add` remains unresolved, with Johannes Sixt arguing for the latter approach.

**Impact**: This fix affects all commands that refresh the index (e.g., `git status`, `git rebase`) and could significantly improve usability for macOS users working with cross-platform repositories.

### `rerere` race condition fix queued for `next`
**Thread root**: 2026/09/02/08-31-37

Thomas Bachem's v8 series fixing a race condition in the `rerere` subsystem is now queued for integration. The series introduces a configurable timeout (`rerere.lockTimeout`) for non-maintenance callers and a `--skip-locked` mode for `git rerere gc` that skips silently when the lock is held.

The patch addresses a long-standing issue where concurrent `rerere` operations contend for the shared `MERGE_RR.lock`, exacerbated by the geometric maintenance strategy introduced in Git 2.54. All prior feedback has been incorporated, including Patrick Steinhardt's suggestion to revert to a shared `RERERE_NOWAIT` flag.

**Impact**: This fix improves the reliability of `rerere` operations, particularly in automated workflows where background maintenance might conflict with user-initiated commands.

### OpenSSL security regression identified in `imap-send`
**Thread root**: 2026/09/07/21-12-07

Patrick Steinhardt identified a **security regression** in Beat Bolli's patch that replaces Git's manual hostname verification in `imap-send` with OpenSSL's `SSL_set1_host()`. The patch removes the `SSL_get_peer_certificate()` check, allowing an attacker to strip the certificate entirely and bypass hostname verification, creating a trivial MITM vector.

The review also raised concerns about the absence of tests for this security-sensitive code path. The patch cannot proceed without addressing this security hole and adding appropriate test coverage.

**Impact**: This regression affects users of `git imap-send` and could expose them to man-in-the-middle attacks if not fixed.

### `git blame` default ignore-revs feature approved for `next`
**Thread root**: 2026/09/11/23-29-44

Ravi Mistry's feature series making `git blame` automatically use `.git-blame-ignore-revs` (or `HEAD:.git-blame-ignore-revs` in bare repositories) when no explicit ignore file is configured has been approved for integration. The v6 revision addresses Junio's polish-only feedback, replacing a "cute hack" with a clearer implementation using `.util = (void *)1` and an explanatory comment.

The series includes security hardening (NUL-byte rejection, promisor fetch prevention, and a 100MB size cap) and thorough test coverage. The feature aligns local behavior with hosting platforms like GitHub and GitLab, reducing friction for users who rely on ignore-revs files.

**Impact**: This change makes `git blame` more convenient for users who maintain ignore-revs files, particularly in large projects with frequent refactoring.

### Incremental connectivity check series under review
**Thread root**: 2026/09/14/09-47-54

Kristofer Karlsson's series introducing an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) continues to receive feedback. The feature reduces the cost of verifying object connectivity during fetches and pushes by processing incoming commits in topological order and tracking trusted objects from parent commits.

SZEDER Gábor confirmed a performance regression in `git log --stat --follow` bisected to commit 304812ed33, which introduced the incremental connectivity check logic. The regression shows a 14× slowdown in the Linux kernel repository and a 2.7× slowdown in the Git repository itself, with the bottleneck identified as the new `propagate_follow_pathspec_to_parent()` logic.

**Impact**: This series could significantly improve performance for large repositories, but the identified regression needs to be addressed before it can proceed.

### `includeIf.hostname` feature ready for final review
**Thread root**: 2026/09/28/23-43-49

Isabella Caselli's implementation of `includeIf.hostname:<pattern>` for Git's configuration system is ready for final review. The series extends the existing `includeIf` mechanism with hostname-based conditions and includes a `git var GIT_HOSTNAME` debugging aid.

Junio C Hamano described the implementation approach as "neat" and "quite nice," with no substantive objections. The only remaining question is whether hostname matching should be case-sensitive, case-insensitive, or support a `hostname/i:` variant.

**Impact**: This feature enables users to conditionally include configuration files based on the machine's hostname, addressing the use case of dotfile sharing across multiple machines.

### ODB alternates refactoring and optimization series posted
**Thread root**: 2026/10/13/07-36-24

Patrick Steinhardt posted a five-patch series refactoring and optimizing the alternates mechanism in the Git object database's "files" backend. The series addresses performance inefficiencies in reading and streaming objects when alternates are present, then performs internal cleanup to simplify the ODB source API.

Justin Tobler provided substantive reviews, confirming the technical rationale and implementation for all five patches. He raised a design-level concern about the `ODB_FOR_EACH_OBJECT_LOCAL_ONLY` flag in the final patch, questioning whether all call sites have legitimate "files" backend-specific reasons.

**Impact**: This series improves performance when working with alternates and simplifies the ODB internals, continuing the broader ODB abstraction effort.

## In brief

- **`git log --follow` performance regression**: SZEDER Gábor confirmed a 14× slowdown in the Linux kernel repository and a 2.7× slowdown in the Git repository, bisected to commit 304812ed33. The bottleneck is in `propagate_follow_pathspec_to_parent()`.
- **`git checkout -m` conflict label preservation**: Phillip Wood's series to make `git checkout -m` recreate conflict markers using original branch names is queued in `next`. The series includes performance optimizations and thorough test coverage.
- **`git fetch` shallow regression**: Rong Bao posted a patch fixing a regression in Git 2.56.0 where `git fetch --all --depth=<n>` or `git fetch --all --unshallow` fails to deepen the history when multiple remotes are configured.
- **Outreachy microprojects**: Several applicants submitted patches for test modernization, i18n improvements, and completion updates, including pluralization fixes for `git describe` and `delta-islands`.
- **CI housekeeping**: Patrick Steinhardt's series modernizing Git's CI configurations continues to receive feedback, with a naming dispute over the `linux-TEST-vars` job remaining unresolved.
- **`git fast-import` crash fix**: Marc Becker's patch fixing a crash when resolving commits or tags created earlier in the same session received maintainer feedback on commit message clarity and test hygiene.
- **`git repo structure` revision filtering**: Mark C. Chu-Carroll's series adding revision-based filtering to `git repo structure` continues to receive feedback on revision-parsing defaults and test coverage.
- **`git mergeconflicts` documentation**: Julia Evans's series introducing a dedicated `gitmergeconflicts(7)` guide received final refinements and maintainer approval.
- **`git detached-head` documentation**: Julia Evans's rewrite of Git's explanation of detached HEAD state is ready for integration, with the new guide named `gitdetached-head(7)` for consistency.
- **`git repack` dry-run regression**: Patrick Steinhardt noted a potential code hygiene opportunity in Siddharth Shrimali's fix for `git repack --drop-filtered --dry-run`.
- **`git log --graph` rendering bug**: herr.kaste reported a rendering error in `git log --graph --all --oneline` where an orphaned commit is drawn with a spurious parent connection.
- **`git log` completion update**: Mirko Faina added shell completion support for `--graph-indent` and `--no-graph-indent` to `git log`.
- **Index lock timeout feature**: Jun Wu's series adding a configurable timeout for Git's index lock received feedback on race-avoidance logic and the user-hostility of the new config knob.