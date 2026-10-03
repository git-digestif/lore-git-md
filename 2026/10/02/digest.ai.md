# Git mailing list daily digest for 2026/10/02

## The day in brief

The Git mailing list saw significant activity around several key topics today. The most consequential developments include: Junio C Hamano reporting test failures in the `git history` signing series after merging it to `seen`; a new RFC proposing a SHA-256 tree digest signing mechanism as an alternative to switching Git's default object format; and Jeff King's proposal for a "limbo" object-format state to ease the transition to Git 3.0. Additionally, several long-running series received substantive reviews, including the MIDX reachability closure fixes and the alternates refactoring.

## Notable threads

### `git history` signing series breaks tests in `seen`

**What changed**: Junio C Hamano reported that the `git history` signing series, which teaches `git history` to sign rewritten commits, breaks `t9902-completion.sh` in the `seen` integration branch. The series also has an edge case in command-line option parsing for `git history split` when invoked with conflicting options.

**Problem/goal**: The series aims to extend commit signing to all `git history` subcommands (`drop`, `fixup`, `reword`, `split`, `squash`) using the same configuration and command-line options already supported by other Git commands.

**Subsystem**: `git history` command, replay API, GPG signing infrastructure

**Impact**: The test failure indicates a mechanical regression in shell completion that needs to be fixed before the series can graduate to `next`. The edge case in option parsing could lead to misleading completions.

**Today's news**: Junio reported two new issues that surfaced after merging the topic to `seen`:
- The series breaks `t9902-completion.sh`, likely because the new `--gpg-sign` and `--no-gpg-sign` options are not handled correctly by the completion script.
- The completion helper for `git history split` outputs all conflicting options when invoked with a sequence like `--gpg-sign --no-dry-run -- --no-gpg-sign`, which may not match the intended precedence rule (last option wins).

[2026/10/02/22-47-13 by Junio C Hamano]

### RFC: SHA-256 tree digest signing mechanism

**What changed**: Scott Chacon proposed a new signing mechanism that embeds SHA-256 digests of tree contents (including submodules) in signed commits and tags, without requiring the repository to use SHA-256 as its object format. The series adds a `--hash=sha256` option to `git commit -S` and `git tag -s`, and a `gpg.treeHash=sha256` config key to make it the default.

**Problem/goal**: Provide a migration path away from SHA-1 for commit/tag signing that avoids the ecosystem-wide disruption of switching Git's default object format to SHA-256 in Git 3.0.

**Subsystem**: Signing infrastructure, commit/tag creation, submodule handling

**Impact**: This proposal could offer similar security benefits to a full SHA-256 transition while avoiding the need for users to migrate their repositories. It's positioned as an alternative to the planned Git 3.0 object format switch.

**Today's news**: Brian M. Carlson firmly rejected the approach, arguing that it fails to address the core problem of SHA-1's collision resistance weaknesses and that the Git project is already committed to a full transition to SHA-256 as the default object format in Git 3.0. Carlson emphasized that security researchers and other users need SHA-256 repositories to store colliding blobs, and the tree digest approach does nothing to enable this. He also noted that the Git 3.0 plan has been public for years and is scheduled for March 2027, with all major forges already supporting SHA-256.

Junio C Hamano raised substantive design questions about the digest algorithm, specifically whether it should distinguish between file modes or symlinks pointing to the same content and whether it should reflect the internal structure of the tree object to detect corruption.

[2026/10/02/19-06-08 by brian m. carlson]
[2026/10/02/15-45-53 by Junio C Hamano]

### Proposal: "limbo" object-format state for empty repositories

**What changed**: Jeff King proposed a new "limbo" object-format state for empty repositories, allowing the server to dynamically adopt the format of the first incoming push. This would solve a usability pain point for first-time pushes to newly created repositories on forges.

**Problem/goal**: When a user pushes to a newly created bare repository on a forge (e.g., GitHub), the repository must already be configured for either SHA-1 or SHA-256. If the client and server disagree, the push fails with a fatal error, forcing users to manually reconfigure or recreate the remote repository.

**Subsystem**: Wire protocol, repository initialization

**Impact**: This proposal would make the Git 3.0 transition smoother by allowing empty repositories to defer the object format decision until the first push, similar to how GitHub already handles the default branch on first push.

