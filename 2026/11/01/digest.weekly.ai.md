## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 73 replies so far from 6 participants over 33 days.*
Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation. After several iterations addressing review feedback on the diff3 example and wording, she posted v5 on October 26 dropping the zdiff3 future-improvement note and fixing indentation; Junio C Hamano accepted the iteration and marked the topic ready for `next`.

### fetch refmap configuration for shallow repos
*Long thread: 66 replies so far from 5 participants over 42 days.*
Harald Nordgren's series keeps a shallow repository from fetching every branch of a new remote by introducing `remote.<name>.fetchRefmap` (renamed from `refmap` in v12) to remap where fetched refs are stored while `remote.<name>.fetch` still decides what is fetched. On October 30 he posted v13, adding a sixth commit that gives `tracking_for_push_dest()` the same fallback to `fetchRefmap` so `@{push}` resolves correctly for `--limited-fetch` remotes, plus a regression test confirming explicit-destination fetch entries are stored in both locations. Phillip Wood and Junio C Hamano debated how `fetch` values and the refmap interact when both carry explicit destinations.

### `git branch --delete-integrated` for squash- and rebase-merged branches
*Long thread: 53 replies so far from 6 participants over 34 days.*
Harald Nordgren's series adds `--delete-integrated <pattern>` to `git branch`, deleting branches whose changes landed upstream via squash or rebase merges even when their tips are not reachable, and naming the landing commit in output as "landed as 9f8e7d6". On November 1 he posted v6, which reads the index before file merges so `.gitattributes` present only in the index are honored, renames variables to `diffopt`, and extends the check to branches merged onto files upstream also changed. Jasper Mayone had tested v4 against master, confirming correct behavior across squash merges, renames, and partial landings, and v5 fixed the usage-line and error-message issues he reported.

