# Git mailing list daily digest for 2026/10/14

## The day in brief
The Git mailing list saw **159 emails** today, with key developments including:
- **`git fast-import` crash fix** queued for `master` after four iterations.
- **`git worktree repair` bugfix series** restructured into five focused patches, addressing Junio’s review.
- **`git push` refspec matching** v2 series posted, targeting 10–100x speedups for large-scale deletions.
- **`git fetch` matching-inspired mode** v9 series now technically complete, awaiting final maintainer feedback.
- **`git stash` performance refactoring** v2 series expanded to seven patches, eliminating redundant index operations.
- **CI housekeeping** v3 series modernizing GitHub Actions/GitLab CI configurations, with naming dispute resolved.
- **`gitmergeconflicts(7)` documentation** finalized and merged to `next`.
- **`git repo structure` revision filtering** v5 patch addressing all prior feedback, now very close to ready.

## Notable threads

### `git fast-import` crash fix (2026/10/10/17-24-38)
**What changed?**
Marc Becker’s four-iteration bugfix for `git fast-import` is now **queued for `master`**. The patch resolves a crash when resolving commits or tags created earlier in the same session but not yet reachable via the object database.

**Why it matters:**
- **Critical correctness fix** for a niche but important code path used by tools like `git p4` and `git svn`.
- **No regressions**: preserves existing behavior for objects outside the cache.
- **Test coverage**: new tests in `t9300-fast-import.sh` for SHA-1 and SHA-256 repositories.

**Today’s delta:**
- [2026/10/14/22-42-49] Junio: "Thanks. This round I have nothing more to add. Will queue."

---

### `git worktree repair` bugfix (2026/09/13/03-20-11)
**What changed?**
Yoichi NAKAYAMA posted **v2 of the five-patch series** fixing a bug where `git worktree repair` could incorrectly modify unrelated worktrees. The series now splits the original refactoring into four smaller commits (patches 1–4) and retains the validation logic in patch 5.

**Why it matters:**
- **Correctness fix**: prevents accidental corruption of unrelated worktrees (e.g., when worktree directories are manually swapped).
- **Incremental refactoring**: addresses Junio’s feedback about commit splitting and error-handling consistency.
- **Test coverage**: expanded `t/t2406-worktree-repair.sh` to verify stricter `.git` file parsing and validation logic.

**Key patches:**
1. [2026/10/14/21-20-32] Aligns `.git` file handling with `repair_gitfile()`.
2. [2026/10/14/21-20-33] Centralizes inferred backlink selection logic.
3. [2026/10/14/21-20-34] Ensures consistent `.git` file parsing in `infer_backlink()`.
4. [2026/10/14/21-20-35] Reduces redundant `.git` file reads.
5. [2026/10/14/21-20-36] Adds validation to prevent repairs on unrelated worktrees.

**Today’s delta:**
- [2026/10/14/22-46-53] Junio: "Makes more sense to have this series as a logical progression of smaller pieces compared to v1."

---

### `git push` refspec matching speedup (2026/10/10/17-24-38)
**What changed?**
Jon Simons posted **v2 of the 12-patch series** speeding up refspec matching for large-scale deletions. The series replaces O(n²) linear traversals with `strmap` lookups, achieving **10–100x speedups** (e.g., 100 delete refspecs to a 100k-ref server drops from 2.5–8.9s to 0.15–0.19s).

**Why it matters:**
- **Performance fix**: addresses a real-world bottleneck in CI/CD pipelines.
- **Correctness fixes**: resolves long-standing quirks in `refname_match()` and duplicate-destination handling.
- **Test coverage**: new perf script (`t/perf/p5516-push-delete-refspec.sh`) and expanded test cases.

**Key patches:**
1. [2026/10/14/03-13-36] Adds input validation for `--force-with-lease`.
2. [2026/10/14/03-13-37] Adds perf test script for baseline measurements.
3. [2026/10/14/03-13-38] Removes `mkpath()` from `refname_match()`, delivering 55–82% of the speedup.
4. [2026/10/14/03-13-43] Adds client-side rejection of duplicate destinations.
5. [2026/10/14/03-13-47] Optimizes `--force-with-lease` overhead, reducing 100-lease timings from 0.45–0.49s to 0.16–0.19s.

**Today’s delta:**
- [2026/10/14/12-15-25] Patrick Steinhardt: Suggested a small correction to the error message in patch 3/12.

---

### `git fetch` matching-inspired mode (2026/09/19/14-47-26)
**What changed?**
Harald Nordgren posted **v9 of the five-patch series** introducing a matching-inspired fetch mode for shallow repositories. The series is now **technically complete**, addressing Junio’s two review points from v8:
1. Moved the `if/else if/else` control-flow reorganization from Patch 2/5 to Patch 3/5.
2. Restored the NULL-check for `branch` in `collect_upstream_from_remote()`.

**Why it matters:**
- **Performance fix**: resolves slow/hanging `git fetch` operations in shallow repositories with many branches.
- **Design consensus**: `remote.<name>.refmap` reuses Git’s existing `--refmap` mechanism.
- **Test coverage**: 119 lines of new tests in `t/t5586-fetch-refmap.sh`.

**Today’s delta:**
- [2026/10/14/07-04-07] Cover letter: "All review concerns from prior versions remain resolved."

---

