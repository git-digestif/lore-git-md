## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 73 replies so far from 6 participants over 33 days.*

Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation. During the week she posted v5, dropping the zdiff3 future-improvement note and fixing the example's indentation and wording. After a final exchange over the resolution rationale, Junio C Hamano accepted the iteration and marked the topic ready for `next`.

### fetch refmap configuration for shallow repos

*Long thread: 66 replies so far from 5 participants over 42 days.*

Harald Nordgren's series keeps a shallow repository from fetching every branch of a new remote by introducing `remote.<name>.fetchRefmap` (renamed from `refmap` in v12) while `remote.<name>.fetch` still decides what is fetched. He posted v12 adding a missing `get_ref_map()` path and v13 giving `tracking_for_push_dest()` the same fallback so `@{push}` resolves correctly for `--limited-fetch` remotes. Phillip Wood and Junio C Hamano debated how explicit-destination `fetch` entries interact with the refmap, and Ben Knoble agreed a future `pushRefmap` would ease his work with notes.

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread: 53 replies so far from 6 participants over 34 days.*

Harald Nordgren's series adds `--delete-integrated <pattern>` to `git branch`, deleting branches whose changes landed upstream via squash or rebase merges and naming the landing commit as "landed as 9f8e7d6". V5 fixed the usage lines and error message that Jasper Mayone had flagged after testing v4 against master, confirming correct behavior across squash merges, renames, and partial landings. V6 reads the index before file merges so `.gitattributes` present only in the index are honored, and renames local variables to `diffopt`. Junio C Hamano reviewed patches 3/4 and 4/4, asking whether ignoring gitattributes in `merge_keeps_upstream()` is intentional.

### More `path.*` keys for `git repo info`

*Long thread: 39 replies so far from 5 participants over 109 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info`: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup`, with absolute and relative suffix formatting, plus a fix so `get_superproject_working_tree()` takes a repository parameter. On November 1 Junio C Hamano stated that the v7 series has seen no interest or review for a full month and that he will discard the topic from his tree unless volunteers give it a final review in a few weeks; Justin Tobler said he will take a look.

### New beginner Git tutorial

*Long thread: 38 replies so far from 6 participants over 25 days.*

Julia Evans's series replaces the old `gittutorial` with a new beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0. She posted v4 making the instructions more Windows-friendly by mentioning Git for Windows' editor-configuration prompt, marking SSH steps as Mac/Linux only with Git Credential Manager suggested on Windows, and adjusting several wordings. V5 replaced an ambiguous "it" with "GCM" and removed trailing whitespace. Junio C Hamano reviewed the patch and asked whether the version is in good shape to merge to `next`.

### reflog: name the branch of each reflog entry

*Long thread: 31 replies so far from 6 participants over 13 days.*

Harald Nordgren posted v3 of the reflog series, dropping v2's `--operations` and `--historical-branch` for a `%gb` placeholder in `--format` that names the branch HEAD was on when each entry was recorded, following switches, renames and rebases. Phillip Wood preferred a format specifier and D. Ben Knoble agreed the reflog is not a true operations log. Ben Knoble argued detaching HEAD for `git am` would make such a change more invisible to users; Junio C Hamano called detaching a flexibility, and Harald Nordgren will use `git reflog @{0}` from now on.

### repack corner cases for cruft-less MIDXs

*Long thread: 30 replies so far from 5 participants over 30 days.*

Taylor Blau's eight-patch series fixes omissions in geometric repacks with `repack.midxMustContainCruft=false` that can make bitmap generation fail, adding tree and tag roots and including required packs in ordinary and incremental MIDX writes. Junio C Hamano questioned the topic's viability after nearly a month of mixed feedback, asking whether it was still improving or past diminishing returns; Taylor Blau stated the topic should be ready to go as-is.

### Documentation: community voting process

*Long thread: 29 replies so far from 13 participants over 25 days.*

