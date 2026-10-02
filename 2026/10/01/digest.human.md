# Git mailing list daily digest for 2026/10/01

## The day in brief
The Git mailing list saw active discussion on several fronts today. A regression in `git filter-branch`'s `--state-branch` feature received final clarifications before merging, while a new `gitmergeconflicts(7)` man page sparked debate about embedding commit details in conflict markers. CI improvements for leak sanitizer reporting advanced, and a long-standing `rerere` race condition fix neared completion. The day also featured proposals for new features like `git restore-mtimes` and stash sharing through remotes, alongside design discussions about incremental rebase helpers and reference storage terminology.

## Notable threads

### `git filter-branch` state branch fix nears completion
The two-patch series fixing a commit mapping inversion regression in `git filter-branch --state-branch` received its final clarifications today. Michele Locati corrected a factual error about the state branch file's format (it stores *original-commit:rewritten-commit*, not the reverse), while Patrick Steinhardt proposed a rewritten commit message that names the regressing commit (f6d855091e) and uses imperative mood. The patch is now technically complete, with comprehensive test coverage that fails on current `master` without the fix and passes with it. The only remaining task is addressing a typo in the patch subject ("fiter-branch"), which Junio C Hamano may edit during merge or request a v3 to fix.

This bugfix targets a regression introduced in Git 2.50.0 that affects users relying on `--state-branch` for incremental repository splitting. The fix is minimal (two lines in `git-filter-branch.sh`) and preserves the documented "from_commit:to_commit" format, making it a strong candidate for merging.

### Conflict marker enhancements proposed for `gitmergeconflicts(7)`
The ongoing effort to introduce a dedicated `gitmergeconflicts(7)` man page took an innovative turn today with Patrick Steinhardt's proposal to embed commit details directly into conflict markers. The suggested format would transform markers from passive indicators into active resolution tools:
```
<<<<<<< HEAD: abcdefg (fruits: add apple, 2026-10-01)
    "cherry",
=======
    "banana",
>>>>>>> add-fruit: 12345678 (fruits: add banana, 2024-02-03)
```
Julia Evans enthusiastically endorsed the idea, highlighting the practical benefits of including commit messages and dates, which provide immediate context about conflicting changes. While she stopped short of fully embracing Patrick's vision of embedding commit IDs directly into markers, she suggested making commit information *discoverable* via commands like `git show HEAD` or `git show add-fruit`.

The discussion also gained momentum around making `merge.conflictstyle=diff3` the default, with both Patrick and Junio C Hamano strongly advocating for this change. Julia's observation that "every time I show people diff3 someone tells me how happy they are" reinforces its practical impact. These enhancements could significantly improve Git's conflict resolution experience and may foreshadow broader changes to default behavior.

### CI improvements for leak sanitizer reporting advance
Harald Nordgren's two-patch series improving GitHub Actions failure reporting for leak sanitizer jobs took a step forward today with a v4 iteration that simplifies escaping logic. The patch now only escapes percent signs (`%`) in annotations, dropping the previous attempt to escape carriage returns (`\r`), which resolves Phillip Wood's concern about overreach for single-line test descriptions.

The series addresses a real pain point for reviewers by making leak sanitizer failures and regular test failures point directly to their exact file and line in the test script. While the critical issue of clickable links in the GitHub Actions UI remains unresolved, the changes are well-motivated and narrowly scoped. The simplification in v4 reduces complexity without sacrificing functionality, making the series ready for broader testing in `next`.

### `rerere` race condition fix nears finalization
Thomas Bachem's three-patch series fixing a long-standing race condition in the `rerere` subsystem received its final design decisions today. The patch introduces `--skip-locked` mode for `git rerere gc` that skips a held lock silently, matching the behavior of other auto-maintenance tasks. The `RERERE_SKIP_LOCKED` flag has been renamed to `RERERE_WARN_LOCKED` to avoid confusion with the new `RERERE_NOWAIT` flag, and caller-specific documentation has been moved to `Documentation/config/rerere.adoc`.

