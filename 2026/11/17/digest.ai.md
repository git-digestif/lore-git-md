# Git mailing list daily digest for 2026/11/17

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 83 replies so far from 6 participants over 60 days; 5 messages that day.*

This series changes `git fetch` behavior in shallow single-branch clones, where `git remote add` sets up a default `+refs/heads/*:refs/remotes/<name>/*` refspec and the next fetch downloads every branch of the remote. It adds `remote.<name>.fetchRefmap` as a configured default for `--refmap`, makes a remote with a fetchRefmap and no fetch refspec fetch the branches local branches track, adds `git remote add --limited-fetch`, and makes that the default in shallow repositories. Earlier, Phillip Wood objected that patch 3 used one variable for both "what to fetch" and "where to store it", and proposed dropping it in favor of combining `.fetchRefmap` and `.fetch`; Matt Hunter identified the blocked topic as `ch/remote-defer-follow-remote-head-validation`. Today Phillip Wood argued that when `.fetchRefmap` is configured and an explicit `src:dst` is passed on the command line, the refmap should be ignored since users expect `src:dst` to override config, while Junio C Hamano countered that `src:dst` overrides `.fetch` but not `.fetchRefmap`, and said he is fine with a refspec carrying a destination simply ignoring the refmap, requiring only a clear one-to-one default/override relationship.

### branch: let --delete-merged find squash merged branches

*Long thread (root 2026/09/29/07-30-30): 58 replies so far from 7 participants over 50 days; 1 message that day.*

This series adds `--delete-integrated` to `git branch`, which deletes a branch when some upstream commit since the fork point contains all of its changes even if the tip is unreachable, as after a GitHub squash or rebase merge, printing `Deleted branch <name> (was <sha>, landed as <sha>)`. Patch 4/4 extends it to branches squash-merged onto files upstream also changed, by merging the branch's file version into the upstream version with `ll_merge` and treating the change as landed when the upstream version is unchanged; that reads file contents and may download blobs in a partial clone. Earlier, Jasper Mayone confirmed v7 addressed his points on index reading, attribute sourcing, and keep-tests, and added a `Tested-by` trailer. Today Harald Nordgren posted v8, whose only change over v7 is reworded commit messages in patch 4/4.

### builtin/rerere: convert to use subcommands

*Long thread (root 2026/10/27/10-46-43): 37 replies so far from 6 participants over 22 days; 5 messages that day.*

This series modernizes `git-rerere(1)`'s custom subcommand parsing so it behaves like other Git commands with subcommands, turning `forget`, `clear`, `status`, `remaining`, `diff`, and `gc` into proper subcommands via `OPT_SUBCOMMAND()`. Earlier, Junio C Hamano reported a backwards-incompatible change: options before the subcommand are no longer accepted, so `git rerere --skip-locked gc` must be spelled `git rerere gc --skip-locked`, and decided to accept breaking it. Today he flagged that `git rerere unknown` now reports "too many arguments" instead of listing subcommands, calling it a regression "more easily spotted by folks", asked that the `gc` commit message describe the deliberate UI regression, questioned whether `args.nr` includes the terminating NULL in the fallback `cmd_rerere_perform()` call, and noted the `diff` documentation still claims extra arguments are passed to the system `diff` command although the command rejects them.

## The day in brief

Two bug reports landed on combine-diff and push: octopus merges with 33+ parents mark wrong deletion columns or no hunks due to an `int` shift in `coalesce_lines()`, and a remote group listing a remote of its own name makes `git push` spawn thousands of nested processes. Elia Pinto's v2 series hardens loose-object size handling against truncation on 64-bit Windows, and Christian Couder posted a series allowing merge results over 2 GiB there by widening `unsigned long` parameters to `size_t`.

## Notable threads

### loose object size checks and a few size truncations

Elia Pinto posted v2 of a five-patch series fixing size truncations on LLP64 platforms (64-bit Windows, where `long` is 32 bits but `size_t` is 64): a check in `unpack_loose_rest()` that the inflated stream matches the header-claimed size, `size_t` for loose object size parameters, a size check in `read_istream_loose()`, `size_t` for `batch_write()`'s length, and `size_t` concatenated-notes sizing to avoid a heap overflow when notes total 4 GiB or more. Junio C Hamano reviewed each patch, approving 1/5, 4/5, and 5/5, asking about `st_add()` use in 2/5, and objecting to 3/5's unconditional `error()` as inconsistent with the helper's existing silent return.