Discussion of Emily Shaffer's proposed voting-process document continued. Toon Claes argued eligible voters should be able to abstain and that voting should be reserved for spread opinions. Emily Shaffer summarized reroll changes--subject line, secret-vote requirement, votes referencing a proposal's commit ID--and listed open questions on the platform (leaning toward CIVS) and voter roll. Junio C Hamano asked to drop the word "code", tally non-patch contributions, and use Message-ID instead of commit ID; Jeff King proposed a human-input exception for extra voters; Jonathan Corbet reported using CIVS for TAB elections.

### Removing `gittutorial-2`

*Long thread: 28 replies so far from 4 participants over 29 days.*

Julia Evans's two-patch series deletes the stale `gittutorial-2`, leaving a stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`, and removes references to it. She posted v3 removing it from `command-list.txt` so `git help --guides` does not advertise the obsolete tutorial. Junio C Hamano acknowledged that but asked that changes to `Documentation/gittutorial.adoc` be left out of this series, since the main tutorial is being rewritten.

### odb: introduce object storage extension

*Long thread: 27 replies so far from 4 participants over 10 days.*

Patrick Steinhardt posted v4 of the series adding the `extensions.objectStorage` extension, modeled on the ref storage extension with `<schema>[://<payload>]` syntax; only the "files" backend exists, whose payload can relocate `.git/objects`. The series adds `GIT_OBJECT_STORAGE_FORMAT`, `git rev-parse --show-object-storage-format`, and `--object-storage-format=` for `git init`, `git clone` and `git submodule`. Karthik Nayak reviewed v4, suggested a commit-message typo fix, and said he is overall happy with the version; Patrick Steinhardt fixed the typo locally without sending a new version.

### advertise fetched refs when backfilling tags

*Long thread: 24 replies so far from 4 participants over 22 days.*

Karthik Nayak's series fixes a shallow-clone regression where refs fetched before tag backfill were not advertised as haves, by appending fetched tips to the transport's `negotiation_include_tips`. He posted v5 adding a patch fixing a pre-existing leak in the remote-helper transport cleanup, then v6 reordering the upstream-info test commit to the front and updating commit messages. Patrick Steinhardt reviewed v6 and stated he is happy with the changes.

### diff-delta: use safe integer arithmetic

*Long thread: 22 replies so far from 5 participants over 5 days.*

Srujith A Poojary posted v9 replacing manual size arithmetic in `create_delta_index()` with the overflow-checked `st_mult`/`st_add`/`st_add3` helpers. After Pablo Sabater corrected the 32-bit struct size and the entries cap, and Junio C Hamano challenged fixed-size claims as implementation-defined, v9 frames the change as defensive--future padding changes die on overflow instead of wrapping--while keeping the exact bounds.

### loose-object-idx migration with batch fsync

*Long thread: 22 replies so far from 4 participants over 9 days.*

Justin Tobler's series fixes ODB transaction commit failures when a compatibility hash is configured, because tmp-objdir migration did not merge the "loose-object-idx" mapping file. He posted v3 reading the source mapping file before acquiring the lock to shrink the critical section and adding a comment to `loose_object_idx_merge()`. Patrick Steinhardt reviewed v3 and said he is happy with it. Karthik Nayak suggested renaming functions for consistency, but Justin Tobler declined the renames as not worth a reroll.

## The period in brief

The week of 2026/10/26 through 2026/11/01 saw heavy, eventful traffic across all seven days, with thirteen long-running threads active simultaneously. Readers should not miss that Julia Evans's `gitmergeconflicts(7)` documentation was marked ready for `next`, Harald Nordgren advanced both the shallow-fetch refmap work to v13 and the `--delete-integrated` branch option to v6, and Patrick Steinhardt's object storage extension reached v4 with a positive review.

## Key developments

### `pack-objects` BUG on filtered `--unshallow` fetches

Patrick Steinhardt traced a `pack-objects` abort in filtered `--unshallow` fetches to `is_not_in_promisor_pack_obj()` calling `BUG()` when an object read fails -- invalid in a partial clone where objects may legitimately be missing. He fixed the crash by skipping missing objects instead of aborting, with regression tests in `t/t5616-partial-clone.sh`, and agreed to pass `OBJECT_INFO_SKIP_FETCH_OBJECT` explicitly for clarity.

