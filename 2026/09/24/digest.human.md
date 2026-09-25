# Git mailing list daily digest for 2026/09/24

## The day in brief
The Git project moved closer to resolving a long-standing reference-transaction hook regression with a v6 patch series that shifts the fix into the transaction layer, while a new `gitmergeconflicts(7)` man page aims to centralize merge conflict guidance. CI and build system improvements dominated the day, with fixes for Windows unit tests, Meson build optimizations, and Rust compatibility on Cygwin. The Git 3.0 timeline saw further clarification, with Junio C Hamano emphasizing its tentative nature and Ramsay Jones reporting successful Rust compilation on Cygwin.

## Notable threads

### Reference-transaction hook regression fix (v6)
**What changed**: Maciej Ciemborowicz posted v6 of the series fixing a regression where the reference-transaction hook receives all-zero OIDs when branches or tags are deleted via high-level commands.

**Problem/goal**: The regression, introduced in Git 2.31, broke external tools (Gerrit, GitLab, CI systems) that rely on the hook to monitor ref changes, as the old OID was lost.

**Subsystem**: Refs, reference-transaction hook, transaction system.

**Kind of change**: Bugfix (regression).

**Impact**: The v6 series abandons the v5 approach of modifying `refs_delete_refs()` and instead moves the fix into the transaction hook layer itself. The new design records a separate "observed old value" for updates where callers didn't supply one, using this value only as hook input without setting `REF_HAVE_OLD` or constraining the update. This ensures the hook always receives the correct old OID for all ref-modifying commands, restoring pre-2.31 behavior.

**Today's developments**: The v6 series was posted, addressing Patrick Steinhardt's architectural concerns by keeping `refs_delete_refs()` as a simple wrapper for unconditional deletions. The patch adds a new mechanism in the transaction layer to resolve and cache old OIDs on demand, storing them in a separate field that doesn't influence transaction safety checks. The series includes updated regression tests and is rebased onto `master`.

**Why it matters**: This fix resolves a persistent regression that has affected external tools for over a year. The architectural pivot to the transaction layer aligns with Patrick's feedback and avoids overloading `refs_delete_refs()`, making it a cleaner, more maintainable solution. The only open question is whether the on-demand resolution of old OIDs introduces unacceptable latency for large batches, which may prompt further discussion.

---

### Git 3.0 timeline and Rust portability
**What changed**: Junio C Hamano clarified that the Git 3.0 timeline is tentative, and Ramsay Jones reported successful Rust compilation on Cygwin.

**Problem/goal**: The Git project is planning a major version bump to Git 3.0, with Rust support as a key goal. However, portability concerns (e.g., NonStop platform) have raised questions about whether Rust can be included in the release.

**Subsystem**: Build system, Rust integration, CI.

**Kind of change**: Meta-discussion (version numbering, portability).

**Impact**: Junio proposed a three-phase timeline (Git 2.98 in December 2026, Git 2.99 in March 2027, Git 3.0 in April 2027) but emphasized that the stabilization period for Git 2.99 may require additional maintenance releases (e.g., 2.99.2 or 2.99.3) before Git 3.0 is finalized. This introduces uncertainty for downstream projects, which must now account for potential delays when aligning their roadmaps. Meanwhile, Ramsay Jones provided concrete evidence that Git's experimental Rust support compiles successfully on Cygwin using an unofficial toolchain, with no new test failures. This is the first public data point on Rust's viability outside mainstream platforms, though NonStop remains an open question.

### Today's developments

- Junio restated the provisional timeline but framed the April 2027 target for Git 3.0 as tentative, deferring the final decision until later in 2026.
- Johannes Schindelin confirmed that the timeline provides sufficient clarity for Git for Windows' roadmap, tying it to Rust-based builds and Git Credential Manager v3.0 integration.
- Ramsay Jones reported that Git's Rust support compiles successfully on Cygwin using an unofficial, unmaintained Rust toolchain (version 1.91.0). The build passes all static checks and matches the non-Rust test suite baseline, with no new regressions. The toolchain's experimental status and lack of official support remain caveats.

**Why it matters**: The Git 3.0 timeline is now explicitly flexible, which may ease pressure on the project but requires downstream projects to plan for potential delays. Ramsay's report provides the first concrete evidence that Rust can compile on non-mainstream platforms, though the toolchain's experimental status limits its immediate practicality. This data point is encouraging for the broader Rust-in-Git effort but does not resolve all portability concerns (e.g., NonStop). The discussion underscores the tension between the project's goals (Rust support in Git 3.0) and the realities of platform compatibility.

