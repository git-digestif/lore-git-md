# Git mailing list daily digest for 2026/10/09

## The day in brief

Git's global config handling inconsistency was resolved with a technically complete series that reads both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` when `--global` is specified. The `rerere` subsystem's race condition fix faced a substantive objection to its warning-and-proceed behavior. Documentation efforts advanced with a new `gitmergeconflicts(7)` guide nearing completion. Performance optimizations for `git push` refspec matching showed 10-100x speedups in large repositories.

## Notable threads

### Global config handling inconsistency fix
**What changed**: Delilah Ashley Wu's two-patch v3 series fixes Git's global config file handling inconsistency where `git config list --global` only showed `$HOME/.gitconfig` while Git actually reads both that and `$XDG_CONFIG_HOME/git/config` for unscoped operations.

**Problem/goal**: The series aligns behavior with documentation and the unscoped `git config list` command by making `--global` operations read both config files.

**Subsystem**: config handling (`config.c`, `builtin/config.c`)

**Impact**: The implementation reuses existing config-reading infrastructure, maintains backward compatibility, and fixes the long-standing inconsistency while preserving existing error handling behavior.

**Today's development**: [2026/10/09/08-51-44] The v3 cover letter confirms the series is now technically complete, addressing all prior feedback except minor structural refinements suggested in Junio's latest review.

### `rerere` race condition fix
**What changed**: Thomas Bachem's three-patch v6 series fixes a race condition in the `rerere` subsystem where concurrent writers contend for the shared `rr-cache` directory lock.

**Problem/goal**: The series makes all `rerere` writers wait for the lock with a configurable timeout, introduces a `--skip-locked` mode for `git rerere gc`, and allows conflict-stopping commands to proceed with a warning if the lock remains held after the timeout.

**Subsystem**: `rerere` (conflict resolution)

**Impact**: The fix addresses a long-standing robustness flaw exacerbated by Git 2.54's geometric maintenance strategy, which triggered `git rerere gc` more aggressively.

**Today's development**: [2026/10/09/09-06-18] Patrick Steinhardt raises a substantive objection to patch 3/3, describing the warning-and-proceed behavior for conflict-stopping commands as "fishy" and proposing it be deferred until real-world users encounter the issue.

### Documentation: `gitmergeconflicts(7)` guide
**What changed**: Julia Evans's six-patch v2 series introduces `gitmergeconflicts(7)`, a new man page consolidating Git's merge-conflict resolution guidance.

**Problem/goal**: The series replaces scattered, duplicated advice in existing man pages with a single, user-tested guide, improving clarity around "ours" vs. "theirs" and providing practical examples.

**Subsystem**: Documentation

**Impact**: The new guide is purely additive, with no behavior changes, and fills a clear gap by centralizing scattered advice on merge conflicts.

**Today's development**: [2026/10/09/18-53-58] Julia confirms she will incorporate Junio's latest refinements into v3, including the relocation of the `gitmergeconflicts(7)` cross-reference in `git-cherry-pick.adoc` to the `SEE ALSO` section.

### `git push` refspec matching performance
**What changed**: Jon Simons's 15-patch v1 series speeds up refspec matching in `git push` for scenarios with many deletions.

**Problem/goal**: The series achieves 10-100x speedups in benchmarked scenarios by replacing linear ref traversals with `strmap` lookups and fixing long-standing behavioral quirks in `refname_match()`.

**Subsystem**: push/refspec matching

**Impact**: The performance improvements target large-repository workflows, particularly when pushing many branch deletions to servers with large refsets (e.g., 100k refs).

**Today's development**: [2026/10/09/19-29-38] The 15-patch series achieves 10-100x speedups in benchmarked scenarios by replacing linear ref traversals with `strmap` lookups and fixing long-standing behavioral quirks in `refname_match()`.

## In brief

