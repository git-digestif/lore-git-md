# Git mailing list daily digest for 2026/08/16

## Long threads

### `git repo info` path keys series

*Long thread: 21 replies so far from 4 participants over 32 days; 1 message that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info`, outputting the locations of repository components: `path.toplevel` for the repository tree, `path.superproject-root` for the superproject tree from submodules, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix` for the prefix offset, each supporting absolute and relative format suffixes, with documentation and tests. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, dropped `path.objects` until pluggable ODB backends arrive, moved the `prefix` member from `startup_info` into `struct repository`, and removed redundant empty-string strbuf adds. Today, Lucas Seiki Oshiro objected that the `path.git-prefix` key seems out of place in `git repo info`, arguing it is "current directory information" rather than "repository information", while acknowledging that the information is relevant.

## The day in brief

A data-loss bug in `git worktree add` with empty input drew discussion over authorship, with René Scharfe declining in favor of Matthias Aßhauer. Lucas Seiki Oshiro questioned whether `path.git-prefix` belongs in `git repo info`. Tian Yuchen asked whether `git index-pack` documentation clarifies its standalone design intent in the `fetch_if_missing` thread. Documentation for `bitmapPseudoMerge.*` options was posted, and `git squash` drew support and workflow suggestions from Matej Dujava and Bradley Morgan.

## Notable threads

### `worktree add` out-of-bounds read fix

Matthias Aßhauer's patch fixes an out-of-bounds read in `worktree_basename` in `builtin/worktree.c` when `path` is empty or consists only of directory separators, which previously returned nonsense data and could make `git worktree add ""` recursively delete the current working directory including `.git`. The fix stops the out-of-bounds read, so the command now exits with a `BUG:` message instead of causing data loss. Today, René Scharfe objected to taking authorship of the fix himself, saying it feels unfair because Matthias Aßhauer did most of the work by identifying the bug and the loop change does not seem very original to him; his main concern is getting the surprisingly impactful bug fixed in a reasonable amount of time. Junio C Hamano said that if Matthias signs the patch off, that would work for him too.

### `fetch_if_missing` moved into `struct repository`

This libification series moves the global `fetch_if_missing` variable, which controls whether a missing-object check prompts a lazy fetch from a promisor remote, into `struct repository`, initialized to 1 by default to keep previous behavior. It adjusts `builtin/fetch-pack.c`, `builtin/fsck.c`, `builtin/rev-list.c`, `builtin/index-pack.c`, and `builtin/pack-objects.c` so the `repo` pointer reaches every call site, keeping a NULL check with a fallback to `the_repository` for `index-pack` since it runs via `RUN_SETUP_GENTLY`. Today, Tian Yuchen asked whether the `git-index-pack` documentation clarifies that the design intent is to operate a standalone `.pack` file, with placing it into a repository being a separate step.

### `bitmapPseudoMerge.*` configuration documentation

kapicharly@icloud.com posted documentation for the experimental `bitmapPseudoMerge.*` configuration options in `Documentation/`, covering how `pattern` groups commits into pseudo-merge groups with capture groups forming sub-groups, and detailing the `decay`, `sampleRate`, `threshold`, `maxMerges`, `stableThreshold`, and `stableSize` parameters along with their default values for controlling pseudo-merge bitmap generation.

## In brief

- In the discussion of adding a `git squash` command, Matej Dujava suggested that when later commits address feedback, one can use `fixup!` or `squash!` commits and then run `git rebase --autosquash HEAD~3`, and noted that a v13 patchset discusses a similar workflow where `git history squash HEAD~3` keeps the message of the first commit.
- Bradley Morgan expressed support for adding a `git squash` function, stating it would be useful and that he gets annoyed with rebase headaches.
