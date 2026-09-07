# Git Mailing List Weekly Digest
**Period: 2026/08/31 -- 2026/09/06**

## The period in brief
This week saw significant progress across several major efforts in the Git project. The period was moderately busy with 6 active days of development, featuring both long-running architectural work and immediate bugfixes. Three developments stand out: the `git replay --linearize` feature reached technical completion, the `--missing-only` option for `git rev-list` was approved for integration after months of refinement, and a critical bug in the `--force-if-includes` safety mechanism was identified and fixed. The Outreachy December 2026 cohort also gained momentum with new mentor volunteers.

---

## Key developments

### `git replay --linearize` reaches technical completion
Toon Claes's nine-iteration series introducing `--linearize` to `git replay` reached v9, addressing all prior feedback including UX terminology refinements and the multi-branch ambiguity. The feature flattens merge commits into a linear history, providing a simpler alternative to Johannes Schindelin's earlier merge-replay implementation. Elijah Newren provided a Reviewed-by on the entire series, signaling readiness for integration. The implementation prevents `--linearize` from being used with multiple branches or `--contained`, and drops merge commits while reparenting subsequent commits onto the last non-merge commit. The series is now technically complete, with only documentation refinements remaining.

---

### `--missing-only` for `git rev-list` approved for integration
Siddharth Asthana's `--missing-only` option for `git rev-list` received final maintainer sign-off and was queued for `next`. This feature directly supports GitLab's Gitaly partial clone workflows by enabling efficient identification of missing objects in a single-pass transaction packing operation. The implementation outputs only missing object IDs without post-processing, preserving existing `--missing=` formatting options. Junio C Hamano approved the feature after the commit message was strengthened to clarify the Gitaly workflow, and the series now includes clear error messages for incompatible options (`--count`, `--disk-usage`). The feature is ready for integration and represents a significant milestone in partial clone support.

---

### `--force-if-includes` safety mechanism bugfix
Tyler Cipriani and Aleksei Sviridkin identified and fixed a critical flaw in the `--force-if-includes` safety mechanism. The feature was incorrectly checking the reflog of the *local* branch whose name matched the *remote* destination branch (e.g., `main` when pushing to `origin/main`), rather than the branch actually being pushed (e.g., `src` when pushing `src:main`). This could lead to false rejections or unintended data loss. The two-patch series fixes the core logic to consult the reflog of the pushed ref and adds special handling for `HEAD`-based pushes and detached HEAD states. The series also introduces a new `advice.forceIfIncludesDetachedHead` config knob and includes eight new test cases covering edge cases. The fix is ready for integration and addresses a long-standing safety concern.

---

### `receive-report` hook series reaches v7
Karthik Nayak posted v7 of the four-patch series introducing the `receive-report` hook for `git-receive-pack`. This hook allows server administrators to intercept and modify the status report sent to clients after ref updates are committed but before the response is finalized. The series is now feature-complete, addressing all prior feedback including unified report generation and code style improvements. The hook is motivated by GitLab's need to implement multi-version concurrency control (MVCC), where the final status must reflect operations occurring after `ref_transaction_commit()`. The design intentionally decouples the reported status from the actual repository state, allowing a hook to report a push as failed while still committing ref updates. The series is queued in `seen` and ready for `next`.

---

### Ref storage terminology unification
Patrick Steinhardt posted an 11-patch series to unify Git's inconsistent terminology for "ref storage format" across CLI options, environment variables, config keys, and documentation. The series systematically renames all occurrences to "ref storage" for consistency, changing `--ref-format=` to `--ref-storage=`, `GIT_REFERENCE_BACKEND` to `GIT_REF_STORAGE`, and `init.defaultRefFormat` to `init.defaultRefStorage`. The series also centralizes URI parsing for reference storage backends, enabling `--ref-storage=` to accept URIs like `files://path/to/repo`. Karthik Nayak provided substantive review, calling the series "relatively quite an easy read" despite its length, while Junio C Hamano raised design-level concerns about whether "ref-storage" is meaningfully clearer than "ref-format." The series prepares for an analogous "object storage" extension and is ready for further review.

---

### `git push` performance from shallow clones
Elijah Newren posted a significant v3 series (six patches) that optimizes `git push` performance from shallow clones by avoiding redundant tree transfers. The core change introduces a tri-state config option `push.shallowExcludeBoundary` (values: `true`/`false`/`abort`) that lets users omit shallow boundary objects from the pack when pushing. The series changes the default value from `false` to `true`, enabling the optimization by default, and adds user-facing advice for splitting multi-ref pushes when shallow boundary exclusions cause failures. The preparatory patches improve error handling in `unpack-objects.c`, `receive-pack.c`, and `shallow.c`, making diagnostics more precise and user-friendly. This work addresses a real-world pain point where `git push` from shallow clones resends the entire toplevel tree (gigabytes in large repos) even for tiny changes. The series is now complete and appears ready for maintainer consideration.

---

### Outreachy December 2026 cohort gains momentum
The Outreachy December 2026 cohort saw increased participation this week, with Pablo Sabater confirming his availability to co-mentor a project. The thread now has four confirmed or potential mentors (Christian Couder, Usman Akinyemi, Kaartic Sivaraam, Pablo Sabater) and two org admin candidates. Christian Couder will submit Git's application imminently, proposing two project ideas: continuing the removal of global state (libifying code) and improving command argument and option parsing. The deadline for mentoring organizations to sign up is September 11, 2026, and Git's application is well-positioned with sufficient volunteer support.

