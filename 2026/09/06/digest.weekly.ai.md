# The Git Project -- Weekly Digest for 2026/08/31 -- 2026/09/06

## The period in brief

The week of 2026/08/31 -- 2026/09/06 was exceptionally active, with six days of high-volume traffic. The mailing list saw **major progress on long-running architectural efforts** (ODB abstraction, ref storage terminology), **critical bugfixes** (shallow clone pushes, `--force-if-includes`), and **feature completions** (`git replay --linearize`, `--missing-only` for `git rev-list`). Two topics dominated: the ODB abstraction effort (Patrick Steinhardt) and the `--missing-only` option for `git rev-list` (Siddharth Asthana), both of which reached integration-ready states. The week also brought clarity to several contentious design questions, including shallow clone push performance and `rerere` lock contention.

---

## Key developments

### ODB abstraction reaches key milestones
Patrick Steinhardt's multi-year effort to abstract Git's object database (ODB) saw significant progress this week. Two new topics landed in `next`: `ps/odb-alternates-at-creation` (8 patches) and `ps/odb-pluggable-fsck` (10 patches). The alternates series removes the ad-hoc alternates writing API, deferring alternates setup to ODB creation, while the fsck series moves fsck checks into backend-specific layers. These changes simplify the ODB interface and prepare for pluggable backends.

The week also saw substantive review of Steinhardt's 8-patch series refactoring ODB alternates handling during repository creation. Justin Tobler raised architectural questions about the growing complexity of `init_db()`'s "skip" flags and whether `git clone --shared` should be restricted to the "files" backend. The series is technically complete and ready for integration, though these design questions may influence future ODB work.

### `--missing-only` for `git rev-list` approved and queued
Siddharth Asthana's `--missing-only` option for `git rev-list` was approved and queued for `next` after a final commit message tweak clarified GitLab's Gitaly workflow. The feature provides a script-friendly way to list only missing object IDs (without `?` prefix) while preserving existing `--missing=` formatting options. It directly supports GitLab's need to efficiently identify objects not present in a partial clone during transaction packing.

The implementation is complete, with all review feedback incorporated. The option requires `--missing=print` or `--missing=print-info`, rejects incompatible options (`--count`, `--disk-usage`), and outputs one OID per line (or `path=`/`type=` fields with `--missing=print-info`).

### `git replay --linearize` reaches v9
Toon Claes posted v9 of the series introducing `--linearize` to `git replay`, which flattens merge commits into a linear history. The series is now technically complete, addressing all prior feedback including UX terminology refinements and the multi-branch ambiguity. The `--linearize` option cannot be used with multiple branches or `--contained`, and merge commits are dropped with subsequent commits reparented onto the last non-merge commit.

Elijah Newren's Reviewed-by on the entire series signals technical completion and readiness for integration. The feature provides a simpler alternative to Johannes Schindelin's earlier merge-replay implementation, offering predictable all-or-nothing flattening behavior.

### `--force-if-includes` safety mechanism fixed
Tyler Cipriani and Aleksei Sviridkin posted a critical bugfix for the `--force-if-includes` safety mechanism, which was incorrectly checking the reflog of the *local* branch whose name matched the *remote* destination branch rather than the branch actually being pushed. This flaw could cause false rejections or unintended data loss.

The two-patch series fixes the core logic to consult the reflog of the pushed ref and adds special handling for `HEAD`-based pushes and detached HEAD states. It also introduces a new `advice.forceIfIncludesDetachedHead` config knob and updates advice messages to be actionable and consistent across transport layers. Eight new test cases cover mismatched local/remote names, `HEAD`-based pushes, and detached HEAD states.

### Ref storage terminology unification
Patrick Steinhardt posted an 11-patch series to unify Git's inconsistent terminology for "ref storage format" across CLI options, environment variables, config keys, and documentation. The series systematically renames all occurrences to "ref storage" for consistency:
- `--ref-format=` → `--ref-storage=` (with backward-compatible aliases)
- `GIT_REFERENCE_BACKEND` → `GIT_REF_STORAGE`
- `init.defaultRefFormat` → `init.defaultRefStorage`

The series also centralizes URI parsing for reference storage backends, enabling `--ref-storage=` to accept URIs like `files://path/to/repo`. Karthik Nayak provided substantive review, while Junio C Hamano raised a design-level concern about whether "ref-storage" is meaningfully clearer than "ref-format."

### `rerere` lock contention resolved
Thomas Bachem posted a series addressing a race condition between `git rebase` and background `rerere gc` maintenance. The v3 patch adds configurable locking timeouts to `rerere_setup()`, allowing foreground operations to wait for the lock rather than failing immediately. The series now includes:
- Configurable timeout (`rerere.lockTimeout`, default 1000ms)
- `BUG()` assertions for incompatible flag combinations
- Clarified documentation about command behavior under lock contention

The patch addresses all prior feedback and resolves the geometric maintenance strategy's lock contention issue introduced in Git 2.54. Junio clarified that after waiting, commands should fail rather than proceed, as a held lock signals another process is actively modifying the `rr-cache`.

### `git push` performance from shallow clones
Elijah Newren posted a significant v3 series (six patches) that optimizes `git push` performance from shallow clones by avoiding redundant tree transfers. The core change introduces a tri-state config option `push.shallowExcludeBoundary` (values: `true`/`false`/`abort`) that lets users omit shallow boundary objects from the pack when pushing.

