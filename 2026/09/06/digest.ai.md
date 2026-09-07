# Git mailing list daily digest for 2026/09/06

## The day in brief
The Git mailing list saw significant activity around performance optimizations for `git push` from shallow clones, with Elijah Newren posting a six-patch v3 series that changes the default behavior of `push.shallowExcludeBoundary` to `true`. Kristoffer Haugsbakk shared new information about Linux kernel policy for AI-assisted contributions, influencing Git's approach to attribution. Design discussions continued on version numbering for the next major release, with Brian M. Carlson advocating for a cautious delay before 3.0. Several bugfixes and refactoring efforts also progressed, including Patrick Steinhardt's ODB alternates handling series and Aleksei Sviridkin's fix for `--force-if-includes`.

## Notable threads

### `git ls-files` performance optimization and AI attribution
The thread on `git ls-files` performance optimization saw new discussion about AI attribution practices. Kristoffer Haugsbakk shared that the Linux kernel's July 2026 policy change (linux/816d9992) now mandates `Assisted-by: LLM` instead of model-specific attribution like `Assisted-by: Codex gpt-5.5`. This provides a project-relevant precedent for Git's handling of AI-assisted contributions, simplifying attribution while avoiding vendor-specific references.

The technical aspects of the `ls-files` optimization remain settled: the patch dramatically improves performance for selective pathspecs (60.7s→1.06s) while maintaining negligible overhead (~5ms) in worst-case scenarios. The attribution discussion is now focused on aligning with emerging best practices across open-source projects.

### `git push` performance from shallow clones
Elijah Newren posted a significant v3 series (six patches) optimizing `git push` performance from shallow clones. The series introduces a tri-state config option `push.shallowExcludeBoundary` (values: `true`/`false`/`abort`) that controls whether shallow boundary objects are omitted from the pack when pushing.

Key developments in v3:
- Patch 1/6 improves error reporting in `unpack-objects.c` by distinguishing missing objects from type mismatches
- Patch 2/6 silences redundant per-ref connectivity error messages in `receive-pack.c`
- Patch 3/6 treats missing boundary commits as traversal endpoints rather than fatal errors
- Patch 4/6 implements the core optimization with the tri-state config option
- **Patch 5/6 changes the default from `false` to `true`**, enabling the optimization by default
- Patch 6/6 adds advice for users to split multi-ref pushes when shallow boundary exclusions cause failures

The series addresses the core inefficiency where `git push` from shallow clones resends entire toplevel trees (gigabytes in large repos) even for tiny changes. The new default (`true`) assumes the server already has the shallow grafts, which is almost always true and avoids wasted work. The `abort` option lets users opt out entirely if they prefer not to choose between performance and correctness.

The false-positive failure mode (where a multi-ref push is rejected because one ref's excluded boundary omits objects needed by another) is now addressed with improved error messages and recovery advice. This directly responds to Derrick Stolee's earlier concerns about usability.

### ODB alternates handling refactoring
Patrick Steinhardt's eight-patch series refactoring ODB alternates handling during repository creation saw continued review. Justin Tobler raised two substantive questions:

1. About the growing complexity of `init_db()`'s "skip" flags, suggesting explicit setup of the refdb and ODB might be cleaner than accumulating skip flags
2. Whether `git clone --shared` and similar options should be restricted to the "files" backend or if their behavior should be left to the backend's discretion

The series removes the ability to write ODB alternates after repository creation, simplifying the ODB interface. It introduces a new option in `odb_source_create_on_disk()` to handle alternates at creation time, eliminating the need for a separate `write_alternates()` callback. This change prepares for alternates to become an implementation detail of the ODB backend.

The patch that removes the last call to `odb_add_submodule_source_by_path()` in `submodule-config.c` received confirmation from Justin Tobler, who verified the safe removal of a 2019 workaround that became unnecessary after earlier patches removed implicit dependencies on `the_repository`.

### Version numbering for next major release
Junio C Hamano initiated a meta-discussion about version numbering for the next major Git release after 2.56. He presented four options: 3.0, 2.99 (transitional), 2.98/2.97 (delayed), or 2.57 (business as usual).

Brian M. Carlson advocated for a cautious delay (option 3: 2.97 or 2.95) to allow time for key work to stabilize, particularly:
- The lowercase-only object IDs series
- SHA-256 support on major forges (upcoming update at Git Merge)
- libgit2's SHA-256 and reftable support

Carlson explicitly excluded Rust platform support as a blocker and dismissed other SHA-256/reftable work as unlikely to be ready in time. His position reflects a preference for a deliberate, staged transition rather than an abrupt version bump.

### `--force-if-includes` bugfix and design discussion
The thread on fixing an uninitialized variable in `--force-if-includes` saw continued design discussion about the fallback timestamp value. Aleksei Sviridkin provided a scenario demonstrating why `timestamp_t date = 0` (epoch start) is the only safe choice, as older reflog entries can persist until explicitly expired and must be checked to prevent unsafe pushes.

Junio C Hamano had suggested alternatives like `gc.reflogExpire` or 90 days as more reasonable, but Aleksei's scenario shows these could allow unsafe pushes in edge cases. The discussion now centers on the trade-off between safety (full reflog scan) and performance (avoiding redundant work).

The patch itself (a one-line initialization in `remote.c`) remains technically ready for integration, with the design discussion likely to inform future refinements to the safety mechanism.

## In brief
- **[format-patch range-diff notes]** Kristoffer Haugsbakk provided documentation clarifications for the abandoned `--[no-]range-diff-notes` feature, confirming the implementation already followed the independent-control design Junio sketched. Junio proposed an alternative rule to simplify the toggle mechanism.
- **[rerere race condition]** Thomas Bachem proposed retaining v3's behavior for conflict stops (warning + proceeding) while keeping lock-timeout failures for other operations, preserving rebase continuity while preventing silent corruption.
- **[push force-if-includes fix]** Tyler Cipriani confirmed the detached HEAD rejection policy remains unchanged in v2 of his series fixing `--force-if-includes` to check the correct reflog.
- **[CI update]** Jeff King noted that the `linux32` job still uses Ubuntu 20.04 (out of LTS) due to i386 support limitations, and the `linux-TEST-vars` job may serve dual purposes.
- **[git clean persistent excludes]** Nicolas Jeanmonod proposed a new `clean.exclude` config key to persistently protect untracked paths from `git clean`, even with `-x`.