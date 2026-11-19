# Git mailing list daily digest for 2026/08/30

## Long threads

### Multi-pack-index recovery when a removed pack is still named

*Long thread: 29 replies so far from 5 participants over 13 days; 1 message that day.*

Elijah Newren's series fixes objects being treated as missing despite being present, a race with concurrent geometric repacking, in the packfile and multi-pack-index code. The current v3 drops the earlier QUICK-reader patch, instead fixing `git mktree --batch` to stop passing `OBJECT_INFO_QUICK`, and renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return so `find_pack_entry()` can recover on `OBJECT_INFO_SECOND_READ` when a stale multi-pack-index names a removed owning pack. Earlier, Jeff King reviewed v3 4/4 and said the workaround looks good to him. Today Junio C Hamano thanked Jeff King for that review and asked whether to mark the series for `next`.

### Converting build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 25 replies so far from 5 participants over 24 days; 1 message that day.*

D. Ben Knoble's three-patch series converts the build-time `USE_NSEC` option into the runtime `core.useNanosec` configuration, changing racy-git detection in `read-cache.c` and `statinfo.c` so index-related commands are less likely to hit racy-git problems on supported filesystems; v5 improved the commit message flow in patch 2. Earlier, Patrick Steinhardt and Junio C Hamano traded review comments on the `#ifdef` strategy, docs, and initialization. Today Junio C Hamano said the patch "looks good to me," but flagged an ordering dependency: `repo_config_values()` must first work on repository instances other than `the_repository` before a `--recurse-submodules` option can load and operate on a submodule index in the same process, since otherwise the submodule's `istate->repo` would not point to `the_repository` and would hit a `BUG()`.

## The day in brief

Junio C Hamano called D. Ben Knoble's runtime `core.useNanosec` patch good while flagging a submodule ordering dependency, and asked whether Elijah Newren's multi-pack-index removed-pack recovery series should be marked for `next`. Nicolas Le Cam reported a use-after-free in `git stash show --src-prefix` that prints freed heap memory since 2.52.0.

## Notable threads

### Use-after-free in `git stash show --src-prefix`

Nicolas Le Cam reported that `git stash show --src-prefix=a/ --dst-prefix=b/` prints fragments of freed heap memory (such as "Uf.txt") instead of the requested prefixes in the diff header, reproducible on 2.52.0 through 2.54.0 but not 2.49.1, while `git diff` with the same flags stays correct. He traced the regression to commit 3ea35c64b, which set `.free_removed_argv_elements = 1` in `show_stash()`; because `OPT_STRING_F` stores pointers into argv elements rather than copying them, freeing those elements leaves `diff_options.a_prefix`/`b_prefix` dangling when the header is emitted. He warned the same hazard applies to any diff option parsed with `OPT_STRING*` into a `struct diff_options` field.

### Worktree lifecycle hooks

Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove`, and `post-worktree-move` hooks in `builtin/worktree.c` and `Documentation/githooks.adoc`, motivated by tools like devenv that need a reliable trigger when linked worktrees appear, move, or are removed, since `post-checkout` does not fire for `--no-checkout` or `--orphan` and nothing fires on move or remove. Today he argued that a wrapper around Git is insufficient because the component needing notification does not control the callers invoking Git (IDEs, coding agents, scripts, or the user directly), and that `git worktree prune` cannot be reproduced in a wrapper since only Git knows which entries it actually removed. He proposed consolidating the three hooks into a single `post-worktree` hook with a fixed interface (`add`/`move`/`remove` with absolute paths), issuing one `remove` event per entry actually pruned and none under `--dry-run`, and asked whether that meets the bar for a native hook.

### Negative pathspecs in `git ls-files` and `git add`

Diogo Castro's patch fixes a bug in `dir.c` where the common prefix computed from only the positive pathspecs is stripped from all pathspecs, so a negative pathspec shorter than that prefix degenerates to an empty string that matches every file (or reads out of bounds), making `git ls-files -- sub/sub/sub/file ':(exclude)nonexistent'` return nothing. Today Diogo objected to extending the common-prefix stripping to negative pathspecs, explaining the optimization avoids unnecessary working-directory walks and that applying it to a negative pathspec like `':!*.md'` could force a full walk even when `a/b/` would suffice. Junio C Hamano cited `git add -- a/b/c a/b/d ':!a/b/x'` and agreed that computing the prefix from positive pathspecs and stripping it from both kinds is incorrect and needs correction.

## In brief

- Harald Nordgren posted v2 of a CI change grouping pull request workflow runs by PR number with `cancel-in-progress` so a newer push cancels the running one; Junio C Hamano flagged that `push` and `pull_request` events for the same commit now fall into separate concurrency groups, so the same commit could be built twice in parallel.
- Junio C Hamano discarded the `die_for_incompatible_opts()` topic for now, saying he did not like the second iteration and liked the preliminary step of the first even less.
- Junio C Hamano posted v2 of an eight-patch refactoring of the checkout/switch/restore code so `cmd_switch()` and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()` instead of the monolithic `checkout_main()`, described as an early rough draft that will not be merged to `seen`.
- Hardik Kumar posted a typofix patch correcting spelling in `versioncmp.c`, `git-gui/git-gui.sh`, and `t/t0022-crlf-rename.sh`; Michael Montalbo reviewed it and said the changes LGTM.
