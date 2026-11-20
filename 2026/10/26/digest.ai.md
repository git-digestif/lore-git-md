# Git mailing list daily digest for 2026/10/26

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread (root 2026/09/24/14-44-15): 73 replies so far from 6 participants over 33 days; 5 messages that day.*

Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page that explains merge conflicts with examples, and links to it from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation instead of re-explaining the process in each. Today she posted v5, which drops the zdiff3 future-improvement note, fixes the diff3 example's indentation and "from the list" wording, and retains "probably" in the resolution explanation; she explained that in such a simple example, spelling out exactly why the resolution is correct risks confusing readers. Junio C Hamano accepted the iteration, noting no one else saw an issue, and marked the topic ready for `next`.

### Fetching from a new remote in a shallow repository

*Long thread (root 2026/09/19/14-47-26): 52 replies so far from 4 participants over 38 days; 7 messages that day.*

Harald Nordgren's five-patch series avoids fetching every branch of a new remote in a shallow repo: it adds a `remote.<name>.refmap` config, lets a refspec-less `git fetch <name>` infer what to fetch from local branches' upstreams, and turns on `--limited-fetch` by default when adding a remote to an already shallow repository. Today Phillip Wood argued that a single config variable should not control both where fetched refs are stored and which refs are fetched, proposing a separate `remote.<name>.fetchRefs` setting for which refs to fetch, and separately asking that the new refmap config be named `remote.<name>.fetchRefmap`; Junio C Hamano agreed that "what to fetch" and "where to store them" already have separate configuration, and that what needs its own setting is the behavior when "what to fetch" is not specified at all.

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread (root 2026/09/29/07-30-30): 47 replies so far from 6 participants over 28 days; 1 message that day.*

Harald Nordgren posted v4 of a series addressing branches merged via "Squash and merge" or "Rebase and merge", whose tips are not reachable from upstream and so are never deleted by `git branch --delete-merged`. The v4 restructuring makes `--delete-merged` again delete only branches whose tips are reachable from their upstream, while a new `--delete-integrated <pattern>` deletes branches that were rebase or squash merged -- including those squashed after upstream changed the same files -- naming the landing commit in output like "Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)". The `branch.<name>.deleteMerged=false` opt-out now exempts branches from both options, and `--dry-run` works with either.

### Removing `gittutorial-2`

*Long thread (root 2026/09/28/20-25-19): 28 replies so far from 4 participants over 29 days; 8 messages that day.*

