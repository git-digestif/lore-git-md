# Git mailing list daily digest for 2026/10/14

## The day in brief
The Git mailing list saw significant activity today with several key developments: the `rerere` race-condition fix series was marked ready for `next`, a major performance optimization for `git push` refspec matching was posted, and multiple documentation improvements (including the new `gitmergeconflicts(7)` guide) advanced toward integration. Outreachy applicants continued to submit test modernization patches, while maintainers provided feedback on CI housekeeping and index lock timeout proposals.

## Notable threads

### `rerere` race-condition fix series ready for `next`
**What changed?**
The two-patch series introducing a configurable lock timeout (`rerere.lockTimeout`) and a `--skip-locked` mode for `git rerere gc` is now technically complete and queued in Junio C Hamano’s `next` branch for integration testing.

### Why it matters

The series fixes a long-standing race condition in the `rerere` subsystem where concurrent writers contend for the shared `rr-cache` directory lock. The geometric maintenance strategy introduced in Git 2.54 exacerbated the issue, but the race is a general robustness flaw: any two concurrent `rerere` operations can collide. The solution is a pragmatic two-part fix: (1) configurable timeout for lock acquisition, and (2) silent lock-skipping for background maintenance tasks. The series addresses all prior feedback from Junio C Hamano and Patrick Steinhardt, with no substantive objections remaining.

### Technical details

- **Files touched**: `rerere.c`, `rerere.h`, `builtin/am.c`, `builtin/gc.c`, `builtin/merge.c`, `builtin/rerere.c`, `builtin/stash.c`, `sequencer.c`, `Documentation/config/rerere.adoc`, `Documentation/git-rerere.adoc`, `t/t4200-rerere.sh`, `t/t7900-maintenance.sh`.
- **New behavior**: Introduces `rerere.lockTimeout` (default 1000 ms) for configurable lock wait times, and `--skip-locked` (hidden/undocumented) for `git rerere gc` to skip held locks silently.
- **Test coverage**: Expanded to verify waiting for the lock, failing after timeout, and handling explicit `rerere` commands (`forget`, `clear`, `gc`).

**Status**: Queued in `next`; no further technical discussion expected.

---

### `git push` refspec matching optimization (10–100x speedup)
**What changed?**
Jon Simons posted v2 of a 12-patch series replacing linear refspec traversals with `strmap` lookups, achieving 10–100x speedups for large-scale branch deletions (e.g., pushing 100 delete refspecs to a server with 100k refs). The v2 squashes failing-test commits into their corresponding fixes and adds four new patches for test coverage of server-side behaviors, refspec ordering guarantees, and performance baseline measurements.

### Why it matters

Client-side refspec matching in `git push` becomes a scalability bottleneck when pushing many branch deletions to large repositories. The current O(n²) implementation leads to multi-second delays, degrading user experience in CI/CD pipelines and large monorepos. The series addresses this by:
1. Removing `mkpath()` overhead in `refname_match()` (55–82% of total speedup).
2. Optimizing `check_push_refs()` and `match_explicit_refs()` with `strmap` lookups.
3. Fixing a long-standing behavioral quirk where leading `./` in refnames were silently stripped.

### Technical details

- **Files touched**: `refs.c`, `remote.c`, `builtin/send-pack.c`, `t/t5516-fetch-push.sh`, `t/t5408-push-porcelain.sh`, `t/t5410-receive-pack.sh`, `t/t5510-fetch.sh`, `t/t5533-push-cas.sh`, `t/perf/p5516-push-delete-refspec.sh`.
- **Performance**: Benchmark timings show 100 delete refspecs dropping from **2.5–8.9s to 0.15–0.19s**.
- **Behavioral changes**: Duplicate destinations are now correctly rejected client-side when pushing to an empty remote, aligning with server-side enforcement.

**Status**: Under review; awaiting substantive technical feedback. The series is well-motivated and addresses a real-world performance pain point.

---

### `gitmergeconflicts(7)` documentation series finalized
**What changed?**
Julia Evans posted the final updates to the `gitmergeconflicts(7)` documentation series, incorporating Junio C Hamano’s requested refinements to the `diff3` example and related phrasing. The series introduces a dedicated man page consolidating merge conflict resolution guidance, replacing duplicated advice across multiple command man pages.

### Why it matters

Git’s documentation currently scatters merge-conflict advice across `git-merge(1)`, `git-rebase(1)`, and other command-specific pages, leading to duplication, omissions, and user confusion. The new guide centralizes this information, improves clarity (e.g., "ours" vs. "theirs"), and provides practical examples. The series has received positive feedback from Junio C Hamano, Jeff King, and Patrick Steinhardt, with all substantive feedback now addressed.

### Technical details

- **Files touched**: `Documentation/gitmergeconflicts.adoc` (new, ~330 lines), `Documentation/Makefile`, `Documentation/meson.build`, `.gitattributes`, `Documentation/git-merge.adoc`, `Documentation/git-rebase.adoc`, `Documentation/git-revert.adoc`, `Documentation/git-cherry-pick.adoc`, `Documentation/git-pull.adoc`, `command-list.txt`.
- **Key improvements**:
  - Exhaustive list of commands that can cause conflicts (`git merge`, `git rebase`, `git cherry-pick`, `git pull`, `git am -3`, `git checkout -m`, `git stash pop`, `git revert`).
  - Side-by-side examples of `diff3` vs. default merge conflict format, demonstrating how the common ancestor reveals intent.
  - Promotion of `merge.conflictstyle=diff3` as a best practice.
  - Clarification of "ours" and "theirs" during rebases.

