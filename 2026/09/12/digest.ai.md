# Git mailing list daily digest for 2026/09/12

## The day in brief

Souma posted the third revision of a series teaching the `git history` subcommands to GPG-sign rewritten commits, while Ariel Keselman sent a v3 that keeps root-ref deletions from taking the packed-refs lock in the files refs backend. Jiri Kuncar also submitted a NULL-guard fix that keeps `git pull` from crashing when a merge head fails to resolve.

## Notable threads

### history: sign rewritten commits

The `git history drop`, `fixup`, `reword`, and `split` subcommands create commits directly and through the replay machinery but currently cannot honor `commit.gpgSign` or an explicit signing request, so users who require signed commits lose that property when rewriting history. Souma posted PATCH v3 of the series "history: support signing rewritten commits": patch 1/2 adds a `sign_commit` field to `replay_revisions_options` and passes it through `pick_regular_commit()` to `create_commit()`/`commit_tree_extended()` when creating replayed commits, and patch 2/2 exposes `-S`/`--gpg-sign[=<key-id>]` and `--no-gpg-sign` on all four subcommands. The commands read `commit.gpgSign` as the default before parsing options, so command-line flags override it and the last flag wins, following the precedence used by rebase, cherry-pick, and revert; the selected key is threaded through direct rewrites and replayed descendants while preserving the original author identity, with the signature recording the current committer's attestation rather than claiming authorship. Changes since v2 include shortened commit messages, renaming the implementation commit from `builtin/history` to `history`, and fixing continuation-backslash formatting in `OPT_HISTORY_GPG_SIGN`.

### refs/files: avoid packed-refs lock for root ref deletion

Deleting a root ref such as `AUTO_MERGE`, `CHERRY_PICK_HEAD`, or `REVERT_HEAD` queues a packed-ref transaction in the files backend even though root refs cannot be packed, so holding `.git/packed-refs.lock` makes `git update-ref --no-deref -d AUTO_MERGE` fail, and post-commit cleanup in a linked worktree with read-only shared metadata reports a packed-refs.lock error after a successful commit. Ariel Keselman posted v3 of the patch, which modifies `files_transaction_prepare` in `refs/files-backend.c` to skip the packed-refs transaction when deleting root refs while keeping loose-ref locking and packed-ref deletion for other refs unchanged. The revision also updates the tests in `t/t0600-reffiles-backend.sh` to run `pack-refs` after creating `AUTO_MERGE` and verify it remains loose, compare the state of all refs around a failed mixed transaction using `git refs list --include-root-refs`, and feed deletes directly to `git update-ref --stdin`; Keselman disclosed that AI assistance generated the patch, tests, and commit message, and reported that the full test suite and all 249 unit tests pass, with t0600 also passing under SHA-256.

### pull: avoid crash of invalid merge head

Jiri Kuncar submitted a patch adding NULL guards for `lookup_commit_reference()` in `builtin/pull.c` to avoid segfaults when a merge head is invalid, which can happen when parallel fetches or gc race on the same repository. A failed lookup is treated as "not up to date", so the caller falls back to a normal merge that reports the broken object instead of crashing. The patch, submitted via GitGitGadget as PR #2223, also adds a test case to `t/t5520-pull.sh` that empties a fetched object and verifies `git pull --no-rebase` fails without crashing.

### doc lint fixes for pack-refs and refs

Inconsistent backtick-quoting in `git-refs(1)` makes the first two commands, `migrate` and `verify`, render bold in man pages and colored differently from the rest in the HTML documentation. Todd Zullinger posted a three-patch series: patch 1/3 updates `Documentation/lint-documentation-style.perl` to match commands as well as options in the synopsis style check by removing the leading `-` requirement from the regex, patch 2/3 converts `git-pack-refs.adoc` and `pack-refs-options.adoc` to the new synopsis style with backtick-quoted option terms and configuration keys, and patch 3/3 applies backtick-quotes to the commands and options in `git-refs.adoc` that were added without quoting after the file's earlier conversion.

### cocci: remove risky "if (!E) free(E)" conversion

René Scharfe agreed with the concern that converting `if (!E) free(E);` into an unconditional `free(E)` turns a no-op into something that could do real damage if the free call itself is wrong, and said he had expected the patch to remove the call entirely instead; he also questioned whether the rule provides value as a guardrail since LeakSanitizer would find a forgotten `free()` given enough test coverage. Junio C Hamano asked whether Scharfe meant rewriting the pattern as `if (!E) ; /* no op free(E) */` or similar, said that could be done, but feared a compiler smart enough would complain and trip `-Werror` when `E` is an obviously side-effect-free expression such as a simple variable reference.

## In brief

- Vsevolod Myalitsin agreed that adding `--global` to the `advice.defaultBranchName` hint is a good solution and proposed not supporting `--system`, `--worktree`, and similar scopes until there is a proven need, choosing between local and global via a boolean flag instead.
- Yoichi NAKAYAMA submitted a patch adding bash completion for the `git worktree repair` subcommand, which optionally completes paths to linked worktrees or worktrees with broken links.
- Harald Nordgren submitted a patch removing the `brew link --force gettext` line from `ci/install-dependencies.sh`, since gettext ships already linked on the macOS CI runner image and the command only emitted a spurious "Already linked" warning.
- Harald Nordgren submitted a patch scoping the perforce, git-lfs, and JGit presence checks in `ci/install-dependencies.sh` to the platforms that attempt to install them (ubuntu-* for all three, macos-* for perforce), since the other platforms never install them yet warned anyway.
- **Root-ref v3 new thread**: Ariel Keselman reported that v3 of the root-ref deletion patch was accidentally sent as a new thread and provided the link.
