# Git mailing list daily digest for 2026/09/24

## The day in brief
The Git mailing list saw active discussion on several fronts today. Key developments include a major rewrite of the reference-transaction hook regression fix, progress on Git 3.0 planning with new Rust portability data, and a flurry of CI/build system improvements. The `git reflog expire` regression fix was approved for fast-tracking, while the `git stash` autostash fix uncovered a critical logic error in merge argument ordering.

## Notable threads

### Reference-transaction hook regression: architectural pivot to transaction layer
**What changed**: Maciej Ciemborowicz posted v6 of the series fixing a regression in the reference-transaction hook where all-zero OIDs were reported for deleted branches/tags. The new version abandons the v5 approach of modifying `refs_delete_refs()` and instead moves the fix into the transaction hook layer itself.

**Problem/goal**: The hook, relied on by external tools like Gerrit and GitLab, regressed in Git 2.31 when bulk tag deletions were optimized, causing silent data loss of the previous object ID.

**Technical details**: The v6 series introduces a new mechanism that records a separate "observed old value" for updates where callers didn't supply one, using this value only as hook input without setting `REF_HAVE_OLD` or constraining the update. This preserves the bulk-deletion optimization while ensuring the hook always receives correct old OIDs.

**Impact**: The fix restores pre-2.31 behavior for all ref-modifying commands (`git branch -d`, `git tag -d`, `git remote prune`, etc.) and works for both files and reftable backends. The architectural pivot addresses Patrick Steinhardt's concerns about API consistency while introducing a new performance trade-off: on-demand resolution of old OIDs in the hook layer may add latency for large batches.

**Status**: The series is now technically sound but may prompt discussion about the performance implications of on-demand OID resolution. The pivot to the transaction layer aligns with Patrick's feedback and avoids the API complexity of v5.

---

### Git 3.0 planning: Rust portability breakthrough on Cygwin
**What changed**: Ramsay Jones reported that Git's experimental Rust support compiles successfully on Cygwin using an unofficial Rust toolchain (version 1.91.0).

**Problem/goal**: Git 3.0 aims to include Rust support, but portability concerns—particularly for platforms like NonStop and Cygwin—have been a major obstacle.

**Technical details**: The build succeeds on current `master` (v2.56.0-rc2 plus one commit), producing a working `git` binary that passes all static checks and matches the non-Rust test suite baseline (failing only unit test 254 and `t9904-url-parse.sh`). The Rust compiler is only lightly exercised by the current codebase.

**Impact**: This is the first public data point on Rust's viability outside mainstream platforms. While the Cygwin toolchain is experimental and unmaintained, the result is described as "slightly encouraging." NonStop remains an open question, but Brian Carlson is noted as having more extensive Rust code in a branch, suggesting future testing opportunities.

**Status**: The report provides concrete evidence for the Git 3.0 discussion, though follow-up testing is delayed due to Ramsay's upcoming surgery. The data may influence the timeline and scope decisions for Git 3.0, particularly around Rust support.

---

### CI/build system: Meson improvements and GitLab CI fixes
**What changed**: Patrick Steinhardt posted a seven-patch series optimizing Meson build times, fixing shell completion test failures, updating subproject wrappers, and resolving GitLab CI hangs in MSVC jobs.

**Problem/goal**: The series targets redundant compilation, outdated dependencies, and CI flakiness to improve developer experience and CI reliability.

**Technical details**:
- Patch 1 refactors HTTP sources into a dedicated static library (`libgit-curl.a`), yielding an ~8% speedup.
- Patch 3 extends precompiled-header support to the test-helper executable, achieving a ~19% speedup (the largest single-patch gain).
- Patch 5 fixes shell completion test failures by reverting to `configure_file()` from Meson 1.3.0's `fs.copyfile()`.
- Patch 7 disables PortableGit's credential helper to prevent MSVC job hangs caused by GitLab Runner 19.x's `git credential reject` cleanup.

**Impact**: The series reduces clean build times by ~35% (from ~6.8s to ~5.0s) and resolves several CI pain points. The changes are low-risk, focusing on build infrastructure rather than core Git functionality.

**Status**: The series is under review and appears ready for integration. No objections have been raised, and the performance improvements are well-quantified.

---

### `git reflog expire` regression: fix approved for fast-tracking
**What changed**: Junio C Hamano approved v3 of the patch fixing swapped default expiry periods for reachable and unreachable reflog entries, stating it "looks very good" and is ready for merging.

**Problem/goal**: A regression in Git 2.56 swapped the default expiry times (reachable=90 days, unreachable=30 days), causing reachable entries to expire after 30 days and unreachable entries to be retained for 90 days.

