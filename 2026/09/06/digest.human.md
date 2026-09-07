# Git mailing list daily digest for 2026/09/06

## The day in brief
The Git mailing list saw active discussion on performance optimizations for `git push` from shallow clones, with Elijah Newren posting a six-patch v3 series that changes the default behavior of `push.shallowExcludeBoundary` to `true`. Kristoffer Haugsbakk shared new guidance on AI attribution following the Linux kernel's policy change, while Junio C Hamano initiated a meta-discussion on version numbering for the next major Git release. Several bugfixes and design proposals also advanced, including a persistent exclude patterns RFC for `git clean`.

## Notable threads

### `git ls-files` performance optimization and AI attribution
The long-running thread about Tamir Duberstein's `git ls-files` performance optimization saw new input today focusing on metadata rather than technical implementation. Kristoffer Haugsbakk shared that the Linux kernel's July 2026 policy change (linux/816d9992) now mandates `Assisted-by: LLM` instead of model-specific attribution like `Assisted-by: Codex gpt-5.5`. This provides a project-relevant precedent for Git's handling of AI-assisted contributions.

The optimization itself, which filters pathspecs early to avoid expensive lstat operations, remains technically settled. The series has demonstrated dramatic speed improvements (60.7s→1.06s) for large repositories when most entries don't match the pathspec, while reducing worst-case overhead to negligible levels (~5ms). The discussion about attribution practices doesn't challenge the technical merits but offers guidance for future AI-assisted contributions to the project.

### `git push` performance from shallow clones
Elijah Newren posted a significant v3 series (six patches) that optimizes `git push` performance from shallow clones by avoiding redundant tree transfers. The core change introduces a tri-state config option `push.shallowExcludeBoundary` (values: `true`/`false`/`abort`) that lets users omit shallow boundary objects from the pack when pushing.

The series has grown to address edge cases and usability concerns raised in review. Patch 5 changes the default value from `false` to `true`, enabling the optimization by default, while patch 6 adds user-facing advice for splitting multi-ref pushes when shallow boundary exclusions cause failures. The preparatory patches (1-3) improve error handling in `unpack-objects.c`, `receive-pack.c`, and `shallow.c`, making diagnostics more precise and user-friendly.

This work directly addresses a real-world pain point where `git push` from shallow clones resends the entire toplevel tree (gigabytes in large repos) even for tiny changes. The new default argues that sending shallow boundaries is almost always wasted work, with minimal compatibility cost. The series is now complete and appears ready for maintainer consideration, though the default change may still draw scrutiny.

### ODB alternates handling refactoring
Patrick Steinhardt's 8-patch series refactoring ODB alternates handling during repository creation saw substantive review engagement today. Justin Tobler raised two architectural questions: first about the growing complexity of `init_db()`'s "skip" flags, suggesting explicit setup of the refdb and ODB might be cleaner; and second about whether `git clone --shared` and similar options should be restricted to the "files" backend or left to backend discretion.

The series removes the ability to write ODB alternates after repository creation, simplifying the ODB interface. This change prepares for alternates to become an implementation detail of the ODB backend, giving backends more control over their configuration. The refactoring defers ODB creation in `git clone` until after URI resolution and alternates collection, ensuring alternates are written with full context. All patches are now complete and the series appears technically ready, though these design questions may influence future ODB abstraction work.

### `--force-if-includes` bugfix and design discussion
The thread about Aleksei Sviridkin's patch fixing an uninitialized variable in the `--force-if-includes` safety mechanism saw continued design discussion today. Junio C Hamano questioned the chosen fallback value (`timestamp_t date = 0`) for the reflog walk, suggesting alternatives like `gc.reflogExpire` or 90 days might be more reasonable. Aleksei responded with a concrete scenario demonstrating why `0` is the only safe choice, as older reflog entries can persist and must be checked to prevent unsafe pushes.

The bug causes the reflog walk to terminate prematurely when the remote-tracking ref has no reflog, leading to false positives in the "remote ref updated since checkout" check. The fix is a one-line initialization in `remote.c` that ensures correct reflog traversal. While the patch itself is ready for integration, this exchange about the fallback value may inform future refinements to the safety mechanism.

## In brief
- **`format-patch` range-diff notes**: Documentation clarification confirmed the abandoned feature already followed the independent-control design Junio sketched, despite the original phrasing obscuring it. Junio proposed an alternative rule to simplify the toggle mechanism.
- **ODB refactoring**: Justin Tobler confirmed the safe removal of the last call to `odb_add_submodule_source_by_path()` in `submodule-config.c`, eliminating a 2019 workaround.
- **`rerere` race condition**: Thomas Bachem proposed retaining v3's behavior for conflict stops (warning + proceeding) while keeping lock-timeout failures for other operations, advancing the error-handling discussion.
- **`--force-if-includes` fix**: Tyler Cipriani confirmed the detached HEAD rejection policy remains unchanged in v2, treating the reflog limitation as a settled trade-off.
- **CI update**: Jeff King noted the `linux32` job still uses Ubuntu 20.04 (out of LTS) due to i386 support, and the `linux-TEST-vars` job may serve dual purposes.
- **Version numbering**: Brian M. Carlson advocated for a cautious delay (2.97 or 2.95) before Git 3.0 to allow time for the lowercase-only object IDs series and other foundational work to stabilize.
- **`git clean` RFC**: Nicolas Jeanmonod proposed a new `clean.exclude` config key to persistently protect untracked paths from `git clean`, even with `-x`.