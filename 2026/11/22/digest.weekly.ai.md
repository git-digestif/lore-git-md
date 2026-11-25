## Long threads

### fetch: limited branch fetching for shallow clones
*Long thread: 86 replies so far from 7 participants over 65 days.*

Harald Nordgren's series prevents `git fetch` in a shallow single-branch clone from downloading every branch of a newly added remote. The week saw the series reach v15, which redesigns the mechanism around `remote.<name>.fetchUpstreams` to explicitly control whether a refspec-less fetch retrieves the upstreams of local branches, so a `remote.<name>.fetchRefmap` alone no longer changes what is fetched. `--refmap` now works without command-line refspecs, and a refspec naming its own destination is stored only there rather than also being mapped through the refmap. `git remote add` gains `--limited-fetch`, which sets the refmap and upstreams config instead of a wildcard fetch refspec, and in a shallow repository it becomes the default unless `-t`/`--track` or `--mirror` is given. Phillip Wood and Junio C Hamano debated whether command-line `src:dst` refspecs should ignore the refmap, settling on the rule that a refspec with its own destination overrides the refmap while the refmap supplies defaults for entries lacking one.

### branch: --delete-integrated for squash-merged branches
*Long thread: 70 replies so far from 7 participants over 55 days.*

Harald Nordgren's series adds `git branch --delete-integrated <pattern>`, which deletes branches whose changes landed upstream via squash or rebase merges, naming the commit where they landed. The series reached v10 this week, fixing a crash where a gitlink path replaced with different regular files on both the branch and upstream caused the command to die with "unable to read blob object"; such branches are now kept. v10 also makes the file merge a plain text merge ignoring merge drivers, finds branches that replaced a file or submodule with a directory, makes `--dry-run` work with `-d` and `-D`, prints from one table of templates, renames the opt-out helper to `opted_out_of_delete_merged()`, and adds tests for `--dry-run`, `--quiet`, and `branch.<name>.deleteMerged=false`. Junio C Hamano reviewed v9 and questioned whether the helper should also let users opt out of `--delete-integrated`, while Jasper Mayone confirmed v10 builds cleanly under `DEVELOPER=1`, passes all 215 t3200 tests, and added a `Tested-by` tag.