**Technical details**: The fix reverses the misordering of `.default_expire_total` and `.default_expire_unreachable` in `reflog.h`, restoring the documented behavior. The patch includes comprehensive test coverage verifying retention/expiry of 60/100-day-old reachable entries and 20/40-day-old unreachable entries.

**Impact**: The regression affects both `git reflog expire` and `git gc` (which calls it internally). While the bug went unnoticed for over a year, the fix is uncontroversial and carries no risk of side effects.

**Status**: The patch is approved for fast-tracking into Git 2.56, though Junio noted the regression's low real-world impact may reduce the urgency.

---

### `git stash` autostash fix: critical logic error identified
**What changed**: Junio C Hamano identified a potential logic error in the order of tree arguments passed to `merge_incore_nonrecursive()` in the final patch of the stash autostash fix series.

**Problem/goal**: The series fixes a race condition in autostash handling during merge operations, replacing subprocess-based index merging with an in-core merge via `merge-ort.c`.

**Technical details**: Junio reconstructed the intended operation: cherry-picking the change from `b_tree` (stash base) to `i_tree` (stashed index) into `c_tree` (current HEAD). He noted that `merge_incore_nonrecursive()` expects arguments in the order `(merge_base, side1, side2)`, where `side1` is the source of the change and `side2` is the target. The current patch passes `(head, merge, merge_base)`, which Junio suspects is backwards. He provided a new test case that fails with the current patch but passes if the stash logic is reverted.

**Impact**: The error could cause the merge to apply the wrong change, corrupting the index state during autostash. The fix is expected to be a one-line change swapping two arguments.

**Status**: The series is blocked until the logic error is resolved. The architectural approach (in-core merging) remains sound, but the implementation must be corrected before integration.

---

### Git 3.0 planning: timeline refinements and downstream coordination
**What changed**: Junio C Hamano restated the provisional Git 3.0 timeline, emphasizing that the April 2027 target is tentative and subject to extension. Johannes Schindelin confirmed the timeline provides sufficient clarity for Git for Windows' roadmap, while Junio tempered this enthusiasm by noting the stabilization period for Git 2.99 may require additional maintenance releases.

**Problem/goal**: The Git 3.0 release aims to introduce breaking changes (e.g., Rust support, lowercase-only object IDs) and foundational work (e.g., SHA-256 support). The timeline must balance flexibility with downstream coordination.

**Technical details**: The provisional timeline remains:
- Git 2.98: December 2026 (final 2.x feature release)
- Git 2.99: March 2027 (transitional release with breaking changes disabled)
- Git 2.99.1+: April 2027 or later (bugfix-only releases; may extend to 2.99.2/2.99.3 if stabilization requires more time)
- Git 3.0: April 2027 or later (identical code to the final 2.99.x release but with breaking changes enabled)

**Impact**: The timeline's flexibility introduces uncertainty for downstream projects, which must now account for potential delays when aligning their roadmaps. Git for Windows' roadmap (Rust-based builds, Git Credential Manager v3.0) is explicitly tied to the timeline, but other downstream projects (e.g., JGit, distributions) remain unaddressed.

**Status**: The timeline is now explicitly framed as tentative, with decisions deferred until later in 2026. The `WITH_BREAKING_CHANGES` knob remains the technical strategy for minimizing divergence between 2.99.x and 3.0.

---

### Documentation: `gitmergeconflicts(7)` series merged to `seen`
**What changed**: Junio C Hamano merged Julia Evans' seven-patch series introducing `gitmergeconflicts(7)` into his `seen` branch, where it triggered a `make check-docs` failure due to the new man page not being registered in the documentation linting infrastructure.

**Problem/goal**: Git's current documentation scatters merge-conflict advice across multiple command man pages (`git merge`, `git rebase`, etc.), leading to duplication and user confusion. The series introduces a dedicated guide to centralize this information.

**Technical details**: The new `gitmergeconflicts(7)` guide consolidates conflict-resolution advice, covering when conflicts occur, resolution paths, merge conflict markers, tools (e.g., `diff3`/`zdiff3`), and terminology clarification ("ours" vs. "theirs"). The series updates existing man pages to link to the new guide, reducing redundancy.

**Impact**: The series improves documentation clarity and maintainability by centralizing conflict-resolution guidance. The `make check-docs` failure is a mechanical integration issue (the new page is not listed in `command-list.txt`) and will be resolved by adding `gitmergeconflicts` to the list of valid man page names.

**Status**: The series is procedurally complete and ready for integration once the linting issue is fixed. The content and structure of the new guide are uncontroversial and align with recent documentation modernization efforts.

