# Git mailing list daily digest for 2026/11/22

## Long threads

### fetch: limited-fetch for new remotes in shallow clones

*Long thread (root 2026/09/19/14-47-26): 86 replies so far from 7 participants over 65 days; 1 message that day.*

This series addresses `git fetch` in a shallow single-branch clone: after `git remote add upstream <url>` sets the default wildcard refspec, the next fetch downloads every branch of the remote, defeating the point of a shallow clone. The series adds `--limited-fetch` and `remote.<name>.fetchUpstreams` so a remote fetches only the branches that local branches track. Today Harald Nordgren posted v15, changing the design so that `remote.<name>.fetchUpstreams` explicitly controls fetching local branches' upstreams (a `fetchRefmap` alone no longer changes what is fetched), making `--refmap` work without command-line refspecs, and stopping refspecs with their own destination from being mapped through refmaps. Patch 6/7 adds `git remote add --limited-fetch`, which sets `fetchRefmap` and `fetchUpstreams` instead of a wildcard fetch refspec and is rejected with `-t`/`--track` or `--mirror`; patch 7/7 makes `git remote add` default to `--limited-fetch` in a shallow repository, with `--no-limited-fetch` overriding. Earlier, Phillip Wood and Junio C Hamano had converged on the rule that a refspec carrying its own destination ignores the refmap.

### branch: --delete-integrated finds squash- and rebase-merged branches

*Long thread (root 2026/09/29/07-30-30): 70 replies so far from 7 participants over 55 days; 2 messages that day.*

This series adds `git branch --delete-integrated <pattern>`, which works like `--delete-merged` but also deletes a branch when some upstream commit contains all of its changes, as after a GitHub "Squash and merge" or "Rebase and merge", naming that commit in the output. Today Junio C Hamano questioned whether the opt-out helper `opted_out_of_delete_merged()` would also need to let people opt out of `delete_integrated()`, saying he is not sure they want to go there. Jasper Mayone reported that v10 builds with no warnings under `DEVELOPER=1` and that all 215 t3200 tests pass with his earlier keep tests added on top; he also verified by hand that the v8 submodule case keeps the branch and exits 0, and that a `merge=union` conflict is not listed by `--dry-run --delete-integrated`, adding a `Tested-by` tag.

### utf8: bounds fixes in utf8_strnwidth() and ANSI escape scanning

*Long thread (root 2026/11/16/18-56-59): 29 replies so far from 3 participants over 7 days; 8 messages that day.*

This series fixes boundary handling in `utf8.c`: `utf8_strnwidth()` disregarded its `len` argument, its comment incorrectly described the return value, and `display_mode_esc_sequence_len()` could read past a counted buffer's end. Today Andy Koppe reviewed v4 and said the series had become unfocused, recommending concentration on the comment fix, the optional end pointer for `display_mode_esc_sequence_len()` with caller changes, and the end guard plus bounded `utf8_width()` call in `utf8_strnwidth()`; he stated `display_mode_esc_sequence_len_all()` is not needed, noted patch 4/5 does not build because `skip` is undeclared, and pointed out that a test comparing pointers to different objects is undefined behaviour. Dirk Gouders agreed the extra function is noise, acknowledged the build failure was a merge-conflict mistake now fixed, accepted the simplified bounds-check formulation, and posted v5 0/4, which drops v4's 5/5, uses an array index in the `_counted` variant to avoid computing `s + len` and O(n^2) degradation, and passes `SIZE_MAX` in the wrapper for NUL-terminated strings.

## The day in brief
Harald Nordgren posted v15 of the limited-fetch series for shallow clones and v5 of the utf8 bounds series was reworked after review. Junio C Hamano posted a six-patch series fixing `strbuf_getwholeline()` not clearing its buffer on EOF, which had caused bugs in shortlog, `clean -i`, and fast-import. Stepan Koltsov posted an fsmonitor patch that cuts `git status` time in a large tree from about 310 ms to about 65 ms.

## Notable threads

### submodule: make '^/' work like '../' but with an absolute path
This RFC adds a submodule URL syntax `^/org/lib.git` that resolves against the superproject's remote from the root of the server, keeping protocol, user, host and port, so a forked superproject at a different depth does not break the URL; unlike `../`, it fails for local-path clones or remotes without a server. Today Junio C Hamano stated he is inclined to perform the proposed squash and mark the topic as ready for `next`, citing Jacob's review as a strong enough endorsement that the design issues were thought through, and asked heavy submodule users to object if they find the new behavior problematic before he merges the topic down.