### More `path.*` keys for `git repo info`
*Long thread: 39 replies so far from 5 participants over 109 days.*
K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup` -- with absolute and relative suffix formatting, plus a fix so `get_superproject_working_tree()` takes a repository parameter. On November 1 Junio C Hamano stated the v7 series had seen no review for a full month and would be discarded unless volunteers gave it a final review within weeks; Justin Tobler said he would take a look.

### doc: new Git tutorial for beginners
*Long thread: 38 replies so far from 6 participants over 25 days.*
Julia Evans's series replaces the old `gittutorial` with a new beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0. On October 30 she posted v5 with only commit-message and whitespace fixes after v4 made instructions Windows-friendly by mentioning Git for Windows' editor prompt, marking SSH steps Mac/Linux only, and suggesting Git Credential Manager on Windows. Richard Kerry had earlier noted that vim and ssh mentions made Windows users feel excluded; Junio C Hamano squashed fixups commenting out the old tutorial in build files and asked whether the version was ready for `next`.

### reflog: name the branch of each reflog entry
*Long thread: 31 replies so far from 6 participants over 13 days.*
Harald Nordgren's series adds a `%gb` placeholder to `--format` (e.g. `git log -g --format='%h %gd: (%gb) %gs'`) naming the branch HEAD was on when each reflog entry was recorded, following switches, renames and rebases. On October 27 he posted v3 dropping earlier `--operations` and `--historical-branch` options; Phillip Wood preferred the format specifier and D. Ben Knoble agreed the reflog is not a true operations log. Ben Knoble argued detaching HEAD for `git am` would make the change more invisible, and Junio C Hamano called detaching a flexibility.

### repack corner cases for cruft-less MIDXs
*Long thread: 30 replies so far from 5 participants over 30 days.*
Taylor Blau's eight-patch series fixes omissions in geometric repacks with `repack.midxMustContainCruft=false` that can make bitmap generation fail, adding tree and tag roots and including required packs in MIDX writes. On October 28 Junio C Hamano questioned the topic's viability after nearly a month of mixed feedback; Taylor Blau stated it should be ready as-is, and on October 29 Junio remarked his question was not meant to be answered by the author.

### Documentation: community voting process
*Long thread: 29 replies so far from 13 participants over 25 days.*
Discussion of Emily Shaffer's proposed voting-process document continued, with Toon Claes arguing eligible voters should be able to abstain and voting reserved for spread opinions. Emily Shaffer summarized reroll changes -- subject line, secret-vote requirement, votes referencing a proposal's commit ID -- and listed open questions on the platform (leaning toward CIVS) and voter roll. Junio C Hamano asked to drop the word "code", tally non-patch contributions, and use Message-ID instead of commit ID; Jeff King proposed a human-input exception for extra voters.

## The period in brief
The week of 2026/10/26 through 2026/11/01 saw seven active days of steady, technical traffic dominated by long-running series reaching late iterations. Readers should not miss Harald Nordgren's v13 of the shallow-fetch refmap work and v6 of `git branch --delete-integrated`, Patrick Steinhardt's fix for a `pack-objects` crash in partial clones, and Linus Torvalds' patch correcting `git describe` distance counting with skewed commit dates.

## Key developments

### Sequencer refuses shallow boundary commits by default
Nakul V S posted v2 of a patch making the sequencer refuse to revert, cherry-pick, or rebase commits at the shallow boundary of a shallow clone, where such a commit looks like a root that added every file and reverting it would empty the working tree. The check runs when each command builds its commit list, so a refused command leaves no state behind; `--allow-shallow-boundary` overrides for revert and cherry-pick only, while rebase always refuses and suggests `git fetch --unshallow`. Junio C Hamano questioned whether rebase onto master was necessary and noted the helper duplicates one from another topic.

### `git describe` distance counting with skewed commit dates
Linus Torvalds submitted a patch to `builtin/describe.c` fixing `git describe` miscomputing commit distance when commit dates are unreliable, as seen in the Linux v7.4 merge window. The fix changes priority queue ordering from `compare_commits_by_commit_date` to `compare_commits_by_gen_then_commit_date` and adds a test in `t/t6120-describe.sh`. Junio C Hamano decided to pick up the fix after Tuomas Ahola conceded his alternative test rewrite failed under Bash.

### `size_t` migration for large objects on LLP64 platforms
Elia Pinto posted multiple series switching object and pack size handling from `unsigned long` to `size_t` to avoid truncation of objects 4GiB and larger on platforms where `long` is 32 bits. The patches cover `builtin/pack-objects.c` streamed blob writing, `object-file.c`, `fsck.c`, `commit.c`, `tag.c`, `unpack-objects`, `index-pack`, `pack-check`, grep, diff, archive, and apply, fixing corruption, false collision errors, and truncated writes.

### `pack-objects` BUG on filtered `--unshallow` of a moved ref
Patrick Steinhardt traced a `BUG()` abort in `builtin/pack-objects.c` during filtered `--unshallow` fetches to `is_not_in_promisor_pack_obj()` calling `BUG()` when an object read fails -- invalid in partial clones where objects may be missing. He fixed the crash by skipping missing objects instead of aborting, with regression tests in `t/t5616-partial-clone.sh`, and also confirmed the same cause behind a separately reported v2.55.0 case.

### `git grep --recurse-submodules` regression fix
Patrick Steinhardt posted a three-patch series fixing `git grep --recurse-submodules <pattern> <tree>` dying with "unable to read tree" on directories inside submodules since Git 2.56.0. The first patch reads trees via the submodule's repository; the other two are belt-and-suspenders for sparse directory entries and `.gitmodules`. Toon Claes approved all three.

### credential-cache daemon startup races
Farid Zakaria posted a two-patch series fixing races in `builtin/credential-cache--daemon.c` when two `git credential-cache store` commands run at once and each starts a daemon, causing the second to unlink the first's socket and lose its credential. Patch 1 ignores `EEXIST` from `mkdir()` and verifies the directory; patch 2 uses `unix_ss_create()` so a second daemon exits while its client talks to the running daemon.

### `git log --follow` performance regression
Miklos Vajna posted v4 speeding up `log --follow` for non-linear history, splitting the `-m --follow` fix into its own patch and adding a fix for history lost when one merge side renamed a path the other side edited. Patrick Steinhardt had reported that commit 304812ed33 slowed `git log --stat --follow` on linux.git from 28 seconds to 6m43s, causing an incident for GitLab.com.

## In brief

- **Removing `gittutorial-2`** -- Julia Evans posted v3 deleting the stale tutorial and removing it from `command-list.txt` so `git help --guides` does not advertise it; Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out since the main tutorial is being rewritten.
- **`odb` object storage extension** -- Patrick Steinhardt posted v4 of the series adding `extensions.objectStorage` with a `files` backend and `--object-storage-format=` for init, clone, and submodule; Karthik Nayak reviewed and said he is overall happy.
- **diff-delta safe integer arithmetic** -- Srujith A Poojary posted v9 replacing manual size arithmetic in `create_delta_index()` with overflow-checked `st_mult`/`st_add`/`st_add3` helpers, framed as defensive against future padding changes.
- **repack `-d` losing objects to mid-run `.keep`** -- qeesung posted v4 writing pack names to a tempfile passed to spawned `pack-objects` via `--ignore-pack-keep-from=<file>`.
- **midx-write skip empty writes** -- Pia Park posted v3 stopping `git multi-pack-index write` from publishing a zero-object MIDX; Taylor Blau approved and Junio C Hamano will queue it.
- **notes concurrent process fix** -- Farid Zakaria's patch makes concurrent `git notes add` fail with "cannot lock ref" instead of silently dropping notes; Kristoffer Haugsbakk called the find "really nice".
- **sigchain concurrent double-pops** -- ian4hu conceded his patch is a band-aid and offered to build a thread-level fix blocking signals when spawning threads, which Phillip Wood called probably the safest approach.
- **sequencer `--empty` for emptied fixup targets** -- Farid Zakaria posted v5 making `git rebase --autosquash --empty=drop` drop a commit whose `fixup!` empties it.
- **loose-object-idx migration with batch fsync** -- Justin Tobler posted v3 fixing ODB transaction failures when a compatibility hash is configured; Patrick Steinhardt reviewed and said he is happy.
- **advertise fetched refs when backfilling tags** -- Karthik Nayak posted v6 of the shallow-clone regression fix appending fetched tips to `negotiation_include_tips`; Patrick Steinhardt reviewed and is happy.
- **Rust gitcore crate move** -- Junio C Hamano wrote "... down." on Mike Hommey's patch moving Rust files into a `rust/` subdirectory, citing effort to limit topics in flight.
- **HTTP retry handling** -- Ravi Mistry posted a 4-patch series fixing off-by-one retry counts, adding exponential backoff, `http.retryStatusCodes` config, and POST retry support.
- **merge-ort config values** -- Thomas Bachem submitted a patch making merge-ort accept `merge.renames` etc. given without a value by reading with `repo_config_get_value()`.
- **worktree remove mid-rebase** -- Jasper Mayone posted a series making `git worktree remove` refuse a worktree mid-rebase or bisect without `--force`.
- **imap-send header order** -- Harald Nordgren posted v5 accepting From, Date and Subject headers in any order and case-insensitively.
- **Git for Windows 2.56.0(4)** -- Johannes Schindelin announced an urgent security release addressing CVE-2026-111497.
- **CI actions pinning** -- Taylor Blau posted v2 pinning GitHub workflow actions to full commit IDs; Junio C Hamano queued it.
- **negotiator fix** -- Taylor Blau posted v3 setting SEEN only after a commit is successfully parsed and queued in both negotiators.
- **various small fixes** -- Matt Hunter's `git init` absolute path restore was marked ready for `next`; Phillip Wood posted a series for `git rebase --continue` cleanup; Kaartic Sivaraam posted v3 improving "not a git repository" errors.