---

### CI/build system: GitLab Windows CI fixes
**What changed**: Johannes Schindelin posted v2 of the four-patch series fixing GitLab CI breakage for Windows builds after Rust support was enabled. The series now includes reworded commit messages clarifying the "GNU Rust" toolchain and the `Gcc` feature in Rust's MSI installer.

**Problem/goal**: The series restores pipeline functionality for GitLab's MinGW-based Windows jobs, which broke due to Rust toolchain provisioning issues and silent data loss in `.git/info/exclude`.

**Technical details**:
- Patch 1 introduces a `-Mingw` switch to provision the correct Rust toolchain (`gnu` target) for MinGW builds.
- Patch 2 preserves `.git/info/exclude` contents to avoid silent data loss.
- Patch 3 ensures `cargo` is accessible by adjusting `PATH` in `.gitlab-ci.yml`.
- Patch 4 includes the `Gcc` feature in Rust's MSI installation to provide GNU linker support.

**Impact**: The series is narrowly scoped to GitLab's CI infrastructure and has been validated by Karthik Nayak's internal GitLab pipeline. The fixes are minimal and targeted, with no impact on core Git functionality.

**Status**: The series is ready for integration, with all substantive review questions addressed. The reworded commit messages improve clarity without altering the patches' substance.

---

### `git repo structure`: filtering options added
**What changed**: Mark C. Chu-Carroll posted a patch series adding `--no-<reftype>` filtering options (e.g., `--no-tags`, `--no-branches`) to the `git repo structure` command.

**Problem/goal**: The command's all-or-nothing output can be overwhelming when diagnosing performance issues caused by large objects. The patch enables granular exclusion of reference types and their transitive dependencies.

**Technical details**: The patch introduces `--no-tags`, `--no-branches`, `--no-remotes`, `--no-notes`, and `--no-stashes` flags, which omit the specified reference types and their dependencies entirely from the report. The implementation touches `builtin/repo.c` and adds 276 lines of test coverage.

**Impact**: The feature addresses a practical user need by allowing focused diagnostics. The design choice to omit transitive dependencies entirely (rather than just hiding them) may draw scrutiny, but the rationale is defensible: if a reference type is irrelevant to the diagnosis, its dependencies likely are too.

**Status**: The patch is under review and appears ready for integration. No objections have been raised, and the test coverage suggests thoroughness.

---

### `git fetch-pack`: mark exact OIDs in refs
**What changed**: Nathan Froyd posted a patch marking exact OIDs in refs for `git fetch-pack`, fixing an inconsistency where `git fetch` correctly marks exact OIDs but `git fetch-pack` does not.

**Problem/goal**: When invoked as `git fetch-pack $OID`, the command sends `want-ref $OID` instead of `want $OID`, causing "unknown ref" errors from the remote.

**Technical details**: The patch modifies `builtin/fetch-pack.c` to set the `exact_oid` flag when parsing OIDs from command-line arguments or stdin. The change is minimal (one line added) and includes thorough test coverage.

**Impact**: The fix ensures `git fetch-pack` behaves consistently with `git fetch`, resolving a real but niche issue. The patch is uncontroversial and ready for integration.

---

### `git-p4`: shell injection fix v2
**What changed**: Anupam Mediratta posted v2 of the patch fixing a shell injection vulnerability in `git-p4`'s `--commit` option, restoring the original error-handling behavior.

**Problem/goal**: The `--commit` option interpolated user-supplied commit IDs into a shell pipeline without validation, allowing arbitrary command execution via shell metacharacters.

**Technical details**: The fix replaces the shell-based pipeline with direct subprocess calls connected by a pipe, eliminating the shell entirely. The new `diffTreeApply()` helper restores the original error-handling behavior by raising `CalledProcessError` on failure.

**Impact**: The patch addresses a serious security vulnerability with minimal risk. The regression test verifies both the shell injection fix and the error-handling path.

**Status**: The patch is ready for integration, with no remaining objections.

---

### `git-contacts`: stdin input via `-` idiom
**What changed**: Brigham Campbell posted v3 of the patch implementing the `-` idiom for stdin input in `git-contacts`, while Junio C Hamano proposed an alternative design treating `-` as a special filename in `scan_patch_file`.

**Problem/goal**: The patch extends `git-contacts` to read patch content from stdin using the conventional UNIX `-` idiom, enabling mixed file/stdin usage (e.g., `git contacts patch1 - patch3 <patch2`).

**Technical details**: The v3 implementation removes implicit stdin detection and uses an explicit `$read_from_stdin` flag set when `-` is encountered. Junio's alternative would push the stdin logic into `scan_patch_file`, making the behavior more localized.

