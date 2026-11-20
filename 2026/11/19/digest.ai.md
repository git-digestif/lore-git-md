# Git mailing list daily digest for 2026/11/19

## Long threads

### fetch: limited branch fetching via `remote.<name>.fetchRefmap`

*Long thread (root 2026/09/19/14-47-26): 85 replies so far from 6 participants over 62 days; 2 messages that day.*

This series lets a remote in a shallow single-branch clone fetch only the branches that local branches track, adding `remote.<name>.fetchRefmap` as a configured default for `--refmap` and a `git remote add --limited-fetch` option. Today Phillip Wood confirmed that `--refmap` overrides the `.fetchRefmap` config, agreed that a refspec with its own destination should ignore the refmap, and proposed a preliminary patch to ignore `--refmap` when the command-line refspec has a destination. Junio C Hamano questioned the examples, noting that `master` as a fetch refspec is shorthand for `master:` and stores nothing, restated the agreement using `+refs/heads/master:refs/heads/main` so the refmap takes precedence, and asked Phillip Wood to confirm his understanding.

### branch: `--delete-integrated` for squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 60 replies so far from 7 participants over 52 days; 1 message that day.*

This series adds `--delete-integrated` to `git branch`, deleting a branch when some upstream commit since the fork point contains all of its changes, as after a GitHub squash or rebase merge, and naming that commit in the output. Today Jasper Mayone confirmed Junio C Hamano's crash report: when upstream has a gitlink and both sides replace it with different regular files, `git branch --delete-integrated origin/main` dies with "fatal: unable to read blob object ...". He fixed it by recording the base mode in `struct branch_change` and requiring `S_ISREG(change->base_mode)` in `change_state_at()`, which keeps the branch and keeps t3200 passing.

### builtin/rerere: convert to subcommands

*Long thread (root 2026/10/27/10-46-43): 42 replies so far from 6 participants over 24 days; 2 messages that day.*

This 13-patch series modernizes `git-rerere(1)` to use `OPT_SUBCOMMAND()` parsing like other Git commands with subcommands. Today Patrick Steinhardt posted v5, which explicitly documents the `--skip-lock` behavior change, fixes the stale `git rerere diff` documentation, fixes the regression where the synopsis was not printed for unknown subcommands, and NULL-terminates argv. Junio C Hamano reviewed patch 11/13, confirming that rejecting `--[no-]rerere-autoupdate` for subcommands fixes a bug, and noted that the help text for the default "perform" action is now hard to discover but acceptable to leave as-is.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread (root 2026/10/21/17-33-36): 42 replies so far from 8 participants over 30 days; 1 message that day.*

This series makes `git worktree remove` refuse a busy worktree, documents reading `%(worktreepath)` safely, and makes `%(worktreepath)` name the worktree rebasing or bisecting branch. Jasper Mayone had already agreed on November 16 to drop 4/5 and 5/5 and stop at 3/5 after Junio C Hamano preferred keeping "+" next to branch his worktree is bisecting. Today he confirmed to Phillip Wood that 4/5 (`%(currentworktree)`) is dropped and will keep `%(HEAD)` extension in mind.

### http: exponential backoff, configurable status codes, and `post_rpc()` retries

*Long thread (root 2026/10/31/15-57-00): 27 replies so far from 4 participants over 20 days; 3 messages that day.*

This series builds on the HTTP 429 retry support: it fixes an off-by-one retry count and Retry-After Trace2 timing, adds exponential backoff when no Retry-After header is present, adds `http.retryStatusCodes`, retries transient errors in `post_rpc()`, and defaults `http.retryAfter` to 10 seconds. Today Patrick Steinhardt proposed documenting that providers are expected to return 429 when overloaded and that configuring other codes is only a workaround for buggy providers; Ravi Mistry agreed and posted v4, which restricts `post_rpc()` retries to read-only services ("git-upload-pack" and "git-upload-archive"), switches the attempt counter to `size_t`, and honors an explicit "Retry-After: 0" or past HTTP-date.

## The day in brief

