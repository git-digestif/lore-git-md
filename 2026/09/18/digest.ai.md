# Git mailing list daily digest for 2026/09/18

## The day in brief
The Git mailing list saw significant activity today, with a **bugfix series for a data-loss race in `git repack`** posted as v2, a **patch fixing `git status --ignored` substring matching**, and an **RFC series introducing a "safe" `strbuf` API** to prevent recursive `die()` calls in trace2. Junio also reviewed Mike Hommey’s Rust reorganization patch, confirming it is ready for `next`, while Mark C. Chu-Carroll posted v4 of his test modernization series.

---

## Notable threads

### Rust infrastructure reorganization (v5) ready for `next`
**What changed?**
Junio C Hamano [2026/09/18/08-20-51] reviewed Mike Hommey’s v5 patch moving Git’s Rust code from `src/` to a dedicated `rust/` subdirectory. The patch applies cleanly on Git 2.56-rc1 and updates the Makefile, meson.build, and CI scripts to reference the new paths. Junio raised a minor stylistic question about whether setting `CARGO_MANIFEST_DIR` in the Makefile would be cleaner than repeatedly passing `--manifest-path`, but confirmed this is not a blocker.

**Why it matters**
This reorganization improves project structure clarity for Git’s mixed C/Rust codebase and supports downstream projects that vendor Git’s C code while excluding Rust. The patch is technically complete and queued for `next`.

**Today’s exchange**
Mike Hommey [2026/09/18/15-02-38] clarified that `current_source_dir` in meson.build refers to the directory containing meson.build, while `project_source_root` would place the target directory at the Git top-level, differing from the Makefile’s behavior.

---

### `git repack` data-loss race fix (v2)
**What changed?**
Qin ShiCheng [2026/09/18/03-03-30] posted v2 of a bugfix series addressing a race in `git repack -d` where concurrent pushes can cause objects to vanish despite `.keep` files. The series replaces the `--honor-pack-keep` mechanism with a snapshot-based approach, ensuring `.keep` files are managed consistently. Key changes:
- Patch 1/5 [2026/09/18/03-03-31] corrected `--stdin-packs=follow` behavior to treat `--keep-pack` packs as "kept-open" (`!`) instead of closed (`^`), allowing traversal through their objects.
- Patch 2/5 [2026/09/18/03-03-32] exposed `clear_kept_pack_cache()` to invalidate the kept-pack cache for cruft-pack walks, fixing a stale-cache bug.
- Patch 3/5 [2026/09/18/03-03-33] optimized `--keep-pack` lookups with binary search, yielding an 11× speed-up.
- Patch 4/5 [2026/09/18/03-03-34] added `--keep-pack-from-file` to handle repositories with many kept packs.
- Patch 5/5 [2026/09/18/03-03-35] eliminated the race by passing `pack-objects` a snapshot of `.keep` packs observed at startup.

**Why it matters**
This fixes a serious data-loss bug where dangling refs could point to nonexistent commits. The series is well-tested, with each patch including a regression test, and merges cleanly into `next` and `seen`.

---

### `git status --ignored` substring matching fix
**What changed?**
René Scharfe [2026/09/18/11-04-06] posted a patch fixing a regression in `git status --ignored` where pathspecs (e.g., `ba`) incorrectly matched ignored directories containing the pathspec as a substring (e.g., `bar/baz/`). The fix ensures `match_pathspec_with_flags()` is called for excluded directories with nested repositories, restoring exact-match behavior.

**Why it matters**
The regression, introduced in commit `95c11ecc73` (2020), caused unintended substring matching, breaking workflows that rely on precise pathspec filtering. The patch includes a new test case in `t/t7061-wtstatus-ignore.sh` to prevent regression.

---

### `git rerere remaining` stage-1 conflict fix
**What changed?**
Junio C Hamano [2026/09/18/12-30-51] posted a patch fixing `git rerere remaining` (and `git mergetool`) skipping consecutive conflicted paths with only stage-1 entries. The fix adds a same-path check (`ce_same_name()`) to `check_one_conflict()` in `rerere.c`, ensuring only entries for the current pathname are skipped.

**Why it matters**
The bug caused `git mergetool` to exit prematurely during large rebases, leaving conflicts unresolved. The fix is minimal (6 lines changed) and aligns with existing logic for stage-2 and stage-3 entries.

---

### "Safe" `strbuf` API RFC
**What changed?**
Derrick Stolee [2026/09/18/13-02-14] posted an RFC series introducing a "safe" `strbuf` API that cannot call `die()` or `exit()`, motivated by trace2’s need to avoid recursive `die()` loops. The series:
- Moved `strbuf` struct definitions to `strbuf-safe.h` [2026/09/18/13-02-15].
- Proactively initialized `GIT_ALLOC_LIMIT` during startup [2026/09/18/13-02-16].
- Refactored `memory_limit_check()` to introduce `safe_memory_limit_check()` [2026/09/18/13-02-17].
- Implemented `sstrbuf_grow()` [2026/09/18/13-02-18], the first safe method.
- Prepared `json-writer.c` for safe API adoption [2026/09/18/13-02-19].
- Added `sstrbuf_init()` and `sstrbuf_release()` [2026/09/18/13-02-20], beginning the conversion of `json-writer.c`.

**Why it matters**
The series addresses a critical problem: trace2 (via `json-writer`) can trigger recursive `die()` loops if memory allocation fails. The safe API returns error codes instead of dying, enabling robust error handling in low-level libraries. The transition is incomplete (callers are not yet error-aware), but the RFC lays the groundwork for broader adoption.

**Likely discussion points**
- Whether the "safe" API is the right approach (vs. hardening the existing API).
- The `sstrbuf_*` naming convention (vs. `strbuf_*_gently`).
- The fate of `GIT_ALLOC_LIMIT` (rename to `GIT_TEST_ALLOC_LIMIT` or keep as-is).

---

### Test modernization (v4)
**What changed?**
Mark C. Chu-Carroll [2026/09/18/17-18-44] posted v4 of a series modernizing three test scripts (`t4001-diff-rename.sh`, `t4009-diff-rename-4.sh`, `t4010-diff-pathspec.sh`) to use current Git test conventions. Changes include:
- Replacing setup functions with `test_expect_success 'setup'` blocks.
- Standardizing capitalization and `expect`/`actual` file naming.
- Using `<<-` here-doc syntax for consistent indentation.

**Why it matters**
The series improves test readability and maintainability without altering behavior. The v4 iteration addresses all prior feedback, including Junio’s suggestions [2026/09/18/07-10-51] to shorten commit subjects and fix inconsistent capitalization.

---

## In brief
- **Git for Windows hardening**: Johannes Scharfe [2026/09/18/07-12-32] argued that the `slen` parameter in `check_signature()` is critical for handling signatures with embedded NULs, while Junio C Hamano [2026/09/18/08-59-11] proposed removing it entirely, citing redundancy.
- **`git diff --no-index -R` fix**: Haokai Ding [2026/09/18/07-19-06] posted a patch fixing file/directory conflicts in `git diff --no-index -R` by swapping filespecs when `reverse_diff` is set.
- **Windows CI follow-up**: Junio C Hamano [2026/09/18/08-42-09] confirmed Rust is not preinstalled on GitLab’s Windows runners, answering an open question from Johannes Schindelin’s CI series.