The series changes the default value from `false` to `true`, enabling the optimization by default, while adding user-facing advice for splitting multi-ref pushes when shallow boundary exclusions cause failures. The preparatory patches improve error handling in `unpack-objects.c`, `receive-pack.c`, and `shallow.c`, making diagnostics more precise and user-friendly.

This work directly addresses a real-world pain point where `git push` from shallow clones resends the entire toplevel tree (gigabytes in large repos) even for tiny changes. The new default argues that sending shallow boundaries is almost always wasted work, with minimal compatibility cost.

---

## In brief

**`receive-report` hook** -- Karthik Nayak's four-patch series introducing the `receive-report` hook for `git-receive-pack` reached v7, addressing all prior feedback and preparing for integration. The hook allows server administrators to intercept and modify the status report sent to clients after ref updates are committed but before the response is finalized. The series is queued in `seen` and ready for `next`.

**`git repack --drop-filtered`** -- Samuel Bronson provided substantive usability feedback on the `git repack --drop-filtered` series, identifying a pain point where the current implementation dies when encountering an index-referenced blob. The author agreed to change the guard from a fatal error to a warning that skips index-referenced blobs.

**Outreachy December 2026 cohort** -- Git's participation in Outreachy gained momentum, with four confirmed or potential mentors (Christian Couder, Usman Akinyemi, Kaartic Sivaraam, Pablo Sabater) and two org admin candidates. The project will propose two ideas: continuing the removal of global state (libifying code) and improving command argument and option parsing.

**`git imap-send --draft`** -- Wolfgang Faust's patch adding a `--draft` option to `git imap-send` to mark uploaded messages with the IMAP `\Draft` flag was updated to address Junio's requests. The v2 patch removes the `Assisted-by: LLM` trailer and improves test cleanup placement.

**`git checkout -m` autostash conflict handling** -- Harald Nordgren's series refining `git checkout -m` behavior when autostash is involved reached v5 and was queued for `next`. The series separates autostash conflict advice from branch-switch confirmation messages, improving output readability during conflict scenarios.

**`git var` extension** -- Junio C Hamano provided substantive review of Andrew Pleeter's `git var` extension patch, requesting documentation improvements, code structure changes, and usability edge case handling. The patch extends `git var` to subsume all `git ident` functionality.

**`git cherry-pick --no-commit` documentation** -- Aleksei Sviridkin's documentation patch clarifying that `CHERRY_PICK_HEAD` is not created when a `--no-commit` cherry-pick fails due to conflicts was updated with revived test coverage. The test verifies the absence of `CHERRY_PICK_HEAD` in this edge case.

**Pathspec handling bugfixes** -- A three-patch series addressing a long-standing edge case in `common_prefix_len()` and a memory-safety issue in `match_pathspec_with_flags()` was queued for merging. The series fixes an edge case where an exclude pathspec appearing first in the list causes `common_prefix_len()` to incorrectly return zero, and prevents a heap-buffer-overflow when negative pathspecs are shorter than the common prefix.

**CI modernization** -- Jeff King updated Git's CI workflows to use Debian 12 instead of Debian 11, citing Debian 11's end-of-LTS status. The patch updates `.github/workflows/main.yml` and `.gitlab-ci.yml` to point to `debian:12` and adjusts the support window comment.

**`git ls-files` performance optimization** -- Tamir Duberstein's optimization filtering pathspecs early to avoid expensive lstat operations was discussed in the context of AI attribution. Kristoffer Haugsbakk shared that the Linux kernel's July 2026 policy change now mandates `Assisted-by: LLM` instead of model-specific attribution.

**`git clean` persistent excludes** -- Nicolas Jeanmonod proposed a new `clean.exclude` config key to persistently protect untracked paths from `git clean`, even with `-x`. The RFC suggests a design similar to `.gitignore` but for `git clean`.

**Version numbering** -- Junio C Hamano initiated a meta-discussion on version numbering for the next major Git release. Brian M. Carlson advocated for a cautious delay (2.97 or 2.95) before Git 3.0 to allow time for foundational work to stabilize.

**Test modernization** -- Junio C Hamano rejected a mechanical replacement of `test -f` with `test_path_is_file` in 63 test scripts, noting the helper functions are designed for assertions, not silent conditionals. Each `test -f` must be analyzed for its semantic purpose.

**macOS regression** -- Ramkumar Ramachandra reported three regressions in Git 2.55.0 on macOS, which were isolated to CrowdStrike Falcon's real-time file scanning racing against Git's high-frequency operations. The issues were resolved by adjusting security software settings.

---

## Looking ahead

The next week is likely to see integration of several major topics that reached completion this week:
- `--missing-only` for `git rev-list` (queued for `next`)
- `git replay --linearize` (technically complete, awaiting integration)
- `receive-report` hook (v7, ready for `next`)
- `--force-if-includes` bugfix (technically complete)
- `rerere` lock contention fix (design settled, v3 in review)

The ODB abstraction effort will continue, with Patrick Steinhardt's alternates refactoring series likely to land. The shallow clone push performance series (Elijah Newren) may see further discussion about its default behavior change.

The Outreachy December 2026 cohort application will be submitted, with project ideas finalized. The version numbering discussion may continue, with a decision about Git 3.0's timeline likely to emerge.

Several documentation efforts are nearing completion, including the `git cherry-pick --no-commit` clarification and the ref storage terminology unification. The `git var` extension and `git ident` redesign may see further iterations based on maintainer feedback.