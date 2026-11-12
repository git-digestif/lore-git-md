# Git mailing list daily digest for 2026/11/11

## The day in brief
The Git project made significant progress on several fronts today. Junio C Hamano endorsed an opt-in toggle for `git log --follow` to address performance regressions, settling a contentious debate. The `git rerere` subcommand modernization was completed with a final patch adding per-subcommand usage strings. Documentation improvements for `git diff` and detached HEAD state advanced, with minor refinements incorporated. Several refactoring efforts moved closer to integration, including the `the_repository` removal from `unpack-trees.c` and archive modules. Test infrastructure improvements and bug fixes also progressed, with a notable fix for a fragile test in the worktree subsystem.

## Notable threads

### `git log --follow` performance regression and opt-in toggle
**What changed**: Junio C Hamano endorsed an opt-in toggle for `git log --follow` in non-linear history, calling it "sensible" and "far better" than reverting the original commit or accepting the performance cost as permanent.

**Problem/goal**: The v5 series reduced a ~14× performance regression in `git log --follow` to ~1.6×, but the remaining gap was unacceptable for GitLab's operational use. The feature tracks file renames across history, particularly in merges.

**Subsystem**: `log-tree.c`, refspec logic, performance optimization

**Impact**: This decision preserves correctness fixes (e.g., `-m --follow` no longer losing history from non-first parents) while addressing deployment constraints for major Git hosts. The toggle will likely be implemented as `--follow=merges` and disabled by default.

**Key technical details**:
- The toggle will decouple from the v5 series to unblock GitLab's deployment
- Junio's endorsement settles the project-level policy debate
- The toggle design must accommodate future use cases like subtree merges

### `git rerere` subcommand modernization completed
**What changed**: The 12-patch series modernizing `git rerere`'s subcommand handling was completed with the final patch adding per-subcommand usage strings.

**Problem/goal**: Replace custom subcommand-parsing logic with Git's standard `OPT_SUBCOMMAND()` mechanism, bringing `git rerere` in line with other multi-subcommand built-ins.

**Subsystem**: `builtin/rerere.c`, command-line interface

**Impact**: This refactoring makes the codebase more maintainable and consistent with other Git commands. The series includes a `BUG()` check to catch invalid flag combinations and completes the `the_repository` removal for this command.

**Key technical details**:
- Final patch adds seven `N_()`-wrapped usage strings
- Each subcommand now has its own usage array
- The man page's `SYNOPSIS` section lists each subcommand on its own line
- Series merged to `master` as of 2025-06-25

### `git diff` documentation rewrite
**What changed**: Junio C Hamano reviewed the v3 patch and signaled intent to merge once the presentation format dispute is settled.

**Problem/goal**: Modernize the EXAMPLES section of the `git diff` man page to improve readability and replace outdated syntax.

**Subsystem**: Documentation, user interface

**Impact**: The revised documentation organizes common `git diff` invocations into five subsections, replaces the `...` syntax with `--merge-base`, and adds cross-references to option definitions. This makes the documentation more accessible to users.

**Key technical details**:
- Uses AsciiDoc horizontal definition lists instead of tables
- Replaces `...` syntax with `--merge-base` for clarity
- Adds cross-references to option definitions
- Includes both `--staged` and `--cached` as synonyms
- The only remaining blocker is Junio's objection to horizontal definition lists due to cut-and-paste issues

### `git history --verbose` design discussion
**What changed**: Patrick Steinhardt clarified that the output format must be generalizable to all `git history` subcommands, suggesting `git range-diff`'s structured format as a model.

**Problem/goal**: Add diff display support to `git history reword` to show the full diff while editing the commit message, mirroring `git commit --amend --verbose`.

**Subsystem**: `git history` (new built-in command), user interface

**Impact**: This feature would improve the user experience for interactive history rewriting. The discussion has shifted from user interface design to technical implementation, with a focus on forward compatibility.

**Key technical details**:
- The output format must accommodate four future use cases: changes to commit layout, commit order, commit messages, and updates to multiple references
- `git range-diff`'s structured, multi-commit format is proposed as a model
- The thread faces a tension between a lightweight unified diff and a structured format
- Six distinct positions remain on opt-in vs. opt-out for diff display

### `the_repository` removal from `unpack-trees.c`
**What changed**: Patrick Steinhardt and Derrick Stolee provided non-blocking endorsements, confirming the series is technically sound.

**Problem/goal**: Remove the `USE_THE_REPOSITORY` macro from `unpack-trees.c` as part of the ongoing `the_repository` removal effort.

**Subsystem**: `unpack-trees.c`, object database

**Impact**: This refactoring makes the codebase more maintainable by eliminating implicit global state. The series is queued for `next` and ready for integration.

**Key technical details**:
- Introduces `unpack_repo()` helper with temporary paranoid consistency checks
- Mechanical rename of local variables named `index` to `istate` for clarity
- No user-visible behavior changes
- The paranoid checks will be revisited once the broader effort stabilizes

## In brief

- **`post-worktree` hook series**: Anders Kaseorg reported a fragile `sed` substitution in `t2401-worktree-prune.sh` that has already caused a test failure in Ubuntu's Git package build.
- **`git repo info` path-keys series**: K Jayatheerth agreed to drop three contested keys (`path.hooks`, `path.cdup`, and `path.toplevel`/`path.superproject-root`) in response to Phillip Wood's objections.
- **`git branch --delete-integrated`**: Jasper Mayone identified performance inefficiencies and correctness limitations in the squash-detection implementation.
- **`git worktree list --merged`**: The v5 series introduced a hashmap for O(1) lookups and added the `%(currentworktree)` atom for conditional formatting.
- **`git mktree --batch` test coverage**: Junio C Hamano queued Roger Koranteng's v2 patch adding test coverage for the `--batch` mode.
- **`git describe` regression fix**: Oswald Buddenhagen provided historical context on the root cause of the regression.
- **ODB loose object backend refactoring**: Patrick Steinhardt accepted Kaartic Sivaraam's suggestion to preserve documentation comments in the new backend file.
- **GitLab CI speedups**: The v3 series documented the `bsdtar`/`libarchive` dependency in `ci/install-sdk.ps1` to address Junio's portability concern.
- **APFS/mmap metadata quirk**: Johannes Sixt added that Git's mmap emulation on Windows exhibits the same delayed timestamp update behavior.
- **`git repack --drop-filtered`**: kongfankaikai proposed that the command should skip blobs referenced by the index and continue processing.
- **Test fix for `t2401-worktree-prune.sh`**: Junio C Hamano provided an Acked-by for Anders Kaseorg's fix to a brittle `sed` substitution.