### `git stash` performance refactoring (2026/10/05/16-35-29)
**What changed?**
Phillip Wood posted **v2 of the seven-patch series** refactoring `git stash` to eliminate redundant index loads and refreshes. The series now reduces `lstat()` calls per changed path by **4 for `create` and 8 for `push`**.

**Why it matters:**
- **Performance improvement**: measurable reduction in filesystem overhead.
- **Code hygiene**: consolidates duplicate `check_changes()` calls and simplifies `reset_tree()`.
- **Test coverage**: existing test suite verifies no regressions.

**Key patches:**
1. [2026/10/14/15-40-13] Introduces `enum create_result` to distinguish "no changes" from "nothing stashed."
2. [2026/10/14/15-40-15] Eliminates a redundant `check_changes()` call in `do_push_stash()`.
3. [2026/10/14/15-40-17] Replaces `reset_tree()` with `create_index_from_tree()` in `stash_working_tree()`.
4. [2026/10/14/15-40-19] Simplifies `reset_tree()` by removing unused parameters.

**Today’s delta:**
- [2026/10/14/18-16-22] Junio: "The original is doubly bad" (approving patch 6/7).

---

### CI housekeeping (2026/09/13/14-47-26)
**What changed?**
Patrick Steinhardt posted **v3 of the eight-patch series** modernizing Git’s CI configurations. The series resolves the naming dispute over the `linux-TEST-vars` job by renaming it to `linux-feature-testbed` (Junio’s preferred choice).

**Why it matters:**
- **Maintenance**: removes outdated dependencies (EOL Docker images, Python 2).
- **Coverage**: preserves reftable/SHA-256 test coverage while simplifying configurations.
- **Test reliability**: fixes a typo in `ci/run-build-and-tests.sh` that disabled breaking changes for the Fedora Meson job.

**Key patches:**
1. [2026/10/14/08-30-27] Adds `SHA1` prerequisite to skip a SHA-1-specific test in SHA-256 repositories.
2. [2026/10/14/08-30-28] Fixes typo in `ci/run-build-and-tests.sh` (restores `WITH_BREAKING_CHANGES`).
3. [2026/10/14/08-30-31] Renames `linux-TEST-vars` to `linux-feature-testbed`.
4. [2026/10/14/08-30-34] Repurposes `linux-reftable` job to test reftable with SHA-256.

**Today’s delta:**
- [2026/10/14/14-28-00] Junio: "I've backported this to apply cleanly on top of Git v2.56.0 for potential `maint` integration."

---

### `gitmergeconflicts(7)` documentation (2026/09/24/14-44-15)
**What changed?**
Julia Evans posted the **final updates** to the `gitmergeconflicts(7)` documentation series, incorporating Junio’s and Patrick Steinhardt’s feedback. The series is now **merged to `next`**.

**Why it matters:**
- **Usability**: centralizes merge conflict advice in a dedicated man page.
- **Clarity**: improves explanations of "ours" vs. "theirs" and promotes `merge.conflictstyle=diff3`.
- **Test coverage**: exhaustive list of commands that can cause conflicts (e.g., `git am -3`).

**Today’s delta:**
- [2026/10/14/19-24-13] Julia: "Will change to '30 days'" (reflog expiry period).
- [2026/10/14/22-08-21] Junio: "Yes, `git am -3` is a three-way merge operation."

---

### `git repo structure` revision filtering (2026/09/24/16-45-02)
**What changed?**
Mark C. Chu-Carroll posted **v5 of the single-patch series** adding revision filtering to `git repo structure`. The patch addresses all prior feedback, including:
- Improved commit message motivation.
- Restored the "empty repository" test.
- Cleaned up control flow redundancies.

**Why it matters:**
- **Diagnostic tool**: enables targeted analysis of subsets of the object graph.
- **Test coverage**: 208 lines of new tests for edge cases (e.g., overlapping includes/excludes).

**Today’s delta:**
- [2026/10/14/12-15-25] Patrick Steinhardt: "I'm now happy with the v5 patch."

## In brief
- **`git fast-import` crash fix** (2026/10/10/19-29-38): v5 posted, addressing all prior feedback.
- **`git describe` i18n** (2026/10/13/17-23-13): Jeff King: "obviously correct" but questions translating debug messages.
- **`git pull --rebase` custom refspecs** (2026/10/13/16-01-09): Junio confirms the issue is real; no patch yet.
- **`git log --follow` regression** (2026/10/13/07-36-24): Miklos Vajna acknowledges the 14× slowdown as a trade-off for correctness.
- **`git stash` completion updates** (2026/10/06/16-41-34): D. Ben Knoble: "dynamically discover subcommands and correct a Zsh-specific formatting issue."
- **Test modernization** (2026/10/14/07-26-46): Junio: "mentors should tell [contributors] to read the review exchanges on other candidates' patches."
- **ODB alternates refactoring** (2026/10/13/13-55-01): v2 posted, addressing Justin Tobler’s error-handling feedback.
- **Reftable backend bugfix** (2026/10/14/14-58-03): Matthew Suozzo adds bounds check to prevent buffer overflow.
- **Files ref backend bugfix** (2026/10/14/15-32-08): Andy Koppe fixes loose ref cache inconsistency with root refs.
- **Sequencer bugfix** (2026/10/14/16-58-24): Andy Koppe fixes ref-name clobbering in `--update-refs` logic.
- **Pretty-printing bugfix** (2026/10/14/17-23-33): Andy Koppe fixes indentation loss after newline in `%w()` format.