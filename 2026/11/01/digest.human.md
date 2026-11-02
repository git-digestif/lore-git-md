# Git mailing list daily digest for 2026/11/01

## The day in brief
The Git mailing list saw significant activity around bugfixes for large-file handling, CI compatibility fixes, and documentation updates. Key developments include Junio Hamano's warning that the `git repo info` path-keys series will be dropped without reviewer engagement, multiple bugfix series addressing 4GiB+ file truncation issues, and a credential-cache daemon race condition fix. Several documentation cleanups and CI fixes also progressed toward integration.

## Notable threads

### Rust reorganization at risk of being dropped
The v5 patch series to reorganize Git's Rust infrastructure into a dedicated `rust/` subdirectory remains at risk of being dropped. Junio C Hamano explicitly stated that Mike Hommey's reaffirmation of personal interest is not sufficient justification to retain the patch, given the lack of broader project enthusiasm and unresolved forward-compatibility concerns around Git 3.0's mandatory Rust components. The patch is technically complete and applies cleanly on Git 2.56-rc1, but without additional advocacy, it will likely be abandoned.

What this changes: The reorganization would improve project structure clarity in Git's mixed C/Rust codebase and support downstream projects that vendor Git's C code while excluding its Rust components. The patch updates the Makefile, meson.build, .gitignore, and CI scripts to reference the new `rust/` locations.

### `git repo info` path-keys series at risk of abandonment
Junio C Hamano issued a final call for reviewers on the v7 patch series adding eight new path-related keys to `git repo info`. The series, which introduces keys like `path.toplevel`, `path.superproject-root`, and `path.cdup`, is mechanically complete and correctness-fixed but has received no substantive technical review in the month since v7 was posted. Without reviewer engagement in the coming weeks, Junio warned the topic will be dropped from his integration branches (`seen`). Justin Tobler committed to reviewing the series within the next day, which may be crucial for its survival.

What this changes: The series exposes filesystem locations of repository components in a scriptable format, enabling more robust automation around repository paths. The new keys support suffix-based formatting (e.g., `path.toplevel.absolute` vs `path.toplevel.relative`) and handle edge cases like bare repositories and submodules.

### Credential-cache daemon race condition fix
Farid Zakaria posted a two-patch series fixing a race condition in the Git credential-cache daemon that could cause lost credentials and orphaned daemons during parallel submodule fetches. The patches address two distinct parts of the race window: concurrent socket directory creation (patch 1) and socket hijacking (patch 2). The fix uses the `unix_ss_create()` helper from the simple-ipc subsystem to safely handle concurrent socket creation, and includes new tests in `t/t0301-credential-cache.sh` to verify the behavior.

What this changes: The credential-cache daemon will now ignore `EEXIST` from `mkdir()` and gracefully exit if it detects an existing socket, preventing credential loss and orphaned processes. This is particularly relevant for users with large repositories that perform parallel submodule fetches over HTTPS.

### Large-file handling bugfixes
Elia Pinto posted multiple bugfix series addressing silent truncation of files 4GiB or larger on platforms where `long` is 32 bits but `size_t` is 64 bits (notably 64-bit Windows). The patches target several subsystems:

- A two-patch series fixes `git archive` (tar/zip formats) and `git apply`
- A two-patch series fixes `git grep` and `git diff`
- A three-patch series fixes integer truncation in `hashwrite()`, `count_lines()`, and `hash_chars()`
- A standalone patch fixes integer overflow in `parse_binary_hunk()` for the `apply` subsystem

What this changes: These fixes prevent silent data corruption when processing large files, ensuring Git correctly handles files up to `SIZE_MAX` without truncation. The changes are particularly relevant for users working with large binary files or repositories containing files ≥4GiB.

### CI fixes for Ubuntu 26.04 compatibility
Harald Nordgren posted a three-patch series restoring four failing CI jobs after GitHub Actions updated the `ubuntu-latest` runner image to Ubuntu 26.04. The patches address CLI interface warnings from `apt` vs `apt-get`, a permission error when creating symlinks during dependency installation, and a temporary Docker networking issue in the `linux32` job. The fixes are minimal and targeted, avoiding broader CI redesign.

What this changes: The CI jobs for `linux32`, `documentation`, `fuzz-smoke`, and `static-analysis` will now pass on the updated runner image, maintaining test coverage and preventing false negatives in the CI pipeline.

## In brief
- **[2026/09/29/07-30-30]** Harald Nordgren posted v6 of the `--delete-integrated` series for `git branch`, incorporating all prior feedback and marking the first two patches as uncontroversial and ready for integration.
- **[2026/10/18/23-55-07]** Matt Hunter proposed an alternative implementation for the `remote.<name>.followRemoteHEAD` lazy-validation series, suggesting folding the new logic into `get_follow_remote_head()` instead of introducing a standalone helper.
- **[2026/10/25/11-07-59]** Omri Sarig posted v6 of the rebase man page update, addressing a commit message wording ambiguity from the v5 review.
- **[2026/10/29/20-39-03]** Edson posted v6 of the `diff-files` parse-options conversion, fixing the broken threading issue noted in prior versions.
- **[2026/10/31/06-55-03]** Junio C Hamano signaled intent to integrate Linus Torvalds's `git describe` commit-distance bugfix after resolving a test implementation debate.
- **[2026/10/31/08-25-49]** Junio noted a hash-agnostic test improvement for the submodule diff header commit abbreviation fix but signaled "Will queue" the patch.
- **[2026/10/31/10-06-42]** Junio reviewed the merge-ort valueless configuration handling fix, calling it "nicely done" and signaling approval.
- **[2026/11/01/06-39-51]** Okhuomon Ajayi posted a patch removing implicit use of `the_repository` in `wt-status.c` as part of the ongoing libification effort.
- **[2026/11/01/10-59-00]** Pushkar Singh posted a patch adding a Trace2 event for `--auto` skip due to lock in the maintenance subsystem.
- **[2026/11/01/18-58-46]** Tuomas Ahola posted a documentation patch updating `CodingGuidelines` to reflect the recent bump in Git's minimum required Perl version to 5.26.0.
- **[2026/11/01/21-06-23]** Cynthia Uche posted an Outreachy proposal draft to extend the `early_scan_options()` API to additional Git commands like `git column`, `git bisect`, and `git check-ref-format`.