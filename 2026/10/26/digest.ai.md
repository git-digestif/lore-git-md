Here is the daily digest for 2026/10/26:

# Git mailing list daily digest for 2026/10/26

## The day in brief
The Git mailing list saw **179 emails** today across **60 active threads**, with key developments in **platform compatibility fixes**, **CI infrastructure improvements**, **performance optimizations**, and **architectural debates** about object storage and fetch modes. The most consequential new developments include:

- **Torsten Bögershausen’s RFC optimization** for the macOS CRLF conversion bug, proposing a cleaner consolidation of the conversion check in `ie_match_stat()`.
- **Patrick Steinhardt’s confirmation** that the redundant `brew link --force gettext` CI patch is now safe for integration after verifying it works in both GitHub Actions and GitLab CI.
- **Junio C Hamano’s design endorsement** of a new `remote.<name>.fetchRefs` config key to resolve the fetch-mode architectural debate, blocking the current `remote.<name>.refmap` series from graduating to `master`.
- **Nine topics graduating to `master`**, including fixes for the sequencer, configuration, documentation, fast-import, CI, ODB, pack-check, repository setup, and blame subsystems.

## Notable threads

### macOS CRLF conversion size-trusting bugfix (read-cache.c)
**What changed?**
Torsten Bögershausen proposed an RFC optimization patch that consolidates the `would_convert_to_git()` check in `ie_match_stat()`, eliminating the double invocation present in Curtis Allen Smith’s v5 patch. The patch reduces the code footprint (8 insertions, 11 deletions) and avoids interfering with Git’s racy Git logic.

**Why it matters**
The optimization directly addresses the architectural debate about where to place the conversion check, offering a cleaner alternative to the v5 approach. While it does not resolve Johannes Sixt’s fundamental objection about size recording during `git add` or Junio C Hamano’s lstat() optimization concern, it provides a step toward code clarity and performance refinement.

**Key technical details**
- **Files touched**: `read-cache.c`
- **New behavior**: Consolidates the conversion check in `ie_match_stat()`, eliminating redundant attribute lookups.
- **Performance**: Reduces the double `would_convert_to_git()` call but does not address the lstat() optimization concern.
- **Status**: RFC, awaiting further discussion on architectural trade-offs.

---

### CI: Remove redundant `brew link --force gettext` (macOS)
**What changed?**
Patrick Steinhardt confirmed the patch is safe after verifying it works in both GitHub Actions and GitLab CI pipelines. The patch removes a redundant `brew link --force gettext` command from the macOS CI dependency script, eliminating spurious "Already linked" warnings.

**Why it matters**
The patch is a low-risk, platform-specific cleanup that reduces log noise in CI pipelines. It has no impact on local development or other platforms, and all prior feedback has been addressed.

**Key technical details**
- **Files touched**: `ci/install-dependencies.sh` (one line deleted)
- **Subsystem**: GitHub Actions and GitLab CI (macOS runner only)
- **Status**: Ready for integration, likely to proceed to `next`.

---

### `git repack`: Fix concurrent-push race leading to data loss
**What changed?**
Qin ShiCheng posted **v4 of the series**, which is now production-ready and rebased onto `master` (post-`ps/odb-files-alternates` merge). The v4 redesign simplifies the approach by introducing `--ignore-pack-keep-from` to explicitly tell `pack-objects` which packs it may delete, eliminating the need for `--honor-pack-keep`.

**Why it matters**
The series fixes a critical race condition where concurrent pushes could cause objects to be deleted despite being referenced by a `.keep` file that appears mid-repack. The v4 redesign addresses all prior review feedback and is now unblocked.

**Key technical details**
- **Files touched**: `builtin/pack-objects.c`, `builtin/repack.c`, `packfile.c`, `repack.c`, and related documentation and tests.
- **New plumbing**: `--ignore-pack-keep-from=<file>` (reads pack names from a file and treats them as if they had no `.keep` file).
- **Behavior changes**: The cruft walk now correctly traverses alternates and `--keep-pack` packs, trading disk space for safety.
- **Status**: Ready for `next`; Taylor Blau has provided a Reviewed-by for patch 3/3.

---

### `remote.<name>.refmap`: Fetch-mode architectural debate
**What changed?**
Junio C Hamano **endorsed the design direction** for v12 of the series, proposing a new `remote.<name>.fetchRefs` config key to decouple "what to fetch" from "where to store it." The current v11 series, which overloads `remote.<name>.refmap`, is now blocked from graduating to `master`.

**Why it matters**
The architectural debate has reached a critical juncture. The maintainer’s endorsement of `remote.<name>.fetchRefs` resolves the core design flaw but requires a substantive redesign of the series. The current v11 implementation remains in `next` but is unlikely to graduate until v12 is implemented.

