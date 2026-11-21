## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 73 replies so far from 6 participants over 33 days.*
Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation. After several iterations addressing review feedback on the diff3 example and wording, she posted v5 on October 26, dropping the zdiff3 future-improvement note and fixing indentation. Following a final exchange over the resolution rationale, Junio C Hamano accepted the iteration and marked the topic ready for `next`.

### fetch refmap configuration for shallow repos
*Long thread: 66 replies so far from 5 participants over 42 days.*
Harald Nordgren's series keeps a shallow repository from fetching every branch of a new remote by introducing `remote.<name>.fetchRefmap` (renamed from `refmap` in v12), which remaps where fetched refs are stored while `remote.<name>.fetch` still decides what is fetched. Phillip Wood and Junio C Hamano debated how the two settings interact when both are configured, with Wood arguing that "what to fetch" and "where to store it" need separate configuration. Nordgren posted v13 on October 30, adding `@{push}` resolution through the refmap and a regression test for explicit-destination fetch entries.

### `git branch --delete-integrated` for squash- and rebase-merged branches
*Long thread: 53 replies so far from 6 participants over 34 days.*
Harald Nordgren's series adds `--delete-integrated <pattern>` to `git branch`, deleting branches whose changes landed upstream via squash or rebase merges even when their tips are not reachable, and naming the landing commit in output. After Jasper Mayone tested v4 against master and reported usage-line and error-message issues, Nordgren posted v5 fixing those and v6 on November 1, which reads the index before file merges so `.gitattributes` in the index are honored and renames variables for clarity.

### More `path.*` keys for `git repo info`
*Long thread: 39 replies so far from 5 participants over 109 days.*
K Jayatheerth's GSoC series adds path keys to `git repo info` including `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup`, with absolute and relative suffix formatting. On November 1, Junio C Hamano stated that the v7 series has seen no interest or review for a full month and that he will discard the topic unless volunteers give it a final review in a few weeks; Justin Tobler said he will take a look.

### New beginner Git tutorial
*Long thread: 38 replies so far from 6 participants over 25 days.*
Julia Evans's series replaces the old `gittutorial` with a new beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0. After Richard Kerry noted that mentions of `vim` and `ssh` made Windows users feel excluded, Evans posted v4 on October 28 with Windows-friendly instructions marking SSH steps as Mac/Linux only and suggesting Git Credential Manager on Windows, then v5 on October 30 with minor cleanups. Junio C Hamano reviewed the series and asked whether it was ready to merge to `next`.

### reflog: name the branch of each reflog entry
*Long thread: 31 replies so far from 6 participants over 13 days.*
Harald Nordgren posted v3 of the series adding a `%gb` placeholder to `--format` that names the branch HEAD was on when each reflog entry was recorded, following switches, renames and rebases. Phillip Wood preferred a format specifier and D. Ben Knoble agreed the reflog is not a true operations log; Ben Knoble argued detaching HEAD for `git am` would make such a change more invisible to users, and Junio C Hamano called detaching a flexibility.

### repack corner cases for cruft-less MIDXs
*Long thread: 30 replies so far from 5 participants over 30 days.*
Taylor Blau's eight-patch series fixes omissions in geometric repacks with `repack.midxMustContainCruft=false` that can make bitmap generation fail, adding tree and tag roots and including required packs in MIDX writes. On October 28, Junio C Hamano questioned the topic's viability after nearly a month of mixed feedback; Blau stated the topic should be ready to go as-is, and Hamano later remarked that his question was not meant to be answered by the series author.

### Documentation: community voting process
*Long thread: 29 replies so far from 13 participants over 25 days.*
Discussion of Emily Shaffer's proposed voting-process document continued, with Toon Claes arguing eligible voters should be able to abstain and that voting should be reserved for spread opinions. Shaffer summarized reroll changes including a secret-vote requirement and votes referencing a proposal's commit ID. Junio C Hamano asked to drop the word "code", tally non-patch contributions, and use Message-ID instead of commit ID; Jeff King proposed a human-input exception for extra voters, and Jonathan Corbet reported using CIVS for TAB elections.

## The period in brief
The week of 2026/10/26 through 2026/11/01 saw seven active days of steady traffic across documentation, fetch configuration, and branch management. The most significant movements were Julia Evans's `gitmergeconflicts(7)` page reaching readiness for `next`, Harald Nordgren advancing both the shallow-fetch refmap and `--delete-integrated` series to new versions, and Patrick Steinhardt fixing a `pack-objects` crash in partial clones.

## Key developments

### `pack-objects` crash on filtered `--unshallow` fetches
Patrick Steinhardt traced a `BUG()` abort in `git pack-objects` when combining a tree filter with `--unshallow` on a moved ref to `is_not_in_promisor_pack_obj()` calling `BUG()` on missing objects, which is invalid in partial clones. He fixed the crash by skipping missing objects instead of aborting, with regression tests in `t/t5616-partial-clone.sh`, and also confirmed the same root cause behind a separately reported v2.55.0 case.

### `git log --follow` performance regression
Commit 304812ed33 slowed `git log --stat --follow` on linux.git from about 28 seconds to 6m43s, causing an incident for GitLab.com. Miklos Vajna posted v4 on October 30 speeding up `--follow` for non-linear history, splitting the `-m --follow` fix into its own patch and adding a fix for history lost when one merge side renamed a path the other side edited.

### Sequencer refuses shallow boundary commits
Nakul V S posted v2 of a patch making the sequencer refuse to revert, cherry-pick, or rebase commits at the shallow boundary of a shallow clone, where such a commit looks like a root that added every file at once. The check runs when each command builds its commit list, so a refused command leaves no state behind; `--allow-shallow-boundary` overrides for revert and cherry-pick, while rebase always refuses and suggests `git fetch --unshallow`. Junio C Hamano questioned whether rebase onto master was necessary and noted the check duplicates one from another topic.