### `git repo info` path keys (GSoC series v9)
*Long thread: 64 replies so far from 6 participants over 124 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.workingtree.*`, `path.superprojectworkingtree.*`, `path.index.*` and `path.prefix` -- and refactors `get_superproject_working_tree()` to take a `struct repository *`. Earlier revisions renamed the keys and dropped `path.hooks` and `path.cdup` after review. This week K Jayatheerth noted that the easiest approach for the planned follow-up series consolidating `repo` and `rev-parse` logic is to combine and optimize the per-test repository setup.

### New detached HEAD guide
*Long thread: 46 replies so far from 5 participants over 41 days.*

Julia Evans's patch adds a `git help detached-head` guide, dual-purpose with the DETACHED HEAD section of `git-checkout.adoc`, explaining what detached HEAD state is and why users land there by accident. She posted v7 adding a sentence about making experimental commits without being on a branch and a clearer commit message. Junio C Hamano asked for a fresh-eyes final pass before merging it down to `next`.

### builtin/rerere: convert to subcommands
*Long thread: 44 replies so far from 6 participants over 25 days.*

Patrick Steinhardt's 13-patch series modernizes `git-rerere(1)` to use `OPT_SUBCOMMAND()` parsing like other Git commands, including a documented backwards-incompatible change requiring options after the subcommand (`git rerere --skip-locked gc` now errors). Patrick posted v5, which documents the `--skip-locked` placement change, fixes the stale `git rerere diff` documentation that wrongly claimed extra arguments go to the system `diff`, fixes the missing-synopsis regression for unknown subcommands, and NULL-terminates argv. Junio C Hamano reviewed patch 11/13, confirming that rejecting `--[no-]rerere-autoupdate` for subcommands fixes a bug, and accepted that the default "perform" help text is now hard to discover. Patrick noted the default-subcommand pattern is acceptable only for bare `git rerere`.

### `credentialStore.lockTimeout` config name
*Long thread: 43 replies so far from 5 participants over 76 days.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred name for credential lock retries, matching other millisecond file-lock settings that omit the unit, with `lockTimeoutMS` kept as a deprecated synonym. She resent v3 with that rationale in the message; Junio C Hamano reviewed it, said it looks good, and marked the topic ready for `next`.

### Worktrees rebasing or bisecting a branch
*Long thread: 42 replies so far from 8 participants over 30 days.*

Jasper Mayone's series makes `git worktree remove` refuse a worktree mid-operation without `--force` and makes `%(worktreepath)` name the worktree rebasing or bisecting a branch. This week he dropped patch 4/5 (`%(currentworktree)`) and agreed to stop the series at 3/5, since Junio C Hamano wanted the "+" marker kept on the branch his own worktree is bisecting and 4/5 existed only to support 5/5. He also answered Junio's question that the new `check_no_operation_in_progress()` and the existing checked-out-branch check answer different questions and should stay separate, with a shared helper proposed as a separate series.

### Bounds fixes in utf8.c
*Long thread: 29 replies so far from 3 participants over 7 days.*

Dirk Gouders's series fixes boundary handling in `utf8_strnwidth()` and `display_mode_esc_sequence_len()` in `utf8.c`. He posted v5, which drops the `display_mode_esc_sequence_len_all()` patch, uses an array index in the `_counted` variant so it never computes `s + len` (avoiding out-of-bounds pointers and O(n^2) degradation), and passes `SIZE_MAX` in the NUL-terminated wrapper. Andy Koppe suggested callers could pass `SIZE_MAX` to sidestep length checks, and Junio C Hamano affirmed that approach after Koppe corrected that `end = s + len` with `SIZE_MAX` would be undefined behavior, so callers must pass a valid length.

## The period in brief

The week of 2026/11/16 through 2026/11/22 saw seven active days of steady, technical traffic on the Git mailing list, with several long-running series reaching significant milestones. Readers should not miss Harald Nordgren's v10 of the `--delete-integrated` branch series fixing a submodule crash, his v15 redesign of limited fetching for shallow clones, and Patrick Steinhardt's reflog-expiration performance fix that cut auto-maintenance from 853ms to under 19ms.

## Key developments

### HTTP retry series restricts post_rpc retries
Ravi Mistry's series fixing HTTP 429 handling reached v4 after Patrick Steinhardt objected that retrying `post_rpc()` is unsafe for non-idempotent POSTs like `git-receive-pack` pushes. The v4 restricts retries to read-only services so pushes are not retried, honors an explicit "Retry-After: 0", and makes `attempt` a `size_t`. Patrick also proposed documenting that providers are expected to return 429 when overloaded, with other codes framed as a workaround for buggy providers.

### combine-diff fixed for merges with more than 32 parents
Aniket Gupta posted a two-patch series fixing the bug reported by Sami Jawhar where combined diffs of merges with more than 32 parents put lines in wrong columns and showed no hunks at 64 parents. Patch 1 changes the parent-bit shift in `coalesce_lines()` from `1<<parent` to `1UL<<parent`, fixing undefined behavior for the 33rd parent and later. Patch 2 makes combine-diff report an error and skip the patch for merges with more parents than the masks can hold instead of printing wrong output, without dying so that `git log --cc` is not cut short.

### refs/reftable: reflog expiration performance
Patrick Steinhardt posted a four-patch series fixing the auto-maintenance "reflog-expire" condition, which treated every reflog entry as unreachable and spawned `git reflog expire` on every run, blocking the foreground for roughly 800ms. The series fixes the condition and skips writing new reftables on dry-run or no-op expires, with `git maintenance run --auto` dropping from 853.2ms to 18.6ms. Karthik Nayak, Toon Claes and Junio C Hamano reviewed the series.

### connect: promisor-remote reply missing from fetch
Christian Couder posted v2 of an eight-patch series fixing a bug where the client does not send a `promisor-remote=...` reply with protocol v2 `fetch` commands, which fails with HTTP servers that handle each request separately. The v2 refactors client-side capability writing so `write_command_and_capabilities()` is the only function used across `ls-refs`, `bundle-uri`, and `fetch`, with the final patches ensuring fetch sends the reply and documenting it in `gitprotocol-v2.adoc`.

### strbuf_getwholeline() must clear the buffer on EOF
Junio C Hamano posted a six-patch series addressing the long-standing API bug where `strbuf_getwholeline()` returns EOF without clearing the output buffer. Patches demonstrate the resulting bugs in shortlog (stale buffer reuse), `git clean -i` ask mode (deleting past EOF), and fast-import (re-parsing a heredoc delimiter at EOF), then fix the core function and convert the `test_expect_failure` demonstrations to `test_expect_success`. He also fixed `mailinfo` leaking a stale last header into the commit log on header-only input.

### submodule: linked worktrees in recursive checkout
Adrian Ratiu posted a three-patch series fixing recursive-checkout failures -- a BUG() crash or a broken submodule -- when a submodule is a linked worktree of its repository. The series replaces the BUG() with an error, gates the `submodulePathConfig` advice on the extension being disabled, and accepts a linked-worktree git dir. Junio C Hamano reviewed, suggesting comment rewording and questioning whether a nested-submodule case could still reach the force-reconnect block.

### Hardening patches for packfiles, bitmaps, and filter-branch
Dominik Maier posted three security-hardening series. One replaces `get_delta_hdr_size()` with `decode_delta_hdr_size()` to bound delta header reads and prevent reading uninitialized stack bytes on corrupt packfiles. Another validates RLW bounds in `ewah_read_mmap()` so corrupt bitmaps are rejected instead of leaking memory. A third requires hex oids in `filter-branch` state-branch maps so entries with slashes or ".." segments die instead of writing outside the temp directory.

### Submodule URLs with '^/' for server-root-relative paths
Stanislav Aleksandrov's RFC proposes resolving submodule URLs starting with `^/` against the superproject remote's root, keeping protocol, user, host and port. Junio C Hamano said he is inclined to perform the proposed squash and mark the topic ready for `next`, citing Jacob's review as a strong endorsement, and asked heavy submodule users to object if the new behavior is problematic.

## In brief

**Optional `--follow` rename tracking** -- Miklos Vajna's patch makes `--follow` rename tracking in non-linear history optional via `--follow=merges`; Patrick Steinhardt judged v2 good and Junio C Hamano marked it ready for `next`.

**Faster object counting in `git repo structure`** -- Patrick Steinhardt's two-patch series makes `builtin/repo` parse commits only as needed and avoid fully parsing trees, reducing memory from 3.4 GiB to 0.7 GiB; Junio marked it ready for `next`.

**push: self-named remote group recurses** -- wheakerd reported that `git push <name>` recurses into thousands of nested processes when a remote group lists a remote of its own name, traced to `push_multiple()` running a child per group member while `add_remote_or_group()` resolves the name as a group first.

**loose object size checks** -- Elia Pinto posted v3 of five patches widening loose-object size parameters to `size_t` end to end, so `git cat-file --batch` no longer writes past 2-4 GiB objects on 64-bit Windows.

**apply: reject corrupt "diff --git" headers** -- Tanga Joris's series makes `git apply --check` report corrupt headers as errors instead of dying on an assertion; Junio argued the 1/3 test only passes because the bogus line names the same file as the next line.

**fetch: --prune-tags clobber changed tags** -- Orgad Shaneh posted v2 restoring the documented behavior of `--prune-tags` force-updating tags that changed on the remote, broken since Git 2.20, by adding "+" to the implied refspec.

**submodule: worktree-specific configuration** -- Yoichi NAKAYAMA posted v2 making submodule settings worktree-specific when `extensions.worktreeConfig` is enabled, keeping `submodule.<name>.gitdir` relative for linked worktrees.

**ls-files: reuse untracked cache** -- Tamir Duberstein updated the series to v4, taking the index lock after `fill_directory()` so optional writes detect concurrent index changes even with `index.skipHash` enabled.

**status, push: rebased push branch** -- Aniket Gupta reported that `--cherry-mark` makes `git status` slower on diverged branches because every commit needs a patch id, and suggested counting without it first.

**T7527 fsmonitor--daemon failures** -- Dirk Gouders reported that T7527 fails 2 of 70 tests with the daemon dying on `BUG("double remove of watch")`, bisected to commit ce48de8b2c, the Linux filesystem change listener.

**repo: revision filtering for `git repo structure`** -- Mark C. Chu-Carroll posted v8 adding a SHA1 prerequisite to tests after Junio reported they break under SHA-256, while Junio raised concerns about dropping "--" and leaking `revs`.

**doc: move BreakingChanges to a manpage** -- Kristoffer Haugsbakk asked whether there is interest in continuing the series moving `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage, noting it never got included in `seen`.

