# The Git Project -- Weekly Digest for 2026/10/26 -- 2026/11/01

## The period in brief

This week (2026/10/26--2026/11/01) saw **635 emails across 212 threads** on the Git mailing list, with **six active days** of development. Traffic was **heavy and eventful**, dominated by architectural debates, critical bugfixes, and long-running feature series reaching maturity. The most consequential developments were **Junio C Hamano’s warning that the Rust reorganization patch may be dropped**, **a flurry of bugfixes for 4GiB+ file truncation on Windows**, and **the `git log --follow` optimization series (v4) dramatically improving performance for merge-heavy histories**. Other key themes included **CI infrastructure consolidation**, **submodule traversal enhancements**, and **ongoing debates about community voting processes**.

---

## Key developments

### Rust reorganization patch at risk of being dropped
The v5 patch series to reorganize Git's Rust infrastructure into a dedicated `rust/` subdirectory remains in limbo after **Junio C Hamano explicitly warned it may be dropped** due to lack of broader project enthusiasm. The patch, posted by Mike Hommey, is technically complete and addresses all prior feedback, but Junio tied its fate to the unresolved discussion about Git 3.0's mandatory Rust components and their impact on downstream projects. Mike reaffirmed his personal interest in the reorganization but provided no new technical arguments or advocacy to sway the decision. The patch remains queued for `next`, but without additional support, it is likely to be abandoned.

The reorganization would improve project structure clarity in Git's mixed C/Rust codebase and support downstream projects that vendor Git's C code while excluding its Rust components. The patch updates the Makefile, meson.build, .gitignore, and CI scripts to reference the new `rust/` locations.

---

### Large-file handling bugfixes address 4GiB+ truncation on Windows
A series of bugfix patches from **Elia Pinto** addressed silent truncation of files 4GiB or larger on platforms where `long` is 32 bits but `size_t` is 64 bits (notably 64-bit Windows). The patches target several subsystems, preventing silent data corruption when processing large files:

- **`git archive` (tar/zip formats) and `git apply`**: Two-patch series fixing truncation in archive creation and patch application.
- **`git grep` and `git diff`**: Two-patch series ensuring correct handling of large files in text searches and diffs.
- **`hashwrite()`, `count_lines()`, and `hash_chars()`**: Three-patch series fixing integer truncation in core object-writing and line-counting functions.
- **`parse_binary_hunk()`**: Standalone patch fixing integer overflow in the `apply` subsystem.

These fixes ensure Git correctly handles files up to `SIZE_MAX` without truncation, addressing a long-standing pain point for users working with large binary files or repositories containing files ≥4GiB. The patches are self-contained and merge cleanly with other in-flight work.

---

### `git log --follow` optimization dramatically improves merge performance
**Miklos Vajna** posted a **v4 series** that fixes bugs and optimizes `git log --follow` for merges, addressing a performance regression introduced in Git 2.56.0. The series:
1. Fixes a correctness bug where `-m --follow` loses history from non-first parents.
2. Fixes a bug where `--follow` misses edits on dropped sides of merges.
3. Introduces a targeted optimization that reduces the number of rename-detection passes needed when following a file through merges.

The optimization cuts runtime for a real-world linux.git case from ~3 minutes to ~37 seconds. The series is technically complete, with all known bugs fixed and performance regressions addressed. The interdiff shows the key technical change: the rename-detection logic now only checks for renames from files that the merge actually deleted, rather than scanning all files in every parent.

---

### CI infrastructure consolidation advances
**Patrick Steinhardt**'s eight-patch series consolidating Git's CI test environment variables across GitHub Actions and GitLab CI workflows saw significant progress. The series reduces duplication, improves maintainability, and ensures consistent test environments by centralizing variable assignments in `ci/lib.sh`. Key improvements include:
- Platform-wide rules replacing per-distribution assignments.
- Removal of redundant CI jobs while preserving test coverage.
- Introduction of a `--feature-testbed` flag for local reproduction of CI environments.
- Restoration of non-Rust test coverage lost after the April 2026 default-enable-Rust commit.

The series received positive feedback from Junio and Jeff King, with only minor implementation questions remaining. Peff suggested an alternative design for the final patch (using an environment variable instead of a command-line flag), which Junio endorsed.

