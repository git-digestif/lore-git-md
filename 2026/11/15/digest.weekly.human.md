# The Git Project -- Weekly Digest for 2026/11/09 -- 2026/11/15

## The period in brief

This week (2026/11/09--2026/11/15) saw **1,247 emails** across six active days, with traffic heavier than average and unusually eventful. The `git repo info` path keys series reached final approval after months of design debate, while a critical performance regression in `git log --follow` was resolved with an opt-in toggle. The community also grappled with fundamental questions about commit identity preservation (`change-id`) and security boundaries. A coordinated security release (Git 2.56.1) addressed six CVEs, and several long-running refactoring efforts—`the_repository` removal, `rerere` modernization, and test suite splitting—made significant progress.

**Do not miss**: the `git repo info` path keys series (now ready for merging), the `git log --follow` performance fix (unblocking GitLab’s deployment), and the `change-id` preservation debate (a rare discussion touching both workflow needs and security boundaries).

---

## Key developments

### `git repo info` path keys series lands

The `[PATCH v9]` series adding four new path-related keys to `git repo info` (`path.working_tree`, `path.superproject_working_tree`, `path.index`, and `path.prefix`) received final approval and is ready for merging. The series resolves a long-standing need for scriptable access to repository component locations, replacing ad-hoc use of `git rev-parse --git-path` with a consistent, user-facing API.

**What happened**: After nine iterations and months of debate, the series converged on naming conventions (underscores `_` instead of hyphens `-`), dropped contested keys (`path.hooks`, `path.cdup`), and optimized tests to avoid redundant repository initialization. Phillip Wood explicitly approved the v9 series, calling it "mechanically correct" and ready for integration. The keys support suffix-based formatting (e.g., `path.working_tree.absolute` vs. `path.working_tree.relative`) and return empty strings in bare repositories, aligning with `git rev-parse` precedent.

**Key participants**: K Jayatheerth (author), Phillip Wood (reviewer), Junio C Hamano (maintainer).

**Status**: Ready for `next`; all design objections resolved.

---

### `git log --follow` performance regression resolved

A June 2026 commit that made `git log --follow` ~14× slower on non-linear history was addressed with an opt-in toggle (`--follow=merges` or `log.follow=merges`), restoring pre-regression performance while preserving correctness improvements for users who need them. The toggle unblocks GitLab’s deployment, which had been stalled by the performance impact.

**What happened**: The original commit enabled rename detection across merges to fix lost history in `-m --follow`, but the performance cost was prohibitive for large repositories. Miklos Vajna introduced a patch that makes the expensive behavior opt-in, with Junio C Hamano endorsing it as the project-level resolution. The toggle preserves the correctness fix while addressing the operational impact.

**Key participants**: Miklos Vajna (author), Junio C Hamano (maintainer), Patrick Steinhardt (GitLab).

**Status**: Fast-tracked for integration; the v5 correctness fixes remain ready for `next`.

---

### `change-id` preservation: workflow needs vs. security boundaries

A heated debate about preserving `change-id` headers during commit rewriting operations (e.g., `rebase`, `amend`) revealed a fundamental tension between workflow needs and security requirements. The discussion centered on whether `change-id` can serve as a stable commit identifier given that users can manually edit the header.

**What happened**: Ewout ter Hoeven argued that preserving `change-id` would eliminate workarounds in tools like `gh-stack` and `glab stack`, which currently rely on heuristics (branch names, patch IDs) that fail in common scenarios. brian m. carlson countered that `change-id` cannot be trusted as a stable identifier because users can edit it, creating a security risk for automated review systems. The thread reframed the problem as a **security boundary question**: any solution must either restrict `change-id` preservation to server-controlled operations or accept that the header cannot be trusted for security-sensitive decisions.

**Key participants**: Ewout ter Hoeven (workflow advocate), brian m. carlson (security advocate), Junio C Hamano (maintainer).

**Status**: No consensus; the discussion remains open.

---

### Security release: Git 2.56.1 addresses six CVEs

A coordinated security release (Git 2.56.1) addressed six CVEs, including remote code execution via malicious submodule gitfiles (CVE-2026-111760), path traversal in the files ref backend (CVE-2026-113408), and stack corruption from multi-signature commits (CVE-2026-114467). The fixes were backported to all supported maintenance tracks (down to `maint-2.43`).