### `git log --follow` performance regression

Miklos Vajna posted v4 speeding up `log --follow` for non-linear history, splitting the `-m --follow` fix into its own patch and adding a fix for history lost when one merge side renamed a path the other side edited. The regression, caused by commit 304812ed33, had slowed `git log --stat --follow` on linux.git from about 28 seconds to 6m43s and caused an incident for GitLab.com.

### Sequencer refuses shallow boundary commits by default

Nakul V S posted v2 of a patch making the sequencer refuse to revert, cherry-pick, or rebase commits at the shallow boundary of a shallow clone, where such a commit looks like a root that added every file at once and reverting it would empty the working tree. The check runs when each command builds its list of commits to replay, so a refused command leaves no sequencer state behind; `--allow-shallow-boundary` overrides the check for revert and cherry-pick only, while rebase always refuses and suggests `git fetch --unshallow`. Junio C Hamano questioned whether rebase onto master was necessary and noted that `is_shallow_boundary_commit()` duplicates one from Harald Nordgren's stagnant topic.

### `git describe` distance counting with skewed commit dates

Linus Torvalds submitted a patch to `builtin/describe.c` fixing `git describe` miscomputing the commit distance when commit dates are unreliable, as seen in the Linux v7.4 merge window. The fix changes the priority queue ordering from `compare_commits_by_commit_date` to `compare_commits_by_gen_then_commit_date` and adds a test in `t/t6120-describe.sh`. Junio C Hamano decided to pick up the fix after Tuomas Ahola conceded his alternative test rewrite failed under Bash.

### `git grep --recurse-submodules <tree>` fix

Patrick Steinhardt posted a three-patch series fixing `git grep --recurse-submodules` failing with "unable to read tree" on nested submodule trees since Git 2.56.0. Only the first patch fixes the bug by reading trees via the submodule's repository, while the other two are belt-and-suspenders for sparse directory entries and `.gitmodules`. Toon Claes approved all three.

### notes: do not drop notes written by a concurrent process

Farid Zakaria's patch makes commands that modify notes fail with "cannot lock ref" instead of silently dropping notes written by a concurrent process, by using the commit the notes were read from as both the parent of the new notes commit and the expected old value of the ref update. Kristoffer Haugsbakk called the find "really nice" and judged the change a pure improvement; Junio C Hamano offered a merge-or-rebase alternative.

### credential-cache daemon startup races

Farid Zakaria posted a two-patch series fixing races in `builtin/credential-cache--daemon.c` when two `git credential-cache store` commands run at once and each starts a daemon: the second daemon unlinks the first's socket and binds its own, so the first's stored credential is lost. Patch 1 ignores `EEXIST` from `mkdir()` and verifies the directory by stat-ing it; patch 2 uses `unix_ss_create()` so a second daemon prints "ok" and exits while its client talks to the running daemon.

### Moving the Rust gitcore crate into a `rust/` subdirectory

Mike Hommey's patch moves all Rust-specific files into a dedicated `rust/` subdirectory, because a top-level `Cargo.toml` implies a bare `cargo build` works and blocks including the Git source as a submodule of a Rust project. Junio C Hamano stated that Hommey's continued interest was not a strong enough reason to keep the topic given his effort to limit topics in flight, and wrote "... down."

## In brief

**Git for Windows 2.56.0(4)** -- Johannes Schindelin announced an urgent security release addressing CVE-2026-111497, fixing corrupt code-signatures in 32-bit MinGit and upgrading brotli to v1.2.1.

**Negotiator fix v3** -- Taylor Blau posted v3 setting SEEN only after a commit is successfully parsed and queued in both the default and skipping negotiators, preventing a "missing parent in priority queue" BUG.

