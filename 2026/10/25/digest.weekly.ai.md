# The Git Project -- Weekly Digest for 2026/10/19 -- 2026/10/25

## The period in brief
This week (2026/10/19--2026/10/25) was **exceptionally active**, with 22 distinct topics appearing across the daily digests. Traffic was **heavy and eventful**, spanning performance optimizations, architectural debates, governance discussions, and usability improvements. The three developments a reader absolutely should not miss are: (1) the **incremental connectivity check** series landing in `next`, offering dramatic speedups for small pushes to large repositories; (2) the **SHA-1/SHA-256 interoperability** effort advancing with pack index v3 support; and (3) the **community voting process** RFC, which could reshape Git’s governance model for contentious decisions.

---

## Key developments

### Incremental connectivity check lands in `next`
**What happened**: Kristofer Karlsson’s six-patch series introducing an opt-in incremental connectivity check for fetches and pushes was merged to `next`. The feature reduces the cost of verifying object connectivity by shifting it from repository size to incoming change size, offering up to **190× speedup** for small pushes to large repositories (14 MB vs. 3.4 GB memory usage) with a modest regression for very long incoming histories (1.5× slower), mitigated by an auto mode fallback.

**Key participants**: Kristofer Karlsson (author), Junio C Hamano (maintainer), Patrick Steinhardt (reviewer).

**Current status**: Merged to `next` as of 2026/10/23. The series introduces new config keys (`transfer.connectivityCheck=full|incremental|auto[=<n>]`, `transfer.connectivityCheckIncrementalMaxCommits=<n>`), a new `rev-list` flag (`--connectivity-check=`), and a new source file (`tree-verify.c`) implementing an inductive trust algorithm. The auto mode (patch 6/6) dynamically switches between incremental and full checks at a configurable 1,000-commit cutoff.

---

### SHA-1/SHA-256 interoperability: pack index v3 support
**What happened**: Christian Couder’s 13-patch v2 series implementing pack index v3 was merged to `next`. The new index format stores object IDs in both primary and compatibility hash algorithms, enabling bidirectional mapping between SHA-1 and SHA-256 repositories. This is a critical step toward full interoperability, allowing packed objects to be mapped between algorithms just as loose objects already can.

**Key participants**: Christian Couder (author), Junio C Hamano (maintainer).

**Current status**: Merged to `next` as of 2026/10/23. The series includes strict validation logic to prevent corruption from malformed indices and touches `pack-write.c`, `packfile.c`, `odb/`, and related headers. The v3 index format includes a variable-length header, per-algorithm tables (short-name, full-name, order-map, CRC, offset), and is designed to be extensible for future hash algorithms.

---

### Community voting process RFC
**What happened**: Emily Shaffer posted a patch defining a formal voting process to resolve contentious decisions when consensus cannot be reached, motivated by the stalled AI contribution policy debate. The proposal sparked heated discussion about eligibility criteria, secret ballots, and the role of corporate contributors. Junio C Hamano endorsed the need for secret ballots, while Kristoffer Haugsbakk and others raised concerns about corporate influence and low-effort participation.

**Key participants**: Emily Shaffer (author), Junio C Hamano (maintainer), Kristoffer Haugsbakk, Patrick Steinhardt, brian m. carlson, Phillip Wood.

**Current status**: The patch is under active discussion, with no clear consensus yet. Key open questions include:
- **Eligibility criteria**: Should voters have at least 50 merged patches (Peff’s original suggestion) or a lower threshold (e.g., five commits)?
- **Secret ballots**: How to implement them fairly while preventing abuse.
- **Activity window**: Whether to require a two-year activity window to prevent low-effort participation.
The outcome could set a precedent for Git’s governance model, particularly for divisive issues like AI contributions.

---

### `git log --follow` performance regression fix
**What happened**: Miklos Vajna posted a four-patch series fixing a severe performance regression in `git log --follow` when tracking renames across merges. The regression, introduced in June 2026, made `--follow` unusably slow (6m43s vs. 28s on the Linux kernel repository). The fix splits diffing into two passes: a fast pass without rename detection to identify unchanged parents, and a fallback to expensive rename detection only when necessary.