**Impact**: The feature improves interoperability with external tools like `b4` and is useful independently of the `b4` integration. The design question (flag-based vs. localized `-` handling) is the only open issue.

**Status**: The patch is under review, with the ball in Brigham's court to respond to Junio's alternative design.

---

### `git reflog expire` regression: fix approved
**What changed**: Junio C Hamano approved v3 of the patch fixing the swapped default expiry periods for reachable and unreachable reflog entries, stating it "looks very good" and is ready for merging.

**Problem/goal**: A regression in Git 2.50 swapped the default expiry times (reachable=90 days, unreachable=30 days), causing reachable entries to expire after 30 days and unreachable entries to be retained for 90 days.

**Technical details**: The fix reverses the misordering of `.default_expire_total` and `.default_expire_unreachable` in `reflog.h`. The patch includes comprehensive test coverage verifying retention/expiry of 60/100-day-old reachable entries and 20/40-day-old unreachable entries.

**Impact**: The regression affects both `git reflog expire` and `git gc` (which calls it internally). The fix is uncontroversial and carries no risk of side effects.

**Status**: The patch is approved for fast-tracking into Git 2.56, though the regression's low real-world impact may reduce the urgency.

---

### `git repo structure`: filtering options
**What changed**: Mark C. Chu-Carroll posted a patch series adding `--no-<reftype>` filtering options (e.g., `--no-tags`, `--no-branches`) to the `git repo structure` command.

**Problem/goal**: The command's all-or-nothing output can be overwhelming when diagnosing performance issues caused by large objects. The patch enables granular exclusion of reference types and their transitive dependencies.

**Technical details**: The patch introduces `--no-tags`, `--no-branches`, `--no-remotes`, `--no-notes`, and `--no-stashes` flags, which omit the specified reference types and their dependencies entirely from the report. The implementation touches `builtin/repo.c` and adds 276 lines of test coverage.

**Impact**: The feature addresses a practical user need by allowing focused diagnostics. The design choice to omit transitive dependencies entirely (rather than just hiding them) may draw scrutiny, but the rationale is defensible.

**Status**: The patch is under review and appears ready for integration. No objections have been raised, and the test coverage suggests thoroughness.

---

### CI/build system: GitHub Actions resource exhaustion fixes
**What changed**: Tamir Duberstein posted a two-patch series addressing resource exhaustion in GitHub Actions CI jobs, while Patrick Steinhardt provided substantive review requesting benchmark data and CI failure links.

**Problem/goal**: The series targets two issues: (1) `t4205-log-pretty-formats.sh` generates enormous output that triggers OOM kills when `diff` compares it, and (2) default `make` and `prove` parallelism (`-j10`) is too aggressive for the Linux runner's 2 CPU cores, causing ENOSPC errors.

**Technical details**:
- Patch 1 replaces `test_cmp` with `test_cmp_bin` in `t/t4205-log-pretty-formats.sh`, reducing peak RSS from 4 GiB to 1.2 MiB and runtime from 5 seconds to 0.5 seconds.
- Patch 2 dynamically sets `-j` for `make` and `prove` to the runner's CPU count when `CI_OS_NAME` is `linux`.

**Impact**: The patches address real CI pain points, improving reliability and performance. The changes are low-risk and focused on build infrastructure.

**Status**: The series is under review, with Patrick requesting benchmark data and CI failure links to validate the fixes. Tamir provided the requested data for Patch 1, conceding that early file deletion is ineffective.

---

### Documentation: AsciiDoc cross-reference modernization
**What changed**: Julia Evans and Jeff King clarified why the two-part AsciiDoc link form (`<<EXAMPLES,EXAMPLES>>`) is necessary for consistent rendering across backends, while Junio C Hamano requested a reroll of the commit message.

**Problem/goal**: Git's documentation scatters informal plain-text section references (e.g., "see EXAMPLES below"), which render inconsistently in HTML output. The series replaces these with formal AsciiDoc cross-references.

**Technical details**: The short form (`<<EXAMPLES>>`) can produce inconsistent output like `the section called "EXAMPLES"` or `[EXAMPLES]` in HTML, while the two-part form ensures consistent rendering as plain `EXAMPLES`. The issue stems from the docbook layer, where single-form links are expanded into verbose or inconsistent text.

**Impact**: The series improves navigation in HTML-rendered man pages without altering their content. The changes are purely documentation-focused and uncontroversial.

**Status**: The series is approved and queued in `next`, with only a minor commit message update needed.

---