---

### Credential-cache daemon race condition fixed
**Farid Zakaria** posted a two-patch series fixing a race condition in the Git credential-cache daemon that could cause lost credentials and orphaned daemons during parallel submodule fetches. The patches address two distinct parts of the race window:
1. Concurrent socket directory creation, using the `unix_ss_create()` helper from the simple-ipc subsystem to safely handle concurrent socket creation.
2. Socket hijacking, where the daemon now gracefully exits if it detects an existing socket.

The fix is particularly relevant for users with large repositories that perform parallel submodule fetches over HTTPS. New tests in `t/t0301-credential-cache.sh` verify the behavior.

---

### `git submodule foreach` gains dependency-ordered traversal
**William Hatfield**'s **v2 series** (3 patches) enhances `git submodule foreach` with new traversal options:
- `--reverse-traversal`: post-order processing (children before parents).
- `--include-superproject`: execute command in superproject after submodules.
- `--reversive`: shorthand combining `--recursive`, `--reverse-traversal`, and `--include-superproject`.

This enables **dependency-sensitive workflows** (e.g., builds/deployments) in complex submodule hierarchies. The series addresses prior feedback on **naming semantics** (`--reverse-traversal` vs `--postorder-traversal`) and **environment handling** (preserves caller’s `GIT_DIR`/`GIT_INDEX_FILE`). The implementation is finalized, but a naming discussion remains ongoing, with Junio preferring algorithmic/directional terms.

---

### `--autosquash` fix for empty commits
**Farid Zakaria**'s **v5 patch** fixes a long-standing edge case in `git rebase --autosquash` where `fixup!`/`squash!` commits that cancel out their target (resulting in empty commits) **failed to honor `--empty` options** (`drop`, `keep`, `stop`). The fix introduces:
- `is_amended_head_empty()` to detect when applying a `fixup!` empties the target commit.
- Persistent state (`$GIT_DIR/rebase-merge/fixup-target`) to track whether the target was originally picked with changes.

This resolves a **real-world workflow issue** at Meta, where internal LLVM patches are managed as single commits using `fixup!` autosquash. The fix enables **automatic dropping of both a commit and its revert** when `--empty=drop` is used. The series is technically approved (Phillip Wood’s review confirms readiness), but a competing design for a `revert!` autosquash directive remains unresolved.

---

### SSH signing verification flaw: `.gitattributes` silently ignored
**Junio C Hamano** identified a **potential correctness issue** in the squash-detection logic's handling of Git attributes. The `merge_keeps_upstream()` function in Harald Nordgren's `git branch --delete-integrated` series passes `the_repository->index` to `ll_merge()`, but the index may not be populated at this point. Since `ll_merge()` consults the index for custom merge drivers and attributes, passing an uninitialized `istate` could cause it to miss in-index attributes, leading to incorrect merge results or silent failures.

The issue affects the **path-by-path three-way merge approximation** introduced to handle upstream modifications in squash-detected branches. If attributes are not loaded, the merge may not respect `.gitattributes` directives, potentially causing false negatives (failing to recognize landed changes) or false positives (incorrectly marking changes as landed). This is the first **substantive technical feedback** on the fourth patch of the series, which had previously been viewed as addressing the last major correctness gap. Resolution will require either a code change (loading the index) or a documentation update (explicitly stating attributes are ignored).

---

### Community voting process: hybrid eligibility model proposed
The discussion about the Git project's community voting process advanced with **Jeff King (Peff) proposing a hybrid eligibility model** that combines an algorithmic threshold with a human-curated list for edge cases. This model reconciles the tension between Phillip Wood's call for a "middle ground" and Emily Shaffer's advocacy for broader inclusivity, while keeping administrative overhead manageable.

The hybrid model would:
1. Define a quantitative rule (e.g., "X commits and active in the last Y years") for most contributors.
2. Supplement this with a human-curated list for edge cases (e.g., former maintainers, deeply involved but inactive contributors).