**Key participants**: Miklos Vajna (author), Junio C Hamano (maintainer).

**Current status**: The series is ready for integration, with comprehensive test coverage and benchmarks showing a speedup to 36s. The implementation avoids the expensive `propagate_follow_pathspec_to_parent()` function, which diffed each commit against *all* of its parents with rename detection enabled, and instead reuses the diffs already being computed in `log_tree_diff()`’s parent loop.

---

### Worktree improvements: `--reflink` and ephemeral branches
**What happened**: Jason Newton’s v3 series introduced copy-on-write worktree creation (`--reflink`) and automatic cleanup of auto-created branches when worktrees are removed (ephemeral branch reaping). The reflink feature avoids full checkout overhead on supported filesystems, while ephemeral branches address a long-standing maintenance nuisance.

**Key participants**: Jason Newton (author), Eric Sunshine, Phillip Wood, brian m. carlson.

**Current status**: The series is technically complete but faces **substantive objections** on both features. Eric Sunshine and Phillip Wood argue the reflink design should default to automatic for tracked files, while the ephemeral branch feature is redundant with `--detach`. Windows ReFS reflink is **permanently disabled** due to silent corruption risks (Git LFS #6312). The interpersonal conflict in the thread has escalated, with Jason dismissing objections as "nitpicking" and Phillip condemning his confrontational tone.

---

### `git branch --delete-merged` enhancements
**What happened**: Harald Nordgren’s v3 series extended `git branch --delete-merged` to detect branches whose changes (but not their tips) have been integrated via squash or rebase merges. The series aims to provide a practical one-step maintenance command for common forge and local workflows.

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), Jasper Mayone.

**Current status**: The series is feature-complete but blocked on a CLI design debate. Junio proposed decoupling strict `--delete-merged` (tip-reachability only) from a new `--delete-irrelevant` (change-based detection), while Harald rejects the split. Jasper Mayone’s testing confirms persistent false positives, reinforcing Junio’s safety concerns. The first patch (refactoring) is uncontroversial and ready for integration.

---

### SSH signing verification fixes
**What happened**: Christian Noé Ramos López posted a v2 security bugfix for Git’s SSH signing verification, addressing two critical flaws: (1) a misleading hint message that included full ref names instead of short names, and (2) a revocation file that failed open when missing or unreadable. The patch ensures signature information is displayed consistently and makes revocation file errors fatal by default, with an escape hatch via Git’s `:(optional)` pathname syntax.

**Key participants**: Christian Noé Ramos López (author), Junio C Hamano (maintainer), Patrick Steinhardt (reviewer).

**Current status**: The patch is ready for integration, with Junio requesting a procedural correction for future rerolls. The fix addresses a security-sensitive code path with no prior test coverage.

---

### `git status` defaults for triangular workflows
**What happened**: Harald Nordgren posted v3 of a patch to make `git status` compare against both `@{upstream}` and `@{push}` by default, targeting users of triangular workflows (e.g., GitHub PRs). The debate centers on whether `@{push}` divergence is inherently less important to monitor by default than `@{upstream}` divergence.

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), D. Ben Knoble.

**Current status**: The patch is under review, with Junio arguing that `@{push}` divergence reflects the user’s own work and is less critical to monitor by default. D. Ben Knoble countered with a user-centric argument, describing workflows where monitoring `@{push}` divergence is valuable (e.g., force-push decisions). The discussion remains unresolved.

---

### `gitmergeconflicts(7)` documentation series
**What happened**: Junio C Hamano queued a six-patch series introducing `gitmergeconflicts(7)`, a new man page consolidating scattered merge-conflict advice across multiple existing man pages. The guide covers conflict markers, "ours/theirs" terminology, `diff3` vs. default formats, resolution tools, and an exhaustive list of conflict-causing commands.

