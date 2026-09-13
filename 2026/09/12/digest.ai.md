# Git mailing list daily digest for 2026/09/12

## The day in brief

Souma posted v3 of a patch series teaching `git history` to sign rewritten commits, addressing prior review feedback. The Git mailing list also saw substantive discussion about a Coccinelle rule removal, with René Scharfe questioning its practical utility and Junio C Hamano explaining compiler warning concerns. Several small but practical patches landed, including shell completion for `git worktree repair`, a refs subsystem bugfix, and CI log noise reductions.

## Notable threads

### Teach `git history` to sign rewritten commits (v3 posted)

Souma posted v3 of a two-patch series that teaches `git history` to sign rewritten commits (`drop`, `fixup`, `reword`, `split`) using the same configuration (`commit.gpgsign`) and command-line options (`-S/--gpg-sign`, `--no-gpg-sign`) already supported by `git commit`. The series extends the replay API to accept a signing key parameter and threads it through the commit-creation call chain, with command-line options overriding configuration and the last option winning.

Today, Souma addressed prior review feedback by trimming commit messages and fixing the `OPT_HISTORY_GPG_SIGN` macro formatting. The core functionality remains unchanged: the preparatory replay API plumbing (patch 1/2) and the main feature implementation (patch 2/2) now have cleaner commit messages and consistent macro formatting. The conceptual question about signing rewritten commits authored by others remains unresolved but is documented in the commit message.

### Remove risky Coccinelle rule for `if (!E) free(E)`

Junio C Hamano’s patch to remove a risky Coccinelle rule that converted `if (!E) free(E);` into an unconditional `free(E)` sparked substantive discussion today. René Scharfe questioned whether the rule was ever useful in practice, suggesting either replacing the guarded `free(E)` with a `BUG()` to force programmer attention or removing the `free(E)` call entirely. René also raised whether LeakSanitizer with sufficient test coverage would catch forgotten frees just as effectively, shifting the discussion to the role of Coccinelle in Git’s CI pipeline.

Junio C Hamano responded by explaining that removing the guarded `free(E)` call entirely could trigger compiler warnings for empty `if` blocks, especially with `-Werror`. This practical concern leaves the door open for further discussion about whether the project should accept such warnings as a trade-off for safety or find another mitigation.

### Avoid unnecessary packed-refs lock for root ref deletion (v3)

Ariel Keselman posted v3 of a patch that avoids unnecessary packed-refs lock acquisition when deleting root refs like `AUTO_MERGE` or `CHERRY_PICK_HEAD`. The patch adds a condition in `files_transaction_prepare()` to skip packed-ref transactions for root ref deletions, addressing a long-standing inefficiency where `git update-ref --no-deref -d AUTO_MERGE` would fail due to the redundant lock.

Today’s v3 incorporates feedback to verify that `AUTO_MERGE` remains loose after `pack-refs` and tests mixed transactions more directly. The patch is minimal and surgical, preserving existing behavior for non-root refs while fixing a real annoyance in linked worktrees with read-only shared metadata.

### Add scope hint for disabling advice messages (boolean simplification proposed)

Vsevolod Myalitsin proposed replacing the `scope_hint` enum with a boolean `is_global_hint` in the advice scope hint mechanism, aligning with Jeff King’s argument that `advice.*` settings were originally intended to be disabled globally. The change simplifies the design by removing all scopes except `--global` and `--local`, resolving build failures and focusing on the immediate goal of accurate scope hints for `--global` settings like `defaultBranchName`.

Today’s proposal explicitly dismisses `--system` and `--worktree` as unnecessary (YAGNI) and treats all other scopes as local by default. The maintainer’s position on this boolean simplification remains unresolved, with Junio C Hamano previously noting that `--local` could still be useful for users working across multiple projects.

## In brief

- **Shell completion for `git worktree repair`**: Yoichi NAKAYAMA added shell completion support for the `git worktree repair` subcommand, updating `contrib/completion/git-completion.bash` to include "repair" in the list of recognized subcommands.
- **CI noise reduction for macOS**: Harald Nordgren removed a redundant `brew link --force gettext` command from the macOS CI dependency installation script, eliminating a spurious "Already linked" warning in every macOS job log.
- **CI noise reduction for GitHub Actions**: Harald Nordgren scoped "missing tool" warnings for Perforce, Git LFS, and JGit to only the platforms that attempt to install them, reducing false-positive warnings on platforms like Alpine or Fedora.
- **Documentation formatting fixes**: Todd Zullinger posted a three-patch series extending the AsciiDoc linting script to enforce backtick-quoting for commands and options in synopsis-style man pages, addressing visual inconsistencies in the `git-refs` and `git-pack-refs` documentation.
- **Prevent `git pull` segmentation fault**: Jiri Kuncar added NULL guards around `lookup_commit_reference()` in `git pull` to prevent crashes when encountering invalid merge heads, treating failed lookups as "not up to date" to allow graceful fallback.