### strbuf_getwholeline() should clear the buffer upon EOF
This six-patch series addresses a long-standing API bug: `strbuf_getwholeline()` returns EOF on `feof()` without clearing the output buffer, an error-prone contract. Today Junio C Hamano posted the series: patches demonstrate existing bugs in shortlog (stale buffer reuse at EOF), `clean -i` ask mode (deleting past EOF), and fast-import (re-parsing a heredoc delimiter at EOF), plus a mailinfo fix so a header-only message does not leak the last header into the commit log; the core fix makes `strbuf_getwholeline()` clear the strbuf upon EOF, converting the demonstrated failures to passing tests.

### dir: skip prep_exclude() when fsmonitor is trusted
This patch changes `git status` with the untracked cache and fsmonitor: `prep_exclude()` is called for every cached directory to drop entries whose `.gitignore` rules changed, but with fsmonitor that is unnecessary because the monitor already drops the entry when anything in the directory changes. Stepan Koltsov posted the patch skipping the call in `valid_cached_dir()` when fsmonitor is on and the entry is valid, and modifying the post-trivial-response recheck to also invalidate directories that have a `.gitignore`, since editing one in place does not change the directory's stat data. He reported that on macOS with about 89,000 directories and a warm untracked cache, `git status --untracked-files=all` went from about 310 ms to about 65 ms, with `read_directory()` time from about 255 ms to about 8 ms.

### difftool: handle invalid paths without crashing
This patch fixes a segmentation fault in `git difftool --dir-diff` when a tree contains an invalid path such as `.git`: `make_transient_cache_entry()` in `builtin/difftool.c` can return NULL, which was dereferenced. Ophir Bleicher posted the patch adding a NULL check and a regression test in `t/t7800-difftool.sh` that expects exit code 1.

### T7527 fails with 2/70 tests
Dirk Gouders reported that t7527 on `origin/master` fails 2 of 70 tests, with a `core.*` file in a temp directory causing `test_cmp` to fail. He specified that the core shows `git fsmonitor--daemon run --detach -` failing with `BUG("double remove of watch for '%s'", w->dir);` at `compat/fsmonitor/fsm-listen-linux.c:144`, and that a bisect identified commit `ce48de8b2c` (fsmonitor: implement filesystem change listener for Linux) as the first bad commit.

## In brief
- Junio C Hamano fixed `mailinfo()` leaking the last header into the commit log on header-only input by making `read_one_header_line()` return -1 on EOF, with a regression test in `t/t5100-mailinfo.sh`.
- René Scharfe made `git archive`'s `--no-add-file` and `--no-add-virtual-file` remove only the entries added by their respective positive forms, via a new `filter_string_list_func()`, with tests in `t/t5000-tar-tree.sh`.
- Wonseok Oh proposed a revised fix for fast-import's inline `M` at EOF, replacing the read loop with one that dies with "expected 'data n' command, found end of input", fixing both an uninitialized object ID and an infinite loop.
- Dirk Gouders posted v5 1/4 adding `display_mode_esc_sequence_len_counted()`, with the NUL-terminated wrapper passing `SIZE_MAX`, and switching `utf8_strnwidth()`, `strbuf_utf8_replace()`, and `strbuf_add_wrapped_text()` to the counted variant.
- Dirk Gouders posted v5 3/4 making `utf8_strnwidth()` check whether skipping ANSI escape sequences reaches the given boundary before calling `utf8_width()`.
- Dirk Gouders posted v5 4/4 passing the remaining byte count as the boundary argument to `utf8_width()` so incomplete multibyte sequences do not read past `len`.
- Hussain Sitamouwala explained the `PARSE_OPT_NONEG` addition in the check-ref-format parse-options conversion and said he is waiting for feedback before preparing v3.
- Dirk Gouchers maintained the "three" count in the gitcli doc fixes and offered a v4 changing "to" to "into" if Junio C Hamano wants it.
- Junio C Hamano said the lazyload comment typo fix looks good and will be picked up and marked ready for `next`.
- Junio C Hamano will pick up the `setup.c` comment fix ("non-existent") and mark the topic ready for `next`.
- Junio C Hamano pointed out that the word "used" is missing in the proposed http.extraheader documentation text.
- Junio C Hamano reviewed the update-index doc formatting patch, suggesting backquotes also for enumeration headers and command names, and expressed a preference against too many narrow low-value patches.
