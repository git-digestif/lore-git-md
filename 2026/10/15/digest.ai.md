# Git mailing list daily digest for 2026/10/15

## The day in brief
The Git mailing list saw active discussion across multiple fronts today. A performance regression in `git log --follow` received a proposed fix and architectural review, while CI housekeeping updates were marked ready for integration. The `includeIf.hostname` feature gained maintainer approval, and several Outreachy contributions advanced test modernization efforts. Documentation improvements and bugfixes rounded out the day's activity.

## Notable threads

### Performance regression in `git log --follow`
Miklos Vajna posted a bugfix patch addressing a severe performance regression in `git log --stat --follow` introduced by commit 304813ed33. The patch optimizes `propagate_follow_pathspec_to_parent()` by only performing rename detection when the path is added relative to a parent, reducing runtime from 6m43s to 29s in the Linux kernel repository. Junio C Hamano proposed an alternative architectural approach, suggesting the optimization logic be moved into `log_tree_diff_flush()` to avoid redundant diff passes. The discussion remains open on the best path forward.

**What changed:** The patch introduces two shortcuts in the follow-path logic, avoiding expensive rename detection when unnecessary.
**Problem:** The regression made `git log --follow` unusably slow in repositories with complex merge structures.
**Subsystem:** Revision walking and rename detection.
**Impact:** Significant performance improvement for users tracing file history across merges.

### CI housekeeping updates ready for integration
Junio C Hamano marked Patrick Steinhardt's eight-patch CI housekeeping series ready for inclusion in `next`. The series modernizes Git's CI configurations by removing outdated dependencies (Python 2, EOL Docker images), eliminating redundant jobs, and improving test coverage. Key changes include updating the 32-bit CI job to use `i386/debian:latest`, renaming the `linux-TEST-vars` job to `linux-feature-testbed`, and repurposing the `linux-reftable-leaks` job to test SHA-256 + reftable combinations. Jeff King's review confirmed all technical concerns were addressed.

**What changed:** CI configurations were updated to use supported images and remove obsolete dependencies.
**Problem:** Outdated CI configurations were causing maintenance burdens and potential reliability issues.
**Subsystem:** CI infrastructure (GitHub Actions, GitLab CI).
**Impact:** Reduced CI complexity while maintaining or improving test coverage.

### `includeIf.hostname` feature gains maintainer approval
Junio C Hamano reviewed and approved the first patch in Isabella Caselli's three-patch series implementing `includeIf.hostname:<pattern>` in Git's configuration system. The series enables users to conditionally include configuration files based on the machine's hostname, addressing the use case of dotfile sharing across multiple machines. Junio described the implementation as "nicely written" and "sensible," confirming it's ready for integration. The series builds on Ignacio Encinas' 2024 work with explicit permission.

**What changed:** Added `git var GIT_HOSTNAME` and implemented `includeIf.hostname:<pattern>` condition.
**Problem:** Users couldn't conditionally include configuration based on hostname.
**Subsystem:** Configuration system.
**Impact:** Improved workflow for users sharing dotfiles across multiple machines.

### Incremental connectivity check series advances
Patrick Steinhardt and Kristofer Karlsson continued their discussion on the incremental connectivity check feature for fetch/push. The series introduces an opt-in mode (`transfer.connectivityCheck=incremental`) that reduces the cost of verifying object connectivity during fetches and pushes. Today's discussion focused on Patrick's review feedback, including type usage improvements, parameter naming, and error handling. The authors agreed to implement a configurable commit-count threshold to dynamically switch between incremental and full checks for very long incoming histories.

**What changed:** The series implements incremental connectivity checking for fetch/push operations.
**Problem:** Full connectivity checks scale poorly with repository size.
**Subsystem:** Object database and connectivity checking.
**Impact:** Up to 190x speedup for small pushes to large repositories.

## In brief
- **`git repo structure` revision filtering:** Mark C. Chu-Carroll's v7 patch adding revision-based filtering to `git repo structure` was marked ready for merging after addressing all feedback.
- **Detached HEAD documentation:** Julia Evans posted v3 of a documentation rewrite for detached HEAD state, with Junio suggesting further refinements to the explanation.
- **Reflog usability improvements:** Harald Nordgren's series adding `--operations` and `--historical-branch` options to `git reflog` sparked discussion about scope and design trade-offs.
- **Loose-ref iterator bugfix:** Venkat Ragavan posted a patch fixing a correctness bug in `cache_ref_iterator_seek()` where loose-ref iteration could misposition when directory names share prefixes with ref names.
- **`git format-rev` deprecation:** Kristoffer Haugsbakk proposed deprecating the experimental `git format-rev` command, noting that `git diff-tree` provides similar functionality but lacks NUL termination support.
- **`git help user-manual` bug:** John Wilson reported a bug where `git help user-manual` fails on Git for Windows due to incorrect filename lookup.
- **Outreachy contributions:** Several Outreachy applicants posted test modernization patches, with Shashank S's patch addressing feedback and being marked ready for merging.
- **Cache-tree crash fix:** Szymon Wilczek posted a bugfix preventing `git rebase` from crashing when another process holds the index lock during an in-process commit.
- **Integer-overflow fixes:** Johannes Schindelin's Coverity-inspired integer-overflow fixes received review feedback from Phillip Wood, focusing on macro implementation details.
- **`the_repository` removal:** Okhuomon Ajayi posted a refactoring patch advancing the ongoing `the_repository` removal effort by updating functions in `merge.c` to use explicit repository parameters.