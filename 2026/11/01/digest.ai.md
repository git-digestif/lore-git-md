# Git mailing list daily digest for 2026/11/01

## The day in brief
The Git mailing list saw significant activity around bugfixes for large-file handling, CI compatibility fixes, and documentation updates. Key developments include Junio C Hamano's warning that the `git repo info` path-keys series will be dropped without reviewer engagement, multiple bugfix series addressing 4GiB+ file truncation issues, and a credential-cache daemon race condition fix. Several documentation cleanups and CI fixes also progressed toward integration.

## Notable threads

### Rust reorganization at risk of being dropped
The v5 patch series to reorganize Git's Rust infrastructure from `src/` to a dedicated `rust/` subdirectory remains at risk of being dropped. Junio C Hamano explicitly stated that Mike Hommey's reaffirmation of personal interest is not sufficient justification to retain the patch, citing lack of broader project enthusiasm and unresolved forward-compatibility concerns around Git 3.0's mandatory Rust components. The patch is technically complete and applies cleanly on Git 2.56-rc1, but without additional advocacy, it may be abandoned.

### `git repo info` path-keys series at risk
Junio C Hamano issued a final call for reviewers on the v7 patch series adding eight new path-related keys to `git repo info`. The series, which introduces keys like `path.toplevel`, `path.superproject-root`, and `path.cdup`, is mechanically complete and correctness-fixed but has received no substantive technical review in the past month. Without reviewer engagement in the coming weeks, Junio warned the topic will be dropped from his integration branches (`seen`). Justin Tobler committed to reviewing the series within the next day.

### `git branch --delete-integrated` series progresses
Harald Nordgren posted v6 of the four-patch series extending `git branch --delete-merged` with a new `--delete-integrated` flag. The series now addresses all prior feedback, including a critical `get_superproject_working_tree()` bug and proper `.gitattributes` support in the squash-detection logic. The first two patches (preparatory refactoring and landing-message helper) are explicitly marked as uncontroversial and ready for integration. The CLI design is finalized, with `--delete-merged` retaining its strict tip-reachability behavior and `--delete-integrated` opting in to change-based detection for rebased and squashed branches.

### Credential-cache daemon race condition fix
Farid Zakaria posted a two-patch series fixing a race condition in the Git credential-cache daemon where concurrent `git credential-cache store` commands could cause one daemon to unlink another's socket. The issue, discovered using Rewind (a deterministic VM for uncovering race conditions), leads to lost credentials and orphaned daemons during parallel submodule fetches. The patches make the daemon ignore `EEXIST` from `mkdir()` and use `unix_ss_create()` to safely handle concurrent socket creation. New tests in `t/t0301-credential-cache.sh` exercise both the `mkdir()` race and the socket hijacking race.

### Large-file handling bugfixes
Elia Pinto posted multiple bugfix series addressing silent truncation of files ≥4GiB on platforms where `long` is 32-bit but `size_t` is 64-bit (notably 64-bit Windows). The patches target:
- `git grep` and `git diff` (two-patch series)
- `git archive` (tar/zip formats) and `git apply` (two-patch series)
- `hashwrite()`, `count_lines()`, and `hash_chars()` (three-patch series)
- Binary patch hunk parsing in `git apply` (single patch)

Each series widens function signatures from 32-bit types to `size_t` to prevent truncation, with the `git diff` series opting to die with a clear error for files ≥4GiB rather than risk generating misleading patches. The fixes are well-motivated and include verification via simulated truncation at 16 bits, though no tests are included due to the impracticality of generating multi-gigabyte inputs.

### CI fixes for Ubuntu 26.04 compatibility
Harald Nordgren posted a three-patch series restoring four failing CI jobs after GitHub Actions updated `ubuntu-latest` to Ubuntu 26.04. The patches:
1. Switch from `apt` to `apt-get` in the `linux32` job to silence CLI interface warnings
2. Run `ln` with `sudo` to work around a permission issue when replacing `/usr/bin/dirname`
3. Pin the `linux32` job to Ubuntu 24.04 to bypass a temporary Docker networking issue

The fixes are minimal and pragmatic, addressing immediate breakage rather than redesigning the CI infrastructure. The series references prior work by Johannes Schindelin in Git for Windows as tested precedent.

### Documentation updates
Several documentation threads saw progress:
- Andy Koppe posted v2 of a five-patch series aligning Git's terminology around "pseudorefs" (FETCH_HEAD, MERGE_HEAD) and "root refs" (HEAD, ORIG_HEAD) with the 2024 redefinition. The series updates the glossary, man pages, code comments, and tests, incorporating Julia Evans's proposal to expand `gitdatamodel.adoc`.
- Omri Sarig posted v6 of a patch updating the `git-rebase` man page to correct outdated references in the "INCOMPATIBLE OPTIONS" section.
- Tuomas Ahola posted a patch updating `Documentation/CodingGuidelines` to reflect the bump in Git's minimum required Perl version from 5.8.1 to 5.26.0.

## In brief
- **[2026/10/18/23-55-07]** Matt Hunter proposed an alternative implementation for the `remote.<name>.followRemoteHEAD` lazy-validation series, folding the new logic into `get_follow_remote_head()` instead of introducing a standalone helper.
- **[2026/10/31/06-55-03]** Junio C Hamano signaled intent to integrate Linus Torvalds's patch fixing `git describe`'s commit distance calculation when commit timestamps are non-monotonic.
- **[2026/10/31/08-25-49]** Junio noted a test could be made hash-agnostic but signaled "Will queue" the patch fixing commit abbreviation in submodule diff headers.
- **[2026/10/31/10-06-42]** Junio reviewed a bugfix for valueless configuration handling in merge-ort, calling it "nicely done" and signaling approval.
- **[2026/11/01/06-39-51]** Okhuomon Ajayi posted a patch removing implicit use of `the_repository` in `wt-status.c` as part of the ongoing `the_repository` removal effort.
- **[2026/11/01/10-59-00]** Pushkar Singh posted a patch adding a Trace2 event for `git maintenance run --auto` skips due to lock, improving observability of silent skips.
- **[2026/11/01/18-58-46]** Tuomas Ahola posted a patch updating `CodingGuidelines` to reflect the recent bump in Git's minimum required Perl version.
- **[2026/11/01/21-06-23]** Cynthia Uche posted an Outreachy proposal draft to extend the `early_scan_options()` API to additional Git commands like `git column`, `git bisect`, and `git check-ref-format`.