# Git mailing list daily digest for 2026/11/06

## Long threads

### reflog: add format %gb

*Long thread (root 2026/10/15/13-13-55): 33 replies so far from 6 participants over 23 days; 1 message that day.*

Harald Nordgren's series adds a `%gb` placeholder to `git log -g --format` that names the branch `HEAD` was on when each reflog entry was recorded, following branch switches, branch renames and rebases and showing "detached at <commit>" for a detached `HEAD`; v4 fixed `%gb` to name the correct branch when `git rebase <upstream> <branch>` runs with `GIT_REFLOG_ACTION` set. Phillip Wood reported that the "detached at " label shows the commit before switching away from a detached `HEAD` rather than the commit where `HEAD` was detached, that parsing "rebase: checkout <revision>" still treats "origin/main" as a branch name and is confused by commit subjects starting with "checkout ", and supplied a failing test covering those issues plus correct handling of ':' in a URL in `$GIT_REFLOG_ACTION`.

### root refs vs pseudorefs documentation

*Long thread (root 2026/10/22/21-28-48): 27 replies so far from 4 participants over 16 days; 6 messages that day.*

Andy Koppe posted v3 of the five-patch series that stops calling root refs pseudorefs across the glossary, `gitdatamodel(7)` and other documentation, explaining the additional semantics of pseudorefs (they can record more than one object ID and extra metadata, and yield the first object ID when used) and clarifying the `FETCH_HEAD` and `MERGE_HEAD` descriptions. Junio C Hamano reviewed patches 1/5 through 4/5, calling the pseudoref rewording "very very good" and the new root ref entry "good", praising the per-worktree ref update as "very nice", welcoming the `core.logAllRefUpdates` "always" correction as fixing a "grave mistake", and confirming that `--include-root-refs` covers all root refs such as `ORIG_HEAD`; he objected to dropping "pseudorefs" from the `--merge` error message.

### ejecting unreviewed topics from 'seen'

*Long thread (root 2026/07/27/03-09-12): 23 replies so far from 5 participants over 103 days; 1 message that day.*

Junio C Hamano has been proposing a policy rule that topics not reviewed for a period are ejected from `seen`, after his aggressive acceptance of topics left many marked "Needs review" for weeks. He agreed with Phillip Wood's earlier suggestion to clarify that ejection is not a permanent rejection, and proposed new heading text for the [Discarded] section of the "What's cooking" report stating that removed topics can be resubmitted if they gain enough interest and show good fit to the system in the future.

### config: read both home and XDG files for --global

*Long thread (root 2025/10/10/01-14-05): 20 replies so far from 5 participants over 393 days; 2 messages that day.*

Delilah Ashley Wu's v3 series makes `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` by routing global-scope reads through `do_git_config_sequence()` with the other scopes ignored, fixing the inconsistency where `git config list --global` only read the former. Junio C Hamano affirmed the t1300 test patch as "OK" and asked whether XDG config is always `~/.config/git/config`; on the implementation patch he asked for a shorter commit message, said the idea "does look sensible", and wondered whether to add a `BUG()` guard in `git_config_system()` against NULL options.

### stash creation: avoid unnecessary index loads and refreshes

*Long thread (root 2026/10/05/16-35-29): 20 replies so far from 2 participants over 33 days; 2 messages that day.*

Phillip Wood's v4 series removes duplicate index loads and refreshes in `git stash push` and `git stash create`, cutting `lstat()` calls per changed path by four for create and eight for push, and reworks temporary index creation to use `create_index_from_tree()` instead of `reset_tree()`. Junio C Hamano reviewed the last two patches, saying patch 6/7 makes sense because `create_index_from_tree()` is a file-scope static function and the refresh is not needed with the current calling sequence, and calling the `reset_tree()` signature simplification in 7/7 an obvious, very nice simplification.

## The day in brief
Junio C Hamano marked the parse-options `--git-completion-helper` topic ready for `next` and proposed clarifying that topics ejected from `seen` can be resubmitted rather than permanently rejected. Andy Koppe's root-refs-vs-pseudorefs documentation reroll drew detailed review from Junio C Hamano, while brian m. carlson confirmed that `git init` built with `WITH_BREAKING_CHANGES=1` still defaults to sha1 and has a fix running through CI.

## Notable threads

### default object format for new repositories in Git v3
Adam Dinwoodie reported that `git init` built with `WITH_BREAKING_CHANGES=1` still creates new repositories using the sha1 object format rather than sha256, confirmed at v2.56.0, `next` and `main`, with repro steps. brian m. carlson confirmed the behavior appears broken and said he has a patch running through CI that he will send once it is green.