**What happened**: The vulnerabilities spanned critical areas (submodules, ref backends, merge machinery) with potential for denial-of-service or arbitrary code execution. Toon Claes (protocol v2 symref validation), Karthik Nayak (multi-signature handling), and Jeff King (30 patches for const-correctness and HTTP header redaction) were key contributors. Git for Windows (Johannes Schindelin) released v2.56.1 with identical fixes.

**Key participants**: Junio C Hamano (maintainer), Toon Claes, Karthik Nayak, Jeff King.

**Status**: Released; fixes integrated into all maintenance tracks.

---

### `git rerere` subcommand handling modernized

Patrick Steinhardt’s 12-patch series modernizing `git rerere`’s subcommand handling was merged to `master`. The series replaces the custom subcommand-parsing logic with Git’s standard `OPT_SUBCOMMAND()` mechanism, aligning it with other multi-subcommand built-ins like `git notes`.

**What happened**: The refactoring is part of the broader effort to remove global state (`the_repository`) and improve code maintainability. The series also added a `BUG()` check to catch invalid flag combinations and per-subcommand usage strings. Karthik Nayak provided final review approval, confirming the series is ready for `next`.

**Key participants**: Patrick Steinhardt (author), Karthik Nayak (reviewer).

**Status**: Merged to `master`.

---

### `git branch --delete-integrated` series reaches v7

Harald Nordgren posted v7 of the series teaching `git branch --delete-integrated` to detect branches whose changes were squash-merged or rebase-merged into an upstream branch, even when the upstream later modified the same files. The series is now technically complete, with all substantive review issues addressed.

**What happened**: The fourth patch now reads the index once before the walk in `find_landed_commits()`, eliminating redundant `repo_read_index()` calls, and adds test cases for edge-case robustness. The commit message explicitly documents the `.gitattributes` resolution behavior and the fact that `--delete-integrated` may download blobs in partial clones. The only remaining technical gap is the author-check mitigation (Jasper’s proposal), which is not yet implemented but does not block integration.

**Key participants**: Harald Nordgren (author), Phillip Wood (reviewer).

**Status**: Ready for final review; no known blockers.

---

### `git apply --check` crash fix revised

A two-part bugfix series for a crash in `git apply --check` when processing malformed patch headers was revised to reject corrupt patch headers outright, aligning with maintainer guidance to enforce strict parsing rules. The series now prevents the parser from proceeding with inconsistent state, resolving the root cause of the crash.

**What happened**: Tanga Joris posted the revised series, which rejects patches with blank lines or indented mode lines after `diff --git`, and patches where `---` appears before `new file mode` or `+++` appears before `deleted file mode`. Siddharth Asthana identified a memory leak in `parse_git_diff_header()` and proposed a preparatory patch to address it. The series is now technically sound, with clear error messages and comprehensive test coverage.

**Key participants**: Tanga Joris (author), Siddharth Asthana (reviewer), Junio C Hamano (maintainer).

**Status**: Ready for final review.

---

### Integer-overflow hardening advances

Johannes Schindelin advanced the integer-overflow hardening series with three key updates, addressing Phillip Wood’s review of the wrapper and date subsystems.

**What happened**: The wrapper subsystem fix now **changes the API contract** for `write_in_full()` to return `0` on success (eliminating overflow risk entirely), and a new `read_in_full_or_error()` helper is introduced for callers that only need success/failure status. The `signed_mult_overflows` and `mult_overflows` macros were dropped due to GCC-only limitations, and the date.c hardening implementation was replaced with a simpler, portable approach (capping user-supplied numbers at `INT_MAX` and guarding multiplications manually).

**Key participants**: Johannes Schindelin (author), Phillip Wood (reviewer).

**Status**: Under review; API changes and macro removals address prior concerns.

---

### `fetchRefmap` series advances to v14

Harald Nordgren posted v14 of the series introducing `remote.<name>.fetchRefmap` and a matching-inspired fetch mode for shallow repositories. The series is now down to five patches after squashing the `@{push}` resolution fix into Patch 3/5.

