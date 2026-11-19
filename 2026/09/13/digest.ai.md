# Git mailing list daily digest for 2026/09/13

## Long threads

### Advice scoping mechanism for `advice.defaultBranchName`

*Long thread (root 2027/08/29/00-49-58): 20 replies so far from 4 participants over 5 days; 1 message that day.*

This thread concerns the advice infrastructure in `advice.c`: the hint shown when initializing a repository suggests disabling the default-branch-name advice with `git config set advice.defaultBranchName false`, but without an explicit scope that command writes to the current repository's configuration, which is useless because the advice appears during `git init` and will not repeat for that repository. The series passes the entire `advice_setting` structure to `vadvise()`, introduces a configuration scope hint for advice settings, and suggests the global configuration scope for the `defaultBranchName` advice so that disabling it applies to future repositories as well. Earlier, the v4 three-patch series was posted, build failures were reported and discussed, and Vsevolod Myalitsin proposed choosing between local and global via a boolean flag rather than supporting `--system` or `--worktree` scopes until a proven need exists. Today, Junio C Hamano agreed that adding `--global` to the advice hint is acceptable since users can still cut-and-paste and tweak it, but clarified that the original intent was for users to decide the scope themselves, and suggested it may not be necessary to mark each advice individually for scope since the message could simply always say `--global` without changing anything else.

## The day in brief

Junio C Hamano judged the v4 series enabling Rust in the Windows CI jobs ready and decided to merge it into `next` and fast-track it to `master` immediately. New series were posted covering a fix for ODB transaction commit failures under batch fsync, validation in `git worktree repair` against modifying unrelated `.git` files and gitdirs, a CMake change for Git for Windows' UCRT64 migration, and Perl-related test fixes on Windows. The advice-scoping discussion continued with agreement that a `--global` hint is acceptable.

## Notable threads

### Using Rust in the Windows CI jobs

Since v2.55.0 Git requires Rust by default, but the Windows CI jobs opted out because each job downloads a minimal Git for Windows SDK containing the GCC toolchain, and Rust is not part of that SDK; the GitHub runners, however, come with Rust pre-installed, configured for the Visual C compiler, so the series picks a GCC-compatible Cargo target under MSYS2/MinGW and switches the Windows CI to build with Rust. Today, Johannes Schindelin confirmed that `CARGO_BUILD_TARGET` works for picking the GCC-compatible target once explicitly exported, since the Makefile does not export it automatically; he posted v3 switching from `CARGO_TARGET`/`--target` to `CARGO_BUILD_TARGET`, then v4 whose only change is removing the now-incorrect commit-message paragraph about `--target`, again asking GitLab CI users to test the series on `saas-windows-medium-amd64`. Junio C Hamano judged v4 ready and decided to merge it in `next` and fast-track it down to `master` immediately, reasoning that Windows has an unusually high ratio of binary users relative to source builders, so the usual cooking period in `next` would benefit it less than other platforms.

### Validating targets in `git worktree repair`

`git worktree repair` in `worktree.c` does not sufficiently validate the cross-references between a linked working tree and its administrative data before repairing them, which can cause the repair to modify the wrong `.git` file or gitdir. Today, Yoichi NAKAYAMA submitted a two-patch series: the first refactors `repair_worktree_at_path()` to read the `.git` file once via `read_gitfile_raw()`, extracts `get_worktree_id()` to obtain the worktree ID from the `.git` file contents, and replaces the `READ_GITFILE_ERR_NOT_A_REPO` handling with an `is_git_directory()` check; the second adds validation so that `repair_gitfile()` verifies the worktree ID recorded in the `.git` file matches the worktree being repaired, preventing corruption when two worktree directories are swapped without `git worktree move`, and `repair_worktree_at_path()` stops the repair when the gitdir path cannot be inferred, preventing corruption of an unrelated gitdir when repairing a copied linked worktree.

### Flushing the transaction packfile before migrating objects in `object-file.c`

This series fixes a commit failure for an ODB transaction containing both a loose object and a "large" blob when `core.fsyncMethod` is `batch`: the transaction's temporary directory is migrated to the main ODB before the packfile is flushed, producing an "unable to rename temporary file" fatal error. Today, Justin Tobler posted the two-patch series: patch 1/2 lifts the `odb_reprepare()` call out of `flush_packfile_transaction()` and requires callers to invoke it explicitly, adding explicit calls in `odb_transaction_files_write_object_stream()` and `odb_transaction_files_commit()`; patch 2/2 moves the `flush_packfile_transaction()` call in `odb_transaction_files_commit()` to occur before objects are migrated to the main ODB, and adds a test case in `t/t1050-large.sh` verifying that adding a large file alongside a loose object under batch fsync leaves no objects in temporary locations.

### CMake Windows build accommodating the UCRT64 migration

Git for Windows is switching from the deprecated MINGW64 toolchain to UCRT64 with v2.56.0, which installs `git.exe` into `/ucrt64/` instead of `/mingw64/`. Today, Johannes Schindelin submitted a one-line patch to `contrib/buildsystems/CMakeLists.txt` changing the x64 Windows CMake build to define `ENSURE_MSYSTEM_IS_SET="UCRT64"` and `MINGW_PREFIX="ucrt64"` instead of the previous MINGW64/mingw64 values, noting the change is required for git-for-windows/git's vs-build CI job though not for core Git, and that it could not be squashed into the earlier series because that series had to land before the migration.

### Fixing Perl-related test failures on Windows

Perl-related tests are skipped in the CI builds for performance reasons and because git svn support was retired in Git for Windows, which caused bit-rot; running the full test suite on Windows/ARM64 while preparing Git for Windows v2.56.0-rc0 surfaced exactly two bugs. Today, Johannes Schindelin submitted a two-patch series: in `t/t9700/test.pl` the platform check changes from `$^O eq 'msys'` to `defined $ENV{MSYSTEM} && $ENV{MSYSTEM} ne 'MSYS'` because MSYS2 Perl now reports `$^O` as `cygwin`, and using `cygwin` would break in actual Cygwin; in `t/t9129-git-svn-i18n-commitencoding.sh` the UTF-8 test cases gain the `!MINGW` prerequisite because Git for Windows (native Win32 using Code Pages) and its MSYS2 Perl interpreter (using Unix locale environment variables) have incompatible encoding conversion concepts.

## In brief

- Todd Zullinger's documentation lint series for `git-refs` and `git-pack-refs` met an objection from Jean-Noël AVILA, who explained the lint regex was deliberately restricted to options because backticked terms are meant to be immutable for translators and extending the match to commands triggers false positives in files such as `git-add.adoc` and `git-push.adoc`; Zullinger said he does not mind dropping the lint patch if the false positives would be more annoying, and would wait before re-rolling without it.
- René Scharfe discussed the Coccinelle `free.cocci` rule and the risky `if (!E) free(E);` pattern, noting GCC accepts `if (!E);` while Clang warns and both accept `if (!E) {}`, and agreeing that the polite course is to leave such flawed code in place rather than add a targeted defense against an inconsequential mistake.
- Orgad Shaneh pinged Junio C Hamano for a response on adding tag content verification to the tests for the `fetch --prune-tags` clobbering series; Junio C Hamano replied that he does not have the earlier discussion in his context window, has no immediate comment, and will respond only after swapping the context back in, but not today.
