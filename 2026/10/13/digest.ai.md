# Git mailing list daily digest for 2026/10/13

## The day in brief
The Git mailing list saw significant activity around platform compatibility fixes for macOS, performance optimizations in the object database, and usability improvements in commands like `git blame` and `git checkout`. Key developments include Junio's scrutiny of a CRLF conversion fix for macOS, Patrick Steinhardt's ODB alternates refactoring series receiving positive reviews, and the `git blame` default ignore-revs feature being approved for `next`. Several Outreachy microprojects were also submitted, focusing on i18n improvements and test modernization.

## Notable threads

### macOS CRLF conversion fix faces maintainer scrutiny
**Thread root:** 2026/01/11/19-51-49

Curtis Allen Smith's patch series addressing a CRLF conversion bug in macOS environments continues to evolve, with Junio C Hamano raising concerns about the patch's interaction with Git's racy Git logic and potential disruption of other performance trade-offs. The series, now at v5, fixes a long-standing issue where Git incorrectly reports modifications when external tools rewrite files with CRLF line endings after checkout.

The core debate remains unresolved: Johannes Sixt argues the fix should target `git add` to ensure the index records the correct post-conversion size, while Curtis maintains the fix belongs in index refresh operations. Junio's latest review signals deeper scrutiny is needed before the patch can progress.

**Key technical details:**
- Affects commands that refresh the index (`git status`, `git rebase`, `git pull --rebase`)
- Introduces `ce_modified_check_fs()` to verify size changes when conversion is active
- Performance impact: 12 microseconds per modified 2.6 KB file in realistic scenarios
- 45 new test lines in `t/t0020-crlf.sh` covering CRLF rewrites and lone-CR edge cases

### ODB alternates refactoring series receives positive reviews
**Thread root:** 2026/10/13/07-36-24

Patrick Steinhardt's five-patch series refactoring and optimizing alternates handling in the Git object database's "files" backend received substantive reviews from Justin Tobler. The series addresses performance inefficiencies in reading and streaming objects when alternates are present, with the first patch achieving a 2x speedup in object lookup.

Justin's reviews confirmed the technical rationale and implementation for all five patches, endorsing the design decision to handle optimizations within the "files" backend rather than introducing new ODB flags. While minor questions were raised about error handling and abstraction boundaries, the series appears ready to progress.

**Key technical details:**
- Reorders loops to prioritize packfile checks over loose-object `stat()` calls
- Moves second object read logic into the "files" backend
- Removes redundant read loop in `do_oid_object_info_extended()`
- Drops the `local` flag from the generic ODB source struct
- Net reduction of 42 lines across 12 files

### `git blame` default ignore-revs feature approved for `next`
**Thread root:** 2026/09/11/23-29-44

Ravi Mistry's patch series making `git blame` automatically use `.git-blame-ignore-revs` as the default ignore-revs file when no explicit file is configured has been approved for the `next` branch. The series, now at v6, addresses all prior feedback and aligns local behavior with hosting platforms like GitHub and GitLab.

Junio praised the v6 implementation as "very cleanly done," particularly the removal of a "cute hack" in favor of clearer code using `.util = (void *)1` with an explanatory comment. The series includes security hardening to handle upstream-controlled content safely, with NUL-byte rejection, promisor fetch prevention, and a 100MB size cap.

**Key technical details:**
- Reads working-tree file in non-bare repositories (matching `.mailmap` behavior)
- Falls back to `HEAD:.git-blame-ignore-revs` in bare repositories
- User-configured ignore files take precedence over the default
- 233 new test lines in `t/t8013-blame-ignore-revs.sh`

### `rerere` race condition fix queued for integration
**Thread root:** 2026/09/02/08-31-37

Thomas Bachem's two-patch series fixing a race condition in the `rerere` subsystem has been queued for integration in the `next` branch. The series introduces a configurable timeout (`rerere.lockTimeout`) for non-maintenance callers and a `--skip-locked` mode for `git rerere gc` that skips silently when the lock is held.

The v8 series incorporates all remaining feedback, including Junio's request to remove redundant documentation about lock behavior and Patrick Steinhardt's suggestion to revert to a shared `RERERE_NOWAIT` flag instead of introducing a dedicated enum. The series addresses a long-standing locking flaw exacerbated by the geometric maintenance strategy introduced in Git 2.54.