**What happened**: The cover letter explicitly ties the series to Matt Hunter’s clarification that Patches 1–3 are sufficient to resolve textual conflicts with Collin’s `ch/remote-defer-follow-remote-head-validation` topic, strengthening the case for a partial merge. The architectural objection to Patch 3/5 (dynamic branch enumeration) is weakened by this context, as the patch is required to resolve the conflicts.

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), Matt Hunter.

**Status**: Ready for final review; strategic discussion about partial merge ongoing.

---

## In brief

**`git repo info` path keys** -- The v9 series adding `path.working_tree`, `path.superproject_working_tree`, `path.index`, and `path.prefix` received final approval from Phillip Wood and is ready for merging.

**`git log --follow` performance fix** -- An opt-in toggle (`--follow=merges`) resolves a ~14× slowdown on non-linear history, unblocking GitLab’s deployment.

**`change-id` preservation debate** -- A fundamental tension between workflow needs (stable commit identity) and security boundaries (editable headers) remains unresolved.

**Security release (Git 2.56.1)** -- Six CVEs addressed, including remote code execution via malicious submodule gitfiles and path traversal in the files ref backend.

**`git rerere` modernization** -- Patrick Steinhardt’s 12-patch series modernizing subcommand handling was merged to `master`.

**`git branch --delete-integrated`** -- The v7 series is technically complete, with all substantive review issues addressed.

**`git apply --check` crash fix** -- The revised series rejects corrupt patch headers outright, resolving the root cause of the crash.

**Integer-overflow hardening** -- API changes and macro removals address prior concerns, advancing the hardening effort.

**`fetchRefmap` series** -- The v14 series is ready for final review, with strategic discussion about partial merge ongoing.

**`git credential` conversion to `OPT_SUBCOMMAND`** -- Wasswa Edson’s v2 series aligns `git credential` with Git’s established patterns for subcommand dispatch.

**`git ls-remote` promisor-remote crash fix** -- Christian Couder’s patch fixes a crash when invoked outside any Git repository with promisor-remote configuration enabled.

**Detached HEAD documentation** -- Julia Evans’s v6 series addresses final style nits and test expectations, creating a standalone guide for detached HEAD.

**`git worktree list` improvements** -- Jasper Mayone’s v5 series extends `%(worktreepath)` and improves worktree safety, using a hashmap for O(1) lookups.

**`git history --verbose` output format** -- The discussion shifted to output format, with Patrick Steinhardt requiring a structured format generalizable to all `git history` subcommands.

**ODB transaction temporary directory creation** -- Justin Tobler’s series makes temporary directory creation eager, addressing a correctness gap where some writes bypass the temporary directory.

**Test modernization for `git fast-import`** -- Karthik Nayak’s 20-patch series splits the monolithic `t9300-fast-import.sh` into smaller, self-contained files.

**Object format interoperability bugs** -- Adrian Ratiu’s 3-patch series fixes bugs in Git’s `compatObjectFormat` interoperability layer.

**Promisor-remote protocol v2 capability** -- Christian Couder’s 3-patch series fixes a bug where the Git client omits the `promisor-remote=...` reply during `fetch` protocol v2 commands.

**`git cat-file --batch` bugfix** -- Caspar Duregger’s series applies `GET_OID_GENTLY` to make error handling uniform for invalid reflog references.

**`git worktree` safety improvements** -- Phillip Wood identified a performance regression in `%(worktreepath)` and proposed architectural alternatives.

**`git diff` EXAMPLES rewrite** -- Julia Evans’s rewrite of the `git diff` man page’s EXAMPLES section is ready for queueing, pending resolution of the presentation format dispute.

**`git mktree --batch` test coverage** -- Roger Koranteng’s patch adds test coverage for `git mktree --batch`, addressing a memory leak.

**`git describe` regression fix** -- Oswald Buddenhagen added historical context to the `git describe` regression.

**APFS/mmap metadata quirk** -- Johannes Sixt broadened the scope of the APFS/mmap issue, noting that Git’s mmap emulation on Windows exhibits the same delayed timestamp update behavior.