**CI actions pinning** -- Taylor Blau posted v2 pinning the CI actions in `.github/workflows/main.yml` to full commit IDs so a moved tag cannot substitute a malicious revision; Junio C Hamano queued the patch.

**imap-send header reordering** -- Harald Nordgren posted v4 and later v5 of the series accepting From, Date and Subject headers in any order and case-insensitively; Patrick Steinhardt is happy with it.

**rebase --continue cleanup** -- Phillip Wood posted a 3-patch series so `git rebase --continue` removes `REBASE_HEAD` and `AUTO_MERGE`; Junio C Hamano reviewed all three patches positively.

**cat-file remote-object-info limit** -- Pablo Sabater proposed removing the 10k-OID limit on `cat-file remote-object-info` and batching internally (default 50k, `--remote-max-batch=<n>`), and discussed a server-side cap with Junio C Hamano.

**"not a git repository" errors** -- Kaartic Sivaraam posted v3 improving these errors to name the inspected directory and its source; Junio C Hamano praised the search-range message and questioned the object-directory hint.

**quarantine alternate-refs perf** -- Kristofer Karlsson proposed skipping the repository's own object directory when listing alternate refs in the receive-pack quarantine; a perf test showed push time with about 200,000 hidden refs dropping from 0.31s to 0.10s.

**git init absolute path** -- Matt Hunter posted v3 restoring the absolute path in `git init`'s output message; Junio C Hamano marked it ready for `next`.

**submodule foreach options** -- William Hatfield posted v2 adding `--reverse-traversal`, `--include-superproject`, and the `--reversive` shorthand to `git submodule foreach`.

**worktree remove mid-rebase** -- Jasper Mayone posted a series making `git worktree remove` refuse a worktree mid-rebase or bisect without `--force`, and making `%(worktreepath)` name such worktrees; Junio C Hamano asked that it be labeled v3.

**gitsecurityboundary.adoc** -- Johannes Schindelin posted `Documentation/gitsecurityboundary.adoc`, defining Git's security boundary around unexpected execution of attacker-controlled code.

**HTTP retry handling** -- Ravi Mistry posted a 4-patch series for HTTP retry handling: fixing an off-by-one retry count and `Retry-After` Trace2 timing, adding exponential backoff, a new `http.retryStatusCodes` config, and retry support for POST requests.

**merge-ort config without value** -- Thomas Bachem submitted a patch making merge-ort accept `merge.renames`, `diff.renames`, and `merge.directoryRenames` given without a value by reading them with `repo_config_get_value()`.

**stash -p performance** -- Jasmeet (Jazz) Bhatia submitted a patch passing the already prepared untracked file list from `do_push_stash()` to `do_create_stash()`, reporting 16.7% faster `stash -p` without fsmonitor in p3904 tests.

**size_t for large objects** -- Elia Pinto posted multiple series switching object, pack, grep, diff, archive and apply size handling from `unsigned long` to `size_t` to avoid truncation of objects 4GiB and larger on platforms where `long` is 32 bits.

**@{push} simple resolution** -- Harald Nordgren posted v2 making `@{push}` resolve to the push remote's same-named branch under `push.default=simple` when `remote.pushDefault` points at a fork; Junio C Hamano reviewed it and will queue it.

**t5702 env fix** -- Jeff King posted a patch replacing `env`-redirect-and-grep checks of `GIT_PROTOCOL` with direct `echo` output compared via `test_cmp`, because GNU coreutils 9.12 quotes values containing `=`.

**root-refs-vs-pseudorefs docs** -- Andy Koppe posted v2 of the documentation series adding both to the ref types in `gitdatamodel(7)` and itemized per-worktree ref lists.

## Looking ahead

K Jayatheerth's GSoC series adding `path.*` keys to `git repo info` faces discard from Junio C Hamano's tree unless Justin Tobler or other volunteers provide a final review within a few weeks. Emily Shaffer's community voting process document remains under discussion with open questions on the voting platform and voter roll, and may see a reroll addressing Junio C Hamano's requests to drop the word "code" and use Message-IDs.