# Git mailing list daily digest for 2026/11/15

## The day in brief
The `git repo info` path keys series received final approval and is ready for merging, while the `change-id` preservation thread saw new arguments about security risks in automated review systems. Integer-overflow hardening patches were revised with simpler, more portable approaches, and a memory leak was identified in the `git apply` crash fix. Test modernization and color-handling improvements also advanced.

## Notable threads

### Preserving `change-id` headers during commit rewriting
What changed: Ewout ter Hoeven presented concrete use cases for preserving `change-id` headers in stacked pull/merge request workflows (GitHub `gh-stack`, GitLab `glab stack`), citing specific issues where branch renames and patch ID mismatches cause redundant replays. Brian m. Carlson countered that `change-id` headers cannot be trusted as stable identifiers because users can manually edit them, creating security risks for automated review systems.

What it affects: External tools (Gerrit, GitButler, Jujutsu) that rely on persistent commit identifiers during rebase, cherry-pick, and amend operations.

Why it matters: The thread must reconcile workflow needs with security concerns, particularly whether client-side rewrites can safely preserve `change-id`s or if server-side control is required.

---
### `git repo info` path keys series ready for merging
What changed: Phillip Wood explicitly approved the `git repo info` path keys series (v9) for merging as-is, marking the end of the review cycle. The series adds seven new keys (`path.workingtree.absolute`, `path.workingtree.relative`, `path.superprojectworkingtree.absolute`, `path.superprojectworkingtree.relative`, `path.index.absolute`, `path.index.relative`, and `path.prefix`) to expose filesystem locations of repository components.

What it affects: Scripting and tooling that needs to locate repository components (working tree, index, superproject, current directory prefix) without accessing Git internals.

Why it matters: The series is mechanically complete, design-resolved, and ready for integration. The unresolved architectural concern (refactoring shared logic into `repo-info.c`) is deferred to follow-up work.

Files touched: `builtin/repo.c`, `Documentation/git-repo.adoc`, `t/t1900-repo-info.sh`, `t/t1500-rev-parse.sh`.

---
### Integer-overflow hardening patches revised
What changed: Johannes Schindelin revised the integer-overflow hardening patches with simpler, more portable approaches. The wrapper subsystem fix now changes the API contract: `write_in_full()` returns `0` on success (eliminating overflow risk), and a new `read_in_full_or_error()` helper is introduced for callers that only need success/failure status. The date.c hardening was withdrawn and replaced with manual overflow checks (capping user-supplied numbers at `INT_MAX` and guarding multiplications).

What it affects: Archive-tar, HTTP, wrapper, and date subsystems, hardening them against undefined behavior when arithmetic exceeds type limits.

Why it matters: The revisions address reviewer concerns about GCC-only macros and platform-specific logic, but leave the date.c hardening unimplemented in this iteration. The wrapper subsystem changes introduce a minor migration burden for callers.

Files touched: `archive-tar.c`, `date.c`, `git-compat-util.h`, `http.c`, `wrapper.c`.

---
### Memory leak identified in `git apply` crash fix
What changed: Siddharth Asthana identified a memory leak in `parse_git_diff_header()` where `patch->def_name` is overwritten without freeing the old value. The leak surfaces in edge cases like empty `diff --git` sections. Asthana suggested addressing it in a preparatory patch to retain test coverage for these edge cases.

What it affects: The `git apply --check` crash fix, which now enforces strict validation of patch headers.

Why it matters: The leak is minor but would allow the test for empty `diff --git` sections to be reinstated, improving coverage. The fix is trivial (a `free()` before assignment) and safe, as callers already zero the `patch` struct.

Files touched: `apply.c`.

---
### ANSI color handling improved in custom log formats
What changed: Andy Koppe fixed lost and leaked ANSI color escape sequences in `git log --format=...` output. Colors that span line breaks are now preserved when tools like `--graph` or color-aware pagers reset colors at line boundaries. Active colors at the end of a line no longer "leak" into subsequent content.

What it affects: Custom log formats with colors, particularly when used with `--graph` or pagers.

Why it matters: The fix improves usability by ensuring colors are self-contained per line, aligning with how default formats already handle colors. The implementation is efficient and includes thorough test coverage.

Files touched: `pretty.c`, `t/t4205-log-pretty-formats.sh`.

---
### Test modernization: `!` → `test_must_fail`
What changed: Hardik Kumar replaced the `!` (NOT) operator with `test_must_fail` in four test scripts (`t5564-http-proxy.sh`, `t6030-bisect-porcelain.sh`, `t7400-submodule-basic.sh`, `t9001-send-email.sh`). The change is mechanical and touches seven instances where Git commands are expected to fail.

What it affects: Test scripts in the HTTP proxy, bisect porcelain, submodule, and `git send-email` subsystems.

Why it matters: `test_must_fail` properly handles edge cases like crashes or signals, whereas `!` would incorrectly pass a test if the command died unexpectedly. This is part of an ongoing effort to adopt more robust test helpers.

Files touched: `t/t5564-http-proxy.sh`, `t/t6030-bisect-porcelain.sh`, `t/t7400-submodule-basic.sh`, `t/t9001-send-email.sh`.

## In brief
- **[2026/11/15/15-13-13 by Okhuomon Ajayi]** Addressed Junio’s objection to the timeout variable naming cleanup by enumerating all existing lock-timeout variables and confirming they are uniformly measured in milliseconds, justifying the renaming of `credentialStore.lockTimeoutMS` to `credentialStore.lockTimeout`.
- **[2026/11/15/12-47-24 by Siddharth Asthana]** Proposed rewording error messages in the `git apply` crash fix to reference the problematic header line (`"---"` before `"new file mode"`) instead of stripped filenames, improving clarity for users.
- **[2026/11/15/04-35-27 by Srujith A Poojary]** Posted v2 of the archive refactoring patch, focusing exclusively on the tar archiver, removing the unused `struct repository *` parameter from `init_zip_archiver()`, and revising the commit message to accurately reflect the scope.
- **[2026/11/15/12-47-42 by Kristofer Karlsson]** Posted v3 of the `struct commit` memory optimization patch, refining comments to clarify safe casting direction, the meaning of `index`, and the source of `date`.
- **[2026/11/15/09-58-29 by Yusuke Hayashi]** Clarified the performance rationale for removing the pre-sizing logic in `get_info_packs()`, arguing that the overhead of dynamic `strbuf` growth is negligible compared to CGI startup and I/O costs in this context.
- **[2026/11/15/10-09-49 by Yikun Wei]** Withdrew the documentation patch after confirming that `config/fmt-merge-msg.adoc` is already transitively included via `config/merge.adoc`.
- **[2026/11/15/07-03-37 by Dirk Gouders]** Resent the documentation patch for `gitcli` (v3) with procedural corrections, incorporating all substantive feedback from Junio. The patch is ready for merging.