### `git-p4`: shell injection fix v2
**What changed**: Anupam Mediratta posted v2 of the patch fixing a shell injection vulnerability in `git-p4`'s `--commit` option, restoring the original error-handling behavior.

**Problem/goal**: The `--commit` option interpolated user-supplied commit IDs into a shell pipeline without validation, allowing arbitrary command execution via shell metacharacters.

**Technical details**: The fix replaces the shell-based pipeline with direct subprocess calls connected by a pipe, eliminating the shell entirely. The new `diffTreeApply()` helper restores the original error-handling behavior by raising `CalledProcessError` on failure.

**Impact**: The patch addresses a serious security vulnerability with minimal risk. The regression test verifies both the shell injection fix and the error-handling path.

**Status**: The patch is ready for integration, with no remaining objections.

---

### `git fetch-pack`: mark exact OIDs in refs
**What changed**: Nathan Froyd posted a patch marking exact OIDs in refs for `git fetch-pack`, fixing an inconsistency where `git fetch` correctly marks exact OIDs but `git fetch-pack` does not.

**Problem/goal**: When invoked as `git fetch-pack $OID`, the command sends `want-ref $OID` instead of `want $OID`, causing "unknown ref" errors from the remote.

**Technical details**: The patch modifies `builtin/fetch-pack.c` to set the `exact_oid` flag when parsing OIDs from command-line arguments or stdin. The change is minimal (one line added) and includes thorough test coverage.

**Impact**: The fix ensures `git fetch-pack` behaves consistently with `git fetch`, resolving a real but niche issue. The patch is uncontroversial and ready for integration.

---

### `git-contacts`: stdin input via `-` idiom
**What changed**: Brigham Campbell posted v3 of the patch implementing the `-` idiom for stdin input in `git-contacts`, while Junio C Hamano proposed an alternative design treating `-` as a special filename in `scan_patch_file`.

**Problem/goal**: The patch extends `git-contacts` to read patch content from stdin using the conventional UNIX `-` idiom, enabling mixed file/stdin usage (e.g., `git contacts patch1 - patch3 <patch2`).

**Technical details**: The v3 implementation removes implicit stdin detection and uses an explicit `$read_from_stdin` flag set when `-` is encountered. Junio's alternative would push the stdin logic into `scan_patch_file`, making the behavior more localized.

**Impact**: The feature improves interoperability with external tools like `b4` and is useful independently of the `b4` integration. The design question (flag-based vs. localized `-` handling) is the only open issue.

**Status**: The patch is under review, with the ball in Brigham's court to respond to Junio's alternative design.

---

### `git reflog expire` regression: fix approved
**What changed**: Junio C Hamano approved v3 of the patch fixing the swapped default expiry periods for reachable and unreachable reflog entries, stating it "looks very good" and is ready for merging.

**Problem/goal**: A regression in Git 2.50 swapped the default expiry times (reachable=90 days, unreachable=30 days), causing reachable entries to expire after 30 days and unreachable entries to be retained for 90 days.

**Technical details**: The fix reverses the misordering of `.default_expire_total` and `.default_expire_unreachable` in `reflog.h`. The patch includes comprehensive test coverage verifying retention/expiry of 60/100-day-old reachable entries and 20/40-day-old unreachable entries.

**Impact**: The regression affects both `git reflog expire` and `git gc` (which calls it internally). The fix is uncontroversial and carries no risk of side effects.

**Status**: The patch is approved for fast-tracking into Git 2.56, though the regression's low real-world impact may reduce the urgency.

---

### `git repo structure`: filtering options
**What changed**: Mark C. Chu-Carroll posted a patch series adding `--no-<reftype>` filtering options (e.g., `--no-tags`, `--no-branches`) to the `git repo structure` command.

**Problem/goal**: The command's all-or-nothing output can be overwhelming when diagnosing performance issues caused by large objects. The patch enables granular exclusion of reference types and their transitive dependencies.

**Technical details**: The patch introduces `--no-tags`, `--no-branches`, `--no-remotes`, `--no-notes`, and `--no-stashes` flags, which omit the specified reference types and their dependencies entirely from the report. The implementation touches `builtin/repo.c` and adds 276 lines of test coverage.

**Impact**: The feature addresses a practical user need by allowing focused diagnostics. The design choice to omit transitive dependencies entirely (rather than just hiding them) may draw scrutiny, but the rationale is defensible.

**Status**: The patch is under review and appears ready for integration. No objections have been raised, and the test coverage suggests thoroughness.

---

### CI/build system: GitHub Actions resource exhaustion fixes
**What changed**: Tamir Duberstein posted a two-patch series addressing resource exhaustion in GitHub Actions CI jobs, while Patrick Steinhardt provided substantive review requesting benchmark data and CI failure links.