**Key technical details**
- **Proposed config key**: `remote.<name>.fetchRefs` (e.g., `*@{upstream}` to fetch only upstream branches of local branches on this remote).
- **Precedence rule**: `remote.<name>.fetch` > `remote.<name>.fetchRefs` > dynamic branch enumeration.
- **Forced updates (`+`)**: Belong on the refmap side (the "where"), not the fetchRefs side (the "what").
- **Status**: Blocked pending v12 implementation.

---

### `gitmergeconflicts(7)`: New documentation guide
**What changed?**
Julia Evans posted **v5 of the series**, incorporating Junio’s final editorial tweaks (indentation and typo fixes) and retaining the original wording of the `diff3` example’s resolution rationale. The series is now cooking in `next` and ready for graduation to `master`.

**Why it matters**
The series consolidates Git’s scattered merge-conflict resolution guidance into a single, example-driven guide, improving clarity and reducing duplication across man pages. The new guide actively promotes `merge.conflictstyle=diff3` as a best practice.

**Key technical details**
- **Files touched**: `Documentation/gitmergeconflicts.adoc` (new), updates to `git-merge.adoc`, `git-rebase.adoc`, `git-revert.adoc`, `git-cherry-pick.adoc`, and `git-pull.adoc`.
- **New symbols**: `gitmergeconflicts(7)` man page.
- **Behavior changes**: Replaces duplicated conflict-resolution sections with cross-references to the new guide.
- **Status**: Merged to `next`; ready for `master`.

---

### `git branch --delete-integrated`: Detect squash-merged and rebase-merged branches
**What changed?**
Harald Nordgren posted **v4 of the series**, introducing `--delete-integrated` to unify rebase and squash detection under a single, opt-in interface. The series now handles upstream modifications via a path-by-path three-way merge approximation.

**Why it matters**
The series extends `git branch --delete-merged` to recognize branches whose changes (but not their tips) have been integrated via squash or rebase merges, addressing a common pain point in repository maintenance. The v4 redesign splits the strict and change-based behaviors into separate flags, addressing safety concerns.

**Key technical details**
- **New CLI option**: `--delete-integrated <pattern>` (opt-in, enables both rebase and squash detection by default).
- **Performance**: Rebase detection remains tree-based (no blob fetches); squash detection may download blobs in partial clones when upstream modifications are present.
- **Safety**: The series includes Junio’s false-positive test case to ensure `--delete-merged` never deletes branches whose changes were independently implemented by another author.
- **Status**: Under review; the author-check mitigation (Jasper Mayone’s proposal) is not yet implemented.

---

### Object storage extension (`extensions.objectStorage`)
**What changed?**
Karthik Nayak **unconditionally approved the entire v3 series**, signaling readiness for integration. The series introduces `extensions.objectStorage` as a pluggable object database backend mechanism, modeled after the ref storage extension.

**Why it matters**
The series is a major step toward Git’s libification, enabling alternative object storage backends (e.g., cloud storage, databases). The v3 refactoring splits the storage format into `(type, payload)` components, improving consistency and reducing URI re-parsing.

**Key technical details**
- **Files touched**: `odb/`, `setup.c`, `repository.c`, `builtin/{clone.c,init-db.c,submodule--helper.c,rev-parse.c,repo.c}`, and related documentation and tests.
- **New config key**: `extensions.objectStorage` (e.g., `files://objs` to relocate objects to `.git/objs`).
- **Behavior changes**: Repository format version bump to `1` for non-default configurations; submodule payload restriction (payloads are stripped).
- **Status**: Technically approved; awaiting Junio’s assessment of the unresolved architectural debate about user-configurable backends.

---

### Community voting process for contentious decisions
**What changed?**
The thread refined the eligibility criteria for the proposed community voting process. **Phillip Wood proposed a hybrid model**: require a minimum number of commits *over all time* (to ensure a track record) but also mandate *at least one recent commit* (to ensure ongoing engagement). **Jeff King provided empirical data** on the distribution of commit counts among contributors, illustrating the trade-off between inclusivity and practicality.

**Why it matters**
The voting process is intended to resolve contentious decisions (e.g., AI contribution policy) when consensus cannot be reached through normal discussion. The hybrid model addresses concerns about "drive-by" contributions while avoiding rigid time windows that might exclude engaged but less prolific contributors.

**Key technical details**
- **Proposed eligibility**: Minimum commits *over all time* + at least one recent commit.
- **Data**: Under a 50-commit threshold and a 2-year window, 42 contributors would qualify; under a 5-commit threshold and a 2-year window, 120 would qualify.
- **Status**: Under discussion; no consensus yet on the threshold or time window.

