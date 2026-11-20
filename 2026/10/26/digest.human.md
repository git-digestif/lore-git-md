# Git mailing list daily digest for 2026/10/26

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 73 replies so far from 6 participants over 33 days; 5 messages that day.*

Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation. Earlier revisions addressed review feedback on the diff3 example and wording. Today she posted v5, dropping the zdiff3 future-improvement note and fixing the example's indentation and "from the list" wording; after a final exchange over the resolution rationale, Junio C Hamano accepted the iteration and marked the topic ready for `next`.

### Avoiding fetching every branch of a new remote in a shallow repo

*Long thread: 52 replies so far from 4 participants over 38 days; 7 messages that day.*

Harald Nordgren's five-patch series makes `git remote add` in an already-shallow repository set up `remote.<name>.refmap` with `--limited-fetch` instead of a wildcard fetch refspec, so a plain `git fetch` does not negotiate history for every branch the remote has. Today Phillip Wood argued that one config variable should not control both which refs are fetched and where they are stored, proposing a separate `remote.<name>.fetchRefs` setting to describe which refs are fetched (and separately suggesting the refmap config be renamed `remote.<name>.fetchRefmap`); Junio C Hamano agreed that "what to fetch" and "where to store them" already have separate configuration, and that the local-branch-tracking heuristic need not remain the only option when "what to fetch" is unspecified.

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread: 47 replies so far from 6 participants over 28 days; 1 message that day.*

Branches merged via squash or rebase are not reachable from upstream, so `--delete-merged` never deletes them. After earlier revisions that loosened `--delete-merged` itself drew objections over false positives, Harald Nordgren posted v4 today: `--delete-merged` again deletes only branches whose tips are reachable from their upstream, while a new `--delete-integrated <pattern>` deletes branches whose changes landed upstream -- comparing object ids of changed files, and also handling files upstream changed -- naming the landing commit in output as "landed as 9f8e7d6". The `branch.<name>.deleteMerged=false` opt-out now covers both options.

### Removing `gittutorial-2`

*Long thread: 28 replies so far from 4 participants over 29 days; 8 messages that day.*

Julia Evans's two-patch series deletes the stale `gittutorial-2`, leaving a stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`, and removes references to it. Today she posted v3 removing it from `command-list.txt` so `git help --guides` does not advertise the obsolete tutorial; Junio C Hamano acknowledged that but asked that changes to `Documentation/gittutorial.adoc` be left out of this series, since the main tutorial is being rewritten.

### New beginner Git tutorial

*Long thread: 28 replies so far from 6 participants over 21 days; 6 messages that day.*

Julia Evans's series replaces the old `gittutorial` with a new beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0 and explicitly not covering branches or collaboration. Today Richard Kerry, a Windows user, said mentions of `vim` and `ssh` make the tutorial feel not aimed at Windows users; Julia Evans proposed splitting authentication setup into a Windows section using Git Credential Manager and a Mac/Linux section using SSH, and plans a new version this week. Junio C Hamano marked the topic "Expecting a reroll".

## The day in brief
The `gitmergeconflicts(7)` documentation series was marked ready for `next`, and a new `--delete-integrated` option for squash- and rebase-merged branches was posted. Patrick Steinhardt traced a `pack-objects` abort in filtered `--unshallow` fetches to a `BUG()` on missing objects in partial clones and plans a fix.

## Notable threads

### `pack-objects` BUG on filtered `--unshallow` of a moved ref
Daniel Zoghalchali reported that since v2.56.0 a fetch combining a tree filter with `--unshallow` aborts in `builtin/pack-objects.c` when the fetched ref has moved past the local tip. Patrick Steinhardt confirmed the reproducer and traced the root cause to `is_not_in_promisor_pack_obj()` calling `BUG()` when an object read fails -- invalid in a partial clone, where objects may legitimately be missing -- and confirmed the same cause behind a separately reported v2.55.0 case. He proposed returning 0 instead of `BUG()` and said he would send a patch the following day.

### `git log --follow` performance regression

Commit 304812ed33 improving `--follow` rename handling for non-linear history slowed `git log --stat --follow` on linux.git from about 28 seconds to 6m43s. Today Patrick Steinhardt reported the regression caused an incident for GitLab.com and hoped Miklos Vajna's fix series would land soon to avoid backporting.

### `repack -d` losing objects to a mid-run `.keep` file
`git repack -d` could lose objects when a `.keep` file appeared mid-run on a pack already slated for deletion. Today qeesung posted v4 of the fix: `repack` writes the names of packs it may delete to a tempfile passed to every spawned `pack-objects` via a new `--ignore-pack-keep-from=<file>` option, while `--honor-pack-keep` still applies to all other packs; a first patch also fixes a stale kept-pack cache in the cruft walk.

## In brief
- Johannes Schindelin announced Git for Windows 2.56.0(4), an urgent security release addressing CVE-2026-111497, fixing corrupt code-signatures in 32-bit MinGit and upgrading brotli to v1.2.1.
- Taylor Blau posted v3 of the negotiator fix, setting SEEN only after a commit is successfully parsed and queued in both the default and skipping negotiators, preventing a "missing parent in priority queue" BUG.
- Patrick Steinhardt objected to moving line-ending config into `repo_config_values`, since populating it only works via `the_repository`; Aniket Gupta held off, and Junio C Hamano asked for a comment warning against further moves there.
- Taylor Blau posted v2 pinning the CI actions in `.github/workflows/main.yml` to full commit IDs so a moved tag cannot substitute a malicious revision; Junio C Hamano queued the patch.