---

### `gitmergeconflicts(7)`: A new man page for merge conflict resolution
**What changed**: Julia Evans posted a seven-patch series introducing `gitmergeconflicts(7)`, a new man page consolidating merge conflict resolution guidance.

**Problem/goal**: Git's current documentation scatters merge-conflict advice across multiple command man pages (`git merge`, `git rebase`, `git revert`, `git cherry-pick`, `git pull`), leading to duplication, omissions, and user confusion. The new guide aims to centralize this information, improve clarity (e.g., clarifying "ours" vs. "theirs"), and provide practical examples based on user feedback.

**Subsystem**: Documentation.

**Kind of change**: Documentation improvement.

**Impact**: The series adds a new `gitmergeconflicts(7)` man page and updates existing command man pages to link to it instead of duplicating conflict-resolution advice. The new guide is organized as a practical, example-driven document with sections covering when conflicts occur, resolution paths, merge conflict markers, tools for handling conflicts, and edge cases (e.g., `git rebase` inverts "ours" and "theirs"). The series also updates the build system to include the new page and adds it to `.gitattributes` for conflict marker detection.

### Today's developments

- The series was posted and merged into Junio's `seen` branch for integration testing, where it triggered a `make check-docs` failure due to the new page not being registered in the documentation linting infrastructure.
- Jeff King identified the root cause: the new `gitmergeconflicts(7)` page is not listed in `command-list.txt`, which the `ta/command-list-guides-sync-lint` topic uses to validate documentation links.
- Junio requested mechanical adjustments to the first patch, including updating `Documentation/meson.build` and folding the `.gitattributes` addition into the first patch to avoid treating it as an afterthought.