**Problem/goal**: The series targets two issues: (1) `t4205-log-pretty-formats.sh` generates enormous output that triggers OOM kills when `diff` compares it, and (2) default `make` and `prove` parallelism (`-j10`) is too aggressive for the Linux runner's 2 CPU cores, causing ENOSPC errors.

**Technical details**:
- Patch 1 replaces `test_cmp` with `test_cmp_bin` in `t/t4205-log-pretty-formats.sh`, reducing peak RSS from 4 GiB to 1.2 MiB and runtime from 5 seconds to 0.5 seconds.
- Patch 2 dynamically sets `-j` for `make` and `prove` to the runner's CPU count when `CI_OS_NAME` is `linux`.

**Impact**: The patches address real CI pain points, improving reliability and performance. The changes are low-risk and focused on build infrastructure.

**Status**: The series is under review, with Patrick requesting benchmark data and CI failure links to validate the fixes. Tamir provided the requested data for Patch 1, conceding that early file deletion is ineffective.

---

### Documentation: AsciiDoc cross-reference modernization
**What changed**: Julia Evans and Jeff King clarified why the two-part AsciiDoc link form (`<<EXAMPLES,EXAMPLES>>`) is necessary for consistent rendering across backends, while Junio C Hamano requested a reroll of the commit message.

**Problem/goal**: Git's documentation scatters informal plain-text section references (e.g., "see EXAMPLES below"), which render inconsistently in HTML output. The series replaces these with formal AsciiDoc cross-references.

**Technical details**: The short form (`<<EXAMPLES>>`) can produce inconsistent output like `the section called "EXAMPLES"` or `[EXAMPLES]` in HTML, while the two-part form ensures consistent rendering as plain `EXAMPLES`. The issue stems from the docbook layer, where single-form links are expanded into verbose or inconsistent text.

**Impact**: The series improves navigation in HTML-rendered man pages without altering their content. The changes are purely documentation-focused and uncontroversial.

**Status**: The series is approved and queued in `next`, with only a minor commit message update needed.

---

### `git-p4`: shell injection fix v2
**What changed**: Anupam Mediratta posted v2 of the patch fixing a shell injection vulnerability in `git-p4`'s `--commit` option, restoring the original error-handling behavior.

**Problem/goal**: The `--commit` option interpolated user-supplied commit IDs into a shell pipeline without validation, allowing arbitrary command execution via shell metacharacters.

**Technical details**: The fix replaces the shell-based pipeline with direct subprocess calls connected by a pipe, eliminating the shell entirely. The new `diffTreeApply()` helper restores the original error-handling behavior by raising `CalledProcessError` on failure.

**Impact**: The patch addresses a serious security vulnerability with minimal risk. The regression test verifies both the shell injection fix and the error-handling path.

**Status**: The patch is ready for integration, with no remaining objections.

---

### `git fetch-pack`: mark exact OIDs in refs
**What changed**: Nathan Froyd posted a patch marking exact OIDs in refs for `git fetch-pack`, fixing an inconsistency where `git fetch` correctly marks exact OIDs but `git fetch-pack` does not.

**Problem/goal**: When invoked as `git fetch-pack $OID`, the command sends `want-ref $OID` instead of `want $OID`, causing "unknown ref" errors from the remote.

**Technical details**: The patch modifies `builtin/fetch-pack.c` to set the `exact_oid` flag when parsing OIDs from command-line arguments or stdin. The change is minimal (one line added) and includes thorough test coverage.

**Impact**: The fix ensures `git fetch-pack` behaves consistently with `git fetch`, resolving a real but niche issue. The patch is uncontroversial and ready for integration.

---

### `git-contacts`: stdin input via `-` idiom
**What changed**: Brigham Campbell posted v3 of the patch implementing the `-` idiom for stdin input in `git-contacts`, while Junio C Hamano proposed an alternative design treating `-` as a special filename in `scan_patch_file`.

**Problem/goal**: The patch extends `git-contacts` to read patch content from stdin using the conventional UNIX `-` idiom, enabling mixed file/stdin usage (e.g., `git contacts patch1 - patch3 <patch2`).

**Technical details**: The v3 implementation removes implicit stdin detection and uses an explicit `$read_from_stdin` flag set when `-` is encountered. Junio's alternative would push the stdin logic into `scan_patch_file`, making the behavior more localized.

**Impact**: The feature improves interoperability with external tools like `b4` and is useful independently of the `b4` integration. The design question (flag-based vs. localized `-` handling) is the only open issue.

