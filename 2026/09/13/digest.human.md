# Git mailing list daily digest for 2026/09/13

## The day in brief
Junio C Hamano approved and fast-tracked a two-patch series enabling Rust compilation in Git’s GitHub Actions Windows CI jobs, resolving a key blocker for the broader Rustification effort. Three new bugfix series were posted, addressing a race in the ODB transaction layer, incorrect `git worktree repair` behavior, and Windows-specific test failures. The Coccinelle rule discussion shifted toward accepting compiler warnings for correctness, while the advice scope hint thread saw Junio propose an even simpler design.

## Notable threads

### Rust compilation in Windows CI now ready for integration
Johannes Schindelin’s two-patch series enabling Rust compilation in Git’s GitHub Actions Windows CI jobs is now ready for integration. The series configures the Rust toolchain to target the GCC ABI (MinGW) instead of the default MSVC ABI, ensuring Cargo produces a static library (`libgitcore.a`) compatible with Git for Windows’ linker. Junio C Hamano [2026/09/13/22-52-34] agreed to merge the series directly into `next` and fast-track it to `master`, bypassing the usual one-week cooking period due to the lower risk of Windows users building from source.

The patches touch `.github/workflows/main.yml`, `ci/lib.sh`, `config.mak.uname`, and the `Makefile`, but do not affect core Git functionality or user-facing features. The series removes the `NO_RUST` opt-out in CI, unblocking the broader Rustification effort. Testing on GitLab’s Windows runners remains an open request but is not a blocker.

### ODB transaction race fix posted
Justin Tobler [2026/09/13/20-26-20] posted a two-patch series fixing a race in the ODB transaction layer during commit when loose objects and large blobs are present with `core.fsync` batching enabled. The bug occurs when a large blob is written to a packfile in a transaction’s temporary directory, but the transaction migrates to the main ODB before the packfile is finalized, leaving it unflushed and causing a "No such file or directory" error.

The fix refactors `odb_maybe_flush_packfile()` to separate ODB reprepare logic from packfile flushing (patch 1/2) and adds a call to flush pending transaction packfiles before migration (patch 2/2). The series includes a new test in `t/t1050-large.sh` that reproduces the bug with a minimal repository setup. The patches are narrowly scoped to `object-file.c` and align with the ongoing ODB abstraction effort.

### Worktree repair validation added
Yoichi NAKAYAMA [2026/09/13/03-20-11] posted a two-patch series preventing `git worktree repair` from incorrectly modifying unrelated worktrees. The bug manifests when cross-references between a linked worktree and its administrative data are not validated before repair, risking corruption of unrelated worktrees in edge cases like manually swapped directories.

The series first refactors code to reduce redundant `.git` file reads and extract the worktree ID (patch 1/2), then adds validation logic using that ID to prevent repairs on unrelated worktrees (patch 2/2). The test script `t/t2406-worktree-repair.sh` is updated with two new test cases covering the validation. The changes are confined to `worktree.c` and the test script, with no broader behavioral changes.

### Windows test fixes for Perl and UTF-8
Johannes Schindelin [2026/09/13/19-11-05] posted a two-patch series fixing Windows-specific test failures that surfaced during full Windows/ARM64 test runs. The first patch adjusts Perl environment detection in `t/t9700/test.pl` to accommodate MSYS2 Perl now reporting itself as `cygwin`, while the second unconditionally skips UTF-8 tests in `t/t9129` on Windows due to fundamental encoding incompatibilities between Git for Windows and MSYS2 Perl.

The patches are narrowly scoped to the test suite and do not affect core Git functionality. The UTF-8 skip is justified by the irreconcilable mismatch between Windows "Code Pages" and Unix-style `LC_ALL` environment variables. The series resolves the only failures encountered during Windows/ARM64 testing in preparation for Git for Windows v2.56.0-rc0.

### Coccinelle rule discussion shifts toward compiler warnings
The discussion about removing a risky Coccinelle rule (`if (!E) free(E)`) took a substantive turn when René Scharfe [2026/09/13/10-45-48] argued that the project should tolerate compiler warnings for the sake of correctness. He noted that GCC accepts `if (!E);` without warning, while Clang warns, and that removing the guarded `free(E)` call entirely is preferable to preserving flawed code for build hygiene.

The thread remains unresolved, with no consensus on whether to adopt René’s blunt approach, Junio’s original rule removal, or another alternative. The discussion now hinges on the project’s tolerance for compiler warnings and the perceived risk of side effects from evaluating `E` in `if (!E)`.

### Advice scope hint design simplified further
Junio C Hamano [2026/09/13/16-32-37] proposed an even simpler design for Vsevolod Myalitsin’s advice scope hint series: the hint could *always* recommend `--global` without needing to mark individual advice settings for their intended scope. This would eliminate the need for the `is_global_hint` field entirely, further simplifying the design.

Junio clarified that the original 2009 design of the `advice.*` config system never intended the scope-less hint to be a literal cut-and-paste command, and that adding scope hints now is an enhancement rather than a bugfix. The proposal conflicts with Jeff King’s argument that some settings (like `defaultBranchName`) are inherently global, but Junio’s stance leaves the door open for a more minimal solution.

## In brief
- **Documentation backtick-quoting**: Todd Zullinger [2026/09/13/14-14-05] plans to drop the lint script change from his v1 series and resubmit v2 with only the mechanical backtick-quoting conversions for `git-refs` and `git-pack-refs`.
- **CMake UCRT64 update**: Johannes Schindelin [2026/09/13/20-41-12] posted a patch updating the CMake build system for Git for Windows’ migration from MINGW64 to UCRT64, ensuring `git.exe` locates its dependencies in the new `/ucrt64/` prefix.