**Status**: Finalized; Junio’s sign-off confirms readiness for merging to `next`.

---

### CI housekeeping series v3 resolves naming dispute
**What changed?**
Patrick Steinhardt posted v3 of an eight-patch CI/build system housekeeping series, resolving the naming dispute over the `linux-TEST-vars` job by renaming it to `linux-feature-testbed` (Junio C Hamano’s preferred choice among 14 alternatives). The series modernizes Git’s CI configurations by removing outdated dependencies (EOL Docker images, Python 2), eliminating redundant jobs, and updating supporting scripts.

### Why it matters

The series reduces CI complexity while maintaining or improving coverage. Key changes include:
- **Naming resolution**: The `linux-TEST-vars` job is now `linux-feature-testbed`, clarifying its purpose (testing non-default Git options like `OPENSSL_SHA1_UNSAFE`).
- **Reftable coverage**: The `linux-reftable-leaks` job is repurposed as `linux-reftable-sha256` to test reftable with SHA-256, while preserving foreign-SCM interoperability coverage in the renamed `linux-reftable` job.
- **Image updates**: The `linux32` job now uses `i386/debian:latest` (supported until 2030), and the `linux-feature-testbed` job uses `ubuntu:latest`.

### Technical details

- **Files touched**: `.github/workflows/main.yml`, `.gitlab-ci.yml`, `ci/install-dependencies.sh`, `ci/lib.sh`, `ci/run-build-and-tests.sh`, `t/t5004-archive-corner-cases.sh`.
- **Net effect**: 20 insertions, 39 deletions, reducing CI complexity.

**Status**: Under review; Junio has backported the series to apply cleanly on Git v2.56.0 for potential `maint` integration. Jeff King’s feedback on wildcard patterns (`*breaking-changes*`) and Icelandic locale support has been addressed.

---

### Configurable index lock timeout series advances
**What changed?**
Jun Wu and Jeff King exchanged clarifications on the configurable index lock timeout series (`core.indexLockTimeout`), addressing edge cases and the pre-existing race condition in `git commit`. The series adds a configurable timeout (default 1 second) for Git’s index lock to reduce contention between background IDE `git status` calls and user-initiated commands.

### Why it matters

The series targets a real-world pain point: IDEs polling `git status` can block user commands like `git add` or `git commit`, leading to "cannot commit" errors. The timeout allows commands to wait briefly for the lock instead of failing immediately. The series also fixes a **20-year-old race condition** in `git commit` where concurrent updates (e.g., from `git add`) could be silently discarded if `git commit` read the index before acquiring the lock.

### Technical details

- **Files touched**: `repository.c`, `builtin/commit.c`, `Documentation/config/core.adoc`, `t/t0032-index-lock-timeout.sh`.
- **New config key**: `core.indexLockTimeout` (integer, milliseconds; 0 = no retry, -1 = indefinite, default = 1000).
- **Behavior**: Commands wait up to the timeout for the lock unless the index was already read by the same process (to avoid write races).

**Status**: Under review; Junio’s feedback on edge cases has been addressed, but the default timeout value (1 second) remains open to debate. Jeff King’s identification of the pre-existing race in `git commit` strengthens the case for the lock-order change.

---

## In brief
- **`git worktree repair` bugfix**: Yoichi NAKAYAMA posted v2 of a five-patch series fixing incorrect repairs of unrelated worktrees. The v2 splits the original refactoring patch into four smaller commits, addressing Junio’s review feedback. **Status**: Under review.
- **Matching-inspired fetch mode**: Harald Nordgren posted v9 of a five-patch series introducing `remote.<name>.refmap` for shallow repositories. The v9 addresses Junio’s two review points from v8. **Status**: Technically complete; awaiting final maintainer feedback.
- **ODB alternates refactoring**: Patrick Steinhardt posted v2 of a six-patch series optimizing alternates handling in the "files" backend. The v2 adds a patch to tighten error handling for corrupt objects. **Status**: Under review; Justin Tobler’s feedback addressed.
- **`git fast-import` crash fix**: Marc Becker posted v4 of a bugfix for a crash when resolving commits or tags created earlier in the same session. **Status**: Queued for `master` (Junio: "Will queue").
- **`git log --follow` regression**: Miklos Vajna acknowledged the 14× slowdown introduced by commit 304813ed33, framing it as a trade-off for correctness. **Status**: Under investigation; no fix posted yet.
- **`git pull --rebase` and custom refspecs**: Junio confirmed the issue is real, citing a FIXME comment in the code. **Status**: Design discussion stage.
- **Test modernization**: Outreachy applicants (Victor Okafor, Shashank S, Srujith A Poojary, Dorine Tipo) posted patches replacing legacy test constructs (`test -f`, `test -d`) with modern helpers (`test_path_is_file`, `test_path_is_dir`). **Status**: Under review; Junio suggested mentors guide contributors to learn from prior review exchanges.
- **Reftable backend bugfix**: Matthew Suozzo posted a patch adding a bounds check in the reftable backend’s object record decoder to prevent potential buffer overflows. **Status**: Under review.
- **Files ref backend bugfix**: Andy Koppe posted a patch fixing an inconsistency in how root refs are handled in the loose ref cache. **Status**: Under review.
- **Sequencer bugfix**: Andy Koppe posted a patch fixing a ref-name clobbering bug in `--update-refs` logic during interactive rebase. **Status**: Approved and queued for integration.
- **Pretty-printing bugfix**: Andy Koppe posted a patch fixing indentation loss after newlines in `%w()` format. **Status**: Under review.