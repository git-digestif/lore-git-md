# Git mailing list daily digest for 2026/09/26

## The day in brief
Kristoffer Haugsbakk posted a redesigned v2 series for `--[no-]range-diff-notes` in `git format-patch`, addressing prior usability concerns. Andrew Pleeter submitted v9 of the `git var` extension series, removing the ambiguous `GIT_SIGNING_KEY` variable. D. Ben Knoble finalized a five-patch bugfix series for `git stash` autostash handling of staged index entries, incorporating all review feedback. A crash in `git pack-objects` during partial clone with `tree:0` filter was reported, and a test flakiness fix for `p5551-fetch-rescan` was proposed.

## Notable threads

### `--[no-]range-diff-notes` v2 for `git format-patch`
Kristoffer Haugsbakk posted v2 of the `--[no-]range-diff-notes` series for `git format-patch`, addressing Junio C Hamano’s usability concerns from v1. The series now consists of two patches: a preparatory refactoring and the new implementation.

The v2 design eliminates the toggle mechanism and arg-less `--range-diff-notes` variant, enforcing explicit separation between patch notes and range-diff notes. The new options are `--range-diff-notes=<ref>` (requiring an explicit ref argument) and `--no-range-diff-notes` (suppressing notes in the range-diff section without fallback). The implementation introduces a `struct rdiff_notes` to track user intent and replaces the custom `revision.c` handler with a standard `parse-options` callback (`rdiff_notes_cb`). The `get_notes_args` function is updated to take only `struct rev_info *` as a parameter, removing a redundant argument.

The first patch refactors `get_notes_args` to directly access `rev->rdiff_log_arg` instead of receiving it as a separate argument, simplifying the codebase for the feature implementation. The second patch implements the new functionality, adding comprehensive test coverage for scenarios like suppressing all range-diff notes, using different notes refs for patches and range-diffs, and ensuring `--range-diff-notes` requires a value. The documentation is updated to reflect the simplified behavior, removing references to the toggle mechanism.

The series is well-structured and addresses the usability issues of v1, but the thread remains closed, and no maintainer decision has been made on whether to accept the redesign.

### `git var` extension series v9
Andrew Pleeter posted v9 of the `git var` extension series, implementing a unified interface for Git’s identity and signing configuration. The series is now split into four patches as previously agreed: (1) string-list conversion for multi-valued variables, (2) `-z` output mode, (3) multi-variable support, and (4) the new identity variables (`GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL`, etc.).

The first patch refactors the internal representation of multi-valued variables to use a `string_list` instead of a newline-joined string, eliminating fragility where delimiters could appear in values. The second patch adds a `-z` output mode that matches `git config -z` format, resolving ambiguity when parsing multi-valued variables or values containing newlines. The third patch implements multi-variable support, allowing `git var` to accept more than one variable argument in a single invocation and omitting unset variables without error. The fourth patch adds the six new identity variables, each backed by a helper function that calls `split_ident_line()` on the corresponding ident string.

The series removes the controversial `GIT_SIGNING_KEY` variable due to ambiguity across signing mechanisms (GPG/SSH/X.509). The exit-code behavior is finalized: multi-variable queries exit 0 if any requested variable is set, omitting unset variables from the output. The series is technically complete and ready for integration, pending Junio’s review of v9.

### `git stash` autostash bugfix v3
D. Ben Knoble posted v3 of a five-patch bugfix series for `git stash`, addressing a race condition in autostash handling of staged index entries when `stash.index=true` is set. The series replaces subprocess-based index merging with in-core merging via `merge-ort.c`, eliminating a SIGSEGV risk and ensuring staged entries are preserved.

The series is now complete and ready for integration. Key changes since v2 include a new test for `stash --index` merges (Patch 3/5), expanded test assertions to verify both worktree and index state preservation (Patch 4/5), and a memory leak fix in the core fix (Patch 5/5). The tree argument order in `merge_incore_nonrecursive()` was corrected from `(head, merge, merge_base)` to `(merge_base, merge, head)` after Junio C Hamano identified the bug in an earlier iteration.