**`git repack --drop-filtered`** -- kongfankaikai proposed that `git repack --drop-filtered` should skip blobs referenced by the index and continue processing.

**`git rerere` safety check** -- Patrick Steinhardt noted a latent safety issue in `enum rerere_flags`.

**Test fix for `t2401-worktree-prune.sh`** -- Anders Kaseorg’s fix for a brittle `sed` substitution was acked by Junio.

**ODB abstraction** -- Patrick Steinhardt and Kaartic Sivaraam reviewed a series moving the loose-object backend into its own source file.

**Config segfault fixes** -- Thomas Bachem’s 15-patch series fixing segfaults from unvalued config keys was queued for `next`.

**`test-tool genbytes`** -- Junio suggested rebasing Kristofer Karlsson’s series to include a newly identified callsite.

**`git archive` refactoring** -- Karthik Nayak flagged a commit-message discrepancy in Srujith A Poojary’s series.

**`git diff` shell function context** -- Johannes Sixt identified the issue as a configuration problem, not a bug.

**Solaris compatibility fixes** -- Two platform-specific fixes were posted for unquoted shell variables and `uname()` error-checking.

**`struct commit` memory optimization** -- Kristofer Karlsson’s patch to reduce `struct commit` size by 10% on 64-bit platforms was approved by Junio.

**GSoC microproject inquiry** -- A student inquired about the availability of a specific test suite fix in `t4150-am.sh`.

**`git repo structure` performance improvements** -- Patrick Steinhardt’s two-patch series achieved an 80% memory reduction and 20% wall-time improvement.

**`git fetch` refspec override semantics** -- Phillip Wood confirmed that command-line refspecs must override `remote.<name>.fetchRefmap` entirely.

**`promisor.fetchOrder` design discussion** -- Christian Couder argued that the current patch’s static client-side order cannot handle multi-remote scenarios.

**`--show-object-format` for `git ls-remote`** -- Patrick Steinhardt approved the v2 patch, signaling readiness for queueing.

**Git v3 default object format** -- brian m. carlson’s patch ensures `git init` correctly defaults to SHA-256 when Git is built with `WITH_BREAKING_CHANGES=1`.

**`git-config` documentation** -- YikunWei added the `fmt-merge-msg` configuration section to the main `git-config(1)` man page.

**`for-each-ref --no-format` bugfix** -- Junio leaned toward making `--no-format` consistently restore the default format across all ref-listing commands.

**`git repack --filter-to-loose`** -- Adithya Chakilam’s two-patch series introduces a new option to improve interoperability with key-value object stores.

**Loose object database cache invalidation** -- Junio confirmed integration of the fix for stale cache entries.

**Timeout variable naming** -- Okhuomon Ajayi addressed Junio’s objection by enumerating all existing lock-timeout variables and confirming they are uniformly measured in milliseconds.

**Archive refactoring** -- Srujith A Poojary posted v2 of the archive subsystem refactoring, focusing exclusively on the tar archiver.

**HTTP backend cleanup** -- Yusuke Hayashi clarified the performance rationale for removing pre-sizing logic in `get_info_packs()`.

**Documentation retraction** -- Yikun Wei withdrew the `fmt-merge-msg` documentation patch after confirming the variables are already transitively included.

**Test modernization** -- Hardik Kumar replaced `!` with `test_must_fail` in four test scripts to handle edge cases like crashes or signals.

**Pretty colors** -- Andy Koppe fixed lost and leaked ANSI color escape sequences in custom log formats.

---

## Looking ahead

The next week is likely to see progress on several fronts:

- **`fetchRefmap` series**: The v14 series is ready for final review, with strategic discussion about whether to merge Patches 1–3 now or insist on the full series.
- **`git branch --delete-integrated`**: The v7 series is technically complete and may see final review and integration.
- **`change-id` preservation**: The debate remains open, with no clear resolution in sight. The discussion may expand to include alternative approaches to commit identity preservation.
- **Integer-overflow hardening**: The series is advancing with API changes and macro removals, and may see further review and integration.
- **`git repo structure` performance improvements**: The two-patch series is ready for review and may see integration.
- **`git worktree` safety improvements**: The v5 series is ready for review, with performance issues addressed.