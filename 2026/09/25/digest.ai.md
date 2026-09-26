# Git mailing list daily digest for 2026/09/25

## The day in brief
The Git mailing list saw significant progress on several fronts today. A regression in `git shortlog` error reporting was fixed, with maintainer Junio C Hamano confirming the solution. The `http.sslVerifyStatus` security feature thread closed its diagnostic phase after SZEDER Gábor confirmed the test failure was caused by using the GnuTLS backend for libcurl. Harald Nordgren's matching-inspired fetch mode for shallow repositories received substantive review, with Junio raising design concerns about automatically fetching the remote's default branch. Documentation efforts continued, with the `gitmergeconflicts(7)` guide making progress on technical accuracy and the cross-reference modernization patch receiving final approval.

## Notable threads

### `http.sslVerifyStatus` test infrastructure follow-up
The diagnostic phase for the `http.sslVerifyStatus` security feature has concluded. SZEDER Gábor confirmed that the test failure was caused by using the GnuTLS backend for libcurl, which lacks full OCSP stapling support. Switching to the OpenSSL backend resolves the issue, and the test suite now passes. Junio C Hamano acknowledged this finding, noting that the root cause is environmental rather than a flaw in the test logic. The thread's focus remains on post-merge test robustness, with no new technical concerns introduced.

The feature, which adds OCSP staple validation to Git's HTTPS certificate verification, was merged to `master` in June 2025 but reverted from `next` due to test infrastructure issues. The core functionality remains intact, and the revert was procedural. This resolution removes the last blocker for the feature to graduate to the next release cycle.

### `--force-if-includes` reflog walk fallback value discussion
The ongoing debate about the fallback timestamp value for `--force-if-includes` reached a critical juncture. Tyler Cipriani provided a substantive review advocating for `0` (epoch start) as the fallback value, arguing that any cutoff risks rejecting valid pushes when matching reflog entries are older than the cutoff. He demonstrated that the performance penalty for `0` is negligible in practice because both local and remote reflogs are typically pruned by `gc`, limiting the reflog walk to the same 90-day window either way.

Junio C Hamano responded with skepticism about the necessity of scanning all reflog entries, reasserting his preference for a 90-day or `gc.reflogExpire` fallback. The unresolved tension between safety (`0`) and performance (cutoff) remains the sole obstacle to integration. The patch is otherwise technically complete, with all prior review feedback addressed.

This discussion highlights a fundamental design trade-off in Git's safety mechanisms. The outcome will determine whether the patch graduates to `next` as-is or prompts a follow-up adjustment to the fallback value.

### Matching-inspired fetch mode for shallow repositories
Harald Nordgren's v3 series implementing the `remote.<name>.refmap`-based "matching-inspired" fetch mode for shallow repositories received substantive review. The series introduces a new fetch mode that dynamically fetches only the branches tracked by local branches at the remote, plus the remote's default branch, addressing the performance problem in shallow repositories where a refspec-less fetch would otherwise negotiate history for every remote branch.

Junio C Hamano raised two significant concerns:
1. Documentation wording for `remote.<name>.refmap` should avoid implying the configuration is inactive in edge cases.
2. Design concern about automatically fetching the remote's default branch, arguing it is overly presumptive and may be wasteful or redundant for some users.

The first concern is a documentation clarification that can be addressed in a follow-up patch. The second concern is more fundamental, questioning whether the forced inclusion of the remote's default branch is justified. This could lead to a significant design change in the series' core behavior.

The series is well-structured and directly targets a real user pain point, but this design question may require a v4 iteration before it can proceed to `next`.

### `gitmergeconflicts(7)` documentation improvements
The documentation effort to introduce `gitmergeconflicts(7)` made significant progress on technical accuracy. The thread focused on clarifying the behavior of `git log --merge -p <path>`, with Junio C Hamano correcting a misunderstanding about the command's output. The `--merge` option implies a `HEAD...<other>` revision range, where `<other>` is context-dependent (e.g., `MERGE_HEAD`, `REBASE_HEAD`, or `CHERRY_PICK_HEAD`). D. Ben Knoble supplemented this with the mechanism of the `--merge` option, explaining that it constructs a symmetric difference traversal.

The approved wording for the guide is now: "will print out all commits which caused the merge conflict for `<filename>`, and the diff of how they changed the file." This clarification ensures the guide provides accurate and practical advice for users resolving conflicts.

The series also received substantive review from D. Ben Knoble, who validated key design choices and provided real-world context for terminology and edge cases. The discussion remains focused on content accuracy and completeness, with mechanical integration issues (e.g., `command-list.txt` registration) being the primary blockers.

### Regression fix for `git shortlog` error reporting
A regression in `git shortlog` and related revision commands, where unknown options were incorrectly reported as `(null)` instead of their actual name, was fixed by Jeff King (Peff). The issue was introduced by commit cd439487 ("revision: manage memory ownership of argv in setup_revisions()", 2025-09-19) and affected all current integration branches.

Junio C Hamano confirmed the fix, providing a simple reproduction test that demonstrates the correct behavior. The series consists of two patches:
1. A minimal fix for a latent bug where known-but-invalid options were misreported as unknown.
2. A broader fix that restores the pre-cd439487 behavior by conditionally nulling `*value` only when `opt->free_removed_argv_elements` is set.

The patches are small, well-motivated, and now have maintainer approval, making them strong candidates for inclusion in the next release.

## In brief
- **Git 3.0 planning**: Kristoffer Haugsbakk proposed a configuration analysis tool to recommend modern Git settings as a less disruptive alternative to changing defaults.
- **Precompiled headers**: SZEDER Gábor addressed an edge case where target-specific `EXTRA_CPPFLAGS` invalidate the precompiled header, proposing a workaround.
- **CI improvements**: Tamir Duberstein provided benchmark data for dynamic parallelism in CI jobs, showing a 2% speedup with `1x CPU count` and a 7% speedup with `2x CPU count`.
- **Documentation cross-reference modernization**: Jeff King approved the v2 patch, which is now ready for integration.
- **`git history` bug**: Nikita Bobko reported that `git history fixup` updates branch refs locked during interactive rebase, causing Git to refuse further ref modifications.
- **Leak sanitizer reporting**: Harald Nordgren improved leak sanitizer failure reporting in GitHub Actions by stopping the test script at the first leak failure and emitting annotations.
- **`git replay` GPG signing**: Patrick Monette added GPG commit signing support to `git replay` via a new `-S` option.
- **Packfile tempfile cleanup**: Royce Remer registered temporary pack files with Git's tempfile subsystem to ensure cleanup on process exit.
- **Line-log whitespace trimming**: Kristofer Karlsson updated the line-log patch to trim trailing blank lines (whitespace-only) from function ranges, aligning with `git grep -W` behavior.
- **JGit pluggable backend validation**: Luca Milanesio provided production-scale validation of JGit's pluggable backend architecture, citing Google's internal Git servers and GerritForge's global refdb backend.