## Long threads

### fetch: limited branch fetching for shallow clones

*Long thread: 86 replies so far from 7 participants over 65 days.*

Harald Nordgren's series evolved through v15 this week, adding `remote.<name>.fetchUpstreams` to explicitly control whether refspec-less fetches retrieve upstreams of local branches, so a `fetchRefmap` alone no longer changes fetch behavior. `--refmap` now works without command-line refspecs, and a refspec naming its own destination is stored only there rather than also being mapped through the refmap. `git remote add --limited-fetch` sets the refmap and upstreams config instead of a wildcard refspec, becoming the default in shallow repositories unless `-t` or `--mirror` is given. Phillip Wood and Junio C Hamano debated whether command-line `src:dst` refspecs should ignore `fetchRefmap`, settling on the rule that a refspec with its own destination ignores the refmap while the refmap supplies defaults for entries lacking one.

### branch: `--delete-integrated` for squash-merged branches

*Long thread: 70 replies so far from 7 participants over 55 days.*

Harald Nordgren posted v9 and v10 this week, fixing a crash Junio C Hamano identified where a gitlink path replaced by different regular files on both branch and upstream caused `read_mmblob()` to die with "unable to read blob object". The fix records the base mode and requires `S_ISREG(change->base_mode)`, keeping such branches. v10 also made the file merge a plain text merge ignoring `.gitattributes` drivers, detected branches that replaced a file or submodule with a directory, made `--dry-run` work with `-d` and `-D`, unified "Deleted"/"Would delete" messages, renamed the opt-out helper to `opted_out_of_delete_merged()`, and added tests for `--dry-run`, `--quiet`, and `branch.<name>.deleteMerged=false`. Jasper Mayone verified v10 builds cleanly under `DEVELOPER=1`, passes all 215 t3200 tests, and added a `Tested-by` tag.

### `git repo info` path keys (GSoC series v9)