**Why it matters**: This series addresses a long-standing gap in Git's documentation by centralizing merge conflict resolution guidance in a single, user-friendly resource. The new guide's example-driven structure and focus on practical workflows align with recent efforts to improve documentation clarity (e.g., Jean-Noël Avila's synopsis-style conversion). The series is well-motivated and likely to improve the user experience for both new and experienced Git users. The only remaining work is mechanical integration (e.g., updating `command-list.txt`), which is straightforward and uncontroversial.

---

### CI and build system improvements
**What changed**: Several patches addressed CI and build system issues, including fixes for Windows unit tests, Meson build optimizations, and Rust compatibility.

**Problem/goal**: Git's CI and build systems have faced intermittent failures and inefficiencies, particularly on Windows and in the Meson build. The patches aim to improve reliability, performance, and compatibility.

**Subsystem**: CI, build system, Windows.

**Kind of change**: CI/build system improvements.

### Impact

- **Windows unit tests**: Karthik Nayak fixed a regression where unit tests were being skipped on Windows CI due to a mismatch in test-slice indexing. The patch updates `ci/run-test-slice.sh` to check for slice "1" instead of "0", aligning it with the one-indexed slice scheme introduced in commit 3141df7ec4.
- **Meson build optimizations**: Patrick Steinhardt posted a seven-patch series optimizing Meson build times, fixing shell completion test failures, updating subproject wrappers, and resolving GitLab CI hangs in MSVC jobs. The series reduces clean build times by ~35% (from ~6.8s to ~5.0s) by avoiding redundant compilation of HTTP sources, using precompiled headers for test helpers and unit tests, and fixing outdated completion helpers. The final patch disables PortableGit's credential helper to prevent MSVC job hangs.
- **Rust compatibility**: Ramsay Jones reported that Git's experimental Rust support compiles successfully on Cygwin using an unofficial toolchain, with no new test failures. This provides the first public data point on Rust's viability outside mainstream platforms.

### Today's developments

- Karthik Nayak's patch fixing Windows unit tests received a tested, substantive review from Patrick Steinhardt, who confirmed the fix aligns with the one-indexed slice scheme.
- Patrick Steinhardt's Meson build optimization series was posted, with each patch targeting a specific inefficiency (e.g., redundant HTTP source compilation, precompiled headers for test helpers, outdated completion helpers). The series also includes a fix for hanging MSVC jobs in GitLab CI.
- Ramsay Jones reported successful Rust compilation on Cygwin, noting that the toolchain is unofficial and unmaintained but that the result is "slightly encouraging."

**Why it matters**: These improvements address real pain points in Git's CI and build systems, particularly for Windows and Meson users. The Meson build optimizations are especially impactful, as they measurably reduce build times and fix test failures. Ramsay's Rust compatibility report provides valuable data for the broader Rust-in-Git effort, though the toolchain's experimental status limits its immediate practicality. The patches are well-scoped and likely to be uncontroversial, with no major open questions.

---

### `git repo structure`: Add filtering options
**What changed**: Mark C. Chu-Carroll posted a patch adding `--no-<reftype>` filtering options to `git repo structure`.

**Problem/goal**: The `git repo structure` command, used for diagnosing performance issues caused by large objects, currently outputs all reference types and their transitive dependencies. Users sometimes need to focus on subsets of repository data, and the all-or-nothing output can be overwhelming.

**Subsystem**: `git repo` command.

**Kind of change**: Feature (new CLI options).

**Impact**: The patch adds `--no-<reftype>` flags (e.g., `--no-tags`, `--no-branches`, `--no-remotes`, `--no-notes`, `--no-stashes`) to exclude specific reference types and their transitive dependencies from the report. The filtering logic is implemented in `count_references` and propagated through the output formatting functions. The patch includes 276 lines of test coverage validating edge cases (e.g., multiple simultaneous filters, output formats like table/lines/nul).

**Today's developments**: The patch was posted, with no review feedback yet. The implementation follows Git's established pattern for disable-only flags (e.g., `--no-verify` in `git push`) and is well-scoped, touching only `builtin/repo.c`, `Documentation/git-repo.adoc`, and the test suite.

**Why it matters**: This patch addresses a practical user need by allowing granular exclusion of reference types, making the `git repo structure` command more useful for diagnosing performance issues. The implementation is clean and follows established patterns, so it is unlikely to be controversial. Reviewers may focus on the test coverage or the choice to omit transitive dependencies entirely (rather than just hiding them from the output), but the rationale is defensible: if a reference type is irrelevant to the diagnosis, its dependencies likely are too.

## In brief
- **`git-p4` shell injection fix (v2)**: Anupam Mediratta posted v2 of the patch fixing a shell injection vulnerability in `git-p4`, restoring the original error-handling behavior while preserving the shell injection mitigation. The patch replaces a shell-based pipeline with direct subprocess calls and includes a regression test.
- **`fetch-pack`: Mark exact OIDs in refs**: Nathan Froyd posted a patch marking exact OIDs in refs for `git fetch-pack`, ensuring it sends `want $OID` instead of `want-ref $OID` when invoked as `git fetch-pack $OID`. The patch adds a single line to set `ref->exact_oid` and includes test coverage for edge cases.
- **`fetch.followRemoteHEAD` lazy validation**: Matt Hunter posted a follow-up patch extending lazy validation to the remote-side parser in `remote.c`, unifying parsing logic for `fetch.followRemoteHEAD` and `remote.<name>.followRemoteHEAD`. The patch introduces a new `struct follow_remote_head_target` and replaces the `follow_remote_head` enum field in `struct remote` with a `char *follow_remote_head_raw` field.
- **`git reflog expire` default expiry fix (v3)**: Pushkar Singh posted v3 of the patch fixing the swapped default expiry periods for reachable and unreachable reflog entries. Junio C Hamano approved the patch, stating it "looks very good" and is ready for merging.
- **`git stash` autostash fix**: Junio C Hamano identified a potential logic error in the order of tree arguments passed to `merge_incore_nonrecursive()` in the final patch of the stash series. He provided a new test case that fails with the current patch, confirming the issue. Phillip Wood also identified a memory leak in the patch, noting that `merge_finalize(&o, &result)` should be called before returning on conflict.
- **Governance documentation**: Antonin Delpeuch clarified that the goal of the proposed governance documentation is to be descriptive, not prescriptive, and that even a factual write-up could surface gaps or inefficiencies in current practice.
- **`git-contacts` stdin input (v3)**: Brigham Campbell posted v3 of the patch implementing the `-` idiom for stdin input in `git-contacts`. Junio C Hamano proposed an alternative design that treats `-` as a special filename in `scan_patch_file` rather than using a `$read_from_stdin` flag.
- **CI resource exhaustion**: Tamir Duberstein provided benchmark data comparing `diff -u` and `cmp` for the first patch in the CI resource-exhaustion series, confirming that `test_cmp_bin` (which uses `cmp`) is a far lighter alternative. He also conceded that early file deletion is ineffective and should be dropped.
- **Afrikaans `git-gui` translation (v2)**: Stéfan Driaan Turvey posted v2 of the patch adding an Afrikaans translation for `git-gui`, addressing procedural requirements (registration in `po/meson.build` and metadata stripping) raised in the initial review.