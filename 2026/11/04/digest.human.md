# Git mailing list daily digest for 2026/11/04

## The day in brief
The Git mailing list saw active discussion on several fronts today. A critical regression in promisor remote lazy-fetch was diagnosed and fixed, with the root cause traced to stale loose object cache entries. The `git describe` command's handling of skewed commit dates sparked debate over algorithmic approaches, with a new prototype offering near-exact results without generation numbers. Outreachy applicants continued to propose projects, with several focusing on reducing global state and improving promisor remote functionality. Performance regressions in `git log --follow` and HTTP retry logic were also prominent topics.

## Notable threads

### Promisor remote lazy-fetch regression fix
**What changed**: Patrick Steinhardt posted a two-patch series fixing a critical regression in promisor remote lazy-fetch that was causing `git merge` to fail with "not our ref" errors. The issue stemmed from stale loose object cache entries not being updated when new objects were written during merges.

**Problem/goal**: The regression, introduced during ODB abstraction work, was preventing Git from recognizing locally written merged blobs, causing unnecessary and failing backfill fetches from promisor remotes.

**Subsystem**: ODB abstraction layer, promisor remote lazy-fetch mechanism, loose object cache.

**Change type**: Bugfix.

**Impact**: The fix addresses a blocker for CI workflows in projects like Kubernetes that rely on blobless partial clones. The regression manifested only when merging an object whose first two hex characters matched those of a newly written object in the same subdirectory.

### Today's developments

- Kristofer Karlsson identified the exact mechanism: stale loose object cache invariant violation exposed by ODB abstraction work
- Patrick Steinhardt confirmed the issue and posted a fix series
- The fix updates the loose object cache's oidtree after writing objects to listed subdirectories

### `git describe` algorithmic reconsideration
**What changed**: Kristofer Karlsson proposed an alternative algorithm for `git describe` that delivers near-exact commit distances without relying on commit generation numbers, potentially obviating the need for the `--exact` opt-out flag.

**Problem/goal**: The current implementation, which uses commit generation numbers to handle skewed commit dates, has performance costs and requires an opt-out mechanism. The new algorithm aims to provide accurate results with similar performance to the current implementation.

**Subsystem**: Commit graph, revision walking.

**Change type**: Algorithmic improvement.

**Impact**: The proposal could eliminate the need for the `--exact` flag while maintaining accuracy, particularly for repositories with non-monotonic commit dates caused by GitLab's Marge-bot workflow.

### Today's developments

- Kristofer posted a prototype that tracks two counters per tag (commits visited vs. commits carrying the tag's bit)
- The algorithm uses an epsilon-based heuristic for commits outside the commit-graph
- Junio acknowledged the proposal as "interesting"
- Linus clarified the root cause of skewed dates: GitLab's Marge-bot `--keep-committers` merge logic

### HTTP retry logic configuration
**What changed**: Brian M. Carlson challenged the default behavior of the HTTP retry logic, arguing that even 429 responses should trigger a non-zero initial delay to avoid overwhelming rate-limited servers.

**Problem/goal**: The current implementation defaults to immediate retry when no Retry-After header is present, which could exacerbate server load during rate-limiting events.

**Subsystem**: HTTP transport.

**Change type**: Design discussion.

**Impact**: The debate centers on whether the default behavior should prioritize fast recovery or server protection, with implications for users interacting with rate-limited hosting backends.

### Today's developments

- Brian argued for a 30-second default delay to avoid overwhelming servers
- The discussion remains unresolved, with no consensus on the appropriate default

### Outreachy proposals for promisor remote improvements
**What changed**: Two Outreachy applicants proposed projects to improve promisor remote functionality, focusing on configurable fetch ordering.

**Problem/goal**: The current implementation uses a hardcoded order for trying multiple promisor remotes, which doesn't account for performance differences or user preferences.

**Subsystem**: Promisor remote, partial clone.

**Change type**: Feature enhancement.

**Impact**: The proposals aim to improve performance and reliability in partial-clone setups with multiple promisor remotes.

### Today's developments

- Yopa Nelly posted a patch implementing `promisor.fetchOrder` configuration
- Shashank S proposed a more comprehensive solution including server-side Protocol v2 extensions
- Both proposals received early feedback on design approaches

## In brief
- **`git log --follow` performance regression**: A severe slowdown (600x) was reported when tracking files through non-linear history, bisected to commit 304812ed33
- **`--force-with-lease` forwarding fix**: Tyler Cipriani posted v2 of a fix for `--force-with-lease` being silently ignored when pushing to remote groups
- **Credential command refactoring**: Wasswa Edson converted `git credential` to use `OPT_SUBCOMMAND` from the parse-options API
- **Path-walk object loss fix**: Derrick Stolee fixed a regression where tagged trees could cause object loss when revisiting paths
- **Documentation fixes**: Cynthia Uche restored a dropped word in the parse-options API documentation
- **CI cleanup**: The defunct `.cirrus.yml` file was removed after confirmation it was unused
- **Global state reduction**: Several Outreachy applicants proposed projects to reduce Git's reliance on global state
- **Commit-graph refactoring**: Patrick Steinhardt posted a ten-patch series consolidating commit-graph write options into a single struct
- **Pre-commit hook policy**: Discussion continued on whether pre-commit hooks should be allowed to mutate the index
- **ODB lockfile fixes**: Justin Tobler posted follow-up patches to recently merged ODB lockfile fixes