**Today's news**: Jeff King proposed that empty repositories advertise a "limbo" state—no object format yet—allowing the server to dynamically adopt the format of the first incoming push. The idea is technically plausible but untested. Peff outlined the core concept: the server advertises `object-format=limbo`, the client responds with its own format, and the server atomically commits to that format before accepting the push. He acknowledged potential race conditions but saw no fundamental blockers.

[2026/10/02/22-44-00 by Jeff King]

### MIDX reachability closure fixes receive substantive review

**What changed**: Jeff King provided substantive reviews of Taylor Blau's eight-patch series fixing corner cases in Git's multi-pack-index (MIDX) and cruft-pack machinery where MIDXs can end up containing objects that are not closed under reachability.

**Problem/goal**: The series fixes four scenarios where MIDXs violate the invariant that they contain all objects reachable from their tips, causing failures in bitmap generation. The scenarios include `--stdin-packs=follow`, incremental repacks, `.keep` packs, and `--write-midx=incremental`.

**Subsystem**: MIDX, cruft packs, `git repack`, `git pack-objects`

**Impact**: These fixes ensure that MIDXs remain reachability-closed, preventing subtle corruption and enabling reliable bitmap generation. The series is a prerequisite for future ODB work.

**Today's news**: Jeff King provided substantive reviews of several patches in the series:
- He sided with Elijah Newren's argument that preserving pack-order correlation (root trees before subtrees) is valuable for delta compression quality, even if the solution is not perfect.
- He raised two technical concerns about patch 2/8: (1) the explanatory comment about deferring tree/tag roots is placed in the wrong function, and (2) the patch calls `prepare_revision_walk()` twice on the same `rev_info` struct, which may introduce subtle bugs.
- He suggested refactoring a double-negation in the conditional logic controlling whether `--keep-pack` arguments are passed to `git pack-objects` during geometric repacks.
- He questioned whether the proposed refactoring to track the preferred pack explicitly in `midx_compaction_step` is strictly necessary, though he endorsed it as a cleaner design.
- He expressed difficulty following the logic of patch 8/8 but deferred to the test coverage, noting that the key change appears correct.

The discussion now centers on the data structure choice for `extra_roots` in patch 2/8, with no hybrid approach proposed yet.

[2026/10/02/23-02-25 by Jeff King]
[2026/10/02/23-13-36 by Jeff King]
[2026/10/02/23-25-29 by Jeff King]
[2026/10/02/23-28-34 by Jeff King]
[2026/10/02/23-41-57 by Jeff King]

### Alternates refactoring series posted

**What changed**: Patrick Steinhardt posted a 13-patch series refactoring the Git object database (ODB) to move alternates handling into the "files" backend, eliminating a conceptual mismatch that caused performance regressions and blocked backend-agnostic extensions.

**Problem/goal**: Alternates were previously managed at the ODB source level, which caused performance regressions, complicated data structures (bitmaps, commit graphs), and blocked backend-agnostic extensions to `GIT_OBJECT_DIRECTORY` and `GIT_ALTERNATE_OBJECT_DIRECTORIES`.

**Subsystem**: ODB core, files backend, commit-graph subsystem

**Impact**: This refactoring simplifies the ODB architecture and enables future work such as pluggable backends. It's a prerequisite for backend-agnostic extensions to environment variables like `GIT_OBJECT_DIRECTORY`.

**Today's news**: Patrick Steinhardt posted the complete 13-patch series, which:
- Introduces `struct odb_files_dir` to manage multiple object directories.
- Refactors `odb_for_each_alternate()` and `odb_find_source()` to work with directories instead of sources.
- Absorbs quarantine logic into `tmp-objdir` and manages it as an object directory.
- Relocates alternates handling into the files backend, making it a backend-specific implementation detail.
- Removes the now-unused `read_alternates` callback from the ODB source interface.

The series touches 52 files, with significant changes to `odb/source-files.c` (482 lines added) and `odb.c` (439 lines removed). It has minor merge conflicts with `seen`, which are documented in the cover letter.

[2026/10/02/10-08-11 by Patrick Steinhardt]

## In brief

