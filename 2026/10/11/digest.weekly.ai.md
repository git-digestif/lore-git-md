# The Git Project -- Weekly Digest for 2026/10/05 -- 2026/10/11

## The period in brief
This week (2026/10/05--2026/10/11) saw **580 emails** across six active days, with traffic slightly above the recent weekly average. The tone was **eventful and occasionally contentious**, dominated by three major arcs: the **Git 3.0 SHA-256 transition debate** sparked by Scott Chacon’s challenge, **AI-assisted contribution policy discussions** following Maciej Ciemborowicz’s disclosure, and **performance and correctness fixes** for long-standing issues in `git repack`, `git blame`, and `git rerere`. Readers should not miss the **Git 3.0 planning notes** from the Contributor’s Summit, the **final approval of `git blame`’s default ignore-revs file**, and the **abandonment of the trace2 hardening effort** after substantive objections.

---

## Key developments

### Git 3.0 planning: two-stage rollout confirmed, SHA-256 and Rust inclusion settled
The Git Contributor’s Summit notes, shared by Kaartic Sivaraam, confirmed the **two-stage rollout plan** for Git 3.0 (2.99 → 3.0) and settled key technical decisions. The 3.0 release will differ only in flipped defaults for breaking changes, minimizing disruption. **SHA-256 support** is confirmed for GitHub (GA November 2026) and GitLab (already public), but interoperability between SHA-1 and SHA-256 repositories remains incomplete on brian m. carlson’s personal branch. **Rust support** drew no objections, but no decision was made on whether it will be mandatory or merely allowed in 3.0. **LTS status** for 2.99 was proposed (Gentoo expressed interest), but no consensus was reached. The notes also highlighted funding gaps for JGit and gitoxide SHA-256 support, with Google declining JGit and gitoxide’s status uncertain.

The discussion revealed **broad consensus on the technical direction** but lingering questions about **downstream synchronization** and **latent bug exposure** during SHA-256 adoption. Follow-up threads on each technical track (SHA-256 interoperability, Rust integration, LTS planning) are expected in the coming weeks.

---

### Git 3.0 SHA-256 transition: strategic debate escalates
Scott Chacon **escalated the debate** over Git 3.0’s SHA-256 transition plan, arguing it is a **GitHub-driven effort lacking broad community buy-in** and proposing his **tree-digest signing mechanism** as a less disruptive alternative. Chacon’s proposal—embedding SHA-256 digests of tree contents in signed commits/tags—would satisfy regulatory mandates for signatures without switching the default object format. brian m. carlson and Patrick Steinhardt countered that the transition is **necessary for regulatory compliance and ecosystem momentum**, with Steinhardt warning that abandoning it would **ossify the ecosystem**, making future breaking changes exponentially harder.

The thread exposed a **governance fault line**: Chacon framed the transition as a **corporate priority** (GitHub’s internal timeline conflicts with the project’s), while Carlson emphasized **personal contributions** and **institutional demand**. The discussion remains unresolved, with Junio C Hamano yet to weigh in. The outcome will determine whether Git 3.0 proceeds with the full SHA-256 transition or adopts a hybrid approach.

---

### AI-assisted contributions: policy debate stalls on legal and process concerns
Scott Chacon’s proposal to **explicitly permit AI-assisted contributions** under clear conditions sparked a **multi-day debate** about enforceability, legal risks, and reviewer bandwidth. The patch introduced an `Assisted-by:` trailer for substantial AI assistance, requiring contributors to review, test, and sign off on submissions. **brian m. carlson** opposed the change on **legal and ethical grounds**, citing risks of license violations and copyright infringement from AI-generated code. **Patrick Steinhardt** and **Junio C Hamano** raised **operational concerns**, noting that loosening the AI policy without addressing reviewer bandwidth would worsen the existing backlog of low-quality submissions.

Maciej Ciemborowicz’s disclosure of a **seven-step AI-assisted workflow**—relying on AI agents for code generation, testing, and review—further polarized the discussion. Ciemborowicz framed his participation as a **moral dilemma**, while maintainers reinforced the expectation that authors engage substantively with reviewer feedback. The thread remains stalled, with no clear path forward. The debate highlights the tension between **productivity tools** and **project norms**, particularly around authorship and review culture.

---

### `git blame` default ignore-revs file lands in `next`
Ravi Mistry’s three-patch series teaching `git blame` to automatically use `HEAD:.git-blame-ignore-revs` as a default ignore-revs file **reached integration-ready status** after Junio C Hamano approved the v3 revision. The series addresses security concerns by **rejecting malformed input** (NUL bytes) and **preventing promisor fetches** in partial clones. The default file lookup logic resolves the blob via `get_oid_with_context()` and skips non-regular tree entries (e.g., committed symlinks). User-configured ignore files take precedence, and empty configurations bypass the default blob entirely.

