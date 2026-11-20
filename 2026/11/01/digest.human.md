# Git mailing list daily digest for 2026/11/01

## Long threads

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread: 53 replies so far from 6 participants over 34 days; 1 message that day.*

Harald Nordgren posted v6 of the series adding `--delete-integrated` to `git branch` in `builtin/branch.c`. The series keeps `--delete-merged` deleting only branches whose tips are reachable from their upstream, while the new `--delete-integrated <pattern>` also deletes branches whose changes landed upstream through GitHub-style squash or rebase merges, printing `Deleted branch topic (was 1a2b3c4, landed as 9f8e7d6)`. V6 reads the index before file merges so `.gitattributes` present only in the index are honored, and renames the `diff_options` variables from `opt` to `diffopt`. Patch 4/4 extends the check to branches merged onto files that upstream also changed, by merging the branch's version into the upstream version and treating the change as landed when the upstream version is unchanged; this reads file contents and may download blobs in a partial clone. Earlier, Jasper Mayone had tested v4 on generated repositories and reported the usage-line and error-message issues that v5 addressed.

### More `path.*` keys for `git repo info`

*Long thread: 39 replies so far from 5 participants over 109 days; 2 messages that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info`: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup`, with absolute and relative suffix formatting, plus a fix so `get_superproject_working_tree()` takes a repository parameter instead of relying on the current working directory. Today Junio C Hamano stated that the v7 series has seen no interest or review for a full month and that he will discard the topic from his tree unless volunteers give it a final review in a few weeks; Justin Tobler said he will take a look.

## The day in brief

Harald Nordgren posted v6 of the `git branch --delete-integrated` series, Farid Zakaria posted a two-patch fix for credential-cache daemon startup races, and Junio C Hamano reviewed the sequencer's refusal to pick or revert shallow boundary commits.

## Notable threads

### Moving the Rust gitcore crate into a `rust/` subdirectory

Mike Hommey's patch moves all Rust-specific files (`Cargo.toml`, `build.rs`, `src/`) into a dedicated `rust/` subdirectory, because a top-level `Cargo.toml` implies a bare `cargo build` works and blocks including the Git source as a submodule of a Rust project. Today Junio C Hamano stated that Hommey's continued interest was not a strong enough reason to keep the topic given his effort to limit topics in flight, and wrote "... down."

### credential-cache daemon startup races

Farid Zakaria posted a two-patch series fixing races in `builtin/credential-cache--daemon.c` when two `git credential-cache store` commands run at once and each starts a daemon: the second daemon unlinks the first's socket and binds its own, so the first's stored credential is lost and its tempfile handler later removes the second's socket. Patch 1 ignores `EEXIST` from `mkdir()` and verifies the directory by stat-ing it; patch 2 uses `unix_ss_create()` so a second daemon prints "ok" and exits while its client talks to the running daemon, registering the socket for removal only after binding succeeds.

### Refusing to pick or revert shallow boundary commits

Nakul V S's patch makes the sequencer refuse to revert or cherry-pick shallow boundary commits in a shallow repository, where such a commit looks like a root commit and reverting it empties the working tree; `--allow-shallow-boundary` overrides the check for revert and cherry-pick, while rebase always refuses and suggests `git fetch --unshallow`. Today Junio C Hamano questioned whether rebase onto master was necessary, asked why `git_config_bool_or_int()` is used for `options.allow-shallow-boundary`, and noted that Nakul's `is_shallow_boundary_commit()` duplicates one from Harald's stagnant topic, suggesting reuse.

## In brief

- Junio C Hamano reviewed Harald Nordgren's v2 resolving "simple" `@{push}` when pushing to another remote, said it looks good, and will queue it.
- Junio C Hamano judged Srujith A Poojary's patch using the submodule repository to abbreviate commit IDs in diff headers "quite straightforward" and will queue it.
- Junio C Hamano decided to pick up Linus Torvalds' `git describe` fix for miscounted distance with skewed commit dates after Tuomas Ahola conceded his test rewrite failed under Bash.
- Matt Hunter suggested `get_follow_remote_head()` could accept the transport and fetch_config parameters to resolve both `remote.<name>.followRemoteHEAD` and `fetch.followRemoteHEAD` without a new helper.
- Harald Nordgren posted v5 of the imap-send series accepting From, Date and Subject headers in any order and case-insensitively, ending the header block at an empty line in CRLF mboxes as well.
- Elia Pinto posted patches making grep use `size_t` and diff die with a clear error for files of 4GiB or more on platforms where `long` is 32 bits.
- Elia Pinto posted patches using `size_t` in archive and apply so files of 4GiB or more are not written truncated on 64-bit Windows.
- Andy Koppe posted v2 of the root-refs-vs-pseudorefs documentation series, adding both to the ref types in `gitdatamodel(7)` and itemized per-worktree ref lists.
