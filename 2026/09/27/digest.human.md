# Git mailing list daily digest for 2026/09/27

## Long threads

### `git repo info` gains more `path.*` keys

*Long thread: 37 replies so far from 5 participants over 74 days; 1 message that day.*

K Jayatheerth posted v7 of the GSoC series adding path keys to `git repo info`, now eight patches covering `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, each with absolute and relative suffix formatting where applicable. Changes since v6 make the superproject bug fix its own commit, remove dead code in the index path, and make `path.cdup` work even when the worktree is outside the current working directory. Earlier, Junio C Hamano had pointed out that `get_superproject_working_tree()` relied on the process working directory rather than the repository being inspected, and that `path.cdup` returned an empty string where `git rev-parse --show-cdup` prints an absolute path; both are addressed in this revision.

### stash `--index` in-core merge cleanup

*Long thread: 28 replies so far from 3 participants over 9 days; 2 messages that day.*

D. Ben Knoble's v3 series "stash: clean up index-mode test merge" fixes the interaction between autostashing, staged index entries, and `stash.index=true` by replacing the shell-out sequence in `builtin/stash.c` with an in-core `merge_incore_nonrecursive` call. Today Junio C Hamano identified a bug in patch 5/5: the patch unconditionally sets `o.verbosity = 0` for the index merge, but the same stack-allocated `merge_options` is reused for a later three-way merge whose verbosity is then never restored, and he proposed moving the quiet handling next to `init_ui_merge_options()`. He also reported that `seen` breaks t5520 when this topic is merged and said he would eject the topic from his tree for the time being.

## The day in brief

K Jayatheerth posted v7 of the series adding `path.*` keys to `git repo info`, and D. Ben Knoble's stash `--index` fix drew a new bug report from Junio C Hamano, who said he would eject the topic from his tree. Phillip Wood opened a discussion on changing default config values, and Daniel Martí posted a v2 of the libsecret credential fix.

## Notable threads

### Changing default config values

In the thread on what comes after Git 2.56, Phillip Wood proposed a consequentialist approach to changing defaults for cosmetic UI settings such as `diff.algorithm`, `diff.colorWords`, `commit.verbose`, and `merge.conflictStyle`, plus a `core.defaults` setting offering evolving "modern defaults" with workflow-specific values like `push.default=current`, enabled by default with a single opt-out. Junio C Hamano responded that Git already takes such an approach, citing repeated changes to the default merge strategy, but argued that net benefit and downside remain speculative until a change has been released for months, and questioned the need to poll `core.defaults` when it would be easier to measure how many users already set a target configuration.

### credential/libsecret loads secrets explicitly

Daniel Martí posted v2 of "credential/libsecret: load secrets explicitly", rewording the commit message to explain that `SECRET_SEARCH_LOAD_SECRETS` is implemented as a separate `GetSecrets` D-Bus call after `SearchItems`, so the patch drops the flag and always loads the one secret used via `secret_item_load_secret_sync()`, which reports errors. An inaccessible item now produces a useful error message instead of the assertion failures from `secret_value_get_text` and `secret_value_unref`, with git still falling back to prompting.

### `format-patch --[no-]range-diff-notes`

Kristoffer Haugsbakk's series teaches `format-patch` options to tweak notes output in the range diff independently of patch notes. Junio C Hamano reviewed patch 2/2 and noted that the second `test_when_finished` in the `--range-diff-notes=not-a-note` test is redundant, finding nothing else questionable; Haugsbakk acknowledged the redundancy and said he will fix it in the next version.

## In brief

- **CI leak-sanitizer patch**: Harald Nordgren's patch stops leak-sanitizer scripts at the first failure and points annotations at the real file and error. Phillip Wood asked if only one leak is reported and noted unexplained escaping; Nordgren agreed to expand the commit message.

- jyotish kumar posted a documentation patch updating `git name-rev` `--annotate-stdin` and `--name-only` descriptions from "SHA-1" to "object ID"; brian m. carlson and Junio C Hamano reviewed it as complete.
- Matthias Goergens submitted a `Documentation/git-remote.adoc` patch clarifying that `git remote set-head` never changes the remote's own `HEAD` and that all its changes are local.
- **Trailer cmd doc patch**: Kristoffer Haugsbakk proposed fixing `trailer.<key-alias>.cmd` examples in `Documentation/git-interpret-trailers.adoc`, rephrasing `help` and `ref` to describe generic commands.

- Jiang Xin said he will rebase the l10n commits onto the new upstream branch to prevent reverted commits from being reintroduced after the rebase-amend regression revert.
- **l10n updates for Git 2.56.0**: Jiang Xin submitted a pull request adding Afrikaans and Brazilian Portuguese and updating eight other languages.

- Kristoffer Haugsbakk mapped his Fastmail and Gmail addresses to `code@khaugsbakk.name` in `.mailmap` so `git check-mailmap` resolves his preferred identity.