**Key participants**: Junio C Hamano (maintainer), Phillip Wood (reviewer).

**Current status**: Merged to `next` as of 2026/10/23. The series replaces duplicated conflict-resolution sections in existing man pages with cross-references to the new guide, improving clarity and reducing maintenance burden.

---

### `remote.<name>.refmap` series
**What happened**: Harald Nordgren’s v10 series introduced a "matching-inspired" fetch mode for shallow repositories, dynamically fetching only branches tracked by local branches at the remote. The series includes a new configuration mechanism (`remote.<name>.refmap`) and a `--limited-fetch` flag for `git remote add`, with shallow repositories defaulting to this mode.

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), D. Ben Knoble.

**Current status**: The series is cooking in `next`, with a bugfix addressing a fallback logic issue in `remote_find_tracking()`. The fallback ensures compatibility with `@{upstream}`, `--delete-merged`, and `git branch --track` operations. Harald committed to adding tests to verify refmap-tracked refs and negative refspec exclusion.

---

## In brief

**CRLF conversion size-trusting bug** -- Junio C Hamano and Torsten Bögershausen debated architectural trade-offs in the macOS CRLF conversion series, with Torsten proposing to relocate the `would_convert_to_git()` check to `ie_match_stat()` to avoid racy Git issues. The performance overhead of the double call remains unresolved.

