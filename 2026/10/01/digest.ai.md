# Git mailing list daily digest for 2026/10/01

## The day in brief
The Git mailing list saw significant activity around **bugfixes for long-standing regressions**, including a `filter-branch` commit mapping inversion and a reference-transaction hook OID reporting issue. **Documentation improvements** advanced with new conflict-resolution guidance and tutorial modernization discussions. **Performance optimizations** for packed-refs and CI failure reporting were refined, while **new features** like `@{p}` shorthand and stash sharing workflows were proposed. The maintainer’s "What’s cooking" report highlighted progress toward Git 3.0, with nine topics graduating to `master`.

---

## Notable threads

### 1. `filter-branch`: Fix commit mapping inversion with `--state-branch`
**What changed?**
Grant Moyer’s v2 patch fixes a regression introduced in Git 2.50.0 where `git filter-branch --state-branch` stored commit mappings in "from_commit:to_commit" format but read them back as "to_commit:from_commit". This caused the map directory to be populated backwards, leading to "Is a directory" errors when `--prune-empty` was used.

**Why it matters**
The bug affects users who rely on `--state-branch` for incremental repository splitting. The fix is minimal (two lines in `git-filter-branch.sh`) and includes expanded test coverage, including a new test case from Michele Locati that directly verifies the commit mapping correctness in incremental runs with `--prune-empty` and `--subdirectory-filter`.

**Today’s developments**
- [2026/10/01/05-36-10] Patrick Steinhardt proposed a rewritten commit message naming the regressing commit (f6d855091e) and using imperative mood.
- [2026/10/01/06-33-06] Michele Locati corrected a factual error: the state branch file’s format is *original-commit:rewritten-commit*, not the reverse, and added his `Signed-off-by` as a co-author.

**Status**
The patch is technically complete, with comprehensive test coverage. The only remaining tasks are clarifying the commit message and addressing a typo in the patch subject ("fiter-branch"). Junio has not indicated whether he will edit the subject during merge or request a v3.

---

### 2. `rerere`: Fix race condition between rebase and background maintenance
**What changed?**
Thomas Bachem’s three-patch series makes `rerere` wait for the `MERGE_RR.lock` with a configurable timeout (`rerere.lockTimeout`) instead of failing immediately. The series also introduces a `--skip-locked` mode for `git rerere gc` to skip held locks silently, matching the behavior of other auto-maintenance tasks.

**Why it matters**
The race condition was exacerbated by the geometric maintenance strategy in Git 2.54, which triggers `git rerere gc` more aggressively. The fix ensures that conflict-stopping commands (e.g., `git rebase`) can proceed with a warning if the lock remains held after the timeout, while post-resolution recording and explicit `git rerere` commands still fail immediately.

**Today’s developments**
- [2026/10/01/08-08-16] Thomas Bachem accepted Patrick Steinhardt’s feedback to rename `--auto` to `--skip-locked` (hidden/undocumented) and move caller-specific documentation to `Documentation/config/rerere.adoc`.

**Status**
The v5 series is ready for final review, with all prior feedback incorporated. Patch 2 will be updated to v6 to address the feedback. The series is well-motivated and narrowly scoped, addressing a real race condition with clear test coverage.

---

### 3. Reference-transaction hook: Fix all-zero OIDs on branch/tag deletion
**What changed?**
Maciej Ciemborowicz’s v6 patch fixes a regression introduced in Git 2.31 where high-level commands (`git branch -d`, `git tag -d`, `git remote prune`, `git fetch --prune`) report all-zero OIDs for both old and new fields in the reference-transaction hook when deleting branches or tags. The patch moves the fix into the transaction hook layer, ensuring the hook always receives the correct old OID without altering transaction semantics.

**Why it matters**
The regression broke external tools (Gerrit, GitLab, CI systems) that rely on the hook to monitor ref changes. The v6 approach resolves architectural concerns raised by Patrick Steinhardt and Junio C Hamano, preserving unconditional deletion semantics while restoring pre-regression behavior.

**Today’s developments**
- [2026/10/01/17-37-05] The author deprioritized this bugfix in favor of addressing a separate but related issue (branch renaming), but the v6 patch remains technically sound and ready for review.

**Status**
The patch is well-tested, with updated regression tests covering batched deletions, remote pruning, and concurrent updates. The only open question is whether the on-demand resolution of old OIDs introduces unacceptable latency for large batches, but this is now considered an acceptable trade-off.

---

### 4. Documentation: Introduce `gitmergeconflicts(7)` man page
**What changed?**
Julia Evans’ patch series introduces a new `gitmergeconflicts(7)` man page to consolidate merge conflict resolution guidance previously scattered across multiple command man pages. The guide is organized as a practical, example-driven document with sections covering when conflicts occur, resolution paths, conflict markers, tools, and terminology.

**Why it matters**
The current documentation duplicates advice across `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull`, leading to omissions and user confusion. The new guide improves clarity (e.g., clarifying "ours" vs. "theirs") and provides practical examples based on user feedback.