Julia Evans posted v3 of her two-patch series removing `gittutorial-2` and all references to it, leaving a stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`; the v3 changelog removes `gittutorial-2` from `command-list.txt` so the guides list does not advertise the obsolete tutorial. Junio C Hamano acknowledged the fix but asked that changes to `Documentation/gittutorial.adoc` be left out of this series, since the main tutorial is being rewritten from scratch and the patch would not merge well in `seen`; he said the topic otherwise needs no further rework.

### New beginner Git tutorial

*Long thread (root 2026/10/06/19-37-00): 28 replies so far from 6 participants over 21 days; 6 messages that day.*

Julia Evans's series replaces the old `gittutorial` with a new beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0 and explicitly not covering branches or collaboration. Today Richard Kerry, a Windows Git user, argued that mentions of vim and ssh make Windows users feel the tutorial is not aimed at them and suggested HTTPS instructions; Julia Evans proposed splitting authentication setup into a Windows section using Git Credential Manager (which Git for Windows ships and which pops up a login dialog on the first `git push`) and a Mac/Linux section using SSH. Junio C Hamano marked the topic "Expecting a reroll."

## The day in brief
The new `gitmergeconflicts(7)` documentation series was marked ready for `next` after its fifth revision, while the shallow-repository fetch series drew a design objection that "what to fetch" and "where to store refs" need separate configuration. A `git log --follow` performance regression was reported as having caused an incident at GitLab.com, and the new beginner tutorial is expecting a reroll after Windows-user feedback.

## Notable threads

### `pack-objects` BUG on filtered `--unshallow` of a moved ref
Daniel Zoghalchali reported that since v2.56.0 a fetch combining a tree filter with `--unshallow` aborts in `builtin/pack-objects.c` with "should_include_obj should only be called on existing objects" when the fetched ref has moved past the local tip. Patrick Steinhardt confirmed the reproducer and identified the root cause as `is_not_in_promisor_pack_obj()` calling `BUG()` when `odb_read_object_info_ext()` returns an error -- invalid in a partial clone where objects may legitimately be missing; he also reproduced the separately reported 2.55.0 case sharing the same root cause, and proposed returning 0 instead of `BUG()` on read error, with a patch to follow.

### Git for Windows 2.56.0(4) security release
Johannes Schindelin announced Git for Windows 2.56.0(4), an urgent security bug fix release addressing CVE-2026-111497: it fixes corrupt code-signatures in the 32-bit MinGit executables and upgrades the included brotli component to v1.2.1.

### `repack -d` losing objects to a mid-run `.keep` file
qeesung posted v4 of a three-patch series fixing `git repack -d` losing objects when a `.keep` file appears on a pack already slated for deletion. Instead of v3's approach of passing kept packs to pack-objects, repack now writes the names of non-kept and cruft packs to a tempfile and passes it to every spawned pack-objects via a new `--ignore-pack-keep-from=<file>` option, which clears `pack_keep` for those packs; `--honor-pack-keep` still applies to all other packs. Patch 1/3 also fixes a stale kept-pack cache in the cruft walk via `repo_invalidate_kept_pack_caches()`.

### `.keep` lockfile ownership in ODB transactions
Justin Tobler's v4 series fixes `git-receive-pack` deleting `.keep` lockfiles belonging to a concurrent process receiving an identical packfile, by moving `.keep` creation to the transaction commit phase. Today Karthik Nayak reviewed the series, asking whether the error path in `write_pack_lockfile()` should use `unlink_or_warn()`, whether `ret = -1` should also be set in the "lockfile write failed" path, and flagging a logical gap in the commit message of 4/4; Tobler had earlier explained that index-pack output may be either "pack" or "keep" because it returns a "pack" line when a lockfile already exists.

### `git log --follow` performance regression
Patrick Steinhardt reported that the `--follow` regression introduced by a June change (28 seconds growing to 6m43s on linux.git) has caused an incident for GitLab.com, and expressed hope that the v3 series fixing it will be landed soon to avoid backporting patches.

## In brief
- Taylor Blau posted v2 of a patch pinning GitHub Actions in the main CI workflow to full commit IDs so a moved tag cannot substitute a malicious action revision; Junio C Hamano queued it.
- Patrick Steinhardt objected to Aniket Gupta's series moving line-ending config into `repo_config_values`, since that structure is only populated for `the_repository` and the patches tie Git more to it rather than libifying; the series is on hold.
- Taylor Blau posted v3 of a two-patch fix setting SEEN only after a commit is successfully parsed in both the default and skipping negotiators, preventing a BUG when two tips share a missing parent.
- Karthik Nayak reviewed Patrick Steinhardt's v3 object storage extension series and judged it good apart from nits; Steinhardt will send a new version. *Long thread (root 2026/10/19/08-55-55): 25 replies so far from 4 participants over 8 days; 7 messages that day.*
- Harald Nordgren posted v3 of a CI series failing the build when perforce, git-lfs, or JGit fail to install, dropping the macOS `xattr` quarantine call and explicit error blocks in favor of `set -e`; GitLab CI passed for the quarantine-removal commit. *Long thread (root 2026/09/12/14-38-02): 20 replies so far from 3 participants over 45 days; 15 messages that day.*
- Marc Strapetz submitted a three-patch series teaching `send-pack` to pass old tips to pack-objects as `--inspect-have` for forced non-fast-forward updates, so unchanged blobs are not resent after a rebase; Junio C Hamano noted the problem sounds like what push negotiation was designed to address.
- Junio C Hamano posted v2 of a patch teaching the config-writing machinery to honor read-only permission bits on files like `~/.gitconfig`, refusing to replace them.