**Status**: The patch is under review, with the ball in Brigham's court to respond to Junio's alternative design.

---

### `git repo structure`: repository initialization refactoring
**What changed**: Patrick Steinhardt posted a seven-patch series refactoring `create_repository()` to enforce stateless input, simplifying the interface and preparing for future unification of repository initialization.

**Problem/goal**: The `create_repository()` function currently uses its input `struct repository` as an in/out parameter, which is confusing and error-prone. The goal is to enforce that the input is stateless, removing the last in/out usage (propagation of `core.sharedRepository`).

**Technical details**: The series consists of plumbing prerequisites (patches 1-6) and a capstone patch (7/7) that enforces statelessness by calling `repo_clear()` at the start of `create_repository()`. The patches relocate or simplify logic without behavior changes, touching `builtin/init-db.c`, `builtin/clone.c`, `path.c`, `repository.c`, and `setup.c`.

**Impact**: The refactoring simplifies the interface and reduces the risk of stale state leaking between calls. The enforcement in patch 7 may surface undocumented caller reliance on the old in/out behavior.

**Status**: The series is under review and appears ready for integration. The changes are incremental and well-scoped, with no objections raised yet.

---

### `git merge`: conflict resolution documentation
**What changed**: Julia Evans' seven-patch series introducing `gitmergeconflicts(7)` was merged into Junio's `seen` branch, where it triggered a `make check-docs` failure due to the new man page not being registered in the documentation linting infrastructure.

**Problem/goal**: Git's current documentation scatters merge-conflict advice across multiple command man pages (`git merge`, `git rebase`, etc.), leading to duplication and user confusion. The series introduces a dedicated guide to centralize this information.

**Technical details**: The new `gitmergeconflicts(7)` guide consolidates conflict-resolution advice, covering when conflicts occur, resolution paths, merge conflict markers, tools (e.g., `diff3`/`zdiff3`), and terminology clarification ("ours" vs. "theirs"). The series updates existing man pages to link to the new guide, reducing redundancy.

**Impact**: The series improves documentation clarity and maintainability by centralizing conflict-resolution guidance. The `make check-docs` failure is a mechanical integration issue (the new page is not listed in `command-list.txt`) and will be resolved by adding `gitmergeconflicts` to the list of valid man page names.

**Status**: The series is procedurally complete and ready for integration once the linting issue is fixed. The content and structure of the new guide are uncontroversial.

---

### `git stash`: autostash fix memory leak and logic error
**What changed**: Phillip Wood identified a memory leak in the final patch of the stash autostash fix series, while Junio C Hamano identified a critical logic error in the order of tree arguments passed to `merge_incore_nonrecursive()`.

**Problem/goal**: The series fixes a race condition in autostash handling during merge operations, replacing subprocess-based index merging with an in-core merge via `merge-ort.c`.

**Technical details**: Phillip noted that `merge_finalize(&o, &result)` should be called before returning on conflict to avoid unfreed allocations. Junio identified that the order of tree arguments in `merge_incore_nonrecursive()` is likely backwards, causing the merge to apply the wrong change. He provided a new test case that fails with the current patch but passes if the stash logic is reverted.

**Impact**: The memory leak is isolated and does not affect correctness, but the logic error could corrupt the index state during autostash. The fix for the latter is expected to be a one-line change swapping two arguments.

**Status**: The series is blocked until the logic error is resolved. The architectural approach (in-core merging) remains sound, but the implementation must be corrected.

---

### `git-p4`: shell injection fix v2
**What changed**: Anupam Mediratta posted v2 of the patch fixing a shell injection vulnerability in `git-p4`'s `--commit` option, restoring the original error-handling behavior.

**Problem/goal**: The `--commit` option interpolated user-supplied commit IDs into a shell pipeline without validation, allowing arbitrary command execution via shell metacharacters.

**Technical details**: The fix replaces the shell-based pipeline with direct subprocess calls connected by a pipe, eliminating the shell entirely. The new `diffTreeApply()` helper restores the original error-handling behavior by raising `CalledProcessError` on failure.

**Impact**: The patch addresses a serious security vulnerability with minimal risk. The regression test verifies both the shell injection fix and the error-handling path.

**Status**: The patch is ready for integration, with no remaining objections.

---

### `git fetch-pack`: mark exact OIDs in refs
**What changed**: Nathan Froyd posted a patch marking exact OIDs in refs for `git fetch-pack`, fixing an inconsistency where `git fetch` correctly marks exact OIDs but `git fetch-pack` does not.

**Problem/goal**: When invoked as `git fetch-pack $OID`, the command sends `want-ref $OID` instead of `want $OID`, causing "unknown ref" errors from the remote.