The feature aligns `git blame` with other Git commands that support ignore-revs files, improving workflows for large-scale code archaeology. The hardening measures—particularly the promisor fetch prevention—address long-standing security concerns about parsing upstream-controlled content. The series is now in `next` and expected to graduate to `master` in the coming weeks.

---

### `rerere` race condition fix finalized after dropping controversial behavior
Thomas Bachem’s two-patch v7 series fixing a race condition in the `rerere` subsystem **reached final form** after dropping the controversial third patch. The series introduces a **configurable timeout** (`rerere.lockTimeout`, defaulting to 1000 ms) for the `MERGE_RR.lock` file, allowing processes to wait briefly for the lock instead of failing immediately. Patch 2 adds a **hidden `--skip-locked` flag** to `git rerere gc`, enabling auto-maintenance tasks to skip silently if the lock is held. The decision to drop the warning-and-proceed behavior reflects feedback from Patrick Steinhardt, who argued it was unnecessary without real-world user reports.

The series addresses a **long-standing robustness flaw** exacerbated by Git 2.54’s geometric maintenance strategy, which triggered `git rerere gc` more aggressively. The fix is conservative, prioritizing correctness over user-facing behavior changes. The series is technically complete and awaits Junio’s final review.

---

### Trace2 hardening abandoned after substantive objections
Derrick Stolee **abandoned the RFC series** (`ds/trace2-tolerate-failed-timestamp`) that aimed to eliminate `die()`-prone helpers in the trace2 subsystem. The series introduced `banned-die.h` to enforce a no-`die()` policy but was deemed **too complex for the benefits it provided**. Jeff King (Peff) had proposed a **ground-up rewrite of trace2** as the only way to fully eliminate `die()`-prone dependencies, but this was not pursued due to complexity.

The thread’s pivot reflects the **challenges of hardening low-level subsystems** without introducing excessive complexity. The core goal—preventing trace2 crashes—remains, but the path forward is now less certain. The abandonment underscores the trade-offs between **defensive programming** and **maintainability** in Git’s codebase.

---

### CI improvements: leak sanitizer v6 resolves annotation link regression
Harald Nordgren’s v6 of the CI leak sanitizer series **resolved the annotation link regression** from v5 by embedding file and line context directly in annotation text. The compromise avoids the earlier issue where annotation links were broken unless the test file was modified in the same pull request. The series now:
1. Stops leak-sanitizer scripts at the first failure (`--immediate`) and annotates leaks with the test script name and sanitizer output.
2. Preserves job-output links while embedding context in the annotation text (e.g., `memory leak logged in t1060 (t1060-object-corruption.sh)`).

Phillip Wood accepted the compromise, and the series is now a candidate for `next`. The improvements target GitHub Actions CI usability, particularly for large test suites.

---

### `git repack` data-loss race fix advances with snapshot-based approach
Qeesung’s v3 series fixing a data-loss race in `git repack -d` **addressed Junio C Hamano’s layering concern** by moving the kept-pack cache invalidation logic from `builtin/pack-objects.c` to `packfile.c`. The series replaces the racy `--honor-pack-keep` mechanism with a **snapshot-based approach**, ensuring `.keep` files are managed consistently. The final patch (5/5) eliminates the race by passing `pack-objects` a snapshot of `.keep` packs observed at repack startup via `--keep-pack-from-file`, replacing the mid-repack directory scan that could miss concurrent `.keep` file creation.

Junio has blocked integration pending stabilization of Patrick Steinhardt’s `ps/odb-files-alternates` topic, preferring not to carry a temporary merge resolution in `seen`. The series is technically complete and addresses a **long-standing correctness issue** in Git’s repack machinery.

---

### Performance optimizations: packed-refs and refspec matching
Two performance optimization series advanced this week:
1. **Karthik Nayak’s packed-refs backend optimization** speeds up operations that rewrite the packed-refs file (e.g., deleting references) by avoiding redundant formatting of unchanged refs. The patch tracks raw byte positions of unchanged refs and writes them directly with `fwrite()`, bypassing `fprintf()`. The optimization reduces `fetch/write-commit-graph` time by ~20% in synthetic benchmarks and is ready for maintainer pickup.
2. **Jon Simons’s refspec matching optimization** for `git push` achieves 10-100x speedups in large repositories by replacing linear ref traversals with `strmap` lookups. The 15-patch series fixes long-standing behavioral quirks in `refname_match()` and targets workflows with many deletions (e.g., 100k refs). The series is technically complete and awaits Junio’s review.

Both series demonstrate Git’s **ongoing focus on large-repository performance**, particularly for operations involving many refs.

---

## In brief

