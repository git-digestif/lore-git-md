# Git mailing list daily digest for 2026/10/20

## The day in brief
The most consequential new developments today were **Torsten Bögershausen’s proposal to relocate the CRLF conversion check** in the macOS compatibility series, **brian m. carlson’s critical correctness concerns** about Windows ReFS silent corruption in the worktree reflink series, and **Junio C Hamano’s rejection of a reviewer’s parameter naming suggestion** in the `the_repository` removal effort. Other notable threads included a **performance regression fix for shallow fetches**, a **bugfix for `git init`’s absolute path output**, and a **discussion about structured output for `git bisect`** to avoid parsing human-readable messages.

---

## Notable threads

### macOS CRLF conversion: architectural debate over check placement
**What changed?**
Torsten Bögershausen proposed relocating the `would_convert_to_git()` check from `ie_modified()` to `ie_match_stat()` to avoid interfering with Git’s racy Git logic. The change would simplify the code by removing a conditional shortcut in `ie_modified()` and letting `ce_modified_check_fs()` handle all conversion cases, including the size-mismatch scenario.

### Why it matters

The proposal addresses a long-standing architectural debate in the macOS CRLF conversion series. While Torsten’s approach preserves correctness and avoids racy Git issues, it reintroduces performance overhead by calling `would_convert_to_git()` twice (once in `ie_match_stat()` and once in `ce_modified_check_fs()`). This trade-off contrasts with Curtis Allen Smith’s v5 placement, which targeted only index-refresh operations and minimized overhead. The discussion remains unresolved, with no clear consensus on whether the double call is acceptable or if a hybrid approach is needed.

### Key details

- **Files**: `read-cache.c`, `ie_match_stat()`, `ie_modified()`
- **Commands affected**: `git diff-files`, `git diff-index` (in addition to index-refresh commands)
- **Performance impact**: measurable overhead in extreme cases (e.g., 138ms for 10,000 modified files)
- **Racy Git interaction**: Torsten’s proposal avoids interference with Git’s racy Git logic, a key concern raised in earlier reviews

---