### defer remote.<name>.followRemoteHEAD validation
The series defers validation of `remote.<name>.followRemoteHEAD` from `handle_config()` to `do_fetch()` via the `get_follow_remote_head()` helper, so commands like `git push` or `git remote -v` no longer warn about a value only `git fetch` consults. Matt Hunter reported that merging a rewrite of the topic into `seen` produces the same conflict in `do_fetch()` as replaying the current topic, and Junio C Hamano endorsed Hunter's approach of passing more context into `get_follow_remote_head()` for the lazy string-to-value conversion as "very cleanly done", asking Colin for his preference on moving the topic forward.

### parse-options: answer --git-completion-helper before command runs
The Outreachy series removes the `NO_PARSEOPT` flag from builtins that now use parse-options (`patch-id`, `diff-pairs`, `fast-import`, `var`) so bash completion can ask them for options via `--git-completion-helper`. Junio C Hamano marked the v5 topic ready for `next`.

### rev-list: handle arguments in early option scan

Azuama Odinaka Maximus's v4 patch makes the early option scan in `cmd_rev_list()` skip arguments of options like `--grep` and `--author` so they are not mistaken for options. Junio C Hamano criticized the hardcoded option-name array as fragile and incomplete, missing options from `diff_merges_parse_opts()` and `diff.c`. After Azuama surveyed the parsing paths and proposed sharing option-recognition logic between the early scan and normal parser, Junio questioned whether keeping a separate early scan is worthwhile, suggesting instead that values like `.fetch_if_missing` be synthesized after all command-line options have been seen.

### git apply: pure rename followed by edit fails with core.fileMode=false
With `core.fileMode=false`, applying a patch series containing a pure rename followed by an edit to the renamed file fails with "error: renamed: wrong type". Aniket Gupta reproduced the bug, showed that taking the whole mode from `patch->old_mode` would let a hand-made patch turn a regular file into a symlink, and proposed `S_IFREG | (patch->old_mode & 07777)` instead; Junio C Hamano asked whether a patch naming the same path twice is valid input and whether the approach works when applying the patch in reverse.

## In brief
- Patrick Steinhardt objected that the v4 `--follow` speedup fix in patch 6/6 does not hold when a file is copied rather than renamed, and asked whether `follow_needs_renames()` returning 0 for unrelated histories is a regression.
- Kristofer Karlsson changed `stream_blob()` in `builtin/unpack-objects.c` to take the streamed blob size as `size_t`, fixing truncation of blobs over 4GB on platforms where `unsigned long` is 32 bits wide.
- Patrick Steinhardt agreed to move ahead with the `odb/source-loose` stale-cache fix for newly written objects; Junio C Hamano accepted it and asked for a NEEDSWORK comment on `OBJECT_INFO_FOR_PREFETCH` about auditing uses of `QUICK`.
- D. Ben Knoble reviewed the branch recursion fix for a nameless start point, correcting the causal chain to `find_tracked_branch()` and suggesting simplified commit messages and tests.
- Derrick Stolee gave an LGTM to the bundle-uri series that stops dying on unresolvable relative URIs; Aniket Gupta conceded the credentials-anonymizing patch covers only one of the messages that can print a password.
- Aniket Gupta fixed `add_patterns_from_buffer()` in `dir.c` to skip ignore-file lines that are blank after trimming, so they no longer become empty patterns matching directory paths.
- brian m. carlson argued that an `ObjectID` with algo 0 passed from C is intended to panic, pointing to a patch on the sha256-interop branch making `ObjectID` members non-public.
- Joy Gundu proposed zero-initializing repository configuration values with `memset()`; Patrick Steinhardt called it obviously correct, while Junio C Hamano preferred C99 designated initializers.
- Patrick Steinhardt asked for direct replies to review comments and a range-diff in the next version of the `--follow` speedup series.
- Patrick Steinhardt posted v3 of the rerere subcommands series; Junio C Hamano said the new `RERERE_NOAUTOUPDATE` comment still lacks an explanation of why the flag is needed. *Long thread (root 2026/10/27/10-46-43): 17 replies so far from 4 participants over 11 days; 9 messages that day.*
- Antonin Delpeuch and Junio C Hamano discussed additions to the governance documentation, including communication channels, security-list membership and localization-maintainer handover.
- Patrick Steinhardt posted a 7-patch CI speedup series covering SDK installation, Meson build type and slow tests; Junio C Hamano called all patches interesting reads that fully made sense.
