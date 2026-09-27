# Git mailing list daily digest for 2026/09/26

## The day in brief

Kristoffer Haugsbakk posted a redesigned v2 of the `--[no-]range-diff-notes` feature for `git format-patch`, addressing earlier usability concerns. Andrew Pleeter submitted v9 of the `git var` extension series, removing the ambiguous `GIT_SIGNING_KEY` variable. D. Ben Knoble finalized a five-patch bugfix series for `git stash` autostash handling of staged index entries, incorporating all review feedback. A crash in `git pack-objects` during partial clone with `tree:0` filter was reported, and a test flakiness fix for `t/perf/p5551-fetch-rescan.sh` was proposed.

## Notable threads

### `--[no-]range-diff-notes` v2 for `git format-patch`

Kristoffer Haugsbakk posted v2 of the `--[no-]range-diff-notes` series for `git format-patch`, addressing Junio C Hamano’s usability concerns from v1. The redesign eliminates the toggle mechanism and arg-less `--range-diff-notes` variant, enforcing explicit separation between patch notes and range-diff notes. The series consists of two patches: a preparatory refactoring of `get_notes_args` to remove a redundant parameter, and the new implementation introducing `--range-diff-notes=<ref>` (requiring an explicit ref argument) and `--no-range-diff-notes` (suppressing notes in the range-diff section without fallback).

The implementation replaces the custom `revision.c` option handler with a standard `parse-options` callback (`rdiff_notes_cb`) and introduces a new `struct rdiff_notes` to track whether the user wants to override the default notes behavior for range-diffs. Test coverage is thorough, verifying scenarios like suppressing all range-diff notes, using different notes refs for patches and range-diffs, and ensuring `--range-diff-notes` requires a value. The series is well-structured, with clear separation between refactoring and new functionality, and the documentation is updated to reflect the simplified behavior.

The v2 design directly addresses the usability issues of v1, making the interaction between `--notes` and `--range-diff-notes` predictable and intuitive. While the thread remains closed and the feature was previously abandoned, this proposal provides a viable path forward if the discussion is revived.

### `git var` extension series v9

Andrew Pleeter posted v9 of the `git var` extension series, implementing a unified, scriptable interface for Git’s identity and signing configuration. The series is now split into four focused patches: (1) internal conversion of multi-valued variables to `string_list`, (2) `-z` output mode matching `git config -z` format, (3) multi-variable support, and (4) the six new identity variables (`GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL`, `GIT_AUTHOR_DATE`, `GIT_COMMITTER_NAME`, `GIT_COMMITTER_EMAIL`, `GIT_COMMITTER_DATE`). The controversial `GIT_SIGNING_KEY` variable was removed due to ambiguity across signing mechanisms (GPG/SSH/X.509).

The series addresses all prior feedback from Junio C Hamano and Phillip Wood, including structural concerns (split into smaller patches), exit-code behavior (multi-variable queries exit 0 even if some variables are unset), and output format (multi-valued variables emitted as repeated `VARIABLE=value` entries). The implementation is table-driven, uses Git’s standard argument parsing infrastructure, and reuses existing ident-parsing logic for the new identity variables. Test coverage is comprehensive, verifying `-z` output, multi-variable queries, argument ordering, and edge cases like unset variables.

The series is technically complete and ready for integration. The unresolved question about a trailing delimiter in `-z` mode (raised by Phillip Wood) is not a blocker and can be addressed in a follow-up if needed. The series remains in `seen` pending Junio’s review of the v9 submission.

### `git stash` autostash bugfix v3

D. Ben Knoble posted v3 of a five-patch bugfix series targeting a race condition in `git stash` autostash handling of staged index entries when `stash.index=true` is set. The series replaces subprocess-based index merging (`git diff-tree` and `git apply`) with in-core merging via `merge-ort.c`, eliminating a SIGSEGV risk and ensuring staged entries are preserved during merge operations. The root cause—a non-reentrant filesystem operation in `save_autostash()`—was resolved by switching to `merge_incore_nonrecursive()` in `do_apply_stash()`.

The series is now complete, incorporating all review feedback. Key changes since v2 include a critical tree argument order fix in `merge_incore_nonrecursive()` (resolving a correctness issue flagged by Junio), a memory leak fix (missing `merge_finalize()`), and expanded test assertions to verify both worktree and index state on failure. The patches are structured as follows: Patch 1 (cleanup) is already merged to `master`; Patches 2–5 (refactoring, test coverage, and the core fix) are finalized and ready for integration. Junio and Phillip Wood have endorsed the changes, and no further revisions are expected.

The series touches `builtin/stash.c`, `merge-ort.c`, and the test suite (`t/t3903-stash.sh`, `t/t7600-merge.sh`). Test coverage now includes conflicted index merges and fast-forward autostash scenarios. The broader argument-order standardization between `merge_ort_nonrecursive()` and `merge_incore_nonrecursive()` was explicitly deferred to a future cleanup.

### `git status --ignored` pathspec substring matching bugfix v2

René Scharfe posted v2 of a bugfix for `git status --ignored` with pathspec substring matching, incorporating Junio C Hamano’s squash-in fix for a NULL-dereference crash in the new `dir_match()` helper. The patch restores exact-match pathspec behavior for ignored directories by ensuring `match_pathspec_with_flags()` is called even when the directory would otherwise be skipped due to an optimization introduced in commit `95c11ecc73`. The interdiff against v1 shows the key change: the `pathspec` check is now explicit (`if (pathspec && !matches_how)`), preventing a NULL-dereference crash when `pathspec` is NULL.

The patch adds two test cases: one in `t/t7061-wtstatus-ignore.sh` verifying the original regression (omitting excluded directories with nested repositories when the pathspec is a prefix match), and one in `t/t2021-checkout-overwrite.sh` verifying safety when `pathspec` is NULL. The commit message now credits Junio for the squash-in and explicitly ties the fix to the regression’s origin. The patch is technically sound, addresses the safety concern raised in review, and is ready for maintainer integration.

## In brief

- **CI: Redundant `brew link --force gettext` removal**: Harald Nordgren clarified that the patch removing a redundant `brew link --force gettext` command from `ci/install-dependencies.sh` is still relevant for GitHub CI builds, resolving Junio C Hamano’s open question. The patch eliminates spurious "Already linked" warnings in macOS CI logs and is ready for final review.
- **`git shortlog` regression fix**: Kristoffer Haugsbakk requested a `.mailmap` update to use `<code@khaugsbakk.name>` for future commits, acknowledging Jeff King’s (Peff) two-patch series fixing the `(null)` regression in `git shortlog` error messages. The series is under maintainer approval and a strong candidate for inclusion in `next`.
- **`git pack-objects` crash in partial clone**: L Naudé reported a crash in `git pack-objects` during `git fetch` in a partial clone configured with `tree:0` filter, triggered by a missing promised tree object. The crash occurs in `is_not_in_promisor_pack_obj()`, called from `process_tree()` in `list-objects.c`, with the assertion `should_include_obj should only be called on existing objects`. The issue is under investigation, with no patch or resolution yet.
- **Test flakiness fix for `t/perf/p5551-fetch-rescan.sh`**: Jon Simons posted a patch adding `--no-deref` to `update-ref` in `t/perf/p5551-fetch-rescan.sh` to fix flakiness introduced by Git commit 3f763ddf28 (fetch: create HEAD symref on first fetch). The change prevents deletion of symref targets, eliminating the conflict that caused the test to abort. The patch is minimal and directly addresses the regression.