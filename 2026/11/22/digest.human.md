# Git mailing list daily digest for 2026/11/22

## Long threads

### Limited fetch for new remotes in shallow clones

*Long thread: 86 replies so far from 7 participants over 65 days; 1 message that day.*

Harald Nordgren posted v15 of his series that keeps `git fetch` in a shallow single-branch clone from downloading every branch of a newly added remote. The redesign makes `remote.<name>.fetchUpstreams` explicitly control whether a refspec-less fetch retrieves the upstreams of local branches, so a `remote.<name>.fetchRefmap` alone no longer changes what is fetched. `--refmap` now also works without command-line refspecs, and a refspec that names its own destination is stored only there instead of also being mapped through the refmap. `git remote add` gains `--limited-fetch`, which sets the refmap and upstreams config instead of a wildcard fetch refspec, and in a shallow repository it becomes the default unless `-t`/`--track` or `--mirror` is given.

### Deleting squash-merged branches with --delete-integrated

*Long thread: 70 replies so far from 7 participants over 55 days; 2 messages that day.*

Harald Nordgren's v10 series adds `git branch --delete-integrated <pattern>`, which deletes branches whose changes landed upstream via squash or rebase merges, naming the commit where they landed. Today Junio C Hamano questioned whether the `opted_out_of_delete_merged()` helper should also let users opt out of `--delete-integrated`, saying he is not sure the project wants to go there. Jasper Mayone reported that v10 builds without warnings under `DEVELOPER=1`, passes all 215 t3200 tests with his earlier keep-tests added, and verified by hand that the submodule case keeps the branch and that a `merge=union` conflict is not listed, adding a `Tested-by` tag.

### Bounds fixes in utf8.c

*Long thread: 29 replies so far from 3 participants over 7 days; 8 messages that day.*

Dirk Gouders posted v5 of his series fixing boundary handling in `utf8_strnwidth()` and `display_mode_esc_sequence_len()` in `utf8.c`. After Andy Koppe called v4 unfocused and flagged a build failure from an undeclared `skip`, v5 drops the `display_mode_esc_sequence_len_all()` patch, uses an array index in the `_counted` variant so it never computes `s + len` (avoiding out-of-bounds pointers and O(n^2) degradation), and passes `SIZE_MAX` in the NUL-terminated wrapper. Gouders accepted Koppe's simplified bounds-check formulation and confirmed the test issues he raised will be fixed.

## The day in brief
Junio C Hamano posted a six-patch series fixing `strbuf_getwholeline()` not clearing its buffer on EOF, which had caused stale-buffer bugs in shortlog, `clean -i`, and fast-import. Harald Nordgren sent v15 of the limited-fetch series for shallow clones, and Junio is inclined to mark the submodule `^/` URL topic ready for `next`. A bisect traced two T7527 failures to the new Linux fsmonitor listener.

## Notable threads

### strbuf_getwholeline() must clear the buffer on EOF
Junio C Hamano posted a six-patch series addressing the long-standing API bug where `strbuf_getwholeline()` returns EOF without clearing the output buffer. Patches demonstrate the resulting bugs in shortlog (stale buffer reuse), `git clean -i` ask mode (deleting past EOF), and fast-import (re-parsing a heredoc delimiter at EOF), then fix the core function and convert the `test_expect_failure` demonstrations to `test_expect_success`, with additional tests for EOF loops in `clean -i` filter mode and fast-import merge-line parsing. He also fixed `mailinfo` leaking a stale last header into the commit log on header-only input.

### Submodule URLs with '^/' for server-root-relative paths
Stanislav Aleksandrov's RFC proposes resolving submodule URLs starting with `^/` against the superproject remote's root (like `../` but absolute), keeping protocol, user, host and port. Today Junio C Hamano said he is inclined to perform the proposed squash and mark the topic ready for `next`, citing Jacob's review as a strong endorsement, and asked heavy submodule users to object if the new behavior is problematic.

### T7527 fsmonitor--daemon failures
Dirk Gouders reported that T7527 on `origin/master` fails 2 of 70 tests, with a core file showing `git fsmonitor--daemon` dying on `BUG("double remove of watch for '%s'", w->dir)` in `compat/fsmonitor/fsm-listen-linux.c`. A bisect identified commit ce48de8b2c, the Linux filesystem change listener, as the first bad commit.

## In brief
- **difftool segfault patch**: Ophir Bleicher proposed a fix for a `git difftool --dir-diff` segfault on invalid paths like `.git` by checking `make_transient_cache_entry()` in `builtin/difftool.c`, adding a regression test.

- Stepan Koltsov skips `prep_exclude()` in `valid_cached_dir()` when fsmonitor is trusted, cutting `git status --untracked-files=all` on a warm cache from about 310 ms to 65 ms on a tree of roughly 89,000 directories.
- René Scharfe made `git archive`'s `--no-add-file` and `--no-add-virtual-file` remove only their own entries via a new `filter_string_list_func()`, instead of clearing the shared list.
- Junio C Hamano fixed `mailinfo` leaking the last header into the commit log on header-only input by returning -1 from `read_one_header_line()` on EOF.
- Wonseok Oh revised his fast-import fix so an inline `M` at EOF dies with "expected 'data n' command" instead of looping or storing an uninitialized object ID.
- Dirk Gouders added `display_mode_esc_sequence_len_counted()` and switched `utf8_strnwidth()`, `strbuf_utf8_replace()`, and `strbuf_add_wrapped_text()` to it.
- Dirk Gouders fixed `utf8_strnwidth()` to stop when ANSI skipping reaches the boundary before calling `utf8_width()`.
- Dirk Gouders passed the remaining length to `utf8_width()` in `utf8_strnwidth()` so incomplete multibyte sequences do not read past the boundary.
