# Git mailing list daily digest for 2026/09/26

## Long threads

### stash: clean up index-mode test merge

*Long thread (root 2026/09/19/21-26-34): 26 replies so far from 3 participants over 8 days; 6 messages that day.*

D. Ben Knoble posted [PATCH v3 0/5] "stash: clean up index-mode test merge", a series in `builtin/stash.c` fixing a bug reported by Eli Barzilay in the interaction between autostashing, staged index entries, and `stash.index=true`. The v3 changelog covers changing the conflict label for the current index to "Current index", fixing a memory leak of `merge_result`, fixing the order of trees passed to `merge_incore_nonrecursive` (merge_base, head, merge) with a new test (3/5) validating it, and making test 4/5 assert more details of the expected state on failure. The final patch (5/5) replaces the shell-out sequence of `diff-tree`/`apply --cached`/`reset` with an in-core `merge_incore_nonrecursive` call, fixing the autostash bug where `git reset` triggered a non-re-entrant `save_autostash()` lock on a deleted `MERGE_AUTOSTASH` ref, and adds a `t/t7600-merge.sh` test for fast-forward merge with `--autostash` and `stash.index`. Earlier, Phillip Wood had proposed a more realistic test checking actual file-content merging with distinct contents for the three merged blobs, and D. Ben Knoble confirmed it fails on the original wrong tree order and passes on the new code, to be included in the next round; Phillip Wood had also spotted the `merge_result` leak, and Junio C Hamano had diagnosed that the transposed tree arguments came from the differing parameter orders of `merge_ort_nonrecursive()` and `merge_incore_nonrecursive()`.

## The day in brief

D. Ben Knoble posted v3 of the stash index-mode merge cleanup, correcting the tree order passed to `merge_incore_nonrecursive` and fixing a `merge_result` memory leak. Andrew Pleeter sent v9 of the `git var` series, adding `-z` output, multiple-variable queries, and broken-out identity variables. L Naudé reported a `pack-objects` abort on a missing promised tree during fetch in a `tree:0` partial clone.

## Notable threads

### pack-objects abort on missing promised tree in partial clone

L Naudé reported that `git pack-objects --exclude-promisor-objects-best-effort` aborts with `BUG: builtin/pack-objects.c:5004: should_include_obj should only be called on existing objects` during `git fetch origin main` in a `tree:0` partial clone of `NousResearch/hermes-agent` on Git 2.55.0. The failure is traced to `is_not_in_promisor_pack_obj()` at `pack-objects.c:5004`, called from `process_tree()` at `list-objects.c:167` for missing tree `d0daf5223f2367bd20fc2deb9d66aed05fcd30e8`, with `remote.origin.promisor=true` and `remote.origin.partialclonefilter=tree:0` set.

### git var: -z output, multiple variables, broken-out idents

Andrew Pleeter posted [PATCH v9 0/4], a reroll splitting the v8 single patch into four patches touching `builtin/var.c`, `Documentation/git-var.adoc`, and `t/t0007-git-var.sh`: converting multi-valued variables (currently only `GIT_CONFIG_GLOBAL`) from newline-joined strings to a `string_list`, adding a `-z` option that terminates each entry with a NUL byte, teaching `git var` to accept more than one variable, and adding `GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL`, `GIT_AUTHOR_DATE` and their `GIT_COMMITTER_*` counterparts via `split_ident_line()`. `GIT_SIGNING_KEY` is dropped because `get_signing_key()` returns a value whose meaning depends on `gpg.format`. In multi-variable mode, variables with no value are omitted and the command exits 0, reserving non-zero status for real errors, while a single variable with no value still exits 1.

### format-patch: learn --[no-]range-diff-notes

Kristoffer Haugsbakk resent PATCH v2 of `kh/format-patch-range-diff-notes` after the previous send failed to reach the list due to a missing `To` header. Patch 2/2 adds `--range-diff-notes=<ref>` and `--no-range-diff-notes` to `format-patch`, implemented via a parse-options callback using a new `rdiff_notes` struct, so the range diff can display different notes than the patches; patch 1/2 simplifies `get_notes_args` to take only a `struct rev_info`.

### dir: skip excluded directory with nested repo on prefix match

René Scharfe posted v2 of the `dir.c` fix for ignored directories containing a nested repository, guarding the nested-repo `dir_match()` call with `pathspec &&` so it is not invoked when pathspec is NULL. He acknowledged the v1 bug after Junio C Hamano had written earlier that `dir_match()` would crash on a NULL pathspec. The series adds a status test in `t/t7061-wtstatus-ignore.sh` and a checkout test in `t/t2021-checkout-overwrite.sh`.

### doc: clarify --follow's single-file limitation

Tamir Duberstein posted v5 of the `Documentation/git-log.adoc` patch clarifying `--follow`: an explicit `--follow` requires exactly one path argument and Git reports an error if none or more than one is given, while the `log.follow` config leaves the command unchanged in those cases. Results for directory arguments and accepted wildcard patterns are unspecified, and `--no-follow` disables the mode.

## In brief

- Harald Nordgren confirmed his `ci/install-dependencies.sh` patch dropping the no-op `brew link --force gettext` on macOS remains relevant, clarifying it silences a spurious "Already linked" warning in GitHub CI macOS job logs.
- Jon Simons updated `t/perf/p5551-fetch-rescan.sh` to pass `--no-deref` to `update-ref --stdin` when deleting child test refs before each measured `git fetch`, fixing repeated runs that otherwise fail with "multiple updates for 'refs/remotes/origin/master'".
- Kristoffer Haugsbakk stated his preferred commit email is `code@khaugsbakk.name` and that he ought to send a `.mailmap` change, and reviewed Jeff King's regression test in `t/t4201-shortlog.sh`, concluding it makes sense as written.