### combine-diff: wrong deletion columns at 33 parents, no hunks at 64

Sami Jawhar reported that `git diff-tree --cc -U0` on octopus merges marks wrong deletion columns at 33 parents (columns 1 and 2 for a line present in parents 2 and 33) and 63 parents, and produces no hunks at 64 parents, identifying `1<<parent` in `coalesce_lines()` at combine-diff.c:249 as shifting an `int` instead of using `1UL<<parent`, which is undefined behavior when shifting by 64.

### push: remote group self-reference causes infinite recursion

wheakerd reported that `git push <name>` recurses when a remote group lists a remote of its own name: with `remotes.origin` set to `origin member`, the command spawned thousands of nested `git push` processes (5,945 "Pushing to origin" lines and up to 5,898 concurrent processes in 12 seconds), pushed to the other member but never to `origin` itself, and exited with status 1. The recursion comes from `push_multiple()` in builtin/push.c running `git push <member>` per group member while `add_remote_or_group()` in remote.c resolves the name as a group first; wheakerd cited an older fetch-side report of the same cycle with a sketched but never-landed fix.

### submodule: linked-worktree submodule hits BUG() in validate_submodule_legacy_git_dir()

Adrian Ratiu traced the `BUG()` hit when switching branches with `submodule.recurse=true` in a superproject worktree whose submodule is a linked worktree to commit 9cf8547320, which made `submodule_move_head()` validate whatever path the submodule's gitfile points to; since a path read from disk can be anything, it should never hit a `BUG()`. He noted the reproducer needs a second worktree because the plain gitdir happens to end in the submodule name, admitted the hint recommending the already-enabled extension was his mistake, and reported a local fix that makes the reproducer switch cleanly in both modes, to be sent in the next few days.

### Allow merges with results over 2 GiB on 64-bit Windows

Christian Couder posted a three-patch series converting `unsigned long` parameters to `size_t` in `write_buf_to_worktree()` (notes-merge.c), `odb_write_object()` and related ODB write paths, then removing the `LONG_MAX` check in `xdl_do_merge()` in xdiff/xmerge.c, so merge results over 2 GiB no longer fail on 64-bit Windows; it also fixes silent truncation of notes of 4 GiB or more on LLP64 platforms.

## In brief

- Elia Pinto posted v2 of "widen a few sizes and counts that are 32 bits everywhere", adding a new 4/4 switching diffstat totals and `scale_linear()` to `uintmax_t`; Junio C Hamano questioned what happens when the wider accumulator itself overflows and noted the `graph_width` capping change is more than a type widening.
- Dirk Gouders is preparing a v2 of the `utf8_strnwidth()` bounds fix after Junio C Hamano argued the real defect is the missing guard before `utf8_width()` and that a post-skip check is too late for mmapped regions.
- Adrian Ratiu will rework the compatObjectFormat series in v2, including fixing `"tree "` and empty `mergetag` header classification in object-file-convert by changing three checks to `<=`.
- Tanga Joris posted v3 of "apply: reject corrupt diff --git headers", fixing a `def_name` leak and naming the offending `---`/`+++` line in errors.
- tsukasa inoue posted v2 fixing the reflog of a symref whose target is updated in the same transaction, adding `ref_transaction_resolve_new_oid()` used by both refs backends.
- Ben Knoble and Junio C Hamano agreed a CLI `--verbose` option for `git history reword` is still wanted alongside any config default.
- Justin Tobler answered review questions on eagerly creating the ODB transaction temporary directory and promised message and error-text fixes in the next version.
- Junio C Hamano posted v2 adding `display_mode_esc_sequence_len_counted()` while keeping the original NUL-terminated API; Andy Koppe suggested accepting colon-separated SGR subparameters.
- Harald Nordgren reviewed the status missing-upstream rewording as "v2 looks good to me"; Junio C Hamano noted such one-line verdicts do not show how deeply the code was read.
- Andy Koppe posted v3 of the pretty color line-break fix, a clarity refactor with the algorithm unchanged.
- Elia Pinto pinged the "tree-walk: use size_t for tree sizes" series, asking whether widening the types or rejecting oversized trees is the right approach.
- Kaartic Sivaraam fixed fast-import to pass `export-marks` rather than `export-marks=<path>` to `check_unsafe_feature()`, so the forbidden-feature error no longer leaks the path.