This fix targets a general robustness flaw in `rerere` locking that was exacerbated by the geometric maintenance strategy introduced in Git 2.54. The series makes all `rerere` writers wait for the lock with a configurable timeout, while allowing conflict-stopping commands to proceed with a warning if the lock remains held. The changes are narrowly scoped to the `rerere` subsystem and address a real-world issue affecting users who rely on incremental repository maintenance.

### Reference transaction hook regression remains deprioritized
Maciej Ciemborowicz reported that they have deprioritized their bugfix for the reference-transaction hook regression in favor of addressing a separate but related issue (branch renaming). The v6 patch remains technically sound and ready for review, fixing a long-standing regression where high-level Git commands report all-zero OIDs for both old and new fields in the reference-transaction hook when deleting branches or tags.

The patch moves the fix into the transaction hook layer, resolving architectural concerns raised by Patrick Steinhardt and Junio C Hamano in earlier iterations. While the author now considers this bug a lower priority, the patch is still a strong candidate for merging as it resolves a persistent regression with a clean, layer-appropriate fix. The only open question is whether the on-demand resolution of old OIDs introduces unacceptable latency for large batches, but this is now considered an acceptable trade-off.

## In brief
- **`git stash create` extension**: Kazumasa Shigeta's patch extending `git stash create` to support `--include-untracked` (`-u`) and `--all` (`-a`) options received maintainer feedback requesting a rewritten commit message that focuses on *why* the patch exists rather than *what* it does. The implementation itself is uncontroversial and addresses a long-standing inconsistency with `git stash push` and `git stash save`.
- **MIDX reachability closure fixes**: Taylor Blau's eight-patch series fixing corner cases in Git's multi-pack-index (MIDX) and cruft-pack machinery received substantive feedback from Elijah Newren, who demonstrated that replacing an `oid_array` with an `oidset` would cause functional regressions in delta compression quality and attribute handling. The discussion now centers on whether the performance benefits of an `oidset` outweigh these functional regressions.
- **`git config --list --show-origin` inconsistency**: Patrick Steinhardt suggested an intermediate solution for the inconsistency where `.git/config` is shown as a relative path while other configuration sources show full paths. He proposed making the path relative to the current working directory when the user is not in the repository root, while flagging potential script compatibility concerns.
- **`git restore-mtimes` proposal**: Artem S. Tashkinov proposed a new command to preserve per-file historical modification timestamps derived from Git commit history. The proposal targets repositories with independently updated blobs where mtime loss is most acute, and asks whether this should be a standalone command, an option to existing commands, or integrated into `git archive`.
- **Stash sharing through remotes**: Hanan Arshad proposed a porcelain workflow for `git stash` to publish, list, import, and remove stashes through remotes. The proposal builds on existing `git stash export`/`import` plumbing and seeks feedback on remote ref namespace, naming scheme, listing behavior, and command naming.
- **Incremental rebase helpers**: The discussion about including an incremental rebase helper in Git's `contrib/` area converged on `git-bisect-rebase` as the leading candidate. Nico Williams clarified that the script's worst-case performance is O(N log N) when every upstream commit introduces conflicts, rather than the O(log N) one might expect from a pure bisection.
- **Integration branch updates**: Junio C Hamano's "What's cooking" report revealed that the next release is named Git 2.98 (not 2.57) to prepare for Git 3.0 in spring 2027. Nine topics graduated to `master`, including reflog expiry, reference storage terminology, ODB alternates, and the new `receive-report` hook.
- **`--no-overwrite-ignore` regression**: Coy Geek reported a regression in the `--no-overwrite-ignore` option where directory rename detection moves a tracked file into a path containing an ignored, untracked file, causing Git to silently overwrite the ignored file instead of aborting the merge. The behavior violates the documented contract of `--no-overwrite-ignore` and risks data loss.