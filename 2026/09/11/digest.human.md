# Git mailing list daily digest for 2026/09/11

## The day in brief
The Git project saw active discussion on several fronts today. The `git history` signing series received final surface-level feedback, while the `git repo info` path keys series faced new edge-case and refactoring requirements. ODB abstraction efforts continued with pluggable fsck checks and alternates handling. CI modernization threads progressed, and a new `--matched-only` option for `git range-diff` was finalized. Several bugfixes and refactoring efforts also advanced toward integration.

## Notable threads

### Teach `git history` to sign rewritten commits
**What changed**: Patrick Steinhardt provided surface-level review feedback on Souma's v2 patch series that teaches `git history` to sign rewritten commits using GPG.

**Problem/goal**: The series enables `git history` to sign commits created during operations like `drop`, `fixup`, `reword`, and `split` using the same configuration and command-line options already supported by `git commit`.

**Subsystem**: `git history` command, replay API, GPG signing.

**Impact**: Users can now maintain cryptographic signatures on rewritten commits, improving auditability in collaborative workflows. The series addresses all prior technical feedback and is close to ready for merging.

**Today's developments**: Steinhardt suggested minor documentation and style cleanups:
- Trim the commit message's in-body documentation of the `sign_commit` parameter in patch 1/3, as it is already documented in-code.
- Remove two redundant sentences from the commit message of patch 2/3: one about the `drop` edge case and another summarizing test coverage.
- Fix a style nit in the `OPT_HISTORY_GPG_SIGN` macro definition (backslash alignment).

### Add path-related keys to `git repo info`
**What changed**: Junio C Hamano identified two high-weight issues in K Jayatheerth's v6 series adding seven new path-related keys to `git repo info`.

**Problem/goal**: The series exposes filesystem locations of repository components (e.g., hooks directory, index file, superproject root) in a scriptable format via `git repo info`. The new keys include `path.cdup`, the inverse of `path.git-prefix`, which was added to address prior design objections.

**Subsystem**: `git repo info` command, repository path handling.

**Impact**: Scripts can now access repository paths without direct access to Git's internals. The series is feature-complete and mechanically sound but faces new requirements before merging.

### Today's developments

1. **Edge-case discrepancy in `path.cdup`**: When `GIT_WORK_TREE` and `GIT_DIR` are set but the current directory is outside the working tree, `path.cdup` returns an empty string while `git rev-parse --show-cdup` returns the absolute path to the working tree root. This undermines the series' goal of consolidating repository path information. The fix will require modifying `get_path_cdup()` to handle the outside-working-tree case, likely by falling back to `get_git_work_tree()` when `repo->prefix` is empty.
2. **Patch 2/7 refactoring**: Junio requested splitting the second patch (introducing `path.superproject-root`) into two: one to fix the `get_superproject_working_tree()` function (with tests in `t1500-rev-parse.sh`) and another to introduce the new keys (with tests in `t1900-repo-info.sh`). This isolates the correctness fix from the feature addition, improving clarity and bisectability.

### Make ODB fsck checks pluggable
**What changed**: Patrick Steinhardt posted v3 of his 10-patch series refactoring Git's object integrity verification (fsck) to make consistency checks pluggable per ODB backend.

**Problem/goal**: The series restructures fsck so that each ODB backend (loose, packed, multi-pack, etc.) can implement its own consistency checks instead of relying on a single monolithic fsck function in `builtin/fsck.c`. This prepares the codebase for future pluggable ODB backends.

**Subsystem**: ODB, fsck.

**Impact**: No user-visible behavior changes; this is a pure refactoring that paves the way for alternate storage systems. The series also fixes a long-standing discrepancy in the `--full` flag's behavior.

**Today's developments**: Steinhardt addressed the last unresolved feedback from Toon Claes:
- Moved the `ODB_FSCK_FULL` filtering logic from the central `odb_fsck()` helper into the "files" backend's `fsck` callback, making the infrastructure more flexible for non-local backends.
- Used early returns in `verify_midx()` for stylistic clarity.
- Replaced a potential `BUG()` assertion with a call to `prepare_repo_settings()` to ensure repository settings are initialized gracefully.
- Clarified that `ODB_FSCK_VERBOSE` and `ODB_FSCK_PROGRESS` are not mutually exclusive, as their outputs do not interfere.

The series is now complete and ready for further review or integration.

### Modernize GitLab CI's Asciidoctor installation
**What changed**: Junio C Hamano confirmed the merge status of Jeff King's patch to modernize GitLab CI's Asciidoctor installation.

**Problem/goal**: Replace the manual `gem install asciidoctor -v 1.5.8` with `apt-get install asciidoctor`, dropping the outdated version pin and relying on the Ubuntu image's default package (2.0.20). This aligns with the thread's broader goal of simplifying CI configuration.

**Subsystem**: CI/build system.

**Impact**: No user-visible changes; the patch ensures CI tests versions users are likely to have installed. The modernization effort is now complete.

**Today's developments**: Junio confirmed that the patch was merged in commit `47ce80527c` ("A bit more for -rc1"), which postdates the v2.56.0-rc0 tag. The final patch in the series (replacing the version-pinned installation) is now queued for application, completing the modernization.

### Add `--matched-only` option to `git range-diff`
**What changed**: Harald Nordgren posted the final version of a feature patch adding the `--matched-only` option to `git range-diff`.

**Problem/goal**: The new option filters `git range-diff` output to show only commits present in both input ranges, addressing the workflow pain point of reviewing rebases or range-diffs where users want to focus on how surviving commits changed rather than seeing added or dropped commits.

**Subsystem**: `git range-diff` command.

**Impact**: Users can now focus on the commits that survived a rebase or range-diff operation, improving the signal-to-noise ratio in review workflows. The implementation reuses existing suppression logic for `--left-only` and `--right-only`.

**Today's developments**: The patch was updated to incorporate Junio C Hamano's final wording for the man page update and mechanical consistency improvements (use of `die_for_incompatible_opt3()`). The series is now resolved and ready for integration.

## In brief
- **ODB alternates handling**: Patrick Steinhardt resolved the last review comment on his 9-patch series refactoring ODB alternates handling during repository creation. The series is now ready for `next`.
- **`--force-if-includes` bugfix**: Tyler Cipriani agreed to address all of Patrick Steinhardt's technical concerns in a forthcoming v4 of his series fixing `--force-if-includes` to check the correct reflog.
- **Rust CI for Windows**: Junio C Hamano and James Le Cuirot provided feedback on Johannes Schindelin's series enabling Rust compilation in Git's GitHub Actions Windows CI jobs. The series will be revised to address Cargo's native build behavior and variable naming.
- **`gitk` modernization**: Junio pulled Johannes Sixt's 8-patch series modernizing `gitk`'s color preference dialog and adding a README note about AI contributions.
- **Coccinelle rules**: Junio C Hamano removed a risky Coccinelle rule that converted `if (!E) free(E);` into an unconditional `free(E)` and added a new rule to explicitly allow unconditional `FREE_AND_NULL(E)` calls.
- **`git blame` ignore revs**: Ravi Mistry proposed making `git blame` automatically look for and use a `.git-blame-ignore-revs` file in the repository root if no explicit ignore file is configured.