Patrick Steinhardt posted v5 of the rerere subcommand conversion and Ravi Mistry posted v4 of the HTTP retry series, both incorporating review feedback. Jasper Mayone confirmed and fixed a crash in `--delete-integrated` and confirmed dropping 4/5 from the worktree series. Dominik Maier sent a batch of hardening patches covering filter-branch, cvsexportcommit, `mv`, ewah bitmaps, and packfile delta headers.

## Notable threads

### filter-branch: validate state-branch map entries
Dominik Maier submitted a patch to `git-filter-branch` that validates each "from:to" line loaded from a `--state-branch` `filter.map`, requiring a non-empty lowercase-hex key and a value of only lowercase hex digits and spaces, so malformed entries with empty keys, slashes, backslashes, or ".." segments die with "Unable to load state from ...:filter.map" instead of writing outside `$tempdir/map`.

### cvsexportcommit: avoid shell in `-W` git calls
Dominik Maier submitted a patch to `git-cvsexportcommit` that chomps `$go_back_to` and converts the `-W` checkout and symbolic-ref calls from single-argument `system()` to list-form `system('git', ...)`, so branch names containing shell metacharacters such as `topic;false` are not interpreted by `/bin/sh`.

### mv: check symlink leading path on force overwrite
Dominik Maier fixed `git mv -f` overwriting an existing target through a symlinked leading path: when the destination beyond the symlink already exists, `lstat` succeeds and `-f` clears `bad`, so the unconditional jump skipped `has_symlink_leading_path()`; the fix jumps only when `bad` is non-NULL, with a regression test in `t/t7001-mv.sh`.

### ewah: validate rlw bounds in `ewah_read_mmap`
Dominik Maier posted a patch validating RLW bounds in `ewah_read_mmap()`: it checks that `buffer_size` is non-zero and fits the mmap length, that the serialized rlw offset points to the final RLW header, and that every literal word count fits the buffer, clamping `next_word()` and `rlwit_init()` as defense in depth, with corrupt-bitmap tests in `t/t5310-pack-bitmaps.sh`.

### packfile: bound delta header size reads
Dominik Maier submitted a patch replacing `get_delta_hdr_size()` with `decode_delta_hdr_size()`, which returns -1 on EOF, unterminated varints, or shift overflow, and bounding `get_size_from_delta()`'s header reads by `stream.total_out` to prevent uninitialized stack reads and out-of-bounds access on corrupt packfiles.

## In brief
- Patrick Steinhardt proposed moving commit-graph generation into `odb_optimize()` to fix the geometric-maintenance commit-graph bug once his pluggable commit-graph series lands.
- Dominik Maier clears the file-static `server_capabilities_v2` in `discover_version()` and `process_capabilities_v2()` so v2 capabilities do not bleed across in-process handshakes.
- Elia Pinto posted v3 of the missing-upstream status message rewording; Junio C Hamano raised an unresolved objection about `push.default=upstream` with a different push remote.
- brian m. carlson posted v3 of the hash algorithm detection fix, which Patrick Steinhardt is happy with; Junio C Hamano confirmed the command-line versus config clarification.
- Junio C Hamano objected that the "structure" subcommand name is misleading for the object statistics reported by the experimental `git repo`.
- Adrian Ratiu confirmed a linked-worktree submodule breakage via the absorb path and agreed to move the `is_worktree` check in v2.
- Patrick Steinhardt confirmed the reftable dry-run expire writes then unlinks a table, making patch 1/4 a performance-only improvement.
- Kaartic Sivaraam posted v2 serializing EXPENSIVE tests with a shared lock to stop parallel memory-hungry tests from OOM-killing CI runners.
- Tanga Joris ran t4300 with `GIT_TEST_LONG=1` on Windows and confirmed both big merge tests pass with the series.
- Dirk Gouders and Andy Koppe discussed dropping the `display_mode_esc_sequence_len()` wrapper in favor of the `_counted` variant. *Long thread (root 2026/11/16/18-56-59): 18 replies so far from 3 participants over 4 days; 10 messages that day.*
- Junio C Hamano argued Git should be strict in detecting malformed "diff --git" lines in input rather than silently ignoring them.
- Adrian Ratiu posted v2 of the compatObjectFormat fixes, rejecting `compatObjectFormat == objectFormat` and copying unknown commit headers unchanged.