*Long thread: 64 replies so far from 6 participants over 124 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.workingtree.*`, `path.superprojectworkingtree.*`, `path.index.*` and `path.prefix` -- and refactors `get_superproject_working_tree()` to take a `struct repository *`. This week he noted that the easiest approach for the planned follow-up series consolidating `repo` and `rev-parse` logic is to combine and optimize the per-test repository setup.

### New detached HEAD guide

*Long thread: 46 replies so far from 5 participants over 41 days.*

Julia Evans posted v7 of her patch adding a `git help detached-head` guide, dual-purpose with the DETACHED HEAD section of `git-checkout.adoc`, adding a sentence about making experimental commits without being on a branch and a clearer commit message. Junio C Hamano asked for a fresh-eyes final pass before merging it down to `next`.

### builtin/rerere: convert to subcommands

*Long thread: 44 replies so far from 6 participants over 25 days.*

Patrick Steinhardt posted v5 of his 13-patch series converting `git rerere` to `OPT_SUBCOMMAND()` parsing, documenting the backwards-incompatible `--skip-locked` placement, fixing stale `git rerere diff` documentation, restoring the synopsis for unknown subcommands, and NULL-terminating argv. Junio C Hamano reviewed patch 11/13, confirming that rejecting `--[no-]rerere-autoupdate` for subcommands fixes a bug, and accepted that the default "perform" help text is now hard to discover. Patrick also agreed that flags should be refused once a non-default action is requested, though he would not break the existing interface now.

### `credentialStore.lockTimeout` config name

*Long thread: 43 replies so far from 5 participants over 76 days.*

Okhuomon Ajayi resent v3 of her Outreachy patch adding `credentialStore.lockTimeout` as the preferred name for credential lock retries, matching other millisecond file-lock settings that omit the unit, with `lockTimeoutMS` kept as a deprecated synonym. Junio C Hamano reviewed it, said it looks good, and marked the topic ready for `next`.

### Worktrees rebasing or bisecting a branch

*Long thread: 42 replies so far from 8 participants over 30 days.*

Jasper Mayone's series makes `git worktree remove` refuse a worktree mid-operation and names the worktree rebasing or bisecting a branch via `%(worktreepath)`. This week he answered Junio C Hamano's question that the new `check_no_operation_in_progress()` and the existing checked-out-branch check answer different questions and should stay separate, with a shared helper proposed as a separate series. He dropped patches 4/5, stopping the series at 3/5, since Junio wanted the "+" marker kept on the branch his own worktree is bisecting.

### Bounds fixes in utf8.c

*Long thread: 29 replies so far from 3 participants over 7 days.*

Dirk Gouders posted v5 of his series fixing boundary handling in `utf8_strnwidth()` and `display_mode_esc_sequence_len()`. After Andy Koppe called v4 unfocused, v5 drops the `display_mode_esc_sequence_len_all()` patch, uses an array index in the `_counted` variant so it never computes `s + len`, and passes `SIZE_MAX` in the NUL-terminated wrapper. Andy Koppe suggested callers could pass `SIZE_MAX` to sidestep length checks, which Junio C Hamano affirmed, though Koppe corrected that `end = s + len` with `SIZE_MAX` would be undefined behavior, so callers must pass a valid length.

## The period in brief

The week of 2026/11/16 through 2026/11/22 saw seven active days of heavy traffic, with several long-running series reaching significant milestones. Harald Nordgren's `--delete-integrated` series reached v10 with a fix for the submodule-to-file crash, while his limited-fetch series for shallow clones reached v15 with a redesigned `fetchUpstreams` configuration. Patrick Steinhardt's `rerere` subcommand conversion reached v5, and Dirk Gouders's utf8 bounds fixes reached v5 after several iterations.

## Key developments

### HTTP retry series restricts retries to read-only services

Ravi Mistry's series fixing HTTP 429 handling reached v4 this week after Patrick Steinhardt objected that retrying `post_rpc()` is unsafe for non-idempotent POSTs like `git-receive-pack` pushes. v4 restricts `post_rpc()` retries to read-only services so pushes are not retried, honors an explicit "Retry-After: 0", and makes `attempt` a `size_t`. Patrick also proposed documenting that providers are expected to return 429 when overloaded, with other codes framed as a workaround for buggy providers.

### combine-diff fixed for merges with more than 32 parents

Aniket Gupta posted a two-patch series fixing the bug Sami Jawhar reported where combined diffs of merges with more than 32 parents put lines in wrong columns and showed no hunks at 64 parents. Patch 1 changes the parent-bit shift in `coalesce_lines()` from `1<<parent` to `1UL<<parent`, fixing undefined behavior for the 33rd parent and later. Patch 2 makes combine-diff report an error and skip the patch for merges with more parents than the masks can hold instead of printing wrong output, without dying so that `git log --cc` is not cut short.

### refs/reftable: reflog expiration performance

Patrick Steinhardt posted a four-patch series fixing the auto-maintenance "reflog-expire" condition, which treated every reflog entry as unreachable and spawned `git reflog expire` on every run, blocking the foreground for roughly 800ms. The series fixes the condition and skips writing new reftables on dry-run or no-op expires, with `git maintenance run --auto` dropping from 853.2ms to 18.6ms. Karthik Nayak, Toon Claes and Junio C Hamano reviewed the series.

### connect: promisor-remote reply missing from fetch

Christian Couder posted v2 0/8 of his series fixing a bug where the client does not send a `promisor-remote=...` reply with protocol v2 `fetch` commands, which fails with HTTP servers that handle each request separately. The v2 refactoring grew from one patch to six, making `write_command_and_capabilities()` the only function used across `ls-refs`, `bundle-uri`, and `fetch`, with final patches ensuring fetch sends the reply and documenting it in `gitprotocol-v2.adoc`.

### strbuf_getwholeline() must clear the buffer on EOF

Junio C Hamano posted a six-patch series addressing the long-standing API bug where `strbuf_getwholeline()` returns EOF without clearing the output buffer. Patches demonstrate resulting bugs in shortlog (stale buffer reuse), `git clean -i` ask mode (deleting past EOF), and fast-import (re-parsing a heredoc delimiter at EOF), then fix the core function and convert the `test_expect_failure` demonstrations to `test_expect_success`. He also fixed `mailinfo` leaking a stale last header into the commit log on header-only input.

### Submodule URLs with '^/' for server-root-relative paths

Stanislav Aleksandrov's RFC proposes resolving submodule URLs starting with `^/` against the superproject remote's root, keeping protocol, user, host and port. Junio C Hamano said he is inclined to perform the proposed squash and mark the topic ready for `next`, citing Jacob's review as a strong endorsement, and asked heavy submodule users to object if the new behavior is problematic.

### T7527 fsmonitor--daemon failures bisected

Dirk Gouders reported that T7527 on `origin/master` fails 2 of 70 tests, with a core file showing `git fsmonitor--daemon` dying on `BUG("double remove of watch for '%s'", w->dir)` in `compat/fsmonitor/fsm-listen-linux.c`. A bisect identified commit ce48de8b2c, the Linux filesystem change listener, as the first bad commit.

## In brief

- **push: self-named remote group recurses** -- wheakerd reported that with `remotes.origin` set to `"origin member"`, `git push origin` spawns thousands of nested `git push` processes, pushes to the other member but never to `origin`, and exits with status 1; the recursion traces to `push_multiple()` running a child per group member while `add_remote_or_group()` resolves the name as a group first.
- **loose object size checks** -- Elia Pinto posted v3 of her series widening loose-object sizes to `size_t` end to end, giving size mismatches their own error message, and ensuring `git cat-file --batch` no longer writes past 2-4 GiB objects.
- **apply: reject corrupt "diff --git" headers** -- Tanga Joris's v3 series makes `git apply --check` report corrupt headers as errors; Junio C Hamano argued the 1/3 test only passes because the bogus line names the same file as the next line, and Tanga confirmed a different name yields "inconsistent old filename".
- **submodule: linked worktrees in recursive checkout** -- Adrian Ratiu posted a three-patch series fixing recursive-checkout failures when a submodule is a linked worktree, replacing the BUG() with an error and accepting a linked-worktree git dir.
- **packfile: bound delta header size reads** -- Dominik Maier replaced `get_delta_hdr_size()` with `decode_delta_hdr_size()`, returning -1 on EOF, unterminated varints, or `size_t` shift overflow, with regression tests in `t/t5300-pack-object.sh`.
- **ewah: validate rlw bounds** -- Dominik Maier added validation of serialized word count, RLW offset, and per-header literal word counts in `ewah_read_mmap()`, rejecting corrupt bitmaps instead of leaking memory or reading out of bounds.
- **filter-branch: check hex oids in state-branch map** -- Dominik Maier's patch requires each "from:to" line from a `--state-branch` `filter.map` to have a non-empty lowercase-hex source, so entries with slashes or ".." segments die instead of writing outside `$tempdir/map`.
- **ls-files: reuse untracked cache** -- Tamir Duberstein updated the series to v4, taking the index lock after `fill_directory()` so optional writes detect concurrent index changes even with `index.skipHash` enabled.
- **status, push: handle rebased push branch** -- Aniket Gupta reported that `--cherry-mark` makes `git status` slower on diverged branches because every commit on both sides needs a patch id, and suggested counting without it first.
- **fetch: --prune-tags clobber changed tags** -- Orgad Shaneh posted v2 restoring the documented behavior of `--prune-tags` force-updating tags that changed on the remote, adding "+" to the implied refspec instead of treating it like `--force`.
- **submodule: worktree-specific configuration** -- Yoichi NAKAYAMA posted v2 making submodule settings worktree-specific when `extensions.worktreeConfig` is enabled, keeping `submodule.<name>.gitdir` relative for linked worktrees.
- **fast-import: inline data at EOF** -- Wonseok Oh revised his fix so an inline `M` at EOF dies with "expected 'data n' command" instead of looping or storing an uninitialized object ID.
- **reflog expire each reflog once** -- Laveesh Bansal posted v3 making `git reflog expire` expire each reflog only once, skipping already-processed reflogs with a strset.
- **cherry-pick onto unborn branch** -- Wonseok Oh submitted a patch clearing the saved HEAD OID to the null OID when cherry-picking onto an unborn branch, so `--abort` reports "cannot abort from a branch yet to be born".
- **docs: http.extraheader not for credentials** -- brian m. carlsson posted a patch clarifying that `http.extraheader` is not for credentials, adding warnings to `http.adoc` and `gitfaq.adoc`.
- **difftool segfault on invalid paths** -- Ophir Bleicher proposed a fix for a `git difftool --dir-diff` segfault on invalid paths like `.git` by checking `make_transient_cache_entry()`.
- **fsmonitor: skip prep_exclude for valid cached dirs** -- Stepan Koltsov skips `prep_exclude()` in `valid_cached_dir()` when fsmonitor is trusted, cutting `git status --untracked-files=all` on a warm cache from about 310 ms to 65 ms.
- **git archive --no-add-file filtering** -- René Scharfe made `--no-add-file` and `--no-add-virtual-file` remove only their own entries via a new `filter_string_list_func()`, instead of clearing the shared list.
- **Optional --follow rename tracking** -- Miklos Vajna's patch making `--follow` rename tracking in non-linear history optional via `--follow=merges` was judged "looks good to me" by Patrick Steinhardt and marked ready for `next` by Junio C Hamano.
- **Faster object counting in git repo structure** -- Patrick Steinhardt's two-patch series making `builtin/repo` parse commits only as needed was marked ready for `next`, with Derrick Stolee calling the memory reduction from 3.4 GiB to 0.7 GiB impressive.