---

## In brief

**`git repack --drop-filtered` usability feedback** -- Samuel Bronson identified a pain point in the `git repack --drop-filtered` series where the current implementation dies when encountering an index-referenced blob, forcing users to restart the entire enumeration process. The author agreed to change the behavior to skip index-referenced blobs with a warning instead of failing.

**`git rerere` lock contention fix** -- Thomas Bachem posted v3 of a patch addressing a race condition between `git rebase` and background `rerere gc` maintenance. The patch adds configurable locking timeouts to `rerere_setup()`, allowing foreground operations to wait for the lock rather than failing immediately. The design is now settled for a two-patch series that also disables background maintenance during rebase.

**`git repo info` path-related keys** -- K Jayatheerth confirmed the fix for a correctness bug in the `path.superproject-root` implementation, where the underlying function was ignoring its `repo` parameter and using `xgetcwd()` instead. The fix will be included in v6 of the series, which adds seven new path-related keys to `git repo info`.

**CI modernization** -- Jeff King updated Git's CI workflows to use Debian 12 instead of Debian 11, citing Debian 11's end-of-LTS status. The patch updates `.github/workflows/main.yml` and `.gitlab-ci.yml` to point to `debian:12` and adjusts the support window comment to reflect the new LTS end date (2028-06-30).

**`git imap-send --draft`** -- Wolfgang Faust posted a patch adding a `--draft` option to `git imap-send` to mark uploaded messages with the IMAP `\Draft` flag. The v2 patch addresses Junio's requests to remove the `Assisted-by: LLM` trailer and improve test cleanup placement. The feature addresses a long-standing usability gap where `git imap-send` lacked a way to signal draft status to IMAP clients.

**`git cherry-pick --no-commit` documentation** -- Aleksei Sviridkin revived the test patch in the `git cherry-pick --no-commit` documentation series, adding a one-line check to guard against regressions in the "tricky" logic of `do_pick_commit()`. The test verifies that `CHERRY_PICK_HEAD` is not created when a `git cherry-pick --no-commit` operation fails due to conflicts.

**`git history` NULL-dereference fix** -- Patrick Steinhardt confirmed a fix "looks good to me" for a NULL-pointer dereference crash in `write_ondisk_index()` when the parent commit's tree object is missing. The fix is ready for integration.

**ODB alternates refactoring** -- Patrick Steinhardt's 8-patch series refactoring ODB alternates handling during repository creation saw substantive review. The series removes the ability to write ODB alternates after repository creation, simplifying the ODB interface and preparing for alternates to become an implementation detail of the ODB backend.

**`parse-options` early argument scanning** -- Christian Couder posted a six-patch series introducing a new `parse-options` sub-API (`early_scan_options()`) to fix argument-parsing bugs in commands like `git bisect` and `git rev-parse`. Junio C Hamano raised substantive design concerns, questioning the long-term viability of the new API and the practical value of the fixes.

**`hooks.allowNoVerify` rejected** -- Junio C Hamano rejected Alessio Attilio's `hooks.allowNoVerify` feature, citing concerns about configuration complexity creep and the narrow scope of the feature. The rejection highlights a philosophical divide about whether Git should provide guardrails for workflow discipline or defer to user education.

**Test modernization** -- Junio C Hamano rejected Mark C. Chu-Carroll's patch series replacing `test -f` with `test_path_is_file` in 63 test scripts, noting the mechanical replacement was incorrect in control-flow contexts where the helper functions are designed for assertions, not silent conditionals.

**`git maintenance` rerere gc heuristic** -- Patrick Steinhardt posted a two-patch series improving the heuristic for the `git maintenance` `rerere gc` task, replacing the hard-coded 60-day cutoff with a dynamic heuristic.

**AI attribution guidance** -- Kristoffer Haugsbakk shared new guidance on AI attribution following the Linux kernel's policy change, which now mandates `Assisted-by: LLM` instead of model-specific attribution. This provides a project-relevant precedent for Git's handling of AI-assisted contributions.

**Version numbering discussion** -- Junio C Hamano initiated a meta-discussion on version numbering for the next major Git release, with Brian M. Carlson advocating for a cautious delay (2.97 or 2.95) before Git 3.0 to allow time for foundational work to stabilize.

**`git clean` RFC** -- Nicolas Jeanmonod proposed a new `clean.exclude` config key to persistently protect untracked paths from `git clean`, even with `-x`.

---

## Looking ahead
The next week is likely to see continued progress on several fronts. The `git replay --linearize` series is ready for integration and may land in `next`. The `--missing-only` option for `git rev-list` is queued for `next` and should graduate to `master` soon. The `receive-report` hook series is also ready for `next` and may see further review attention. The Outreachy application deadline (September 11) will focus efforts on finalizing project ideas and mentor commitments. The `git push` performance optimization from shallow clones may see further discussion about the default behavior change, while the ref storage terminology unification series could advance with additional review. Several bugfix series (`--force-if-includes`, `rerere` lock contention, `git repo info` path-related keys) are nearing completion and may be integrated.