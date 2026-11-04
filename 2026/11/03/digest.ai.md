Here is today's Git mailing list digest:

# Git mailing list daily digest for 2026/11/03

## The day in brief
The Git project made progress on several fronts today. A critical regression in promisor remote lazy-fetch was reported, threatening CI workflows for projects like Kubernetes. The `git repo info` path-keys series reached a design consensus on `/dev/null` handling but faces abandonment risk due to lack of reviewer engagement. Junio C Hamano formalized a strict intake policy for the `seen` branch, requiring trusted reviewer endorsement to prevent clutter. Several bugfixes and refactorings advanced, including fixes for `git log --graph` rendering issues and HTTP transport resilience improvements.

## Notable threads

### Critical regression in promisor remote lazy-fetch
Patrick Ohly reported a critical regression in Git 2.54.0 that breaks promisor remote lazy-fetch in partial clones. The issue, introduced by commit `5946a564` ("odb/source: make `read_object_info()` pluggable"), causes `git merge` to fail with "not our ref" when attempting to lazily fetch missing objects. The regression blocks CI workflows for projects like Kubernetes, which rely on blobless partial clones. No root cause analysis or proposed fix is yet available, but the bisect points squarely at the ODB abstraction work.

**What changed?**
The ODB abstraction layer altered how lazy fetches are triggered, specifically the interaction between `read_object_info()`, the `OBJECT_INFO_SECOND_READ` flag, and reprepare paths. The issue affects Git 2.54.0 and `master`, with Git 2.52.0 and earlier unaffected.

**Why it matters**
Partial clones are a critical feature for large repositories, enabling efficient workflows by deferring object downloads until needed. This regression breaks that promise, forcing users to revert to older Git versions or abandon partial clones entirely. The impact is particularly severe for CI systems, where the failure is deterministic and blocks critical workflows.

### `git repo info` path-keys series reaches design consensus
The `[PATCH v7]` series adding path-related keys to `git repo info` reached a design consensus on `/dev/null` handling. K Jayatheerth and Justin Tobler agreed to special-case `/dev/null` in `path.hooks.absolute` and `path.hooks.relative` to return `/dev/null` verbatim, improving usability and Windows compatibility. However, the series remains at risk of abandonment due to persistent lack of reviewer engagement. Junio C Hamano reiterated that the series will be dropped from `seen` within weeks unless reviewers step in.

**What changed?**
The series now includes 24 new keys with suffix-based formatting support, such as `path.toplevel.absolute`, `path.superproject-root.relative`, and `path.cdup`. The `/dev/null` handling for `path.hooks` keys diverges from `git rev-parse --git-path hooks` precedent but is uncontested.

**Why it matters**
The series exposes filesystem locations of repository components in a user-facing, scriptable format, addressing a long-standing gap in Git's introspection capabilities. The `/dev/null` special case improves usability for scripts that need to detect disabled hooks, but the series' broader value hinges on reviewer engagement to resolve remaining design questions, particularly the inclusion of deprecated `path.grafts` keys.

### Junio formalizes strict intake policy for `seen` branch
Junio C Hamano formalized a strict intake policy for the `seen` branch, requiring **trusted reviewer endorsement** for inclusion. The policy rejects Patrick Steinhardt's middle-ground proposal to require only "one human review that was somewhat favorable on the overall direction," insisting instead on feedback from reviewers whose judgment Junio trusts. The policy aims to prevent cluttering `seen` with "hopeless" topics that lack serious contributor effort, such as ensuring compatibility with in-flight topics.

**What changed?**
The policy raises the bar for `seen` inclusion from "any favorable comment" to "a trusted reviewer's approval of the direction." Junio frames this as necessary to avoid wasting project resources on topics that are unlikely to progress. The policy is now active, with Junio having sent notices to authors of stagnant topics.

**Why it matters**
The policy shifts the balance between reviewer productivity and contributor accessibility. While it reduces maintainer workload and avoids unceremonious ejections, it may discourage newcomers or less-connected contributors whose patches lack early attention from trusted reviewers. The policy's long-term impact on review velocity and patch quality remains to be seen.

### `git log --graph` rendering bug fixed
Caspar Duregger submitted a patch fixing a rendering error in `git log --graph --all --oneline` where orphaned commits were incorrectly drawn with a spurious parent connection. The fix modifies `graph_is_visual_root()` in `graph.c` to avoid indenting commits when there are columns to their right, preventing misalignment between `graph_output_pre_root_line()` and `graph_output_commit_line()`. Pablo Sabater reviewed and approved the patch, which is now ready for integration.