- **`git history` signing**: Souma’s patch series (v5) to teach `git history` to sign rewritten commits is **unblocked** after resolving a completion test regression. The series is cooking in `seen` and ready to graduate to `next`.
- **Shallow clone safeguards**: Patrick Steinhardt proposed that Git **warn or refuse by default** when users attempt destructive operations on shallow boundary commits. Junio C Hamano clarified that shallow clones **preserve original commit IDs** but logically rewrite boundary commits as root commits during traversal. The discussion is ongoing.
- **ZIP timestamp bugs**: René Scharfe posted a patch omitting the Extended Timestamp (UT) extra field for out-of-range commit times, preventing silent timestamp wrap. The fix prioritizes correctness over tool compatibility and is ready for review.
- **`git fast-import` crash fix**: Aniket Gupta confirmed a fix for crashes when resolving unreachable in-session commits. The patch adds a cache lookup in `parse_from_existing` and is ready for integration.
- **Global config handling**: Delilah Ashley Wu’s v3 series fixes Git’s global config file handling inconsistency, making `--global` operations read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`. The series is technically complete and ready for Junio’s final review.
- **`gitmergeconflicts(7)` guide**: Julia Evans’s six-patch v2 series introduces a new man page consolidating Git’s merge-conflict resolution guidance. The series is nearing completion, with Junio’s latest refinements to be incorporated into v3.
- **Outreachy mentor reorganization**: Git’s participation in the Outreachy December 2026 cohort was finalized with a mentor reorganization. The project will proceed with three approved ideas but can only mentor two interns due to mentor availability.
- **CI infrastructure debate**: Patrick Steinhardt pushed back against Junio C Hamano’s proposal to remove the `linux32` and `linux-TEST-vars` jobs, arguing that i386 coverage and exotic configuration testing remain valuable. The discussion highlights the trade-offs between CI maintenance burden and test coverage.
- **`git stash create` redesign**: Kazumasa Shigeta and Phillip Wood narrowed the scope of proposed changes to `--include-untracked` and `--all`, ruling out `--patch` and positional pathspecs. The discussion remains unresolved.
- **`git fetch --prune-tags` bugfix**: Orgad Shaneh gently pinged Junio C Hamano for a substantive response on the series aligning implementation with documentation. The thread remains stalled.
- **`git checkout -m` conflict labels**: Phillip Wood’s v2 series to preserve original conflict labels is **ready for integration**, pending resolution of the conflict style question.
- **`git fetch` incremental connectivity check**: Patrick Steinhardt and Junio C Hamano discussed trust boundary detection and error handling, with Junio suggesting **non-ref roots** (e.g., index entries) for connectivity checks.
- **Test modernization**: Muhammed Dilshad A updated `t/t0450-txt-doc-vs-help.sh` to use Git’s helper functions and posted a three-patch series migrating mergesort tests to Clar. Both series are ready for review.
- **`--relative` with combined diffs**: Muhammed Dilshad A posted v2 of a series fixing `--relative` handling in combined diffs, addressing path normalization and rename handling.
- **Windows performance regression**: Daniel Gullberg reported a performance regression in `safe.directory` ownership checks with unreachable UNC paths, causing 25-second stalls on Windows.
- **`git repo structure`**: Mark C. Chu-Carroll’s v4 series adds revision-based filtering support, but remains blocked by unaddressed feedback on commit message motivation and the double ref-walk inefficiency.
- **`git(1)` man page rewrite**: D. Ben Knoble approved Julia Evans’ v2 revision, which incorporates feedback to use `git help push` as an alternative to `git push --help`.
- **Push negotiation bug**: Jens Röcker reported a bug where a pre-push hook repacking the local object database causes Git to resend common history. Patrick Steinhardt proposed a fix.
- **`git status`/`git push` enhancements**: Harald Nordgren posted a six-patch series to better handle rebased push branches, introducing more granular tracking of divergence and adjusting advice messages.
- **Clock-skew warning**: Junio C Hamano raised fundamental objections to Devi Srinivas Vasamsetti’s patch, questioning its usefulness and noting the warning cannot address the root problem.
- **`git worktree repair` validation**: Yoichi NAKAYAMA agreed to split patch 1/2 into smaller commits to separate behavioral changes from refactoring.
- **`git branch --delete-merged` squash detection**: Harald Nordgren posted v2 of the series, adding a `--no-squashed` flag to skip slower squash detection.
- **OpenSSL compatibility patch for `imap-send`**: Xiangfei Ding posted a patch updating `imap-send.c` to stop assuming `ASN1_STRING` data is null-terminated, addressing an upcoming breaking change in OpenSSL derivatives.
- **Documentation update**: Jon Simons and Junio C Hamano discussed wording for a `SubmittingPatches` update discouraging the "add failing test, then fix" anti-pattern.

---

## Looking ahead
The next week is likely to see continued discussion on **Git 3.0 planning**, particularly around **SHA-256 interoperability** and **Rust integration**. The **AI-assisted contributions policy debate** may see further input from Junio C Hamano or the Project Leadership Committee, though no resolution is imminent. **Performance optimization series** (packed-refs, refspec matching) are expected to advance toward integration, as are **correctness fixes** for `git repack` and `git rerere`. The **`git blame` default ignore-revs file** and **`rerere` race condition fix** are both in `next` and may graduate to `master`. Finally, the **Outreachy project selection** process will conclude, with interns expected to begin work in early November.