# Git mailing list daily digest for 2026/11/19

## Long threads

### fetch: limited branch fetching via remote.<name>.fetchRefmap

*Long thread: 85 replies so far from 6 participants over 62 days; 2 messages that day.*

In a shallow single-branch clone, `git remote add` installs the default refspec, so the next fetch downloads every branch of the new remote. Harald Nordgren's series adds `remote.<name>.fetchRefmap`, makes a refmap-only remote fetch the branches local branches track, and adds `git remote add --limited-fetch`. Today Phillip Wood confirmed that `--refmap` overrides the `.fetchRefmap` config and agreed that a refspec with its own destination should ignore the refmap; Junio C Hamano restated the agreement using `+refs/heads/master:refs/heads/main` so the refmap takes precedence, and asked Phillip to confirm his understanding.

### branch: --delete-integrated for squash-merged branches

*Long thread: 60 replies so far from 7 participants over 52 days; 1 message that day.*

Harald Nordgren's series adds `--delete-integrated` to `git branch`, deleting a branch when an upstream commit since the fork point contains all its changes, as after a squash or rebase merge. Today Jasper Mayone confirmed Junio C Hamano's reading: when a gitlink becomes different regular files on both sides, the command dies with "fatal: unable to read blob object" and deletes nothing, not a segfault. He fixed it by recording the base mode and requiring `S_ISREG(change->base_mode)` in `change_state_at()`, keeping the branch and t3200 passing.

### builtin/rerere: convert to subcommands

*Long thread: 42 replies so far from 6 participants over 24 days; 2 messages that day.*

Patrick Steinhardt's series modernizes `git-rerere(1)` to parse subcommands with `OPT_SUBCOMMAND()`. Today he posted v5, which documents the backwards-incompatible `--skip-locked` placement, fixes the stale `git rerere diff` documentation and the missing-synopsis regression for unknown subcommands, and NULL-terminates argv. Junio C Hamano reviewed patch 11/13, confirming that rejecting `--[no-]rerere-autoupdate` for subcommands fixes a bug, and accepted that the default "perform" help text is now hard to discover.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread: 42 replies so far from 8 participants over 30 days; 1 message that day.*

Jasper Mayone's series makes `git worktree remove` refuse a worktree mid-operation without `--force` and makes `%(worktreepath)` name the worktree rebasing or bisecting a branch. Today he dropped patch 4/5 (`%(currentworktree)`) and agreed to stop the series at 3/5, since Junio C Hamano wanted the "+" marker kept on the branch his own worktree is bisecting and 4/5 existed only to support 5/5.

### http: exponential backoff and retry configuration

*Long thread: 27 replies so far from 4 participants over 20 days; 3 messages that day.*

Ravi Mistry's series fixes the HTTP 429 retry count and Trace2 timing, adds exponential backoff, `http.retryStatusCodes`, `post_rpc()` retries, and a 10-second `http.retryAfter` default. Today Patrick Steinhardt proposed documenting that providers are expected to return 429 when overloaded, with other codes framed as a workaround for buggy providers; Ravi Mistry agreed and posted v4, which restricts `post_rpc()` retries to read-only services so pushes are not retried and honors an explicit "Retry-After: 0".

## The day in brief

Ravi Mistry posted v4 of the HTTP retry series, restricting retries to read-only services. Jasper Mayone fixed `--delete-integrated` handling of gitlink-to-file changes and dropped `%(currentworktree)` from the worktree series. Dominik Maier sent hardening patches for packfile delta headers, EWAH bitmaps, and filter-branch state loading.

## Notable threads

### packfile: bound delta header size reads
Dominik Maier replaced `get_delta_hdr_size()` with `decode_delta_hdr_size()`, returning -1 on EOF, unterminated varints, or `size_t` shift overflow, and bounded `get_size_from_delta()`'s reads by `stream.total_out`; the old code read uninitialized stack bytes or past the 20-byte `delta_head[]` on corrupt packfiles. `patch_delta()` handles the error return, with regression tests in `t/t5300-pack-object.sh`.

### ewah: validate rlw bounds in ewah_read_mmap
Dominik Maier added validation of the serialized word count, RLW offset, and per-header literal word counts in `ewah_read_mmap()`, which previously trusted them and could leak memory or read out of bounds. Corrupt bitmaps are now rejected, `next_word()` and `rlwit_init()` are clamped as defense in depth, and `t/t5310-pack-bitmaps.sh` covers the corrupt cases.

### filter-branch: check hex oids in state-branch map
Dominik Maier's patch requires each "from:to" line from a `--state-branch` `filter.map` to have a non-empty lowercase-hex source and a hex-or-space target before writing `../map/$from_commit`, so entries with slashes, backslashes, or ".." segments die with "Unable to load state from ...:filter.map" instead of writing outside `$tempdir/map`; `t/t7003-filter-branch.sh` covers traversal and backslash entries.

## In brief
- Elia Pinto posted v3 of the loose-object size series: size mismatches get their own error message, loose-object sizes use `size_t` end to end, and `git cat-file --batch` no longer writes past 2-4 GiB objects.
- Dominik Maier cleared the file-static `server_capabilities_v2` vector between handshakes so v2 capabilities no longer bleed across remotes, fixing "mismatched object format" errors in `git remote show r1 r2`.
- Dominik Maier fixed `git mv -f` so a forced overwrite still runs the symlinked-leading-path check when the target beyond the symlink exists.
- Elia Pinto posted v3 rewording the missing-upstream `git status` message; Junio C Hamano flagged a remaining case where a different pushRemote still gets a `git push` suggestion.
- **Patrick Steinhardt on geometric maintenance fix**: He proposed moving commit-graph generation into `odb_optimize()` to fix the bug, once his pluggable series lands.