### Worktree reflink: Windows ReFS silent corruption risk
**What changed?**
brian m. carlson raised **critical correctness concerns** about the proposed `FSCTL_DUPLICATE_EXTENTS_TO_FILE` implementation for Windows ReFS in the worktree reflink series. The underlying bug (Git LFS #6312) can cause silent corruption when cloning extents, making the feature unsafe for production use until Microsoft provides a fix.

### Why it matters

The reflink feature is a major performance improvement for users managing many worktrees, but the Windows implementation now faces a **data integrity risk** that cannot be mitigated in Git’s codebase. The discussion has shifted from implementation details to whether the feature should be **disabled entirely on Windows** until the underlying bug is fixed. This could delay or derail the Windows portion of the series, which was previously considered uncontroversial.

### Key details

- **Files**: `copy.c` (Windows-specific reflink logic)
- **Risk**: silent corruption when cloning extents on ReFS
- **Workaround**: disable Windows reflink until Microsoft fixes the bug
- **Impact**: macOS and Linux reflink implementations remain unaffected

---

### `the_repository` removal: maintainer rejects parameter naming suggestion
**What changed?**
Junio C Hamano rejected a reviewer’s suggestion to rename the parameter from `r` to `repo` in a refactoring patch for `merge-blobs.c`, arguing that `r` is the **idiomatic parameter name** for `struct repository *` in the codebase. The exchange highlights the tension between readability and consistency in the ongoing `the_repository` removal effort.

### Why it matters

The `the_repository` removal is a major refactoring effort to make Git’s internals more modular and reusable as a library. While the parameter naming debate is minor, it reflects broader questions about **codebase consistency** and **contributor expectations**. Junio’s rejection signals that the project prioritizes idiomatic conventions over marginal descriptiveness, which may influence future contributions to the effort.

### Key details

- **Files**: `merge-blobs.c`
- **Function**: `fill_mmfile_blob()`
- **Parameter**: `struct repository *r` (unchanged)
- **Context**: part of the ongoing `the_repository` removal project

---

### Shallow fetch performance regression fix
**What changed?**
Harald Nordgren posted a **revised patch (v10)** for the shallow fetch performance regression, fixing a protocol-layer API misuse by switching from `packet_write_fmt` to `packet_writer` to preserve sideband multiplexing. The patch resolves a merge conflict with Junio’s `ch/fetch-followremotehead-lazy-validation` topic and is now **technically complete**.

### Why it matters

The regression caused multi-minute delays in shallow fetches from large repositories (e.g., 900k commits) due to inefficient history traversal. The fix aligns protocol v2 behavior with v0 by directly advertising shallow grafts when no depth is specified, avoiding redundant traversal. The series is now **ready for final maintainer review** before merging to `next`.

### Key details

- **Files**: `shallow.c`, `upload-pack.c`
- **Performance impact**: 7 minutes → 1 second for reported CI workflows
- **Test coverage**: `t/t5537-fetch-shallow.sh` (new test for sideband-all shallow fetch)

---

### `git init` absolute path output regression
**What changed?**
Matt Hunter posted a **bugfix patch** to restore the pre-20e38cba3926 behavior of `git init` and `git init --reinit` printing the absolute path of the initialized repository (e.g., `/home/user/repo/.git`) instead of the relative `.git/` path. The regression was introduced in the current development cycle (Git 2.56), making the fix urgent to avoid shipping the behavior change.

### Why it matters

The regression affects user-facing output and could break scripts or tooling that rely on the absolute path. The patch is **minimal and uncontroversial**, with Junio C Hamano noting that the absolute path is more useful in command transcripts. The discussion has shifted to whether a **regression test** should be added, with Patrick Steinhardt advocating for one and Junio raising broader concerns about untested user-facing details in Git’s output.

### Key details

- **Files**: `builtin/init-db.c`
- **Regression introduced**: 20e38cba3926 (setup: split up concerns of `init_db()`, 2026-09-10)
- **Test coverage**: none added yet; discussion ongoing

---

### Structured output for `git bisect`
**What changed?**
Theodore Ts'o proposed adding a `--format=json` option to `git bisect` to provide structured, machine-readable output, framing it as a **Git 3.0 feature**. The discussion was sparked by a real-world dependency on `git bisect`’s human-readable output in the Linux kernel’s `ktest.pl` tool, which broke due to a recent formatting change.

### Why it matters

The thread highlights the **fragility of parsing human-readable output** and the need for structured alternatives in automation. Ts'o’s proposal addresses a broader problem: downstream tools (e.g., `ktest.pl`, `xfstests-bld`) rely on Git’s output as a de facto API, even though it is not guaranteed to be stable. The discussion remains **informational**, with no patch yet, but it could inform future work on structured output in Git.

### Key details

- **Proposed feature**: `git bisect --format=json`
- **Use case**: automated bisection harnesses (e.g., `ktest.pl`, `xfstests-bld`)
- **Target**: Git 3.0 (forward-looking)
- **Current fragility**: parsing strings like "is the first bad commit" or relying on exact output formatting

---

## In brief
- **CRLF conversion**: Torsten Bögershausen proposed relocating the `would_convert_to_git()` check to `ie_match_stat()` to avoid racy Git issues, but the performance trade-off remains unresolved.
- **Worktree reflink**: brian m. carlson raised critical correctness concerns about Windows ReFS silent corruption, advocating disabling Windows reflink until Microsoft fixes the bug.
- **`the_repository` removal**: Junio C Hamano rejected a reviewer’s suggestion to rename a parameter from `r` to `repo`, prioritizing idiomatic conventions.
- **Shallow fetch performance**: Harald Nordgren posted a revised patch (v10) fixing a protocol-layer API misuse, making the series ready for final review.
- **`git init` regression**: Matt Hunter posted a bugfix to restore absolute path output, with discussion ongoing about adding a regression test.
- **Structured output**: Theodore Ts'o proposed `--format=json` for `git bisect` to address fragility in parsing human-readable output.
- **Hook configuration**: Adrian Ratiu confirmed the current behavior of `hook.<event>.enabled` was intentional, leaving the decision to accept the patch or update the documentation unresolved.
- **Alternate ODB reload**: Junio C Hamano suggested deferring the `stat_validity` update in `read_alternates()` until after a successful read.
- **`git stash` refactoring**: Phillip Wood posted v3 of a series optimizing redundant index operations, with Junio praising the most interesting step.
- **`git imap-send`**: Harald Nordgren agreed to use `istarts_with()` instead of `starts_with()` to comply with RFC 5322.
- **`git status` default behavior**: Harald Nordgren plans to expand the commit message to establish historical context for the current default (`@{upstream}`).
- **`git rebase` doc fix**: Omri Sarig posted v3 of a patch fixing stale mentions of `--preserve-merges` and `--allow-empty-message`, now queued for `maint`.
- **`git fsck` object count**: Aniket Gupta posted a patch to correct a misleading progress message in `git fsck --verbose`.
- **`git mktree --batch` tests**: Roger Koranteng added basic test coverage for `git mktree --batch`, addressing a gap that allowed a memory leak to go unnoticed.
- **`git init` tutorial**: Julia Evans and Junio C Hamano discussed minor wording and formatting tweaks, with the series now one mechanical fix away from merging.
- **Outreachy inquiries**: Wasswa Edson and Tanga Joris sought clarification on project scoping for "Implement promisor remote fetch ordering" and "Improve how command arguments and options are scanned and parsed," respectively.