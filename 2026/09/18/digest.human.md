# Git mailing list daily digest for 2026/09/18

## The day in brief
The Git mailing list saw a flurry of activity today, with a **v2 bugfix series for a repack data-loss race** landing, **Rust infrastructure reorganization queued for `next`**, and an **RFC introducing a "safe" `strbuf` API** to prevent fatal errors in trace2. A **test modernization series** also reached v4, addressing prior feedback.

## Notable threads

### Rust infrastructure reorganization queued for `next`
The long-running effort to reorganize Git’s Rust code into a dedicated `rust/` subdirectory reached a milestone today. Junio C Hamano reviewed Mike Hommey’s v5 patch, confirming it applies cleanly on Git 2.56-rc1 and correctly updates the build system (Makefile, meson.build, CI scripts) to reference the new locations. The patch moves Rust sources from `src/` to `rust/src/` while keeping build artifacts in their original locations, addressing project hygiene and downstream vendoring concerns.

Junio raised a minor stylistic question about whether setting `CARGO_MANIFEST_DIR` in the Makefile would be cleaner than repeatedly passing `--manifest-path`, but this is not a blocker. Mike clarified that `current_source_dir` in meson.build refers to the directory containing meson.build, while `project_source_root` would place the target directory at the Git top-level, differing from the Makefile’s behavior. The patch is now queued for `next`, marking a key step toward Git 3.0’s mandatory Rust components.

### Repack data-loss race fix (v2)
Qin ShiCheng posted v2 of a critical bugfix series addressing a race condition in `git repack -d` where concurrent pushes could cause objects to vanish despite `.keep` files. The series replaces the `--honor-pack-keep` mechanism with a snapshot-based approach, ensuring `.keep` files are managed consistently. Key changes in v2 include:
- **Patch 1/5**: Corrects `--stdin-packs=follow` behavior to treat `--keep-pack` packs as "kept-open" (`!`) instead of closed (`^`), allowing traversal through their objects.
- **Patch 2/5**: Fixes a stale kept-pack cache in cruft-pack walks by exposing `clear_kept_pack_cache()`.
- **Patch 3/5**: Optimizes `--keep-pack` lookups with binary search, yielding an 11× speed-up for repositories with many kept packs.
- **Patch 4/5**: Adds `--keep-pack-from-file` to handle repositories where the number of kept packs exceeds command-line limits.
- **Patch 5/5**: Eliminates the race by passing `pack-objects` a snapshot of `.keep` packs observed at repack startup.

The series is well-tested, with each patch including a test that fails without it. The fix is production-tested and merges cleanly into `next` and `seen`, addressing a serious data-loss bug that would become critical once `--honor-pack-keep` is removed.

### "Safe" strbuf API RFC
Derrick Stolee posted an RFC series introducing a subset of the `strbuf` API that cannot call `die()` or `exit()`, motivated by trace2’s need to avoid recursive fatal errors during memory allocation failures. The series is structured in six patches:
1. Moves `strbuf` struct definitions to `strbuf-safe.h` to enable the safe API.
2. Proactively initializes `GIT_ALLOC_LIMIT` during Git’s startup to avoid `die()`-prone environment parsing.
3. Refactors `memory_limit_check()` to introduce `safe_memory_limit_check()`, which never calls `die()`.
4. Implements `sstrbuf_grow()`, the first safe method, using `srealloc()` and returning an error code.
5. Switches `json-writer.c` to include `strbuf-safe.h` to prepare for safe API adoption.
6. Adds `sstrbuf_init()` and `sstrbuf_release()` to the safe API and begins converting `json-writer.c` to use them.

The series is technically sound, with CodeQL used to verify the safety property. However, the transition is incomplete: `json-writer` methods are now "safe," but their callers (e.g., trace2) are not yet error-aware. Stolee plans to split this into two phases if the RFC is accepted. Key discussion points include the `sstrbuf_*` naming convention, the fate of `GIT_ALLOC_LIMIT`, and the transition plan for callers.

### Test modernization (v4)
Mark C. Chu-Carroll posted v4 of a test modernization series updating three legacy test scripts (`t4001-diff-rename.sh`, `t4009-diff-rename-4.sh`, `t4010-diff-pathspec.sh`) to use current Git test conventions. The changes are purely mechanical, improving readability and maintainability without altering behavior. Key improvements include:
- Replacing setup functions with `test_expect_success 'setup'` blocks.
- Standardizing capitalization in assertions and test names.
- Adopting `expect`/`actual` file naming and `<<-` here-doc syntax.
- Consistent tab-based indentation.

The series reduces the total line count by 20 across the three files and addresses all prior feedback from Junio C Hamano. The patches are uncontroversial and part of the ongoing "test modernization" effort.

## In brief
- **`git status --ignored` substring matching**: René Scharfe posted a patch fixing a regression where ignored directories were incorrectly matched when the pathspec was a substring of the directory name. The fix ensures `match_pathspec_with_flags()` is called for excluded directories with nested repositories.
- **`git rerere remaining` bugfix**: Junio C Hamano posted a patch fixing a bug where consecutive conflicted paths with only stage-1 entries were incorrectly skipped. The fix adds a same-path check (`ce_same_name()`) to `check_one_conflict()` in `rerere.c`.
- **`git diff --no-index -R` file/directory conflicts**: Haokai Ding posted a patch fixing a regression where file/directory conflicts were incorrectly reported as deletions or additions instead of being reversed. The fix swaps filespecs of the early queue entry when `reverse_diff` is set.
- **Git for Windows hardening**: Johannes Scharfe and Junio C Hamano debated a patch addressing GPG signature-prefix matching. Johannes argued for propagating a `slen` parameter to handle signatures with embedded NULs, while Junio proposed removing the parameter entirely, citing redundancy.
- **GitHub Actions Windows CI**: Junio noted that Rust is not preinstalled on GitLab’s Windows runners (`saas-windows-medium-amd64`), confirming Johannes Schindelin’s earlier question. This does not affect the merged series, which is scoped to GitHub Actions.