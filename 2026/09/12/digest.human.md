# Git mailing list daily digest for 2026/09/12

## The day in brief

Souma posted v3 of the series teaching `git history` to sign rewritten commits, addressing prior review feedback. A discussion about Coccinelle’s role in Git’s CI pipeline continued, with Junio C Hamano explaining why removing a guarded `free(E)` call entirely could trigger compiler warnings. Several small but practical patches landed, including shell completion for `git worktree repair`, a fix for unnecessary packed-refs locks during root ref deletion, and crash prevention in `git pull` when encountering invalid merge heads.

## Notable threads

### Teach `git history` to sign rewritten commits (v3 posted)

Souma posted v3 of the series that teaches `git history` to sign rewritten commits (`drop`, `fixup`, `reword`, `split`) using the same configuration (`commit.gpgsign`) and command-line options (`-S/--gpg-sign`, `--no-gpg-sign`) already supported by `git commit`. The series now consists of two patches: a preparatory refactoring of the replay API plumbing (patch 1/2) and the main feature implementation (patch 2/2). Today’s update addressed prior review feedback by trimming commit messages and fixing the `OPT_HISTORY_GPG_SIGN` macro formatting.

The core functionality remains unchanged: rewritten commits now respect signing configuration and command-line options, with the same precedence rules as `git commit`. The conceptual question—whether signing rewritten commits authored by someone else is desirable or misleading—remains unresolved but is documented in the commit message. The series touches `replay.c`, `replay.h`, `builtin/history.c`, the `git-history` documentation, and four test scripts (`t3451`–`t3454`). All prior review feedback from Patrick Steinhardt has been incorporated, and the series appears close to ready for integration.

### Coccinelle rule removal: trade-offs and compiler warnings

The discussion about Junio C Hamano’s patch removing a risky Coccinelle rule that converted `if (!E) free(E);` into an unconditional `free(E)` continued today. René Scharfe questioned whether the rule was ever useful in practice and suggested replacing the guarded `free(E)` with either a `BUG()` or removing it entirely. He also raised whether LeakSanitizer with sufficient test coverage would catch forgotten frees just as effectively, shifting the discussion to the role of Coccinelle in Git’s CI pipeline.

Junio responded by explaining that removing the guarded `free(E)` call entirely could trigger compiler warnings for empty `if` blocks (e.g., `if (!E) ;`), especially with `-Werror`. This practical concern leaves the door open for further discussion about whether the project should accept such warnings as a trade-off for safety or find another way to silence them. The thread highlights the balance between static analysis, runtime tools, and build hygiene in Git’s development process.

### Avoid unnecessary packed-refs lock for root ref deletion (v3 posted)

Ariel Keselman posted v3 of a patch addressing a long-standing inefficiency in the refs subsystem. The patch avoids unnecessary packed-refs lock acquisition when deleting root refs (like `AUTO_MERGE`, `CHERRY_PICK_HEAD`, etc.), which are never packed. The change is motivated by practical workflows: holding `.git/packed-refs.lock` currently causes `git update-ref --no-deref -d AUTO_MERGE` to fail, breaking post-commit cleanup in linked worktrees with read-only shared metadata.

The implementation is minimal: a single condition in `files_transaction_prepare()` checks `!is_root_ref(update->refname)` before initiating a packed transaction. The patch preserves existing behavior for non-root refs and includes thorough tests covering both the happy path and mixed transactions. Feedback from Patrick Steinhardt has been incorporated, and the patch appears ready for integration.

### Prevent `git pull` segmentation fault on invalid merge heads

Jiri Kuncar posted a bugfix patch preventing `git pull` from crashing when encountering an invalid merge head. The patch adds NULL guards around calls to `lookup_commit_reference()` in `builtin/pull.c`, treating failed lookups as "not up to date" rather than segfaulting. This can happen if a merge head reference is corrupted, possibly due to concurrent operations like parallel fetches or garbage collection.

The patch includes a new test in `t/t5520-pull.sh` that simulates the scenario by corrupting an object and verifying graceful failure. The fix is minimal and well-motivated, addressing a real-world issue with clear test coverage.

## In brief

- **Shell completion for `git worktree repair`**: Yoichi NAKAYAMA added shell completion support for the `git worktree repair` subcommand, updating `contrib/completion/git-completion.bash` to include the new subcommand and extend path completion for linked worktrees.
- **macOS CI dependency cleanup**: Harald Nordgren removed a redundant `brew link --force gettext` command from the macOS CI dependency installation script, reducing log noise since gettext is already linked on the CI runner image.
- **Scoped CI tool warnings**: Harald Nordgren scoped "missing tool" warnings for Perforce, Git LFS, and JGit to only the platforms that attempt to install them, eliminating false-positive warnings on platforms like Alpine or Fedora.
- **AsciiDoc linting and formatting**: Todd Zullinger posted a three-patch series extending the AsciiDoc linting script to enforce backtick-quoting for commands in synopsis-style man pages and applying the fix to `git-pack-refs` and `git-refs` documentation. The changes align with the ongoing effort to standardize man-page formatting.
- **Advice scope hint simplification**: Vsevolod Myalitsin proposed replacing the `scope_hint` enum with a boolean `is_global_hint` to simplify the advice scope hint mechanism, aligning with Jeff King’s argument that `advice.*` settings were originally intended to be disabled globally. The maintainer’s position on this simplification remains unresolved.