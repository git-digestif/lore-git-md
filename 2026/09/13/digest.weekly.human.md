# Git Mailing List Digest 2026/09/07 -- 2026/09/13

## The period in brief
This week (2026/09/07--2026/09/13) saw sustained activity across Git's core subsystems, with 24 distinct topics receiving attention. Traffic was heavy and eventful: the `git history squash` feature reached technical completion, the `receive-report` hook series overcame a critical design flaw, and Rust compilation in Windows CI was enabled. Three things a reader should not miss: the `git history` signing series nearing integration, the ODB abstraction effort advancing with pluggable fsck checks, and the Windows CI Rust blocker being resolved.

---

## Key developments

### `git history` signing series nears integration
Souma's series teaching `git history` to sign rewritten commits (`drop`, `fixup`, `reword`, `split`) using GPG reached v3, addressing all prior review feedback from Patrick Steinhardt. The series now consists of two patches: a preparatory refactoring of the replay API and the main feature implementation. Rewritten commits now respect the same signing configuration (`commit.gpgsign`) and command-line options (`-S/--gpg-sign`, `--no-gpg-sign`) as `git commit`, with identical precedence rules. The conceptual question—whether signing rewritten commits authored by someone else is desirable or misleading—remains unresolved but is documented in the commit message. The series touches `replay.c`, `replay.h`, `builtin/history.c`, the `git-history` documentation, and four test scripts (`t3451`–`t3454`). Junio C Hamano has not yet weighed in, but the series appears ready for integration.

### ODB abstraction effort advances with pluggable fsck checks
Patrick Steinhardt posted v3 of his 10-patch series refactoring Git's object integrity verification (fsck) to make consistency checks pluggable per ODB backend. The series restructures fsck so that each ODB backend (loose, packed, multi-pack, etc.) can implement its own consistency checks instead of relying on a single monolithic fsck function in `builtin/fsck.c`. This prepares the codebase for future pluggable ODB backends and fixes a long-standing discrepancy in the `--full` flag's behavior. Steinhardt addressed the last unresolved feedback from Toon Claes by moving the `ODB_FSCK_FULL` filtering logic into the "files" backend's `fsck` callback, making the infrastructure more flexible for non-local backends. The series is now complete and ready for further review or integration.

### Windows CI Rust blocker resolved
Johannes Schindelin's two-patch series enabling Rust compilation in Git's GitHub Actions Windows CI jobs was approved and fast-tracked by Junio C Hamano. The series configures the Rust toolchain to target the GCC ABI (MinGW) instead of the default MSVC ABI, ensuring Cargo produces a static library (`libgitcore.a`) compatible with Git for Windows' linker. The patches touch `.github/workflows/main.yml`, `ci/lib.sh`, `config.mak.uname`, and the `Makefile`, but do not affect core Git functionality or user-facing features. The series removes the `NO_RUST` opt-out in CI, unblocking the broader Rustification effort. Junio agreed to merge the series directly into `next` and fast-track it to `master`, bypassing the usual one-week cooking period due to the lower risk of Windows users building from source.

### `receive-report` hook series overcomes critical design flaw
Karthik Nayak posted v10 of the `receive-report` hook series, addressing a critical design flaw identified by Junio C Hamano. The flaw—a `BUG()` call in the `switch` statement that assumed the client's requested protocol version was always valid—was fixed by replacing the `default: BUG(...)` case with an explicit `REPORT_STATUS_UNKNOWN` case that `break`s, ensuring the code silently skips the status report when the client does not request one. The series also fixed a memory leak in the `override_cmds_error()` helper by introducing an `error_string_owned` flag to track ownership of `cmd->error_string`. The hook enables server administrators to filter or modify the status report sent to clients after ref updates, with GitLab's MVCC use case as the primary motivation. The series is now feature-complete with all prior feedback incorporated and appears ready for graduation to `master`.

### `git history squash` reaches technical completion
Harald Nordgren's v15 reroll of the `git history squash` feature enforced strict case-sensitive matching for autosquash markers (e.g., rejecting `fixup! ABCDEF` while accepting `fixup! abcdef`), aligning with Git's historical convention of emitting only lowercase hexadecimal OIDs. The series is now technically complete, addressing all prior feedback on autosquash marker resolution, shape-based validation, ref protection for local branches, and the `--no-edit` workflow. Junio C Hamano's "Will replace" sign-off from v7 signals intent to queue it for the next release. The feature collapses a commit range into its oldest ancestor while preserving descendant history, avoiding the repeated conflict stops of a rebase-based approach. The series touches `builtin/history.c`, `sequencer.c`/`sequencer.h`, `advice.c`/`advice.h`, `Documentation/git-history.adoc`, `t/t3455-history-squash.sh`, and `t/t9902-completion.sh`.