- **[2026/10/09/20-17-09]** Junio approves the second patch in the blame ignore-revs series, noting the `parse_oidset_line()` refactoring belongs in patch 1/2.
- **[2026/10/09/08-09-59]** Harald Nordgren accepts Junio's suggestions for the matching-inspired fetch mode series, including rewording the `git-fetch` man page and refactoring patch 2/4.
- **[2026/10/09/05-41-02]** Patrick Steinhardt distinguishes between small, easily reasoned changes (acceptable for AI assistance) and large, complex series (risky due to the author's limited context to vet AI-generated nuances).
- **[2026/10/09/21-21-40]** Karthik Nayak disengaged from reviewing the `reference-transaction` hook series because his feedback was consistently met with a new version (N+1) rather than substantive engagement with the previous round (N).
- **[2026/10/09/12-06-24]** Julia Evans declines to document the rationale behind `conflict-marker-size=32` in the new `gitmergeconflicts(7)` guide, arguing the topic is too niche for the target audience.
- **[2026/10/09/15-08-46]** The v3 mergesort test series is now structured as four patches: Clar migration, test simplification, edge-case coverage, and final cleanup.
- **[2026/10/09/14-00-05]** Muhammed Dilshad A's patch modifies `unpack-trees.c` to enforce untracked-file protection in sparse checkouts, ensuring the operation aborts rather than proceeding.
- **[2026/10/09/11-11-30]** Harald Nordgren proposes a new high-level command, `git full-clean`, that would combine `git branch --delete-merged` (with squash/rebase detection) and all pruning operations under sensible defaults.
- **[2026/10/09/19-05-15]** Junio clarifies that `git stash create` should eventually support all `stash push` options that create a stash, making the two commands interchangeable for scripting.
- **[2026/10/09/23-41-37]** Junio resolves the licensing ambiguity for the `sha1dc-accel` series by confirming Git's GPL-2-only stance is compatible with incorporating MIT-licensed code, provided the original license and copyright notices are preserved.
- **[2026/10/09/09-13-23]** Phillip Wood's v3 submission confirms all technical feedback from Junio on v2 has been addressed, including the correctness bug fix for `.git/MERGE_LABELS` writing.
- **[2026/10/09/17-34-01]** Sphinx proposes a safeguard mechanism for `git revert` that would refuse by default when attempting to edit a shallow boundary commit, with an override flag (`--allow-shallow-boundary`).
- **[2026/10/09/13-11-26]** Julia Evans's v2 patch updates the advice message to recommend `git merge --continue` instead of `git commit`, aligning it with the advice given for rebase, revert, and cherry-pick.
- **[2026/10/09/13-56-54]** Muhammed Dilshad A commits to simplifying the mergesort test setup and rewriting the commit message for clarity before sending v3.
- **[2026/10/09/10-22-13]** Patrick Steinhardt opposes the AI policy change on operational grounds, arguing that loosening the AI policy without addressing reviewer bandwidth would worsen the existing backlog of low-quality submissions.
- **[2026/10/09/13-29-27]** Phillip Wood endorses making revocation file errors fatal by default for SSH signing verification, using Git's `:(optional)` pathname prefix to allow an escape hatch.
- **[2026/10/09/22-31-29]** Karthik Nayak plans to revise the shallow-fetch regression test to directly verify that no objects are re-fetched during tag backfilling, using `git cat-file` to inspect the repository state.
- **[2026/10/09/02-38-08]** David Yang reports a bug where `git diff` produces distorted output compared to GNU `diff -u`, including a GitHub link to the problematic commit.
- **[2026/10/09/11-31-58]** Patrick Steinhardt's patch updates `t/t5004-archive-corner-cases.sh` to skip a test that extracts SHA-1 objects from a ZIP archive when running in a SHA-256 repository.
- **[2026/10/09/13-40-04]** Elia Pinto's patch updates `git status` and `git checkout` to clarify the message when a branch tracks a remote branch that does not exist, suggesting `git push` only when it would actually create the branch.
- **[2026/10/09/21-16-12]** Junio's review of the Windows credential helper fix flags potential issues with wide-character handling in the fix, particularly in pointer arithmetic and comparison operations on `LPCWSTR` types.
- **[2026/10/09/19-42-42]** Ramyres Aquino reports a bug where `git svn clone` fails with a `file://` URL on Windows, providing reproduction steps and error details.