Peff volunteered Junio or the Project Leadership Committee (PLC) as arbiters of the human-curated list, suggesting a one-time "big initial list" with periodic updates. Jonathan Corbet provided a **tested review** of CIVS (Condorcet Internet Voting Service), confirming its practicality for occasional polls but highlighting a **usability hurdle**: voters must proactively sign up on the platform, which could create friction for Git's email-centric workflow.

---

### HTTP transport gains resilience with exponential backoff and retries
**Ravi Mistry**'s four-patch series improved Git's HTTP transport resilience with:
- Exponential backoff (1s, 2s, 4s, ...) when Retry-After headers are absent.
- The `http.retryStatusCodes` config option to retry transient 5xx errors (opt-in).
- Retries for HTTP POST requests, which handle data transfers during fetch/clone and partial clone operations.

The series targets real-world pain points for users operating at scale and includes thorough test coverage. The patches fix an off-by-one retry count bug and a race in Retry-After header parsing, making Git more robust in unreliable network conditions.

---

## In brief

- **`git repo info` path-keys series at risk**: Junio issued a final call for reviewers on the v7 patch series adding eight new path-related keys to `git repo info`. The series is mechanically complete but has received no substantive technical review. Justin Tobler committed to reviewing it, which may be crucial for its survival.
- **`git stash` performance improvement**: Jasmeet Bhatia's patch improved `git stash` performance by reusing the already-prepared index state and untracked file list, showing a 16-18% improvement for `git stash -p` and 6% for `git stash push` on the Linux kernel repository.
- **`git describe` commit distance fix**: Linus Torvalds submitted a patch correcting `git describe`'s commit distance calculation when commit dates are non-monotonic, prioritizing commit generation numbers over raw commit dates.
- **`git worktree list` filtering**: The discussion pivoted from a `--branch` option to extending `git for-each-ref`'s `%(worktreepath)` atom, but raised concerns about safety and composability.
- **`git grep --recurse-submodules` regression**: Patrick Steinhardt posted a series to fix a regression where the command fails when grepping nested submodule trees, ensuring objects are read from the correct repository instance.
- **`gitsecurityboundary(7)` man page**: Johannes Schindelin introduced a new man page to define Git's security boundary, reducing misaligned security reports.
- **CI fixes for Ubuntu 26.04**: Harald Nordgren posted a three-patch series restoring four failing CI jobs after GitHub Actions updated the `ubuntu-latest` runner image.
- **`@ {push}` revision suffix alignment**: Harald Nordgren's patch aligned `@{push}` resolution with `push.default=simple` behavior in triangular workflows.
- **Test modernization**: Multiple patches modernized test infrastructure, including Kaartic Sivaraam's series to make `clar`-based unit tests ignore `-l`/`--long-tests`.
- **Dead-code removals**: Junio removed unused functions like `author_ident_sufficiently_given()` and `verify_uptodate()`.
- **`the_repository` removal**: Okhuomon Ajayi removed the last uses of `the_repository` in `fsmonitor.c` and `wt-status.c` as part of the ongoing libification effort.
- **Documentation improvements**: Julia Evans's beginner-focused tutorial replacing `gittutorial` was merged to `next`, and Tuomas Ahola updated `CodingGuidelines` to reflect the minimum required Perl version (5.26.0).

---

## Looking ahead

The next week is likely to see continued progress on several fronts:

- **Rust reorganization**: The patch may be dropped unless additional advocacy emerges. If retained, it will need to address forward-compatibility concerns for Git 3.0.
- **Community voting process**: The hybrid eligibility model will likely see further discussion, with decisions needed on the algorithmic threshold, human-curated list maintenance, and voting platform.
- **`git log --follow` optimization**: The v4 series is technically complete and may be integrated, dramatically improving performance for users with merge-heavy histories.
- **Large-file handling**: The bugfix series for 4GiB+ file truncation on Windows will likely be integrated, addressing a long-standing pain point for users working with large files.
- **CI infrastructure**: Patrick Steinhardt's consolidation series is poised for integration, improving maintainability and consistency across CI workflows.
- **Submodule traversal**: William Hatfield's `git submodule foreach` enhancements may be integrated, enabling dependency-sensitive workflows in complex submodule hierarchies.
- **`git repo info` path-keys**: The series may be integrated if Justin Tobler's review is positive, exposing filesystem locations of repository components in a scriptable format.