**fsmonitor daemon directory watch** -- John Mellor reported the Linux fsmonitor daemon does not watch a directory moved into the working tree from outside, only its subdirectories, traced to `rename_dir()` not calling `add_watch(path)`.

**diff-lib check_removed optimization** -- Maik Riechert posted a patch making `check_removed()` return immediately for `CE_FSMONITOR_VALID` entries, cutting `git diff HEAD` on 100,000 files from 57ms to 26ms.

**clone --revision under protocol v2** -- Andrei Șova fixed `git clone --revision` failing under protocol v2 when passed a valid commit hash in a different hash algorithm.

**submodule --jobs handling** -- Abhishek Agarwal proposed making `git submodule update --jobs=0` use `online_cpus()` instead of aborting.

**rev-parse --output-object-format** -- Ophir Bleicher made `git rev-parse --output-object-format=sha1` raise an error instead of printing a wrong hash in SHA-1-compatible SHA-256 repositories.

**pluggable commit-graph v2** -- Patrick Steinhardt posted v2 adding a commit that verifies trailer hashes across commit graph chains.

**packfile lifecycle series** -- Patrick Steinhardt posted a six-patch series making `pack_max_fds` enforcement process-wide and skipping `do_not_close` packs when closing.

**fast-import EOF handling** -- Wonseok Oh revised his fix so an inline `M` at EOF dies with "expected 'data n' command" instead of looping.

**reflog expire each reflog once** -- Laveesh Bansal posted v3 making `git reflog expire` skip already-processed reflogs with a strset.

**cherry-pick onto unborn branch** -- Wonseok Oh submitted a patch clearing the saved HEAD OID to the null OID so `--abort` reports "cannot abort from a branch yet to be born".

**http.extraheader documentation** -- brian m. carlsson posted a patch clarifying that `http.extraheader` is not for credentials.

**difftool segfault fix** -- Ophir Bleicher proposed a fix for a `git difftool --dir-diff` segfault on invalid paths like `.git`.

**fsmonitor prep_exclude skip** -- Stepan Koltsov skips `prep_exclude()` in `valid_cached_dir()` when fsmonitor is trusted, cutting `git status --untracked-files=all` from 310ms to 65ms on a warm cache.

**git archive --no-add-file filtering** -- René Scharfe made `--no-add-file` and `--no-add-virtual-file` remove only their own entries via a new `filter_string_list_func()`.