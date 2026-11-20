# Git mailing list daily digest for 2026/11/17

## Long threads

### fetch: limited branch fetching for shallow clones

*Long thread: 83 replies so far from 6 participants over 60 days; 5 messages that day.*

This series lets a remote fetch only the branches that local branches track, via `remote.<name>.fetchRefmap` and `git remote add --limited-fetch`, so that adding a remote to a shallow single-branch clone does not download every branch. Today Phillip Wood and Junio C Hamano debated whether a command-line `src:dst` refspec should also ignore `remote.<name>.fetchRefmap`. Phillip Wood argued that users expect a command-line `src:dst` to override config rather than add to it, while Junio C Hamano countered that `src:dst` overrides `.fetch` while `.fetchRefmap` supplies defaults for `--refmap`; he said he is completely fine with a refspec carrying a destination simply ignoring the refmap, so the refmap applies only to entries lacking a destination.

### branch: `--delete-integrated` for squash-merged branches

*Long thread: 58 replies so far from 7 participants over 50 days; 1 message that day.*

This series adds `--delete-integrated` to `git branch`, deleting a branch when some upstream commit since the fork point contains all of its changes, as after a GitHub squash or rebase merge, and naming that commit in the output. Today Harald Nordgren posted v8, whose only change from v7 is reworded commit messages in patch 4/4, the patch that also catches branches squash-merged onto files upstream had changed by merging the branch's version with `ll_merge`.

### builtin/rerere: convert to subcommands

*Long thread: 37 replies so far from 6 participants over 22 days; 5 messages that day.*

Patrick Steinhardt's series replaces `git-rerere`'s custom subcommand parsing with `OPT_SUBCOMMAND()` so it behaves like other Git commands with subcommands. Today Junio C Hamano accepted the backwards-incompatible option placement (`git rerere --skip-locked gc` now errors), asked that the `gc` commit message describe the deliberate UI regression, questioned whether `args.nr` includes the terminating NULL in the `OPT_SUBCOMMAND()` fallback, and noted the `diff` documentation still claims extra arguments are passed to the system `diff` although the subcommand rejects them.

## The day in brief
wheakerd reported that `git push <name>` recurses into thousands of nested processes when a remote group lists a remote of its own name. Sami Jawhar found `combine-diff` mislabels deletion columns beyond 32 parents because `1<<parent` shifts an int. Elia Pinto's v2 series hardens loose-object size handling against truncation on 64-bit Windows.

## Notable threads

### push: self-named remote group recurses
wheakerd reported that with `remotes.origin` set to `"origin member"`, `git push origin` spawns thousands of nested `git push` processes, pushes to the other member but never to `origin`, and exits with status 1. He traced the recursion to `push_multiple()` in `builtin/push.c` running a child per group member while `add_remote_or_group()` in `remote.c` resolves the name as a group first, and noted `git push <group>` is new in 2.55.0 with no test covering a self-named group.

### loose object size checks and size truncations
Elia Pinto posted v2 of five patches: size checks in `unpack_loose_rest()` and `read_istream_loose()`, widening loose-object size parameters to `size_t`, a `size_t` length in `batch_write()`, and `size_t` sizing of concatenated notes to avoid heap overflow on 64-bit Windows. Junio C Hamano reviewed each patch, approving 1/5, 2/5, 4/5 and 5/5, and objecting that 3/5's unconditional `error()` is inconsistent with the silent `return -1` below it.

### combine-diff: wrong columns at 33 parents, no hunks at 64
Sami Jawhar reported that `git diff-tree --cc -U0` on octopus merges marks wrong deletion columns at 33 parents (columns 1 and 2 for a line in parents 2 and 33) and 63 parents, and produces no hunks at 64 parents. He identified `1<<parent` in `coalesce_lines()` at `combine-diff.c:249` as shifting an int rather than using `1UL<<parent`, causing undefined behavior when shifting by 64.

## In brief
- Junio C Hamano added `display_mode_esc_sequence_len_counted()` for counted buffers while keeping the NUL-terminated original; Dirk Gouders proposed an early return and unit tests, and Andy Koppe suggested accepting colon-separated SGR subparameters.
- Adrian Ratiu traced a `BUG()` in `validate_submodule_legacy_git_dir()` for linked-worktree submodules to commit 9cf8547320 and said he will send a fix.
- Elia Pinto posted v2 widening sizes and counts to `size_t`/`uintmax_t` across csum-file, the diff rewrite path, and diffstat totals; Junio C Hamano questioned the Rust `hashwrite()` u32 contract and the `spanhash` `cnt` member.
- Christian Couder posted a series converting `unsigned long` to `size_t` in notes-merge and the odb write paths, dropping the `LONG_MAX` check in `xdl_do_merge()`.
- Tanga Joris posted v3 rejecting corrupt `diff --git` headers in `git apply`, fixing a `def_name` leak and naming the offending `---`/`+++` line in errors.
- tsukasa inoue posted v2 fixing the reflog of a symref whose target is updated in the same transaction, via `ref_transaction_resolve_new_oid()`.
- Dirk Gouders and Junio C Hamano discussed bounds handling in `utf8_strnwidth()`; Dirk Gouders is preparing a v2.
- Adrian Ratiu reported two new `compatObjectFormat` findings -- a `"tree "` header with trailing space is copied, and an empty `mergetag` line keeps an unconverted object ID -- and plans fixes in v2.