**Worktree reflink** -- brian m. carlson raised critical correctness concerns about Windows ReFS silent corruption in the worktree reflink series, advocating disabling Windows reflink until Microsoft fixes the bug (Git LFS #6312). The macOS and Linux implementations remain unaffected.

**`the_repository` removal** -- Junio C Hamano rejected a reviewer’s suggestion to rename a parameter from `r` to `repo` in a refactoring patch, prioritizing idiomatic conventions over marginal descriptiveness. The exchange highlights broader questions about codebase consistency in the ongoing effort.

**Shallow fetch performance regression** -- Harald Nordgren posted a revised patch (v10) fixing a protocol-layer API misuse in the shallow fetch performance regression series, making it ready for final review. The fix aligns protocol v2 behavior with v0 by directly advertising shallow grafts.

**`git init` absolute path output** -- Matt Hunter posted a bugfix to restore the absolute path output in the "Initialized empty Git repository" message, addressing a regression introduced in Git 2.56. The patch is uncontroversial but sparked debate about adding a regression test.

**Structured output for `git bisect`** -- Theodore Ts'o proposed adding a `--format=json` option to `git bisect` to provide structured, machine-readable output, motivated by downstream tools (e.g., `ktest.pl`) that parse Git’s human-readable output. The discussion remains informational, with no patch yet.

**Hook configuration** -- Adrian Ratiu confirmed the current behavior of `hook.<event>.enabled` was intentional, leaving the decision to accept the patch or update the documentation unresolved.

**Alternate ODB reload** -- Junio C Hamano suggested deferring the `stat_validity` update in `read_alternates()` until after a successful read, addressing a race condition in the alternate object database reload series.

**`git stash` refactoring** -- Phillip Wood’s seven-patch series optimizing redundant index operations in `git stash` was queued for `next`, with Junio praising the most interesting step.

**`git imap-send` OpenSSL compatibility** -- Patrick Steinhardt raised critical concerns about the OpenSSL 4.0 compatibility macros in Beat Bolli’s v3 series, including a security regression (certificate stripping) and a behavioral regression (redundant `verify_peer()` call). The series remains blocked.

**ZIP archive date-handling** -- René Scharfe’s patch to omit the Extended Timestamp (UT) extra field for out-of-range commit times is ready for integration after a test infrastructure fix.

**`git stash` shell completion** -- D. Ben Knoble’s series to improve `git stash` completion faces design questions about Zsh-specific fixes and the removal of deprecated `stash save` completion. Junio endorsed the second patch as a clear improvement.

**Index lock timeout** -- Jun Wu’s configurable timeout for index locks (`core.indexLockTimeout`) sparked debate about configurability vs. root-cause fixes. Mihai Cosma proposed an alternative approach deferring lock acquisition until write time, fixing a silent data-loss bug in the `index.skipHash` code path.

**Connectivity check for deepening fetches** -- Igor Wiedler’s fix for a performance regression in shallow fetches over protocol v2 is ready for integration after addressing a Windows CI failure.

**Reftable backend buffer overflow** -- Matthew Suozzo’s patch to fix a potential buffer overflow in the reftable backend’s object record decoder faces design questions about the allocation helper.

**Loose ref cache inconsistency** -- Andy Koppe’s patch to fix an inconsistency in how root refs are handled in the loose ref cache is ready for integration, pending resolution of an architectural debate.

**ODB race condition** -- Justin Tobler’s series to fix a race condition in `.keep` lockfile cleanup received final approval and is ready for integration.

**GitLab CI whitespace jobs** -- Patrick Steinhardt’s patch to fix shallow clone failures in GitLab CI jobs sparked a debate about the jobs’ long-term value.

**GitHub Actions macOS runners** -- Junio C Hamano posted v2 of a CI update to split macOS jobs between `macos-15` and `macos-26`, following Patrick Steinhardt’s suggestion.

**`upload-pack` performance regression** -- Harald Nordgren’s patch to fix a performance regression in shallow fetches over protocol v2 needs revision to address a protocol-compliance issue.

**`git-rebase` documentation** -- Omri Sarig’s v4 patch to correct outdated and missing option links in the `git-rebase` man page is ready for merging.

**`git bundle` memory safety** -- Olof Fairbrook’s patch to fix a memory safety issue in `git bundle create` was approved and queued for integration.

**`the_repository` removal** -- Shashank Srikumar’s patch to eliminate implicit use of `the_repository` in `prune_packed_objects()` received final clarification from Junio C Hamano.

**Rust integration** -- Andy Koppe’s patch to fix a Makefile bug for custom Cargo target directories sparked a debate about implementation mechanism.

**Remote configuration** -- Colin Hinton’s patch to defer validation of `remote.<name>.followRemoteHEAD` faces design questions about error message clarity and syntax unification.

**ZIP archive pluralization** -- Keryle Fondji’s patch to use `Q_()` for plural-form error messages in the ZIP backend received feedback on commit message clarity.

**`git fsck` pluralization** -- Aniket Gupta’s patch to use `Q_()` for the verbose "Checking connectivity" message was approved and queued for integration.

**`git status` triangular workflows** -- Harald Nordgren’s patch to make `git status` compare against both `@{upstream}` and `@{push}` by default faces questions about test coverage and justification.

**Skipping negotiator correctness** -- Taylor Blau posted a patch to fix a correctness issue in the skipping negotiator’s queue management.

**`MyFirstContribution` tutorial** -- Junio C Hamano posted a patch to reorder the tutorial to emphasize review discussion before sending v2.

**Untracked cache performance** -- Tamir Duberstein’s v3 series teaching `git ls-files` to reuse and update the untracked cache populated by `git status` is under active review, with no objections recorded yet.

**Test modernization** -- Multiple patches replaced low-level shell test constructs with semantic helpers (e.g., `test -e` → `test_path_is_file`), part of an ongoing community-wide effort.

**i18n improvements** -- Outreachy microprojects converted hardcoded English plural forms to `Q_()` for proper translation, with several patches nearing completion.

**Governance** -- XuRua’s RFC proposed Git-level throttling to curb "comma programmer" workflows and reduce cloud compute waste, sparking debate about upstream policy.

**Documentation** -- Updates to `MyFirstContribution.adoc` emphasized review discussion before sending v2 patches.

**CI/build system** -- Taylor Blau proposed pinning GitHub Actions to full commit SHAs for security, sparking a debate about upstream policy vs. downstream constraints.

**`gitbreaking-changes(7)` manpage** -- Junio C Hamano signaled tentative approval for the series, contingent on the post-transition awkwardness being manageable.

**`git stash` performance** -- Phillip Wood and Junio C Hamano finalized stylistic details for the `pw/stash-optimize-check_changes-calls` series, now queued for `next`.

**Community Summit notes** -- The discussion around the voting process was directly tied to the unresolved AI contribution policy debate.

**`git remote set-password`** -- brian m. carlson rejected the proposal outright, citing project policy against encouraging URL-embedded credentials.

**Terminology cleanup** -- Andy Koppe’s five-patch series aligned Git’s documentation, code comments, and tests with the 2024 terminology change narrowing "pseudoref" to FETCH_HEAD and MERGE_HEAD only.

**ODB transaction race condition** -- Patrick Steinhardt approved Justin Tobler’s v4 bugfix series targeting a race condition in Git’s ODB transaction machinery.

**`git log --follow` test coverage** -- Miklos Vajna posted the first patch in a series adding test coverage for `--follow` behavior across merge commits.

**Maintainer capacity** -- Junio C Hamano’s "What's cooking" report highlighted a backlog of ~100 topics, with only ~30 in or near `next`, raising concerns about capacity.

**`git worktree list --branch`** -- Jasper Mayone implemented `--branch=<branch>` filtering for `git worktree list`, following Junio C Hamano’s suggestion. Phillip Wood proposed a refactoring to reduce code duplication.

**`test-lint` performance** -- Tyler Cipriani posted a four-patch series addressing the slowdown in `test-lint` caused by the `test-grep-lint` check, reducing runtime from ~25 seconds to ~1 second for five consecutive runs.

**`diff-delta.c` safe integer helpers** -- Srujith A. Poojary’s patch replacing manual integer arithmetic with safe helpers in `diff-delta.c` is under review, with Junio demanding a clearer explanation of the theoretical overflow risk.

**Reflog pluralization fix** -- Aniket Gupta’s v2 patch fixing a pluralization error in reflog error messages is ready to land.

**`git-rebase` man page update** -- Omri Sarig’s v4 patch updating the `git-rebase` man page to correct outdated references is ready for merging.

**`the_repository` removal** -- Dorine Tipo’s patch mechanically refactoring `partial_clone_register()` and `partial_clone_get_default_filter_spec()` to accept a `struct repository *` parameter is part of the broader effort to eliminate global state.

**`git reflog expire` dry-run output** -- Laveesh Bansal’s bugfix for a UI inconsistency in `git reflog expire --dry-run --verbose` is ready for integration.

**Completion regression fix** -- Tanga Joris’s patch removing the `NO_PARSEOPT` flag from four built-in commands (`patch-id`, `diff-pairs`, `fast-import`, `var`) fixes a completion regression.

**`gitk` cherry-pick context menu** -- Johannes Sixt reviewed a patch adding a "Cherry-pick with -x" context-menu action to `gitk`, questioning the necessity of the feature and suggesting UI refinements.

**Advice scope hint** -- Junio C Hamano queued the `jc/advice-config-set-global` topic (v5), which adds a scope hint to Git’s advice settings, ensuring the command to disable an advice message includes the correct config scope (`--global`).

---

## Looking ahead
The next week is likely to see continued discussion and potential resolution on several fronts:

1. **Community voting process**: The eligibility criteria and secret ballot implementation will likely dominate governance discussions, with a possible vote on the AI contribution policy looming.
2. **Worktree improvements**: The `--reflink` and ephemeral branch series may see a reroll addressing the substantive objections, though the interpersonal conflict could delay progress.
3. **`git status` defaults**: The triangular workflows patch may see a v4 addressing Junio’s latest critique, or the debate could stall pending broader community input.
4. **SHA-1/SHA-256 interoperability**: The pack index v3 series is now in `next`, but follow-up work on bidirectional mapping and interoperability testing is likely.
5. **Performance optimizations**: The incremental connectivity check and shallow fetch regression fixes are now in `next`, but follow-up work on edge cases (e.g., stack overflow risk in tree verification) may emerge.