# Git mailing list daily digest for 2026/10/09

## The day in brief
Git's global config handling inconsistency was resolved with a technically complete v3 series, while a critical `rerere` race condition fix faced new objections. Documentation improvements for merge conflicts gained maintainer approval, and a performance-focused `git push` series showed promising 10-100x speedups. Security issues in SSH signing verification and Windows credential parsing were identified, with fixes in progress.

## Notable threads

### Global config handling inconsistency fixed
**[2025/10/10/01-14-05]**
Delilah Ashley Wu's v3 series fixing Git's global config file handling inconsistency is now technically complete. The series ensures `git config list --global` reads both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`, aligning behavior with unscoped operations. All prior feedback has been addressed, including Windows path normalization concerns and structural refinements from Junio C Hamano. The implementation uses ignore flags and reuses `do_git_config_sequence()` to avoid duplicating config-file reading logic.

**What changed:** The v3 cover letter confirms the series is ready for integration, with all feedback incorporated except minor structural refinements.

**Impact:** This fixes a long-standing inconsistency while maintaining backward compatibility and preserving existing error handling behavior.

---

### `rerere` race condition fix faces new objections
**[2026/09/02/08-31-37]**
Patrick Steinhardt raised a substantive objection to patch 3/3 of Thomas Bachem's three-patch series fixing a race condition in the `rerere` subsystem. The patch implements warning-and-proceed behavior for conflict-stopping commands when the lock remains held after timeout. Steinhardt describes this behavior as "fishy" and proposes deferring the patch until real-world users encounter the issue, arguing it may be premature without evidence of user impact.

**What changed:** Steinhardt's objection introduces uncertainty about the necessity of patch 3/3, which was previously considered technically ready.

**Impact:** The series' architectural settlement is now in question, potentially delaying integration of the full fix.

---

### Merge conflict documentation improvements approved
**[2026/09/24/14-44-15]**
Julia Evans' documentation series introducing `gitmergeconflicts(7)` gained maintainer approval. The series consolidates scattered merge-conflict advice into a single, user-tested guide. Junio C Hamano approved patches 2/6, 3/6, and 6/6 for merging if unchanged in v3, and patches 4/6 and 5/6 with minor wording changes. The new guide improves clarity around "ours" vs. "theirs" terminology and promotes `merge.conflictstyle=diff3` as a best practice.

**What changed:** Junio's approval signals readiness for integration, with v3 addressing all remaining feedback.

**Impact:** This improves Git's documentation by centralizing scattered advice without altering behavior.

---

### Performance-focused `git push` series shows 10-100x speedups
**[2026/10/09/19-29-38]**
Jon Simons' 15-patch series targeting client-side refspec matching in `git push` demonstrates 10-100x speedups in benchmarked scenarios. The series replaces linear ref traversals with `strmap` lookups and fixes long-standing behavioral quirks in `refname_match()`. Key improvements include removing `mkpath()` overhead (55-82% of total speedup) and preparing for efficient `strmap` lookups in the final patch.

**What changed:** The series was submitted with detailed benchmark data and a disciplined test-before-code approach.

**Impact:** This significantly improves `git push` performance for large repositories, particularly in deletion workflows.

---

### SSH signing verification flaws identified
**[2026/10/08/06-56-22]**
Two critical flaws in Git's SSH signing verification were identified: (1) `valid-before` timestamps are checked against the committer date (which the signer controls), and (2) revocation file errors fail open instead of closed. Phillip Wood and Patrick Steinhardt agreed on fixes: document the `valid-before` behavior's limitations and make revocation file errors fatal by default, using Git's `:(optional)` pathname prefix for an escape hatch.

**What changed:** Design consensus was reached on actionable fixes for both issues.

**Impact:** These fixes harden Git's SSH signing verification, aligning it with the GPG backend's behavior.

---

### Windows credential helper bug identified
**[2026/10/09/13-45-18]**
A bug in the Windows credential helper (`contrib/credential/wincred/git-credential-wincred.c`) was reported, where multi-line secret blobs are parsed incorrectly. The current implementation uses unsafe string operations that can cause buffer overreads. Junio C Hamano's review raised concerns about wide-character handling correctness in the proposed fix.

**What changed:** The bug was identified and a fix proposed, though Junio's review suggests further refinement is needed.

**Impact:** This affects Windows users storing credentials with multi-line secrets (e.g., OAuth refresh tokens).

---

## In brief
- **[2026/09/11/23-29-44]** `git blame` default ignore-revs file: Junio approved the code for patches 2/2, with minor wording changes needed for patches 4/6 and 5/6.
- **[2026/09/14/11-31-17]** `git repo structure` revision filtering: Qin ShiCheng acknowledged Junio's suggestion to wait for `ps/odb-files-alternates` to stabilize before sending v4.
- **[2026/09/19/13-33-44]** `reference-transaction` hook: Patrick Steinhardt distinguished between small, easily reasoned changes (acceptable for AI assistance) and large, complex series (risky due to author's limited context).
- **[2026/09/29/07-30-30]** `git branch --delete-merged`: Harald Nordgren proposed a new `git full-clean` command combining branch cleanup and pruning operations.
- **[2026/09/29/11-25-40]** `sha1dc-accel`: Junio resolved licensing ambiguity, confirming Git's GPL-2-only stance is compatible with MIT-licensed code.
- **[2026/09/30/20-20-48]** `git checkout -m` conflict labels: Junio accepted Phillip Wood's v3 series for integration.
- **[2026/10/03/08-54-42]** Shallow clone safeguards: Sphinx proposed a safeguard mechanism for `git revert` to refuse by default when editing shallow boundary commits.
- **[2026/10/07/03-42-05]** Test infrastructure cleanup: Junio's feedback on commit message clarity was addressed in v3.
- **[2026/10/07/14-29-53]** CI housekeeping: Junio endorsed the SHA-256 coverage trade-off in patch 8/8.
- **[2026/10/08/22-10-37]** Shallow fetch regression: Karthik Nayak plans to revise the test to directly verify no objects are re-fetched during tag backfilling.
- **[2026/10/09/02-38-08]** `git diff` distortion: David Yang reported `git diff` producing distorted output compared to GNU `diff -u`.
- **[2026/10/09/13-40-04]** Upstream branch advice: Elia Pinto's patch updates `git status` to clarify messages when tracking a non-existent remote branch.
- **[2026/10/09/19-42-42]** `git svn clone` on Windows: Ramyres Aquino reported `git svn clone` failing with `file://` URLs on Windows.