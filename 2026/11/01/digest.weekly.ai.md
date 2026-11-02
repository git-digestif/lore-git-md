# Git Mailing List Weekly Digest
**Period: 2026/10/26 -- 2026/11/01**

## The period in brief
This week saw **485 emails across 167 threads** on the Git mailing list, with traffic slightly above average for late October. The most consequential developments were **critical bugfixes** (repack data-loss, 64-to-32 truncation, credential-cache races), **performance optimizations** (`git log --follow`, `git stash`), and **architectural debates** (Rust reorganization, SSH signing verification, community voting). Three series reached production-ready status: Qin ShiCheng’s repack data-loss fix, Elia Pinto’s large-file handling fixes, and Ravi Mistry’s HTTP transport resilience improvements.

---

## Key developments

### Repack data-loss fix lands
Qin ShiCheng’s **v4 series** fixing a race condition in `git repack` was declared production-ready after merging the prerequisite `ps/odb-files-alternates` topic. The redesign introduces `--ignore-pack-keep-from` to explicitly tell `pack-objects` which packs it may delete, eliminating the problematic `--honor-pack-keep` entirely. Taylor Blau provided the final `Reviewed-by`, and benchmark data showed negligible overhead in realistic scenarios. The series touches `builtin/pack-objects.c`, `builtin/repack.c`, and six test files.

### 64-to-32 truncation fixes address silent data corruption
Elia Pinto’s **multi-series effort** to fix 64-to-32-bit truncations on LLP64 platforms (notably Windows) reached critical mass. The patches target silent data corruption in `git archive`, `git apply`, `git grep`, `git diff`, and the object database. Key changes include converting `unsigned long` to `size_t` for size parameters and hardening `hashwrite()` against truncation. The series is technically complete and awaits integration, with Patrick Steinhardt’s procedural feedback addressed in v3.

### HTTP transport gains resilience
Ravi Mistry’s **four-patch series** added exponential backoff, configurable retry status codes, and POST request retries to Git’s HTTP transport. The changes target real-world pain points for users operating at scale, including:
- Fixing an off-by-one retry count bug
- Introducing exponential backoff (1s, 2s, 4s, ...) when Retry-After headers are absent
- Adding `http.retryStatusCodes` to retry transient 5xx errors (opt-in)
- Extending retries to HTTP POST requests (data transfers during fetch/clone)

The series includes thorough test coverage and is ready for integration.

### `git log --follow` optimization for merges
Miklos Vajna’s **v4 series** fixed correctness bugs and optimized `git log --follow` for merge-heavy histories. The series addresses a performance regression introduced in Git 2.56.0 that made `git log --follow` significantly slower when processing merges. The optimization reduces the number of rename-detection passes needed when following a file through merges, cutting runtime for a real-world linux.git case from ~3 minutes to ~37 seconds. The series is technically complete and ready for integration.

### Rust reorganization at risk
Mike Hommey’s **v5 patch series** reorganizing Git’s Rust infrastructure into a dedicated `rust/` subdirectory remains in limbo. Junio C Hamano explicitly stated the patch will be dropped without broader project enthusiasm, despite being technically complete. The reorganization addresses immediate project hygiene concerns while establishing a foundation for Git 3.0’s mandatory Rust components. The patch updates Makefile, meson.build, .gitignore, and CI scripts to reference the new locations.

### Community voting process advances
The discussion around Git’s community voting process saw significant progress. Jeff King (Peff) proposed a **hybrid eligibility model** combining an algorithmic threshold (e.g., "X commits and active in the last Y years") with a human-curated list for edge cases. This model reconciles the tension between inclusivity and practicality, addressing concerns from Phillip Wood and Emily Shaffer. Jonathan Corbet provided a tested review of CIVS (Condorcet Internet Voting Service), confirming its practicality but highlighting a usability hurdle: voters must proactively sign up on the platform.

---

## In brief

**Architectural debates**
- **SSH signing verification flaw**: Junio C Hamano identified a correctness issue in Harald Nordgren’s `git branch --delete-integrated` series where `.gitattributes`-driven merge behavior could be silently ignored due to an uninitialized index state.
- **Notes subsystem retry mechanism**: Kristoffer Haugsbakk and Junio debated a retry mechanism for `git notes add` to allow non-conflicting updates to proceed, leveraging Git’s existing merge/rebase machinery.

**Performance improvements**
- **`git stash`**: Jasmeet Bhatia’s patch eliminated duplicate work in stash creation by reusing the already-prepared index state and untracked file list, showing a 16-18% improvement for `git stash -p`.
- **`git describe`**: Linus Torvalds fixed commit distance calculations for skewed timestamps by prioritizing commit generation numbers over raw commit dates.

**Bugfixes**
- **`git grep --recurse-submodules`**: Patrick Steinhardt posted a series to fix a regression where the command fails when grepping nested submodule trees.
- **Credential-cache daemon race condition**: Farid Zakaria fixed a race condition that could cause lost credentials and orphaned daemons during parallel submodule fetches.
- **Shallow fetch with tags**: A bugfix for shallow fetches with tags was merged to `seen`, restoring correct ‘have’ reporting during negotiation.
- **MIDX incremental layers**: Taylor Blau’s v2 series fixed a regression where MIDX writes published empty layers without a reverse index.

**CI and test infrastructure**
- **CI tooling**: Harald Nordgren’s v3 series made Perforce, Git LFS, and JGit installation mandatory on relevant platforms, failing fast if tool installation fails.
- **CI compatibility**: Harald Nordgren posted a three-patch series restoring four failing CI jobs after GitHub Actions updated the `ubuntu-latest` runner image to Ubuntu 26.04.
- **Unit test infrastructure**: Jeff King (Peff) completed an eight-patch series refactoring Git’s unit test infrastructure to integrate `clar`-based unit tests with the shell test suite.

**Documentation**
- **`gitsecurityboundary(7)`**: Johannes Schindelin introduced a new man page defining Git’s security boundary, reducing misaligned security reports.
- **Beginner tutorial**: Julia Evans’s documentation series replacing the outdated `gittutorial` man page was merged to `next`.
- **`.gitignore`**: Junio C Hamano proposed a rewrite of the `gitignore(5)` man page to explicitly state that the `*` wildcard matches strings of any length, including zero.

**Refactoring and cleanup**
- **`the_repository` removal**: Multiple patches removed implicit use of `the_repository` in `fsmonitor.c`, `name-hash.c`, and `wt-status.c`.
- **Dead-code removals**: Junio removed unused functions like `author_ident_sufficiently_given()` and `verify_uptodate()`.
- **Function encapsulation**: Junio posted a series marking functions as `static` where they are no longer needed outside their source files.

**New features and enhancements**
- **`git submodule foreach`**: William Hatfield’s v2 series enhanced the command with dependency-ordered traversal options (`--reverse-traversal`, `--include-superproject`, `--reversive`).
- **`git branch --delete-integrated`**: Harald Nordgren’s v6 series for detecting squash-merged and rebase-merged branches incorporated all prior feedback.
- **`git worktree list` filtering**: Eric Sunshine proposed extending `git for-each-ref`’s `%(worktreepath)` atom to report worktree paths even during rebase/bisect.

---

## Looking ahead
The next week is likely to see integration of the **repack data-loss fix**, **64-to-32 truncation fixes**, and **HTTP transport resilience improvements**. The **Rust reorganization** and **community voting process** discussions will continue, with decisions pending on both fronts. The **`git log --follow` optimization** and **`git stash` performance improvements** are also strong candidates for integration. Ongoing efforts like the **`the_repository` removal** and **test modernization** will see continued activity, with new patches expected in these areas.