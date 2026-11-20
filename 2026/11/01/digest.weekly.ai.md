## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 73 replies so far from 6 participants over 33 days.*

Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull`. After v5 dropped a zdiff3 note and fixed example indentation, Junio C Hamano accepted the iteration and marked the topic ready for `next`.

### fetch refmap and `@{push}` resolution for limited-fetch remotes

*Long thread: 66 replies so far from 5 participants over 42 days.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository by renaming `remote.<name>.refmap` to `remote.<name>.fetchRefmap` (v12) and giving `tracking_for_push_dest()` the same fallback (v13), so `@{push}` resolves correctly for `--limited-fetch` remotes. Phillip Wood and Junio C Hamano debated how `fetchRefmap` interacts with explicit-destination `fetch` entries, with v13 storing such refs in both locations.

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread: 53 replies so far from 6 participants over 34 days.*

Harald Nordgren's v6 adds `--delete-integrated <pattern>`, which deletes branches whose changes landed upstream via squash or rebase merges and names the landing commit as "landed as 9f8e7d6". V6 fixes the usage line and error message reported by Jasper Mayone, reads the index before file merges so `.gitattributes` are honored, and renames variables for clarity. Junio C Hamano reviewed the final patches.

### More `path.*` keys for `git repo info`

*Long thread: 39 replies so far from 5 participants over 109 days.*

K Jayatheerth's GSoC series adds `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup` to `git repo info`. Junio C Hamano stated that v7 has seen no review for a month and will be discarded unless volunteers give it a final review; Justin Tobler said he will take a look.

### doc: new Git tutorial for beginners

*Long thread: 38 replies so far from 6 participants over 25 days.*

Julia Evans's series replaces `gittutorial` with a new beginner tutorial covering `git init` through `git push`. V4 made instructions Windows-friendly by marking SSH steps Mac/Linux only and suggesting Git Credential Manager on Windows; v5 fixed an ambiguous pronoun. Junio C Hamano squashed fixups removing references from other documents.

### reflog: name the branch of each reflog entry

*Long thread: 31 replies so far from 6 participants over 13 days.*

Harald Nordgren's v3 adds a `%gb` placeholder to `--format` that names the branch HEAD was on when each reflog entry was recorded. D. Ben Knoble argued detaching HEAD for `git am` would make the change invisible; Junio C Hamano called detaching a flexibility, and Nordgren will use `git reflog @{0}` instead.

### repack corner cases for cruft-less MIDXs

*Long thread: 30 replies so far from 5 participants over 30 days.*

Taylor Blau's eight-patch series fixes omissions in geometric repacks with `repack.midxMustContainCruft=false` that can make bitmap generation fail. Junio C Hamano questioned whether the topic was past diminishing returns; Taylor Blau stated it should be ready to go as-is.

### Documentation: community voting process

*Long thread: 29 replies so far from 13 participants over 25 days.*

Emily Shaffer's proposed voting-process document continued with discussion of eligible voters, abstentions, and platform choice (leaning toward CIVS). Junio C Hamano asked to drop the word "code", tally non-patch contributions, and use Message-ID instead of commit ID; Jeff King proposed a human-input exception for extra voters.

### Removing `gittutorial-2`

*Long thread: 28 replies so far from 4 participants over 29 days.*

Julia Evans's v3 deletes the stale `gittutorial-2`, leaving a stub redirecting to `gitdatamodel` and `gitcore-tutorial`, and removes it from `command-list.txt`. Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out since the main tutorial is being rewritten.

### object storage extension in the object database

*Long thread: 27 replies so far from 4 participants over 10 days.*

Patrick Steinhardt's v4 introduces `extensions.objectStorage` with a `files` backend whose payload can relocate `.git/objects`, plus `--object-storage-format=` for `git init`, `git clone`, and `git submodule`. Karthik Nayak reviewed v4 and said he is overall happy.

### advertise fetched refs when backfilling tags

*Long thread: 24 replies so far from 4 participants over 22 days.*

Karthik Nayak's v6 fixes a shallow-clone regression where refs fetched before tag backfill were not advertised as haves, by appending fetched tips to `negotiation_include_tips`. Patrick Steinhardt reviewed v6 and stated he is happy with the changes.

### diff-delta: use safe integer arithmetic

*Long thread: 22 replies so far from 5 participants over 5 days.*

Srujith A Poojary's v9 replaces manual size arithmetic in `create_delta_index()` with overflow-checked `st_mult`/`st_add`/`st_add3` helpers. Pablo Sabater corrected the 32-bit struct size and entries cap, and Junio C Hamano challenged fixed-size claims as implementation-defined.

### loose: fix migrating "loose-object-idx" with batch fsync

*Long thread: 22 replies so far from 4 participants over 9 days.*

Justin Tobler's v3 fixes ODB transaction commit failures when a compatibility hash is configured, by merging the temporary directory's loose-object-idx mapping atomically. Patrick Steinhardt reviewed v3 and said he is happy with it.

## The period in brief

The week of 2026/10/26 through 2026/11/01 saw steady traffic across seven active days, with several long-running series reaching late iterations. Readers should not miss Patrick Steinhardt's fix for the `pack-objects` BUG on partial clones, Harald Nordgren's v6 of `git branch --delete-integrated`, and Linus Torvalds' fix for `git describe` distance counting with skewed dates.

## Key developments

### `pack-objects` BUG on filtered `--unshallow` of a moved ref

Patrick Steinhardt traced a crash in `builtin/pack-objects.c` during filtered `--unshallow` fetches to `is_not_in_promisor_pack_obj()` calling `BUG()` on missing objects, which is invalid in partial clones. He fixed it to skip missing objects instead of aborting, with regression tests in `t/t5616-partial-clone.sh`. The same root cause explained a separately reported v2.55.0 case, and the fix also addresses a `git log --follow` performance regression that caused an incident for GitLab.com.

### `git grep --recurse-submodules <tree>` dies on submodule directories

Patrick Steinhardt posted a three-patch series fixing "fatal: unable to read tree" when `git grep --recurse-submodules` reaches a directory inside a submodule. The root cause was commit 93cd344e34, which stopped registering the submodule ODB as a source; the fix reads trees via the submodule's repository. Toon Claes approved all three patches.

### Sequencer refuses shallow boundary commits by default

Nakul V S posted v2 of a patch making `git revert`, `git cherry-pick`, and `git rebase` refuse commits at the shallow boundary of a shallow clone, where such a commit looks like a root that added every file and reverting it would empty the working tree. `--allow-shallow-boundary` overrides the check for revert and cherry-pick only; rebase always refuses and suggests `git fetch --unshallow`. Junio C Hamano reviewed the patch and noted duplication with a helper from another topic.

### `git describe` distance counting with skewed commit dates

Linus Torvalds submitted a patch fixing `git describe` miscomputing commit distance when commit dates are unreliable, as seen in the Linux v7.4 merge window. The fix changes the priority queue ordering from `compare_commits_by_commit_date` to `compare_commits_by_gen_then_commit_date`. Junio C Hamano decided to pick up the fix after Tuomas Ahola conceded his alternative test rewrite failed under Bash.

### `size_t` for streamed large blobs and objects

Elia Pinto posted multiple series switching object and pack size handling from `unsigned long` to `size_t` to avoid truncation of objects 4GiB and larger on LLP64 platforms. The patches cover `pack-objects` streamed blobs, `object-file.c`, `fsck.c`, `commit.c`, `tag.c`, `unpack-objects`, `index-pack`, `pack-check`, grep, archive, and apply, fixing corruption, dies, and false collision errors.

### Notes: do not drop notes written by a concurrent process

Farid Zakaria's patch makes commands that modify notes fail with "cannot lock ref" instead of silently dropping notes written by a concurrent process, by using the read commit as both the parent and expected old value of the ref update. Kristoffer Haugsbakk called the find "really nice" and judged the change a pure improvement.

### credential-cache daemon startup races

Farid Zakaria posted a two-patch series fixing races in `builtin/credential-cache--daemon.c` when two `git credential-cache store` commands run at once and each starts a daemon. The second daemon previously unlinked the first's socket; now it prints "ok" and exits while its client talks to the running daemon.

### `repack -d` losing objects to a mid-run `.keep` file

qeesung posted v4 of the fix for `git repack -d` losing objects when a `.keep` file appears mid-run on a pack slated for deletion. The fix writes pack names to a tempfile passed to spawned `pack-objects` via `--ignore-pack-keep-from=<file>`, while `--honor-pack-keep` still applies to other packs.

## In brief

- **Git for Windows 2.56.0(4)** -- Johannes Schindelin announced an urgent security release addressing CVE-2026-111497, fixing corrupt code-signatures in 32-bit MinGit and upgrading brotli to v1.2.1.
- **Negotiator fix v3** -- Taylor Blau posted v3 setting SEEN only after a commit is successfully parsed and queued in both default and skipping negotiators, preventing a "missing parent in priority queue" BUG.
- **CI actions pinned** -- Taylor Blau posted v2 pinning GitHub workflow actions to full commit IDs so a moved tag cannot substitute a malicious revision; Junio C Hamano queued it.
- **imap-send reordered headers** -- Harald Nordgren posted v4 and v5 accepting From, Date and Subject headers in any order and case-insensitively; Patrick Steinhardt is happy with it.
- **`git rebase --continue` cleanup** -- Phillip Wood posted a 3-patch series removing `REBASE_HEAD` and `AUTO_MERGE`; Junio C Hamano reviewed all three positively.
- **`cat-file remote-object-info` limit** -- Pablo Sabater proposed removing the 10k-OID limit and batching internally with a default of 50k.
- **"not a git repository" errors** -- Kaartic Sivaraam posted v3 naming the inspected directory and its source; Junio C Hamano praised the search-range message.
- **Quarantine alternate refs perf** -- Kristofer Karlsson proposed skipping the repository's own object directory when listing alternate refs, dropping push time with 200,000 hidden refs from 0.31s to 0.10s.
- **`relative_url_gently()`** -- Aniket Gupta added this so an unresolvable relative URI in a bundle list is reported as an error instead of dying.
- **`git init` absolute path** -- Matt Hunter posted v3 restoring the absolute path in output; Junio C Hamano marked it ready for `next`.
- **`git submodule foreach` options** -- William Hatfield posted v2 adding `--reverse-traversal`, `--include-superproject`, and `--reversive`.
- **Worktree handling** -- Jasper Mayone withdrew `git worktree list --branch` and proposed making `%(worktreepath)` name worktrees mid-rebase or bisect; he also posted a series making `git worktree remove` refuse such worktrees without `--force`.
- **`git repo structure` concern levels** -- Mark C. Chu-Carroll posted v4 of degree-of-concern reporting; Junio C Hamano spotted a copy-and-paste error in the largest tree calculation.
- **`NO_PARSEOPT` removal** -- Tanga Joris posted v3 moving completion-helper handling into `show_usage_with_options_if_asked()`.
- **Makefile ccache** -- Harald Nordgren posted v2 exporting `CCACHE_SLOPPINESS += pch_defines,time_macros` for precompiled header caching.
- **`gitsecurityboundary.adoc`** -- Johannes Schindelin posted a document defining Git's security boundary around unexpected execution of attacker-controlled code.
- **HTTP retry handling** -- Ravi Mistry posted a 4-patch series fixing retry counts, adding exponential backoff, `http.retryStatusCodes`, and POST retry support.
- **merge-ort config without value** -- Thomas Bachem submitted a patch making merge-ort accept `merge.renames` etc. given without a value by reading them with `repo_config_get_value()`.
- **`@{push}` simple resolution** -- Harald Nordgren posted v2 resolving "simple" `@{push}` when pushing to another remote; Junio C Hamano said it looks good and will queue it.
- **`stash -p` performance** -- Jasmeet (Jazz) Bhatia submitted a patch passing the untracked file list to skip redundant index work, reporting 16.7% faster `stash -p`.
- **Test fixes** -- Jeff King posted a patch to `t/t5702` replacing `env`-redirect-and-grep with direct `echo` comparison due to GNU coreutils 9.12 quoting changes.
- **Root-refs documentation** -- Andy Koppe posted v2 adding root-refs and pseudorefs to `gitdatamodel(7)` ref types.
- **sigchain recovery** -- ian4hu's patch repairs concurrent double-pops of the signal chain; Phillip Wood diagnosed the root cause and proposed blocking signals when spawning threads.
- **Sequencer `--empty` for fixup targets** -- Farid Zakaria posted v5 making `git rebase --autosquash --empty=drop` drop commits emptied by `fixup!`/`squash!`.
- **midx-write empty writes** -- Pia Park's v3 stops `git multi-pack-index write` from publishing a zero-object MIDX; Junio C Hamano will queue it.
- **Loose-ref seeking** -- Junio C Hamano asked for objections to making Venkat Ragavan S's loose-ref seeking fix ready for `next`.
- **odb/files alternates reload** -- Patrick Steinhardt posted v3 restoring alternates reloading on object lookup retry; Christian Couder flagged a race condition that Steinhardt will adapt.

## Looking ahead

Mike Hommey's patch moving the Rust `gitcore` crate into a `rust/` subdirectory was rejected by Junio C Hamano with "... down." K Jayatheerth's `path.*` keys for `git repo info` will be discarded from Junio's tree unless volunteers provide a final review within a few weeks. Julia Evans's new beginner tutorial is expected to see a reroll addressing remaining review comments.