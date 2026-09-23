# Git mailing list daily digest for 2026/09/22

## The day in brief
Erik Cervin-Edin posted v3 of the `--fixup` message-option unification series, completing the feature matrix for non-interactive fixup commits. Junio C Hamano raised a layering concern in the `git repack` concurrent-push fix, while Maciej Ciemborowicz posted v4 of the reference-transaction hook regression fix, addressing all remaining feedback. The Git v3.0 timeline discussion continued, with Johannes Schindelin requesting a provisional release plan from Junio.

## Notable threads

### `--fixup` message-option unification (v3)
**What changed?**
Erik Cervin-Edin posted v3 of the series enabling non-interactive message specification for all `git commit --fixup` variants. The only change since v2 is the removal of the redundant `die_for_incompatible_opt3()` function in patch 2/2, which became obsolete after refactoring.

### Why it matters

The series completes the feature matrix for `--fixup` commits, allowing users to specify messages via `-m`, `-F`, `-c`, and `-C` options across all fixup types (`--fixup`, `--fixup=amend:`, `--fixup=reword:`). This eliminates the last inconsistency in Git's fixup workflow, making it fully scriptable and non-interactive.

### Technical details

- Files touched: `builtin/commit.c`, test suite
- Unified message handling through `prepare_to_commit()` for all variants
- Special handling for amend-style messages (strips "amend!" lines)
- Comprehensive test coverage in `t/t7500` and `t/t7501`
- No changes to core data structures or on-disk formats

### Reference-transaction hook regression fix (v4)
**What changed?**
Maciej Ciemborowicz posted v4 of the series fixing the regression where the reference-transaction hook received all-zero OIDs when branches, tags, or remote refs were deleted. This version addresses all remaining feedback, including granular error reporting for partial failures and internal API consistency.

### Why it matters

The regression, introduced in Git 2.31, broke external tools (Gerrit, GitLab, CI systems) that rely on the hook to monitor ref changes. The fix restores the pre-2.31 behavior where the hook receives meaningful old OID values while preserving the performance gains of the original optimization.

### Technical details

- Files touched: `refs.c`, `builtin/branch.c`, `builtin/tag.c`, `builtin/fetch.c`, `builtin/remote.c`
- Core change: `refs_delete_refs()` now uses `REF_TRANSACTION_ALLOW_FAILURE` and returns failed refs via an optional `string_list` parameter
- UI now reports "[deleted]" for successful prunes and omits failed refs from dangling symref checks
- Test coverage expanded to include partial prune scenarios for both fetch and remote operations
- No performance regression (~0.85s for 10k packed tags)

### Git repack concurrent-push fix (layering concern)
**What changed?**
Junio C Hamano raised a layering concern about the unchecked downcast in patch 2/5 of the `git repack` concurrent-push fix series. The code assumes every ODB source is a `files`-backend source and calls `odb_source_files_downcast()` without first checking the source type, which will trigger a `BUG()` if the source is from a different backend.

### Why it matters

The concern is architectural: the current implementation violates layering principles by exposing backend-specific details in generic code. While the immediate fix is correct, the unchecked downcast could cause problems when a non-files ODB backend is introduced.

### Technical details

- Files touched: `object-file.c` (patch 2/5)
- Proposed alternatives: check source type before downcasting or push packfile management details down to the files backend layer
- The fix is forward-looking and does not block the immediate patch
- Likely respondents: Patrick Steinhardt (ODB abstraction lead), Justin Tobler (ODB-focused series author)

### Git v3.0 timeline discussion
**What changed?**
The Git v3.0 timeline discussion continued, with Johannes Schindelin requesting Junio C Hamano outline a provisional release plan in the next "What's cooking" report. The Contributor's Summit recap highlighted Rust support as a goal (but not guaranteed due to portability concerns) and SHA-256 interoperability as technically complete but not upstreamed.

### Why it matters

The discussion aims to align the community on the Git 3.0 release timeline, scope, and breaking-change expectations. A provisional plan would help downstream projects coordinate, though the roadmap remains fluid pending public release of the summit notes and further community feedback.

### Technical details

- Provisional timeline: Git 2.98 (December 2026), v2.99 (transitional release), v3.0 (spring 2027)
- Rust support: Goal for v3.0 but not guaranteed due to portability concerns (e.g., NonStop platform)
- SHA-256 interoperability: Technically complete but not upstreamed; ecosystem gaps (e.g., JGit)
- Breaking changes: `bc/restrict-hex-to-lowercase` in preparation

## In brief
- **`git-p4` security fix**: Anupam Mediratta posted a patch replacing a shell-based pipeline with direct subprocess calls to eliminate a shell injection vulnerability in the `--commit` option.
- **`git reflog expire` regression fix**: Pushkar Singh posted a patch swapping the values of `.default_expire_total` and `.default_expire_unreachable` to restore the documented behavior (reachable=90d, unreachable=30d).
- **`git stash` autostash fix**: D. Ben Knoble finalized the v2 design, using `merge_finalize()` for cleanup and fixed labels for ancestor/branch names.
- **Test modernization**: Mark C. Chu-Carroll posted v6 of the `t40*` diff test modernization series, addressing all prior feedback on rebase errors and commit separation.
- **Documentation cross-references**: Julia Evans posted a patch replacing informal plain-text section references with AsciiDoc link syntax across 38 man pages, approved by Junio C Hamano.
- **Git for Windows 2.56.0-rc2**: Johannes Schindelin announced the release candidate, dropping Windows 8.1 support and fixing several bugs, including a 32-bit installer issue and a Vim regression.
- **CI workaround**: Johannes Schindelin posted a patch skipping flaky HTTP/2 authentication tests in Git's Debian 12 CI job due to an intermittent failure in curl 7.88.1.