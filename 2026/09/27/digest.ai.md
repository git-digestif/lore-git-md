# Git mailing list daily digest for 2026/09/27

## Long threads

### `git repo info` path keys, GSoC series v7

*Long thread (root 2026/07/16/01-21-31): 37 replies so far from 5 participants over 74 days; 1 message that day.*

K Jayatheerth posted v7 of the GSoC patch series (8 patches) that adds `path.*` keys to `git repo info`, exposing the paths of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, with absolute and relative suffix formatting. Changes since v6 make the `get_superproject_working_tree()` bug fix its own commit (the function now takes a `struct repository *` instead of relying on `xgetcwd()`, so `--git-dir` contexts work), remove dead code in the index path, and make `path.cdup` return a correct value even when the worktree is outside the current working directory, matching `git rev-parse --show-cdup`. Earlier, Junio C Hamano had asked what `path.cdup` returns outside the work tree and suggested splitting the superproject fix from the new keys; K Jayatheerth agreed to the split and to the dead-code removal.

### stash index-mode merge cleanup

*Long thread (root 2026/09/19/21-26-34): 28 replies so far from 3 participants over 9 days; 2 messages that day.*

D. Ben Knoble's v3 series "stash: clean up index-mode test merge" fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true`, where the final patch replaces the shell-out `diff-tree`/`apply --cached`/`reset` sequence in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. Today Junio C Hamano identified a bug in patch 5/5: the patch unconditionally sets `o.verbosity = 0` for the index merge, but the same stack-allocated `merge_options` is reused for a later three-way merge whose verbosity is then never restored; he proposed moving the quiet handling next to `init_ui_merge_options()`. He also said that `seen` "seems to break" t5520 when this topic is merged and that he would eject the topic from his tree for the time being.

## The day in brief

K Jayatheerth posted v7 of the GSoC series adding `path.*` keys to `git repo info`. Junio C Hamano found a verbosity bug in the stash index-merge series and said `seen` "seems to break" t5520 when merged. Daniel Martí posted v2 of the libsecret credential fix, and jyotish kumar's `name-rev` doc patch updating SHA-1 wording to object IDs drew positive reviews.

## Notable threads

### credential/libsecret: load secrets explicitly
Daniel Martí posted v2 of the patch fixing `credential/libsecret` so that `keyring_get()` in `credential/libsecret.c` drops the `SECRET_SEARCH_LOAD_SECRETS` flag and instead always loads the one secret it uses via `secret_item_load_secret_sync()`, which reports errors. He explained that the flag is implemented as a separate `GetSecrets` D-Bus call that skips locked or deleted items and ignores load failures, leaving `secret_item_get_secret()` returning NULL and producing assertion failures before git falls back to prompting; with the patch an inaccessible item yields a useful error message instead, keeping the same number of D-Bus calls and a single code path.

### changing default config values
In the discussion about what follows Git 2.56, Phillip Wood proposed a consequentialist approach to changing defaults for largely cosmetic UI settings such as `diff.algorithm`, `diff.colorWords`, `commit.verbose`, and `merge.conflictStyle`, allowing changes when the expected net benefit outweighs the downside, and proposed a `core.defaults` setting providing evolving "modern defaults" with workflow-specific values, enabled by default with a single opt-out. Junio C Hamano responded that Git already takes such an approach, citing repeated changes to the default merge strategy, but argued that net benefit and downside remain speculative until a change has been released and distributed for months, and questioned the need to poll `core.defaults`, suggesting it would be easier to measure how many users already set a target configuration.

### CI leak-sanitizer failure annotations
Harald Nordgren's patch makes leak-sanitizer CI scripts stop at the first failure and points both failure and leak annotations at the real file and line with the actual error. Phillip Wood asked whether stopping at the first failure means only one leak is reported when there are two, noted the patch adds escaping and changes annotations beyond displaying the backtrace without explaining why, and described strange scrolling behavior on the linked CI page. Harald Nordgren replied that the script already reported every subsequent test as "not ok" after one failure, burying the real leaks in noise, and said he would look into the scrolling and expand the commit message.

### name-rev documentation hash wording
jyotish kumar posted a patch to `Documentation/git-name-rev.adoc` changing the `--annotate-stdin` description from "40-character SHA-1 hexes" to "full-length object ID hexes" and the `--name-only` description from "SHA-1" to "object ID", since `name-rev` handles object IDs according to the active hash algorithm. brian m. carlsson agreed it was a documentation bug, reviewed the patch as much better and complete with no other "SHA-1" or "40" instances left, and Junio C Hamano concurred, noting "object ID" is the best term for `--name-only`.

### format-patch range-diff notes options
Kristoffer Haugsbakk's v2 series teaches `git format-patch` `--[no-]range-diff-notes` to tweak notes output in the range diff independently of patch notes. Junio C Hamano reviewed patch 2/2 and noted that the second `test_when_finished` in the `--range-diff-notes=not-a-note` test is redundant, finding nothing else questionable; Kristoffer Haugsbakk acknowledged the redundancy, explained that without a Range-diff there is no notes section, and said he will fix it in the next version.

## In brief
- Matthias Goergens submitted a documentation patch clarifying in `Documentation/git-remote.adoc` that `git remote set-head` never changes the remote repository's own `HEAD`; every change it makes is local and Git provides no client-side way to change a remote's default branch.
- Kristoffer Haugsbakk fixed the `trailer.<key-alias>.cmd` examples in `Documentation/git-interpret-trailers.adoc`, rephrasing the `help` and `ref` trailer examples to describe generic commands instead of specific scripts.
- Jiang Xin submitted a pull request of l10n updates for Git 2.56.0, adding Afrikaans and Brazilian Portuguese and updating Catalan, French, Irish, Indonesian, Swedish, Turkish, Ukrainian, and Simplified Chinese, plus refreshing `po/AGENTS.md`.
- Kristoffer Haugsbakk mapped his Fastmail and Gmail addresses to his canonical `code@khaugsbakk.name` in `.mailmap`, enabling `git check-mailmap` lookups usable as a trailer command.
- Jiang Xin said he will rebase all l10n commits onto the new upstream branch to prevent reverted commits from being reintroduced and will try to restore the reverted l10n entries.
