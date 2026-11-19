# Git mailing list daily digest for 2026/09/12

## The day in brief

Today's most consequential posts were a crash fix for `git pull` when a merge head does not resolve, the third revision of a refs-backend change that skips the packed-refs lock when deleting root refs, and a new revision of a series adding GPG signing to the `git history` rewrite commands. Smaller items covered bash completion, CI script cleanups, documentation lint fixes, and advice scoping.

## Notable threads

### pull: avoid crash of invalid merge head

Jiri Kuncar submitted a patch adding NULL guards for `lookup_commit_reference()` in `builtin/pull.c`. Invalid merge-head references can arise when parallel fetches or gc race on the same repository, and the current code segfaults on them. The patch treats a failed lookup as "not up to date", so the caller falls back to a normal merge that reports the broken object instead of crashing. It comes with a new test case in `t/t5520-pull.sh` that truncates a fetched object and verifies `git pull --no-rebase` fails cleanly rather than crashing.

### refs/files: avoid packed-refs lock for root ref deletion

Ariel Keselman posted v3 of a patch to the files refs backend. Deleting a root ref such as `AUTO_MERGE`, `CHERRY_PICK_HEAD`, or `REVERT_HEAD` currently queues a packed-ref transaction even though root refs can never be packed, so holding `.git/packed-refs.lock` makes the deletion fail -- which breaks post-commit cleanup in linked worktrees with read-only shared metadata. The change modifies `files_transaction_prepare` in `refs/files-backend.c` to skip the packed transaction for root-ref deletions while keeping loose-ref locking and packed-ref deletion for other refs unchanged. The v3 tests in `t/t0600-reffiles-backend.sh` verify `AUTO_MERGE` stays loose after `pack-refs`, compare all refs around a failed mixed transaction, and feed deletes directly to `git update-ref --stdin`. Keselman disclosed AI assistance in generating the patch and tests, and reported that the full test suite and all 249 unit tests pass.

### history: sign rewritten commits

Souma posted v3 of a two-patch series teaching the `git history` rewrite commands to honor commit signing. The `history` commands create commits directly and via the replay machinery but currently ignore `commit.gpgSign` and explicit signing requests, so users who require signed commits lose that property when rewriting history. The series adds a `sign_commit` field to `replay_revisions_options` in the replay API and exposes `-S`/`--gpg-sign[=<key-id>]` and `--no-gpg-sign` across `git history drop`, `fixup`, `reword`, and `split`. The commands read `commit.gpgSign` as the default, let command-line options override it with the last option winning, and apply the selected key to every new commit including both halves of a split and replayed descendants, while preserving the original author identity.

## In brief

- René Scharfe questioned a coccinelle rule that converts `if (!E) free(E);` into an unconditional `free(E)`, arguing it turns a no-op into a potentially damaging call; Junio C Hamano asked whether rewriting it as an explicit no-op comment would instead trip `-Werror`.
- Vsevolod Myalitsin agreed that suggesting `--global` in the `advice.defaultBranchName` hint is the right fix and proposed limiting advice scoping to a local-versus-global boolean until other scopes have a proven need.
- Yoichi NAKAYAMA added bash completion for the `git worktree repair` subcommand, completing paths to linked worktrees or worktrees with broken links.
- Todd Zullinger posted a three-patch series fixing inconsistent backtick-quoting in the `git-refs` and `git-pack-refs` documentation and extending the style lint to match commands as well as options.
- **Drop no-op gettext link on macOS**: Harald Nordgren submitted a patch to remove `brew link --force gettext` from macOS CI, which only produced a spurious "Already linked" warning.

- Harald Nordgren scoped the perforce, git-lfs, and JGit presence warnings in `ci/install-dependencies.sh` to the platforms that actually install them.
- Ariel Keselman noted that v3 of the root-ref-deletion patch was accidentally sent as a new thread.