- **`git var` extension series**: Andrew Pleeter confirmed the series is technically complete and ready for maintainer integration, acknowledging Phillip Wood's surface-level feedback on test style. [2026/10/02/17-41-38 by Andrew Pleeter]
- **`gitmergeconflicts(7)` documentation**: Junio C Hamano provided detailed examples demonstrating why `merge.conflictstyle=diff3` is essential for correct conflict resolution, showing how the common ancestor reveals the intent behind conflicting changes. [2026/10/02/17-58-53 by Junio C Hamano]
- **`git stash create` extension**: Kazumasa Shigeta accepted Junio's critique that the commit message focused too much on *what* the patch does rather than *why* it exists, and committed to rewriting it to emphasize the inconsistency being fixed. [2026/10/02/09-04-26 by 重田一聖]
- **`git config --list --show-origin`**: Brian M. Carlson supported switching to an absolute, canonicalized path for `.git/config`, aligning with how other paths in the output are already handled. [2026/10/02/21-01-02 by brian m. carlson]
- **`git fetch` commit-graph optimization**: Kristofer Karlsson clarified that the seeds passed to `write_commit_graph()` are **additive**, extending the existing graph rather than replacing it, and provided benchmark numbers showing a ~20% improvement in a synthetic setup. [2026/10/02/12-40-44 by Kristofer Karlsson]
- **`git remote pushDefault` extension**: Harald Nordgren posted a two-patch series extending `remote.pushDefault` to accept a space-separated list of remote names, pushing to the first available remote in the list. [2026/10/02/07-17-50 by Harald Nordgren via GitGitGadget]
- **`git bisect` plumbing**: Jeff King suggested using `git show bisect/bad` to retrieve the commit identified as the culprit after a successful `git bisect` run, avoiding parsing porcelain output. [2026/10/02/22-11-54 by Jeff King]
- **`git config` pager documentation**: Todd Zullinger expanded the `core.pager` section to show more ways users can override the `LESS` environment variable settings that Git applies by default. [2026/10/02/23-41-52 by Todd Zullinger]
- **`git branch --delete-merged`**: Junio C Hamano questioned whether defaulting to `**` (matching all upstreams) is a good comparison to other branch-listing options that default to the current branch. [2026/10/02/17-11-23 by Junio C Hamano]
- **`git rerere` lock contention fixes**: Thomas Bachem posted the v6 series incorporating all feedback, including renaming `--auto` to `--skip-locked` for `git rerere gc` and adding test coverage for the merge recreated by `git rebase -r`. [2026/10/02/11-11-29 by Thomas Bachem via GitGitGadget]
- **`git-contacts` stdin feature**: Junio C Hamano queued the v6 update in `next` for integration testing, marking it ready to graduate to `master` after cooking. [2026/10/02/14-50-00 by Junio C Hamano]
- **`gitmergeconflicts(7)` documentation**: Julia Evans proposed a new "WHAT IS A MERGE CONFLICT?" section for the guide, adopting a lightweight, example-driven approach that avoids technical jargon. [2026/10/02/17-39-44 by Julia Evans]
- **`git stash pop` custom conflict-label options**: Harald Nordgren withdrew the patch after Junio C Hamano and Phillip Wood questioned the fundamental motivation for exposing these options in user-facing commands. [2026/10/02/07-21-26 by Harald Nordgren]
- **`git restore-mtimes` RFC**: Junio C Hamano proposed a concrete algorithm for computing historical mtimes and raised an edge case (file concatenation) that the original proposal did not address. [2026/10/02/01-28-09 by Junio C Hamano]
- **`git refs` subcommand grouping**: Patrick Steinhardt posted the v2 series incorporating Junio's feedback, including a preparatory refactoring of `usage_with_options_internal()` and extending `OPT_SUBCOMMAND_F()` to accept a help string. [2026/10/02/08-09-45 by Patrick Steinhardt]
- **`git repo info` path keys**: Junio C Hamano confirmed he will replace the previous iteration with K Jayatheerth's v2 series. [2026/10/02/17-03-51 by Junio C Hamano]
- **`git submodule` merge corruption**: Jeff King suggested using AddressSanitizer (ASan) to detect the invalid memory access directly, rather than relying on the platform-dependent glibc allocation behavior. [2026/10/02/22-23-35 by Jeff King]
- **Windows memory usage regression**: Jeff King proposed that the 1 GB per process may reflect *virtual* memory from Git's memory-mapped packfiles on Windows, and suggested checking whether the 1 GB figure is virtual or resident. [2026/10/02/22-19-29 by Jeff King]
- **`git branch -m` reference-transaction hook**: Brian M. Carlson encouraged Никита Поникаров (or others) to submit manually written, higher-quality patches following Git's contribution guidelines, framing the fix as both a correctness issue and a performance opportunity for `git remote rename`. [2026/10/02/19-37-36 by brian m. carlson]