Phillip Wood proposed an expanded test case for `stash --index` merges, exercising content-level conflicts. The author confirmed the test scenario passes with the fix and fails on the original code. The discussion also resolved a minor implementation detail about diff output comparison, confirming raw diff output is sufficient for test comparisons.

The series touches `builtin/stash.c`, `merge-ort.c`, and the test suite (`t/t3903-stash.sh`, `t/t7600-merge.sh`). All prior feedback is incorporated, and the series is technically complete. Junio and Phillip Wood have endorsed the fixes, and no further revisions are expected.

### `git status --ignored` pathspec substring matching bugfix v2
René Scharfe posted v2 of a bugfix for `git status --ignored` with pathspec substring matching. The patch incorporates Junio C Hamano’s squash-in fix for a NULL-dereference crash in the new `dir_match()` helper, adding a defensive NULL check and a test case to verify safety when `pathspec` is NULL.

The patch restores exact-match pathspec behavior for ignored directories by ensuring `match_pathspec_with_flags()` is called even when the directory would otherwise be skipped due to an optimization introduced in commit `95c11ecc73`. The interdiff against v1 shows the key change: the `pathspec` check is now explicit (`if (pathspec && !matches_how)`), preventing a NULL-dereference crash. The added test in `t/t2021-checkout-overwrite.sh` verifies that `git checkout` does not segfault when an untracked nested repository exists without a pathspec.

The patch is technically sound, addresses the safety concern raised in review, and includes thorough test coverage. It is ready for maintainer integration.

### Crash in `git pack-objects` during partial clone with `tree:0` filter
L Naudé reported a crash in `git pack-objects` during a `git fetch` in a partial clone configured with `tree:0` filter. The failure occurs at `builtin/pack-objects.c:5004` with the assertion `should_include_obj should only be called on existing objects`, triggered when the code encounters a promised tree (`d0daf5223f2367bd20fc2deb9d66aed05fcd30e8`) that is not present locally and cannot be fetched.

The crash happens in `is_not_in_promisor_pack_obj()`, called from `process_tree()` in `list-objects.c`, during the `repack_local_links()` code path in `index-pack`. The reporter confirms the tree is missing via `git cat-file` and `git rev-list --missing=print`, and notes the repository has `remote.origin.promisor=true` and `remote.origin.partialclonefilter=tree:0`. The issue is tied to the `repack_local_links()` code path, suggesting the crash occurs during repacking of promisor objects.

No minimal reproduction is provided, but the conditions (partial clone with `tree:0` filter and a missing promised tree) are clearly identified. The crash is non-deterministic but reproducible under the described conditions.

### Test flakiness fix for `t/perf/p5551-fetch-rescan`
Jon Simons posted a patch adding `--no-deref` to `update-ref` in `t/perf/p5551-fetch-rescan.sh` to fix flakiness. The test became flaky after Git commit 3f763ddf28 introduced automatic creation of a `HEAD` symref during `git fetch`. Without `--no-deref`, `update-ref` fails when trying to delete both the symref target and the symref itself, causing the test to abort.

The change is minimal—one line—and directly addresses the regression. The commit message clearly explains the problem and cites the relevant prior commit. The patch is well-scoped and uncontroversial, resolving the flakiness in `p5551` without side effects.

### CI/build system patch for macOS
Harald Nordgren clarified that the CI/build system patch removing a redundant `brew link --force gettext` command is still relevant. The change targets GitHub CI builds, not local macOS development, and silences spurious "Already linked" warnings in macOS job logs. The patch is minimal, safe, and empirically validated by a passing macOS CI pipeline. All prior feedback is addressed, and the thread is unblocked.

## In brief
- Kristoffer Haugsbakk requested a `.mailmap` update to use `<code@khaugsbakk.name>` for future commits in response to the `git shortlog` regression fix series.
- René Scharfe acknowledged a NULL-dereference bug in the `dir.c` bugfix and confirmed the v2 patch incorporates Junio’s squash-in fix.