# Git mailing list daily digest for 2026/11/04

## The day in brief
The Git mailing list saw active discussion on several fronts today. A critical regression in promisor remote lazy-fetch was diagnosed and fixed, with the root cause traced to stale loose object cache entries. The `git describe` command's handling of skewed commit dates sparked debate over algorithmic approaches, with a new prototype offering near-exact results without relying on generation numbers. Outreachy applicants continued to propose projects, with several focusing on reducing global state and improving promisor remote functionality. Meanwhile, bugfixes for `git push --force-with-lease`, `git maintenance`, and credential-cache daemon races were refined and queued for integration.

## Notable threads

### Promisor remote lazy-fetch regression fix
**What changed**: Patrick Steinhardt posted a two-patch series fixing a critical regression in promisor remote lazy-fetch where newly written merged blobs couldn't be read back due to stale loose object cache entries. The fix updates the cache's oidtree after writing objects to listed subdirectories.

**Problem/goal**: The regression, introduced during ODB abstraction work, caused `git merge` to fail with "not our ref" errors when attempting to fetch objects that actually existed locally. This blocked CI workflows for projects like Kubernetes using blobless partial clones.

**Subsystem**: ODB abstraction layer, promisor remote lazy-fetch mechanism, loose object cache.

**Kind of change**: Bugfix (performance regression).

**Impact**: High - the regression affected all operations writing loose objects after listing their subdirectory in partial clones, including `git merge`, `git checkout`, and `git rebase`. The fix restores correct behavior without requiring full cache invalidation.

**Key technical details**:
- Root cause: Loose object cache's oidtree didn't reflect newly written objects in subdirectories already listed by prefetch
- Trigger condition: Merging an object whose first two hex characters matched those of a newly written object in the same subdirectory
- Fix: New `loose_cache_mark_seen()` function updates oidtree after writing objects
- Test coverage: New test in `t/t5616-partial-clone.sh` validates the fix with 50 mergeable files

### `git describe` algorithmic debate
**What changed**: Kristofer Karlsson proposed an alternative algorithm for `git describe` that delivers near-exact commit distances without relying on commit generation numbers, potentially making the `--exact` opt-out flag unnecessary.

**Problem/goal**: The current fix for skewed commit dates (from GitLab's Marge-bot workflow) uses commit generation numbers with an opt-out `--exact` flag. The new algorithm offers similar accuracy without the performance cost of `ensure_generations_valid()`.

**Subsystem**: `git describe` command, commit graph generation numbers.

**Kind of change**: Algorithmic improvement (feature enhancement).

**Impact**: Medium - affects users working with repositories containing non-monotonic commit dates, particularly the Linux kernel. The new approach could eliminate the need for the `--exact` flag while maintaining accuracy.

**Key technical details**:
- Current approach: Uses commit generation numbers with opt-out `--exact` flag
- New algorithm: Tracks two counters per tag (commits visited vs. commits carrying the tag's bit)
- Performance: Roughly the same as current implementation
- Edge case handling: Epsilon-based heuristic (1 week) for commits outside commit-graph
- Test results: Produces correct distances for regression case, matches `rev-list --count` on last merge window

### Outreachy proposals for promisor remote improvements
**What changed**: Two Outreachy applicants proposed projects to improve promisor remote functionality: configurable fetch ordering and a Protocol v2 extension for advertising preferred priorities.

**Problem/goal**: Current promisor remote behavior uses a hardcoded fallback order (configuration file order) when fetching missing objects. The proposals aim to make this configurable and allow servers to advertise preferred priorities.

**Subsystem**: Promisor remote, partial clone, Protocol v2.

**Kind of change**: Feature enhancements.

**Impact**: Medium - improves performance and reliability for users with multiple promisor remotes, particularly in large-scale deployments.

**Key technical details**:
- **Yopa Nelly's proposal**: `promisor.fetchOrder` config (comma-separated list of remote names)
- **Shashank S's proposal**: Protocol v2 extension gated by `promisor.sendFields` config
- Both approaches maintain backward compatibility
- Edge cases: Handling unlisted remotes, preserving `partialClone`-last rule
- Test coverage: Existing `t5710` fixture doesn't trigger lazy fetches; new tests needed

### Credential-cache daemon race condition fix
**What changed**: Farid Zakaria posted v2 of a three-patch series fixing race conditions in the credential-cache daemon where concurrent `git credential-cache store` commands could cause credential loss.

**Problem/goal**: The races occurred when multiple daemons tried to create the socket directory or hijack the socket, leading to credential loss, orphaned daemons, and inaccessible passwords.

**Subsystem**: Credential-cache daemon, simple-ipc subsystem.

**Kind of change**: Bugfix.

**Impact**: High - affects users with large repositories during parallel submodule fetches over HTTPS. The fix prevents credential loss and orphaned processes.

**Key technical details**:
- Root cause: Concurrent daemon startup races
- Fix: Uses `unix_ss_create()` for safe socket creation
- New behavior: Second daemon prints "ok" and exits if socket exists
- Test coverage: New tests in `t/t0301-credential-cache.sh` verify "ok" response and credential preservation
- Architectural concern: Mixing `unix_ss_*` and `tempfile` APIs (not yet addressed in v2)

## In brief

- **`git push --force-with-lease` regression**: Tyler Cipriani posted v2 of a fix for `--force-with-lease` being silently ignored when pushing to remote groups, addressing feedback on commit message clarity and a typo in the `Reported-by` trailer.

- **`git maintenance` lock handling**: Aniket Gupta reported a regression where background-scheduled maintenance runs fail with an error exit code instead of silently skipping, and prefetch tasks emit spurious lock warnings.

- **HTTP retry configuration**: Brian M. Carlson objected to the default `http.retryAfter=0` (immediate retry) when no Retry-After header is present, arguing for a non-zero default to avoid overwhelming rate-limited servers.

- **Commit-graph write refactoring**: Patrick Steinhardt posted a ten-patch series consolidating commit-graph write options into a single struct, moving verification into the pluggable ODB backend, and modernizing function names.

- **Path-walk object loss fix**: Derrick Stolee posted a bugfix for a regression in Git 2.55.0 where tagged trees containing blobs at already-visited paths caused object loss during pack-objects/path-walk operations.

- **`git credential` parse-options conversion**: Wasswa Edson posted a patch converting `git credential` to use `OPT_SUBCOMMAND` from the parse-options API, replacing manual `strcmp()`-based subcommand dispatch.

- **Documentation fix**: Cynthia Uche restored the word "stopping" that was accidentally dropped in 2009 from the description of `PARSE_OPT_KEEP_UNKNOWN_OPT` in `api-parse-options.adoc`.

- **`git log --follow` performance regression**: A severe performance regression in `git log --follow` was reported, with a 600x slowdown in large repositories when tracking files through non-linear history, bisected to commit 304812ed33.

- **Outreachy proposals**: Several Outreachy applicants proposed projects, including reducing Git's global state, refactoring built-in commands to use the parse-options API, and improving promisor remote functionality.

- **CI cleanup**: Tuomas Ahola posted a cleanup patch to delete the defunct `.cirrus.yml` file, which was confirmed unused by Patrick Steinhardt and Carlo Arenas.