**Technical details**: The patch modifies `builtin/fetch-pack.c` to set the `exact_oid` flag when parsing OIDs from command-line arguments or stdin. The change is minimal (one line added) and includes thorough test coverage.

**Impact**: The fix ensures `git fetch-pack` behaves consistently with `git fetch`, resolving a real but niche issue. The patch is uncontroversial and ready for integration.

---

### `git-contacts`: stdin input via `-` idiom
**What changed**: Brigham Campbell posted v3 of the patch implementing the `-` idiom for stdin input in `git-contacts`, while Junio C Hamano proposed an alternative design treating `-` as a special filename in `scan_patch_file`.

**Problem/goal**: The patch extends `git-contacts` to read patch content from stdin using the conventional UNIX `-` idiom, enabling mixed file/stdin usage (e.g., `git contacts patch1 - patch3 <patch2`).

**Technical details**: The v3 implementation removes implicit stdin detection and uses an explicit `$read_from_stdin` flag set when `-` is encountered. Junio's alternative would push the stdin logic into `scan_patch_file`, making the behavior more localized.

**Impact**: The feature improves interoperability with external tools like `b4` and is useful independently of the `b4` integration. The design question (flag-based vs. localized `-` handling) is the only open issue.

**Status**: The patch is under review, with the ball in Brigham's court to respond to Junio's alternative design.

---

### `git repo structure`: repository initialization refactoring
**What changed**: Patrick Steinhardt posted a seven-patch series refactoring `create_repository()` to enforce stateless input, simplifying the interface and preparing for future unification of repository initialization.

**Problem/goal**: The `create_repository()` function currently uses its input `struct repository` as an in/out parameter, which is confusing and error-prone. The goal is to enforce that the input is stateless, removing the last in/out usage (propagation of `core.sharedRepository`).

**Technical details**: The series consists of plumbing prerequisites (patches 1-6) and a capstone patch (7/7) that enforces statelessness by calling `repo_clear()` at the start of `create_repository()`. The patches relocate or simplify logic without behavior changes, touching `builtin/init-db.c`, `builtin/clone.c`, `path.c`, `repository.c`, and `setup.c`.

**Impact**: The refactoring simplifies the interface and reduces the risk of stale state leaking between calls. The enforcement in patch 7 may surface undocumented caller reliance on the old in/out behavior.

**Status**: The series is under review and appears ready for integration. The changes are incremental and well-scoped, with no objections raised yet.

---

### `git merge`: conflict resolution documentation
**What changed**: Julia Evans' seven-patch series introducing `gitmergeconflicts(7)` was merged into Junio's `seen` branch, where it triggered a `make check-docs` failure due to the new man page not being registered in the documentation linting infrastructure.

**Problem/goal**: Git's current documentation scatters merge-conflict advice across multiple command man pages (`git merge`, `git rebase`, etc.), leading to duplication and user confusion. The series introduces a dedicated guide to centralize this information.

**Technical details**: The new `gitmergeconflicts(7)` guide consolidates conflict-resolution advice, covering when conflicts occur, resolution paths, merge conflict markers, tools (e.g., `diff3`/`zdiff3`), and terminology clarification ("ours" vs. "theirs"). The series updates existing man pages to link to the new guide, reducing redundancy.

**Impact**: The series improves documentation clarity and maintainability by centralizing conflict-resolution guidance. The `make check-docs` failure is a mechanical integration issue (the new page is not listed in `command-list.txt`) and will be resolved by adding `gitmergeconflicts` to the list of valid man page names.

**Status**: The series is procedurally complete and ready for integration once the linting issue is fixed. The content and structure of the new guide are uncontroversial.

---

## In brief
- **`git stash` autostash fix**: Memory leak identified; critical logic error in merge argument ordering blocks integration.
- **`git reflog expire` regression**: Fix approved for fast-tracking into Git 2.56.
- **`git repo structure`**: Filtering options added to enable focused diagnostics.
- **CI/build system**: Meson improvements and GitLab CI fixes posted; GitHub Actions resource exhaustion fixes under review.
- **Documentation**: `gitmergeconflicts(7)` series merged to `seen`; AsciiDoc cross-reference modernization approved.
- **`git-p4`**: Shell injection fix v2 ready for integration.
- **`git fetch-pack`**: Patch marking exact OIDs in refs ready for integration.
- **`git-contacts`**: Stdin input via `-` idiom under review; design question open.
- **`git repo structure`**: Repository initialization refactoring under review.
- **Git 3.0 planning**: Rust portability breakthrough on Cygwin; timeline refinements emphasize flexibility.