**What changed?**
The patch adds a condition (`graph->commit_index + 1 >= graph->num_columns`) to `graph_is_visual_root()` to prevent spurious indentation for orphaned commits with right-side columns. The fix ensures orphaned commits appear as standalone nodes without spurious indentation, regardless of `--graph-indent` setting.

**Why it matters**
The bug affects users who rely on `git log --graph` to visualize repository history, particularly in repositories with orphaned branches or unrelated histories. The fix restores correct rendering and improves the reliability of Git's graph-drawing logic.

### HTTP transport resilience improvements advance
Ravi Mistry's four-patch series improving Git's HTTP transport resilience advanced to v2, addressing all feedback from Junio C Hamano. The series introduces exponential backoff for retries, configurable retry status codes (`http.retryStatusCodes`), and retries for HTTP POST requests. Junio's review praised the extraction of `http_retry_wait()` but noted minor documentation and implementation refinements. The series is now effectively ready for merging, pending any further review.

**What changed?**
The v2 series clarifies error recovery rationale, improves error messages (including credential redaction), and adds test coverage for exponential backoff and credential handling. The `http.retryStatusCodes` configuration option defaults to "429" but allows users to specify additional retryable status codes (e.g., 5xx errors).

**Why it matters**
The series improves Git's resilience when interacting with hosting backends and reverse proxies under high concurrency. The exponential backoff and configurable retry codes address real-world pain points for users who experience transient server errors, while POST retries ensure data transfers (e.g., during fetch/clone) survive temporary failures.

## In brief
- **`git range-diff --matched-only`**: Junio clarified that the three-revision form of `git range-diff` cannot replace `--matched-only` in the rebasing scenario, reaffirming the feature's necessity. The patch remains queued in `seen` but at risk of being dropped unless a reviewer steps in.
- **`git diff --word-diff` bug**: Jean Abou Samra reported a bug where `git diff --word-diff` produces incorrect output when comparing files with both textual changes and whitespace additions. Phillip Wood confirmed the issue and explained its root cause: the word-diff logic splits hunks at context lines, which breaks when blank lines are introduced.
- **`git push --force-with-lease` silently ignored**: Patrick Peroša reported a bug where `git push --force-with-lease` is silently ignored when pushing to a remote group unless an explicit `<refname>` is provided. The flag is dropped during the internal decomposition of the group push into individual remote pushes.
- **`git branch --switch` RFC**: Johannes Welsch proposed adding a `-s`/`--switch` option to `git branch` to create and switch to a new branch in one step. Junio C Hamano questioned the necessity, noting redundancy with `git checkout -b` and `git switch -c`.
- **`git describe` regression fix**: Linus Torvalds and Patrick Steinhardt converged on a systemic resolution for `git describe`'s commit distance calculation when commit timestamps are skewed. The fix will use commit generation numbers by default, with an opt-out `--no-exact` flag and a warning mechanism for fallback behavior.
- **`git maintenance` lock-handling**: Pushkar Singh agreed to split the patch into two (behavioral change and Trace2 event) and deduplicate error-message formatting, addressing Patrick Steinhardt's review feedback.
- **`git reflog` `%gb` placeholder**: Phillip Wood identified a regression in the `%gb` format placeholder's branch-name detection when `GIT_REFLOG_ACTION` is set, causing incorrect labels during rebase operations.
- **`git bundle-uri` crash fix**: Aniket Gupta posted v2 of a patch fixing a crash in `git bundle-uri` when processing relative URIs with excessive `../` components. The patch addresses all feedback, including error-message improvements and test coverage for credential handling.
- **`git check-ref-format` refactoring**: Husain Sitamuwala posted v2 of a patch converting `git check-ref-format` to the `parse-options` API, addressing all feedback. Karthik Nayak declined to review v2 on procedural grounds, citing insufficient review window for v1.
- **`git grep --recurse-submodules` fix**: Patrick Steinhardt posted v2 of a bugfix series for `git grep --recurse-submodules`, addressing all feedback. The series is now ready for integration testing.
- **`git diff` EXAMPLES rewrite**: Julia Evans and Junio C Hamano debated the presentation format and content of the EXAMPLES section in `git-diff(1)`. Junio objected to horizontal definition lists, `--staged` promotion, and the omission of `git diff AUTO_MERGE`. Julia conceded the latter two points but the presentation format dispute remains unresolved.
- **Pre-commit hook policy**: Junio C Hamano proposed documenting that pre-commit hooks must not mutate the index and exploring code-level enforcement. The discussion reframed the original bug report as a policy matter, with Chris Torek explaining the internal mechanics behind the observed behavior.

## Summary statistics
- **Total emails today**: 143
- **Active threads before filtering**: 49
- **Notable threads**: 6
- **In-brief items**: 15