**Key technical details:**
- Configurable timeout (`rerere.lockTimeout`, default 1000 ms)
- `--skip-locked` mode for `git rerere gc` (hidden/undocumented)
- Test coverage in `t/t4200-rerere.sh` and `t7900-maintenance.sh`
- Conservative design: fails outright if the lock remains held after timeout

### Incremental connectivity check series faces design questions
**Thread root:** 2026/09/14/09-47-54

Kristofer Karlsson's series introducing an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) continues to face design questions. The feature aims to reduce the cost of verifying object connectivity during fetches and pushes by processing incoming commits in topological order.

Junio has signaled strong interest in the feature but raised questions about a potential dynamic fallback mechanism for very long incoming histories. SZEDER Gábor confirmed a performance regression in `git log --follow`, showing a 14× slowdown in the Linux kernel repository, which may inform the connectivity check design.

**Key technical details:**
- New config key: `transfer.connectivityCheck=incremental`
- New internal `rev-list` flag: `--verify-trees-incremental`
- 316-line implementation in `tree-verify.c`
- Up to 190x speedup for small pushes to large repos

## In brief

- **`git fast-import` crash fix:** Marc Becker's patch fixing a crash when resolving unreachable in-session commits received maintainer feedback requesting commit message clarifications and test hygiene improvements. [2026/10/08/10-01-18]

- **`git repo structure` revision filtering:** Mark C. Chu-Carroll's patch adding revision-based filtering to `git repo structure` continues to face questions about revision-parsing defaults and test coverage for trees/blobs. [2026/09/28/23-43-49]

- **`includeIf.hostname` implementation:** Isabella Caselli's series implementing `includeIf.hostname:<pattern>` for Git's configuration system is ready for final review, with all prior feedback addressed. [2026/09/11/23-29-44]

- **`git mergeconflicts` documentation:** Julia Evans's documentation series introducing `gitmergeconflicts(7)` received polish-only feedback from Junio, who provided a squashable patch to revise the `diff3` example. [2026/09/24/16-45-02]

- **`git repack` regression fix:** Siddharth Shrimali's patch fixing a regression in `git repack --drop-filtered --dry-run` is queued for integration, with Patrick Steinhardt noting a potential follow-up cleanup. [2026/09/28/23-43-49]

- **`git log --follow` performance regression:** Patrick Steinhardt forwarded a user report of a severe slowdown in `git log --stat --follow`, bisected to commit 304812ed33. SZEDER Gábor confirmed the regression and isolated the bottleneck. [2026/10/13/13-55-01]

- **Outreachy microprojects:** Several Outreachy applicants submitted microprojects, including pluralization fixes for `git describe` and `delta-islands.c`, and test modernization in `t0081-find-pack.sh` and `t2030-unresolve-info.sh`. [2026/10/13/14-25-29, 2026/10/13/16-44-19, 2026/10/13/10-37-16, 2026/10/13/15-04-50]

- **Index lock timeout feature:** Jun Wu's series adding a configurable timeout (`core.indexLockTimeout`) for Git's index lock received substantive feedback from D. Ben Knoble and Junio, focusing on the performance rationale and race-avoidance logic. [2026/10/13/17-23-13]

- **`git log --graph` rendering bug:** herr.kaste reported a rendering error in `git log --graph --all --oneline` where an orphaned commit is drawn with a spurious parent connection. [2026/10/13/19-00-00]

- **`git log` completion:** Mirko Faina added shell completion support for `--graph-indent` and `--no-graph-indent` to `git log`. [2026/10/13/19-50-49]

- **`git fetch` regression fix:** Rong Bao posted a patch fixing a regression in `git fetch --all --depth=<n>` and `git fetch --all --unshallow` where the flags are not forwarded to fetcher subprocesses. [2026/10/13/14-44-39]

- **`git pull --rebase` and custom refspecs:** jyotish kumar raised a concern about `git pull --rebase` miscomputing the fork-point when custom fetch refspecs are configured. [2026/10/13/16-01-09]

- **CI housekeeping updates:** Patrick Steinhardt's CI housekeeping series continues to face a naming dispute for the `linux-TEST-vars` job, with Junio proposing 14 alternative names. [2026/10/07/14-52-01]

- **fsmonitor bug report:** Erik Cervin-Edin reported a bug in the Linux fsmonitor daemon where it fails to track edits in renamed directories. [2026/10/13/13-39-28]