# Git mailing list daily digest for 2026/11/06

## Long threads

### reflog: add %gb format placeholder

*Long thread: 33 replies so far from 6 participants over 23 days; 1 message that day.*

Harald Nordgren's series adds a `%gb` placeholder to pretty formats naming the branch `HEAD` was on when each reflog entry was recorded, read from entries left by branch switches, renames and rebases, with "detached at <commit>" for a detached `HEAD`. Phillip Wood reported that the "detached at " label shows the commit before switching away rather than where `HEAD` was detached, that parsing "rebase: checkout <revision>" treats "origin/main" as a branch name and is confused by commit subjects starting with "checkout ", and supplied a failing test covering those issues and ':' in a URL in `$GIT_REFLOG_ACTION`.

### root refs vs pseudorefs documentation

*Long thread: 27 replies so far from 4 participants over 16 days; 6 messages that day.*

Andy Koppe's series reworks the glossary and `gitdatamodel(7)` to distinguish root refs like `ORIG_HEAD` from pseudorefs like `FETCH_HEAD` and `MERGE_HEAD`, and stops calling root refs pseudorefs across docs, refs code and tests. Koppe posted v3, explaining pseudorefs' additional semantics in the glossary, clarifying the `FETCH_HEAD` and `MERGE_HEAD` descriptions, and rewording two commit messages. Junio C Hamano reviewed patches 1-4, calling the pseudoref rewording "very very good" and the `core.logAllRefUpdates` correction a fix for a "grave mistake", with wording suggestions and a flag that non-root paths outside `refs/` are still treated as refs.

### Clarifying ejected topics can return

*Long thread: 23 replies so far from 5 participants over 103 days; 1 message that day.*

Agreeing with Phillip Wood that ejection is not a permanent rejection, Junio C Hamano proposed new [Discarded] heading text stating removed topics "can be resubmit if they gained enough interest and showed good fit to the system in the future."

### config: read both home and XDG files for `--global`

*Long thread: 20 replies so far from 5 participants over 393 days; 2 messages that day.*

Delilah Ashley Wu's series makes `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` via `do_git_config_sequence()`. Junio C Hamano affirmed the t1300 test patch, asked whether XDG config is always `~/.config/git/config`, asked Wu to shorten the config_sequence commit message, and wondered about adding a `BUG(...)` guard in `git_config_system()` against NULL callers.

### stash creation: avoid unnecessary index loads and refreshes

*Long thread: 20 replies so far from 2 participants over 33 days; 2 messages that day.*

Phillip Wood's seven-patch series removes duplicate index loads and refreshes in `git stash push` and `git stash create`, cutting `lstat()` calls per changed path by four for create and eight for push. Junio C Hamano reviewed the last two patches, calling 6/7 sensible since `create_index_from_tree()` is file-scope static and 7/7 an obvious, very nice simplification of `reset_tree()`'s signature.

## The day in brief

Andy Koppe rerolled the root-refs-vs-pseudorefs docs series to v3 with Junio C Hamano's reviews largely approving it, while Phillip Wood found edge cases in the `%gb` reflog placeholder. Patrick Steinhardt questioned whether the `--follow` speedup regresses sub-tree merges and objected that its fix fails for copied files, and brian m. carlsson confirmed `git init` with `WITH_BREAKING_CHANGES=1` still defaults to sha1 with a fix in CI.

## Notable threads

### rev-list: handle arguments in early option scan
Azuama Odinaka Maximus's patch makes the early option scan in `cmd_rev_list()` skip separate arguments of options like `--grep` so they are not mistaken for options. Junio C Hamano criticized the hardcoded option-name array as fragile and incomplete; Odinaka Maximus surveyed the parsing paths and proposed sharing recognition logic between the early scan and the normal parser, while Hamano questioned whether a separate early scan is the right direction, suggesting option defaults be synthesized after all options are seen.

### log: speed up `--follow` following renames for non-linear history
The series speeds up `git log --follow` across renames in non-linear history and fixes cases where the speedup lost history. Patrick Steinhardt noted that with unrelated histories `follow_needs_renames()` finds no merge bases and always returns 0, asking whether that regresses the sub-tree-merge use case, and objected that patch 6/6's fix fails when a file is copied rather than renamed, with a failing test.

### default object format for new repositories in Git v3
Adam Dinwoodie reported that `git init` built with `WITH_BREAKING_CHANGES=1` still creates repositories with the sha1 object format rather than sha256, confirmed at v2.56.0, `next` and `main`, with repro steps. brian m. carlsson confirmed the behavior appears broken and said he has a patch running through CI that he will send once it is green.

## In brief
- Aniket Gupta reproduced the `git apply` failure on a pure rename followed by an edit with `core.fileMode=false` and proposed `S_IFREG | (patch->old_mode & 07777)`; Junio C Hamano questioned whether the input is valid and whether the fix works in reverse.
- Patrick Steinhardt proposed moving ahead with the `odb/source-loose` stale-cache fix for newly written objects; Junio C Hamano accepted it and asked for a NEEDSWORK comment auditing `QUICK` uses of `OBJECT_INFO_FOR_PREFETCH`.
- Kristofer Karlsson changed `stream_blob()` in `builtin/unpack-objects.c` to take the blob size as `size_t`, fixing truncation of blobs over 4GB where `unsigned long` is 32 bits.
- Junio C Hamano endorsed Matt Hunter's approach of passing more context into `get_follow_remote_head()` for lazy `remote.<name>.followRemoteHEAD` resolution, calling it "very cleanly done".
- Derrick Stolee gave an LGTM to Aniket Gupta's bundle-URI series skipping unresolvable relative URIs instead of dying; Gupta conceded the credentials patch covers only one message and offered a corrected v4.
