# Git mailing list daily digest for 2026/11/01

## Long threads

### branch: let --delete-integrated find branches merged onto changed files

*Long thread (root 2026/09/29/07-30-30): 53 replies so far from 6 participants over 34 days; 1 message that day.*

Harald Nordgren posted v6 of his series adding a `--delete-integrated <pattern>` option to `git branch` in `builtin/branch.c`. The feature treats a branch as merged when some upstream commit contains all of its changes, covering GitHub "Squash and merge" and "Rebase and merge" branches whose tips are not reachable from the upstream and therefore survive `git branch --delete-merged`; the output names the landing commit ("Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)"). Changes in v6 read the index before file merges so `.gitattributes` present only in the index are honored, and rename the `diff_options` variables from `opt` to `diffopt`. Patch 4/4 extends the check to branches squash-merged onto files that upstream also changed, by merging the branch's version into the upstream version and treating the change as landed when the upstream version is unchanged; this reads file contents and may download blobs in a partial clone. Earlier, v4 had restructured the feature so `--delete-merged` again deletes only reachable tips, Jasper Mayone had tested v4 against master and reported it deletes squash and rebase merges while keeping partly-landed and never-merged branches, and Junio C Hamano had questioned whether the index is read before `ll_merge()` consults gitattributes.

### add more path keys to git repo info

*Long thread (root 2026/07/16/01-21-31): 39 replies so far from 5 participants over 109 days; 2 messages that day.*

K Jayatheerth's GSoC series adds `path.*` keys to `git repo info` exposing paths of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, each with absolute and relative suffix formatting. Previously, v7 made the `get_superproject_working_tree()` fix its own commit so the superproject is determined from the repository being inspected rather than the current working directory, removed dead code around `repo_get_index_file()`, and made `path.cdup` work when the worktree is outside the current working directory. Today, Junio C Hamano stated that the v7 series has seen no interest or review for a full month and that he will discard the topic from his tree unless volunteers give it a final review in a few weeks; Justin Tobler said he will take a look at the topic.

## The day in brief

Junio C Hamano reviewed Harald Nordgren's `@{push}` fix for `push.default=simple` when pushing to another remote, said it looks good, and will queue it; he set a review deadline for K Jayatheerth's month-unreviewed `git repo info` path-keys series, which Justin Tobler volunteered to examine. Harald Nordgren posted v6 of `--delete-integrated` for `git branch` and v5 of the `imap-send` header fix, while Farid Zakaria detailed two startup races in `credential-cache--daemon`.

## Notable threads

### remote: resolve "simple" @{push} when pushing to another remote

Harald Nordgren's two-patch v2 series in `remote.c` makes `@{push}` resolve to where `git push` goes when `push.default=simple` and `remote.pushDefault` point at a fork, instead of failing with "cannot resolve 'simple' push to a single destination", so `git status` can compare the branch with its push destination. The v2 changes move the "simple" resolution out of `branch_get_push_1()` into its own helper in a preparatory commit. Today, Junio C Hamano reviewed the series, said it looks good, and stated he will queue it.

### credential-cache--daemon: do not take over a running daemon's socket

Farid Zakaria posted a two-patch series fixing races in `builtin/credential-cache--daemon.c` when two `git credential-cache store` commands run simultaneously and each tries to start a daemon: the second daemon unlinks the first's socket and binds its own, losing the first's stored credential, leaving `git credential-cache exit` to stop only the second daemon, and causing the first daemon's tempfile handler to remove the second's socket on exit. He found the issue with Rewind, a deterministic VM he is building. The first patch ignores `EEXIST` from `mkdir()` and stats the directory to verify permissions, and the second uses `unix_ss_create()` instead of `unix_stream_listen()` so a second daemon prints "ok" and exits while its client talks to the running daemon, registering the socket for removal only after binding succeeds.

### imap-send: fix with reordered message headers

Harald Nordgren posted v5 of his four-patch `imap-send` series, which stops rejecting patches sent with `git send-email` (which write Subject before Date) by accepting From, Date and Subject headers in any order within each message's header block. The new v5 change ends the header block at an empty line in CRLF mboxes as well, so headers in the body or later messages no longer count, with a test added. The series also makes `imap-send` wait for the tunnel command to finish when `imap.tunnel` is set, drops the special case for "From git-send-email" lines, and matches header names case-insensitively per RFC 5322.

### sequencer: refuse to pick or revert shallow boundary commits

Nakul V S's patch makes `revert`, `cherry-pick` and `rebase` refuse shallow boundary commits -- commits at the edge of a shallow clone's history whose parents are hidden, which look like root commits and would empty or refill the working tree when replayed. `revert` and `cherry-pick` gain `--allow-shallow-boundary` to override, remembered across `--continue`, while `rebase` always refuses and suggests `git fetch --unshallow`. Today, Junio C Hamano questioned whether rebasing onto current master was necessary, asked about a vision for later supporting `git commit --amend` and hand-typed todo lines, questioned the use of `git_config_bool_or_int()` for `options.allow-shallow-boundary`, and suggested checking conflicts with `next` and `seen` and reusing the `is_shallow_boundary_commit()` function from Harald Nordgren's stagnant object-name topic.

### move rust gitcore crate to a different subdirectory

Mike Hommey's v5 patch moves all Rust-specific files (`Cargo.toml`, `build.rs`, `src/*.rs`) into a dedicated `rust/` subdirectory, because a top-level `Cargo.toml` implies `cargo build` works standalone (it does not) and prevents including the git source as a submodule of a Rust project. Today, Junio C Hamano wrote that the reason given was "not a strong enough reason to keep this one" as he tries to keep the number of topics in flight low, and added "... down."

## In brief

- Junio C Hamano decided to pick up Linus Torvalds's `git describe` skewed-commit-dates fix and move it forward after objecting to Tuomas Ahola's test rewrite, which registered a clean-up action from inside a subshell.
- Junio C Hamano judged Srujith A Poojary's submodule commit-abbreviation patch "quite straightforward" and said he will queue it.
- Elia Pinto posted a two-patch series so grep uses `size_t` for search buffers and diff dies with a clear error instead of truncating files of 4GiB or more on 64-bit Windows.
- Elia Pinto posted a two-patch series using `size_t` in archive tar/zip writers and `git apply` so files of 4GiB or more are not written truncated or corrupt on 64-bit Windows.
- Harald Nordgren submitted a three-patch CI series fixing jobs failing on Ubuntu 26.04, including pinning linux32 to ubuntu-24.04 to restore networking.
- Andy Koppe posted v2 of the "root refs vs pseudorefs" documentation series, adding root refs and pseudorefs to ref types in `gitdatamodel(7)`.
- Junio C Hamano acknowledged Jeff King's correction about the fetch test and queued the t5702 env-quoting resilience patch.
- Junio C Hamano gave a positive verdict on Thomas Bachem's merge-ort patch accepting valueless rename settings.
- Elia Pinto posted a patch making `parse_binary_hunk()` reject binary hunks whose declared size exceeds `INT_MAX` as corrupt.
- Pushkar Singh submitted a patch emitting a Trace2 data event when `git maintenance run --auto` skips due to an existing lock.
