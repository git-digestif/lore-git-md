# Git mailing list daily digest for 2026/08/30

## Long threads

### packfile recovery when a multi-pack-index names a removed pack

*Long thread (root 2026/08/18/22-34-04): 29 replies so far from 5 participants over 13 days; 1 message that day.*

Elijah Newren's series addresses objects reported as missing despite being present, caused by a race with geometric repacking. The v3 series drops the earlier patch handling QUICK readers, instead fixing `git mktree --batch` to stop passing `OBJECT_INFO_QUICK`, renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return value (`MIDX_FILL_MISS` / `MIDX_FILL_HIT` / `MIDX_FILL_OWNER_UNAVAILABLE`), and makes `find_pack_entry()` recover when a stale multi-pack-index names a removed owning pack by scanning the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ`. Earlier, Jeff King reviewed v3 3/4 and 4/4 and found both good, endorsing the rename and the MIDX-specific scan loop. Today, Junio C Hamano thanked Jeff King for his review of the packfile recovery patch (v3 4/4) and asked whether to mark the series for `next`.

### converting build-time USE_NSEC into runtime core.useNanosec

*Long thread (root 2026/08/07/11-56-23): 25 replies so far from 5 participants over 24 days; 1 message that day.*

D. Ben Knoble's three-patch series converts the build-time `USE_NSEC` option into the runtime `core.useNanosec` configuration variable, because build-time is too early for distributing Git; enabling it at runtime makes index-related commands such as `git diff` less likely to hit "racy git" problems on supported filesystems. The series also aligns `repo_config_values_init` with the struct declaration and adds a Meson `man_base_url` option. The v5 revision, posted earlier, improved the commit message flow in patch 2. Today, Junio C Hamano said the patch "looks good to me," but flagged an ordering dependency: using `repo_config_values()` here requires it to first work on repository instances other than `the_repository` before a `--recurse-submodule` option such as `git diff --recurse-submodules` can load and operate on a submodule index in the same process, since otherwise the submodule's `istate->repo` would not point to `the_repository` and would hit a `BUG()`.

## The day in brief

Junio C Hamano asked whether Elijah Newren's multi-pack-index removed-pack recovery series should be marked for `next` after Jeff King's approving review, while separately flagging a submodule-related ordering dependency on D. Ben Knoble's runtime `core.useNanosec` series. Nicolas Le Cam reported a use-after-free in `git stash show --src-prefix`/`--dst-prefix` that has printed freed heap memory since 2.52.0.

## Notable threads

### use-after-free in git stash show prefix options

Nicolas Le Cam reported that `git stash show --src-prefix=a/ --dst-prefix=b/` prints fragments of freed heap memory (such as "Uf.txt") instead of the requested prefixes in the diff header, reproducible on 2.52.0 through 2.54.0 but not 2.49.1, while `git diff` with the same flags remains correct. He traced the regression to commit 3ea35c64b, which set `.free_removed_argv_elements = 1` in `show_stash()`; because `OPT_STRING_F` stores pointers into argv elements rather than copying them, freeing those elements leaves `diff_options.a_prefix`/`b_prefix` dangling when the diff header is emitted. He warned that the same hazard applies to any diff option parsed with `OPT_STRING*` into a `struct diff_options` field, not only the prefix options.

### worktree lifecycle hooks

Domen Kožar, maintainer of the devenv developer environment manager, argued that a wrapper around Git is insufficient for worktree lifecycle notification because the component needing the notification does not control the callers invoking Git -- IDEs, coding agents, other worktree tools, scripts, or the user directly -- and that `git worktree prune` cannot be reliably reproduced in a wrapper since only Git knows which entries it actually removes. To address concerns about hook proliferation, he proposed consolidating the three separate hooks into a single `post-worktree` hook with a fixed interface -- `post-worktree add <id> "" <new-path>`, `post-worktree move <id> <old-path> <new-path>`, and `post-worktree remove <id> <old-path> ""` -- using absolute paths, issuing one `remove` event per entry actually pruned (none under `--dry-run`), and reporting only operations that have taken effect and cannot be undone. He asked whether this meets the bar for a native hook before spending time on a reroll.

### negative pathspecs in git ls-files and git add

Diogo Castro's patch fixes a bug where `git ls-files` and `git add` compute the common prefix of only the positive pathspecs and then strip that many bytes from all pathspecs, positive and negative alike; with `git ls-files -- sub/sub/sub/file ':(exclude)nonexistent'` the negative pathspec degenerates to "" and matches every file, returning no results, and reading past the end of the string when the common prefix is longer than the negative pathspec. Diogo objected to extending the common-prefix stripping to negative pathspecs, since that optimization avoids unnecessary working-directory walks and applying it to a negative pathspec such as `':!*.md'` could force a full walk. Junio C Hamano cited `git add -- a/b/c a/b/d ':!a/b/x'` and agreed that computing the common prefix from only positive pathspecs and then stripping it from both is incorrect and needs correction.

### canceling stale pull request workflow runs

Harald Nordgren posted v2 of a CI change that groups pull request workflow runs by pull request number (`<workflow>-<pull-request-number>`) with `cancel-in-progress` true for `pull_request` events, so a newer push to a pull request cancels the currently running run, while non-pull-request events keep `<workflow>-<sha>` grouping with `cancel-in-progress` false; the only new content versus v1 is an expanded commit message. Junio C Hamano raised an unresolved objection: if a user has CI enabled on their own repository, pushes a commit, and opens a pull request, the `push` and `pull_request` events for the same commit would now fall into separate concurrency groups, so the same commit could be built and tested twice in parallel instead of the second run being stopped early by `skip-if-redundant`.

### die_for_incompatible_opts() variadic helper

Junio C Hamano discarded the `die_for_incompatible_opts()` topic for now, stating that he did not like the second iteration and did not like the 1/2 preliminary step of the first iteration even less. The series had proposed a variadic replacement for the `die_for_incompatible_optN()` family (N from 2 to 4) that can check an arbitrary number of mutually exclusive options.

## In brief

- Junio C Hamano posted v2 of an eight-patch series repartitioning the checkout/switch/restore code so that `cmd_switch()` and `cmd_restore()` dispatch directly to `checkout_branch()` or `checkout_paths()` without the monolithic `checkout_main()`; he described it as a fairly early rough draft for illustration only that will not be merged to `seen`, with patch 8/8 moving `post_checkout_hook()` from `builtin/checkout.c` to `checkout.c` and giving it a `struct repository` parameter.
- Hardik Kumar posted a typofix patch correcting "fractionnal" to "fractional" in `versioncmp.c`, "occurence" to "occurrence" in `git-gui/git-gui.sh`, and "similiarity" to "similarity" in `t/t0022-crlf-rename.sh`; Michael Montalbo reviewed it and said the changes LGTM.