### `git describe` distance counting with skewed dates
Linus Torvalds submitted a patch fixing `git describe` miscomputing commit distance when commit dates are unreliable, as seen in the Linux v7.4 merge window. The fix changes the priority queue ordering to `compare_commits_by_gen_then_commit_date` and adds a test for a tag dated before its parent. Junio C Hamano decided to pick up the fix after Tuomas Ahola conceded his alternative test rewrite failed under Bash.

### `git grep --recurse-submodules` on tree objects
Root reported that since Git 2.56.0, `git grep --recurse-submodules <pattern> <tree>` dies with "fatal: unable to read tree" on directories inside submodules, traced to commit 93cd344e34. Patrick Steinhardt posted a three-patch series reading submodule trees via their repository, with Toon Claes approving all three.

### Notes concurrency and credential-cache races
Farid Zakaria posted a patch making commands that modify notes fail with "cannot lock ref" instead of silently dropping notes written by a concurrent process, and a two-patch series fixing races in `credential-cache--daemon` startup when two store commands run at once, where the second daemon unlinks the first's socket.

### ODB object storage extension
Patrick Steinhardt posted v4 of the series adding the `extensions.objectStorage` extension with a `files` backend whose payload can relocate `.git/objects`, plus `--object-storage-format=` for `git init`, `git clone`, and `git submodule`. Karthik Nayak reviewed v4 and said he is overall happy with the version.

## In brief

**Git for Windows 2.56.0(4)** -- Johannes Schindelin announced an urgent security release addressing CVE-2026-111497, fixing corrupt code-signatures in 32-bit MinGit and upgrading brotli to v1.2.1.

**CI actions pinned** -- Taylor Blau posted v2 pinning the CI actions in `.github/workflows/main.yml` to full commit IDs; Junio C Hamano queued the patch.

**Negotiator fix** -- Taylor Blau posted v3 setting SEEN only after a commit is successfully parsed and queued in both negotiators, preventing a "missing parent in priority queue" BUG.

**repack -d .keep file race** -- qeesung posted v4 writing pack names to a tempfile passed to spawned `pack-objects` via `--ignore-pack-keep-from=<file>`.

**sigchain double-pop** -- ian4hu's patch repairs aftermath of concurrent signal chain pops; Phillip Wood diagnosed the root cause and proposed blocking signals when spawning threads.

**Sequencer --empty handling** -- Farid Zakaria posted v5 making `git rebase --autosquash --empty=drop` drop commits emptied by fixup/squash.

**Alternates reloading** -- Patrick Steinhardt posted v3 restoring `info/alternates` reloading on object lookup failure; Christian Couder flagged a race in the stat validity check.

**MIDX empty writes** -- Pia Park posted v3 stopping `git multi-pack-index write` from publishing zero-object MIDXs; Junio C Hamano said he will queue it.

**Loose-ref seeking** -- Junio C Hamano asked for objections to making Venkat Ragavan S's loose-ref seeking fix ready for `next`.

**imap-send headers** -- Harald Nordgren posted v4 and v5 accepting From, Date and Subject headers in any order and case-insensitively.

**rebase --continue cleanup** -- Phillip Wood posted a 3-patch series removing `REBASE_HEAD` and `AUTO_MERGE`; Junio C Hamano reviewed positively.

**cat-file remote-object-info** -- Pablo Sabater proposed removing the 10k-OID limit and batching internally.

**Error messages** -- Kaartic Sivaraam posted v3 improving "not a git repository" errors to name the inspected directory.

**git init output** -- Matt Hunter posted v3 restoring the absolute path in `git init`'s output; marked ready for `next`.

**submodule foreach** -- William Hatfield posted v2 adding `--reverse-traversal`, `--include-superproject`, and `--reversive`.

**worktree handling** -- Jasper Mayone proposed making `%(worktreepath)` name worktrees mid-rebase or bisect, and posted a series making `git worktree remove` refuse such worktrees without `--force`.

**HTTP retry** -- Ravi Mistry posted a 4-patch series for HTTP retry handling with exponential backoff and `http.retryStatusCodes`.

**merge-ort config** -- Thomas Bachem submitted a patch making merge-ort accept `merge.renames` etc. given without a value.

**size_t conversions** -- Elia Pinto posted multiple series switching object, pack, grep, diff, archive, and apply size handling from `unsigned long` to `size_t` to fix 4GiB+ objects on LLP64 platforms.

**Rust crate move** -- Mike Hommey's patch moving Rust files into a `rust/` subdirectory was declined by Junio C Hamano, who wrote "... down."

**gitsecurityboundary** -- Johannes Schindelin posted `Documentation/gitsecurityboundary.adoc` defining Git's security boundary.

**root-refs documentation** -- Andy Koppe posted v2 adding root-refs and pseudorefs to `gitdatamodel(7)`.

**Test fixes** -- Jeff King posted a patch to `t/t5702` replacing `env`-redirect checks due to GNU coreutils 9.12 quoting changes.

**Removing `gittutorial-2`** -- Julia Evans posted v3 deleting the stale tutorial and redirecting to `gitdatamodel` and `gitcore-tutorial`.

**diff-delta safe arithmetic** -- Srujith A Poojary posted v9 replacing manual size arithmetic with overflow-checked helpers.

**Tag backfill haves** -- Karthik Nayak posted v6 fixing shallow-clone tag backfill to advertise fetched refs as haves.

**loose-object-idx migration** -- Justin Tobler posted v3 fixing ODB transaction failures with compatibility hashes; Patrick Steinhardt reviewed positively.