**Today’s developments**
- [2026/10/01/05-14-03] Patrick Steinhardt proposed embedding commit details (IDs, messages, dates) directly into conflict markers, transforming them into active resolution tools.
- [2026/10/01/12-10-11] Julia Evans enthusiastically endorsed Patrick’s proposal, highlighting the practical benefits of including commit messages and dates.

**Status**
The series is under review, with the first patch merged into Junio’s `seen` branch for integration testing. The discussion has elevated two concrete improvements: (1) commit accessibility and (2) promoting `merge.conflictstyle=diff3` as a best practice. The guide is expected to actively promote these enhancements, and the discussion may foreshadow a broader shift in Git’s default behavior.

---

### 5. CI: Improve leak sanitizer failure reporting in GitHub Actions
**What changed?**
Harald Nordgren’s two-patch series improves GitHub Actions failure reporting by linking leak sanitizer failures and regular test failures directly to the test script line where they occur. The first patch stops leak-sanitizer scripts at the first failure (using `--immediate`) and annotates leaks with the test script name and full sanitizer output in a collapsible log group. The second patch ensures regular test failures and fixed known breakages point to their exact file and line in the test script.

**Why it matters**
The current CI output buries failures at the end of test runs, producing only a generic "Process completed with exit code 1" message. The series provides actionable failure annotations while balancing early failure (simpler debugging) and comprehensive reporting (catching all leaks).

**Today’s developments**
- [2026/10/01/18-44-13] Harald Nordgren simplified the escaping logic in v4: `github_escape_message_` now only escapes percent signs (`%`), dropping the previous attempt to escape carriage returns (`\r`).
- [2026/10/01/19-49-08] Phillip Wood identified an escaping consistency issue: the patch escapes percent signs in new annotations but not in existing ones, which could cause GitHub to misparse those annotations.

**Status**
The series is cooking in `next` and appears ready for broader testing. The changes are well-motivated, narrowly scoped, and address a real pain point for reviewers. The remaining open questions are usability refinements rather than correctness issues.

---

## In brief
- **`gittutorial-2` removal**: Julia Evans’ series to remove the obsolete `gittutorial-2` document was merged to `master`, with a follow-up patch addressing a build break in `seen`. The thread now centers on whether the older `gittutorial` should be preserved or removed as part of a broader cleanup. [2026/10/01/22-21-30]
- **`git stash create`**: Kazumasa Shigeta’s patch extending `git stash create` to support `--include-untracked` (`-u`) and `--all` (`-a`) options received substantive review from Junio C Hamano, who requested the commit message be rewritten to focus on *why* the patch exists rather than *what* it does. [2026/10/01/17-03-13]
- **`git config --list --show-origin`**: Patrick Steinhardt suggested an intermediate solution for displaying `.git/config` as a relative path when the user is not in the repository root, flagging potential script compatibility concerns. [2026/10/01/06-37-01]
- **`@p` shorthand**: Junio C Hamano provided historical context for the 2015 `@{push}` introduction, noting the asymmetry with `@{u}` was already noticed in 2014 but not acted upon. The patch to add `@{p}` as a shorthand for `@{push}` remains queued in `seen`. [2026/10/01/03-29-50]
- **`git stash pop` conflict labels**: Junio C Hamano and Phillip Wood questioned the fundamental motivation for extending custom conflict-label options to `git stash pop`, given their origin as internal machinery for `git checkout -m`. [2026/10/01/17-58-40]
- **Unexpected history divergence**: A user reported unexpected history divergence requiring force-pushes, asking whether this could originate from Git’s reference tracking, fetch logic, or background updates. [2026/10/01/07-19-56]
- **Stash sharing workflow**: Hanan Arshad proposed a porcelain workflow for sharing stashes through remotes, seeking feedback on remote ref namespace, user-provided names vs. object-derived identifiers, listing behavior, and command naming. [2026/10/01/08-53-09]
- **`git restore-mtimes`**: Artem S. Tashkinov proposed a new command to preserve per-file historical modification timestamps derived from Git commit history, asking whether this should be a standalone command, an option to existing commands, or integrated into `git archive`. [2026/10/01/09-43-25]
- **Subcommand grouping**: Patrick Steinhardt’s series introducing subcommand grouping to the parse-options API and applying it to `git refs` received structural feedback from Junio C Hamano, who suggested refactoring the option-formatting logic into a helper function. [2026/10/01/17-46-24]
- **Incremental rebase helpers**: The discussion about including a helper script for incremental rebase conflict resolution in `contrib/` converged on `git-bisect-rebase` as the leading candidate, with Nico Williams endorsing Alejandro Colomar’s simplified implementation. [2026/10/01/21-01-01]
- **`--no-overwrite-ignore` regression**: Coy Geek reported a regression in the `--no-overwrite-ignore` option where directory rename detection moves a tracked file into a path containing an ignored, untracked file, causing Git to silently overwrite the ignored file instead of aborting the merge. [2026/10/01/23-54-12]
- **What’s cooking**: Junio C Hamano’s report highlighted progress toward Git 3.0, with nine topics graduating to `master` and 25 cooking in `next`. Notable graduations include reflog expiry, reference storage terminology, ODB alternates, and the new `receive-report` hook. [2026/10/01/22-48-09]