---

## In brief

**`git history` signing series** -- Souma posted v3 of the series teaching `git history` to sign rewritten commits, addressing all prior review feedback. The series is now ready for integration.

**ODB alternates refactoring** -- Patrick Steinhardt posted v5 of his 9-patch series refactoring ODB alternates handling, incorporating minor naming and documentation improvements. The series removes the ability to write ODB alternates after repository creation, simplifying the ODB interface.

**`uploadpack.lazyFetchTrusted` series** -- Christian Couder and Junio C Hamano resolved the last open nit in the series, which introduces a server-side protected configuration variable to mark repositories as trusted for lazy fetching. The series is now ready for integration.

**MinGW/Windows build adjustments** -- Johannes Schindelin posted v3 of his 12-patch series upstreaming Git for Windows-specific build and runtime adjustments. The series addresses hard-coded assumptions, MSYS2 integration, linking behavior, deprecated build artifacts, a locale-handling regression, and direct `git.exe` invocation.

**`git repo info` path keys** -- Junio C Hamano identified two high-weight issues in K Jayatheerth's v6 series adding path-related keys to `git repo info`: an edge-case discrepancy in `path.cdup` and a refactoring requirement for patch 2/7. The series exposes filesystem locations of repository components in a scriptable format.

**`--force-if-includes` bugfix** -- Tyler Cipriani posted v3 of the series fixing `--force-if-includes` to check the correct reflog, addressing all of Patrick Steinhardt's technical concerns. The series is now ready for integration.

**Rust CI for Windows** -- Johannes Schindelin's series enabling Rust compilation in Git's GitHub Actions Windows CI jobs was revised to address Cargo's native build behavior and variable naming. The series is now ready for integration.

**`gitk` modernization** -- Junio pulled Johannes Sixt's 8-patch series modernizing `gitk`'s color preference dialog and adding a README note about AI contributions.

**Coccinelle rules** -- Junio C Hamano removed a risky Coccinelle rule that converted `if (!E) free(E);` into an unconditional `free(E)` and added a new rule to explicitly allow unconditional `FREE_AND_NULL(E)` calls.

**`git blame` ignore revs** -- Ravi Mistry proposed making `git blame` automatically look for and use a `.git-blame-ignore-revs` file in the repository root if no explicit ignore file is configured.

**Shell completion for `git worktree repair`** -- Yoichi NAKAYAMA added shell completion support for the `git worktree repair` subcommand, updating `contrib/completion/git-completion.bash`.

**macOS CI dependency cleanup** -- Harald Nordgren removed a redundant `brew link --force gettext` command from the macOS CI dependency installation script.

**Scoped CI tool warnings** -- Harald Nordgren scoped "missing tool" warnings for Perforce, Git LFS, and JGit to only the platforms that attempt to install them.

**AsciiDoc linting and formatting** -- Todd Zullinger posted a three-patch series extending the AsciiDoc linting script to enforce backtick-quoting for commands in synopsis-style man pages.

**Advice scope hint simplification** -- Vsevolod Myalitsin proposed replacing the `scope_hint` enum with a boolean `is_global_hint` to simplify the advice scope hint mechanism. Junio C Hamano later proposed an even simpler design: the hint could *always* recommend `--global` without needing to mark individual advice settings.

**ODB transaction race fix** -- Justin Tobler posted a two-patch series fixing a race in the ODB transaction layer during commit when loose objects and large blobs are present with `core.fsync` batching enabled.

**Worktree repair validation** -- Yoichi NAKAYAMA posted a two-patch series preventing `git worktree repair` from incorrectly modifying unrelated worktrees by adding validation logic using the worktree ID.

**Windows test fixes** -- Johannes Schindelin posted a two-patch series fixing Windows-specific test failures that surfaced during full Windows/ARM64 test runs.

**CMake UCRT64 update** -- Johannes Schindelin posted a patch updating the CMake build system for Git for Windows' migration from MINGW64 to UCRT64.

---

## Looking ahead
The next period is likely to see integration of several high-profile series: the `git history` signing series, the `receive-report` hook, the ODB alternates refactoring, and the Windows CI Rust enablement. The ODB abstraction effort will continue with pluggable fsck checks and ongoing alternates handling work. The `git repo info` path keys series may see a v7 addressing the edge-case discrepancy in `path.cdup` and the requested refactoring. The advice scope hint discussion remains unresolved, with Junio's minimalist proposal still on the table. The Coccinelle rule discussion may conclude with a decision to tolerate compiler warnings for the sake of correctness.