---

### Nine topics graduated to `master`
Junio C Hamano listed nine topics that graduated to `master`, covering a range of subsystems:

1. **ak/sequencer-head-refdup**: Fixes a bug in the sequencer where the HEAD reference name could be clobbered during worktree lookups.
2. **ic/includeif-hostname**: Introduces `includeIf:hostname:<pattern>` for Git configuration and adds `git var GIT_HOSTNAME`.
3. **je/doc-promote-git-help**: Updates the `git` man page to better guide users toward `git help`.
4. **mb/fast-import-reference-pending**: Fixes a crash in `git fast-import` when the `from` command references newly created commits or tags.
5. **ps/ci-housekeeping-and-modernizations**: Modernizes CI infrastructure by dropping Python 2 support, updating OS images, and improving reftable test coverage.
6. **ps/odb-files-alternates-cleanups**: Optimizes the ODB "files" backend by prioritizing packfile checks across alternates before loose objects.
7. **ps/pack-check-bigfile-threshold**: Corrects a boundary check in `git fsck` to avoid false corruption reports for packed objects whose sizes match `core.bigFileThreshold`.
8. **ps/setup-enforce-repo-passed-to-create-repository-has-no-state**: Refactors repository initialization to treat the repository object as a stateless out-parameter.
9. **rm/blame-default-ignore-revs**: Makes `git blame` and `git annotate` automatically use `.git-blame-ignore-revs` as the default ignore list.

---

## In brief
- **`git imap-send`**: Harald Nordgren’s series to handle non-canonical email header order and fix tunnel-command lifecycle issues is nearly complete, with only minor follow-up tweaks expected.
- **`git reflog`**: Junio C Hamano pushed back on Harald Nordgren’s proposal to refactor multi-step commands to work on detached HEAD by default, leaving the architectural debate unresolved.
- **`git worktree list --branch`**: Phillip Wood proposed a `--format='--branch=%(refname:lstrip=2)'` tweak to simplify the composability example and implementation.
- **`git repo structure --concern`**: Mark C. Chu-Carroll posted v2 of the series, addressing feedback on memory leaks, CI/CD test flakiness, and config key naming inconsistency.
- **`gitk`**: Ralf Habacker added a "Cherry-pick with cross-reference" context-menu item to invoke `git cherry-pick -x`.
- **`git archive-zip`**: kerylefondji7@gmail.com posted v7 of the patch to pluralize the "path too long" error message, addressing all prior feedback.
- **Partial clone documentation**: Edson documented the two-pass fetch order for promisor remotes and the security model for server-advertised remote acceptance.
- **Shared object store**: Massimiliano Cialdi proposed a design inquiry for a shared object store to reduce redundant storage in embedded workflows.
- **`git init`**: Matt Hunter’s patch to restore absolute path output in the "Initialized empty Git repository" message is ready for integration.
- **`git revert`/`git cherry-pick`**: Nakul V S’s patch to refuse to operate on shallow boundary commits by default is under review, with Phillip Wood proposing a shared helper function for the check.
- **`git check-ref-format`**: Fien Dora added regression tests for `--no-allow-onelevel` and `--allow-onelevel` option precedence.
- **`CodingGuidelines`**: Phillip Wood’s patch to officially permit compound literals in C code is ready for integration.
- **`git send-pack`**: Marc Strapetz’s series to optimize force-pushes of rebased feature branches is under review, with Junio C Hamano questioning its redundancy with the existing push negotiation mechanism.
- **Test modernization**: Kaartic Sivaraam’s series to make `clar`-based unit tests ignore the `-l`/`--long-tests` argument passed by `meson test` is ready for integration.
- **`git notes-cache`**: Okhuomon Ajayi’s patch to eliminate implicit use of `the_repository` in `notes_cache_match_validity()` is ready for integration.
- **`git combine-diff`**: Okhuomon Ajayi’s patch to replace direct uses of `the_repository` in `combine-diff.c` is ready for integration.
- **Glossary terminology**: Junio C Hamano rejected Julia Evans’s patch to make "object ID" the canonical term for the hash, insisting "object name" must remain the canonical term.
- **`git worktree list`**: Jasper Mayone’s `--branch=<branch>` filtering patch is under review, with Phillip Wood proposing a `--format` tweak to simplify the implementation.
- **Unit test CI compatibility**: Kaartic Sivaraam’s patch to make `clar`-based unit tests ignore the `-l`/`--long-tests` argument is ready for integration.
- **Git for Windows 2.56.0(4)**: Johannes Schindelin announced a security-focused bugfix release addressing corrupt code-signatures in 32-bit MinGit executables and a vulnerability in the included brotli component (CVE-2026-111497).