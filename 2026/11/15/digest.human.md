# Git mailing list daily digest for 2026/11/15

## The day in brief
The `git repo info` path keys series received final approval and is ready for merging, while the `change-id` preservation thread saw new arguments about workflow benefits and security risks. Integer-overflow hardening patches advanced with API changes and macro removals, and a bugfix for `git apply --check` gained polish with memory-leak and error-message improvements.

## Notable threads

### `change-id` preservation: workflow vs. security
The long-running discussion about preserving `change-id` headers during commit rewriting operations saw two consequential exchanges today.

Ewout ter Hoeven [2026/11/15/11-03-05] countered brian m. carlson’s privacy and utility objections with concrete use cases from GitHub’s `gh-stack` and GitLab’s `glab stack`. These tools struggle to maintain commit identity across rewrites, relying on heuristics (branch names, patch IDs) that fail in common scenarios: branch renames break tracking, rewritten commits trigger redundant replays, and patch ID mismatches cause cascading errors. Ewout cited specific GitHub and GitLab issues where users report dozens of incorrectly replayed commits, arguing that `change-id` preservation would eliminate these workarounds by making Git the authoritative source of commit identity.

brian m. carlson [2026/11/15/16-24-24] responded with a critical trust issue: **change-id headers cannot be relied upon as stable identifiers because users can manually edit them**. This creates a security risk for automated review systems that might assume a preserved `change-id` implies an unchanged patch. A malicious actor could exploit this by reusing a reviewed `change-id` for an entirely different commit, tricking the server into accepting unvetted code. Brian’s objection reframes the problem as a **security boundary question**: any solution must either restrict `change-id` preservation to server-controlled operations or accept that the header cannot be trusted for security-sensitive decisions.

The thread’s tension now centers on whether these security requirements can coexist with the workflow needs, or if the two are fundamentally incompatible.

### `git repo info` path keys: final approval
The `[PATCH v9]` series adding four new path-related keys (`path.working_tree`, `path.superproject_working_tree`, `path.index`, and `path.prefix`) to `git repo info` received explicit approval for merging.

Phillip Wood [2026/11/15/10-33-45] confirmed the range-diff between v8 and v9 is mechanically correct and stated, *"I'd be happy if this is merged as-is."* The only remaining concern—test efficiency, specifically creating a new repository for each test—was acknowledged but not treated as a blocker. The series is now uncontroversial, with all design objections resolved, naming conventions settled, and Phillip’s explicit approval. Junio can queue it without further changes.

The keys introduced are:
- `path.working_tree.absolute`/`path.working_tree.relative`
- `path.superproject_working_tree.absolute`/`path.superproject_working_tree.relative`
- `path.index.absolute`/`path.index.relative`
- `path.prefix` (relative path from the top-level working tree to the current directory, e.g., `sub/dir/`)

### Integer-overflow hardening: API changes and macro removals
Johannes Schindelin advanced the integer-overflow hardening series with three key updates today, addressing Phillip Wood’s review of the wrapper and date subsystems.

In [2026/11/15/19-24-46], Johannes revised the wrapper subsystem fix to **change the API contract** rather than just add defensive checks. `write_in_full()` now returns `0` on success (eliminating overflow risk entirely), and a new `read_in_full_or_error()` helper is introduced for callers that only need success/failure status. The commit message’s original claim that "git never passes such large counts" is softened, acknowledging that future callers might inadvertently trigger the issue. This aligns with Phillip’s citation of a 2019 bugfix where `ssize_t` was narrower than `size_t` on NonStop platforms.

Johannes then [2026/11/15/19-29-57] **dropped the `signed_mult_overflows` and `mult_overflows` macros entirely**, acknowledging the approach as flawed due to its GCC-only limitation and operand-width assertion concerns. This leaves the series without a portable overflow-checking mechanism for signed arithmetic.

Finally, [2026/11/15/19-35-12] withdrew the original date.c hardening implementation and proposed a simpler, portable approach: capping user-supplied numbers at `INT_MAX` and guarding multiplications manually. This avoids compiler-specific overflow detection and handles both signed and unsigned `time_t` uniformly.

### `git apply --check` crash fix: polish
The two-part bugfix series for a crash in `git apply --check` when processing malformed patch headers gained final polish today.

Siddharth Asthana [2026/11/15/12-39-29] identified a **memory leak** in `parse_git_diff_header()`: the function overwrites `patch->def_name` without freeing the old value, causing leaks when processing multiple `diff --git` headers. The fix is trivial—a `free()` before assignment—and both callers already zero the `patch` struct, making it safe. Asthana suggested addressing this in a **preparatory patch** to retain the test case for empty `diff --git` sections, which would improve coverage without complicating the main fix.

In a separate review [2026/11/15/12-47-24], Asthana proposed rewording error messages to reference the problematic header line (`"---"` before `"new file mode"`) instead of stripped filenames, making the messages more actionable for users. The patch now rejects corrupt patch headers outright, aligning with Junio’s guidance to reject any format not generated by `git diff` or `git format-patch`.

## In brief
- **Timeout variable naming**: Okhuomon Ajayi [2026/11/15/15-13-13] addressed Junio’s substantive objection by enumerating all existing lock-timeout variables and confirming they are uniformly measured in milliseconds, justifying the renaming of `credentialStore.lockTimeoutMS` to `credentialStore.lockTimeout`.
- **Archive refactoring**: Srujith A Poojary [2026/11/15/04-35-27] posted v2 of the archive subsystem refactoring, focusing exclusively on the tar archiver and removing an unused `struct repository *` parameter from `init_zip_archiver()`.
- **`struct commit` memory optimization**: Kristofer Karlsson [2026/11/15/12-47-42] posted v3 of the patch, refining comments to clarify safe casting direction, the meaning of `index`, and the source of `date`.
- **HTTP backend cleanup**: Yusuke Hayashi [2026/11/15/09-58-29] clarified the performance rationale for removing pre-sizing logic in `get_info_packs()`, arguing that dynamic `strbuf` growth overhead is negligible compared to CGI startup and I/O costs.
- **Documentation retraction**: Yikun Wei [2026/11/15/10-09-49] withdrew the `fmt-merge-msg` documentation patch after confirming the variables are already transitively included via `config/merge.adoc`.
- **Test modernization**: Hardik Kumar [2026/11/15/18-15-21] replaced `!` with `test_must_fail` in four test scripts to handle edge cases like crashes or signals.
- **Pretty colors**: Andy Koppe [2026/11/15/21-35-03] fixed lost and leaked ANSI color escape sequences in custom log formats by post-processing output to ensure each line is self-contained.