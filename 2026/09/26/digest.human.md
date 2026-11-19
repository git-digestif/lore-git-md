# Git mailing list daily digest for 2026/09/26

## Long threads

### stash: clean up index-mode test merge

*Long thread: 26 replies so far from 3 participants over 8 days; 6 messages that day.*

D. Ben Knoble posted v3 of a five-patch series that cleans up the index-mode merge in `builtin/stash.c`, fixing a bug reported by Eli Barzilay in the interaction between autostashing, staged index entries, and `stash.index=true`. The final patch replaces the shell-out sequence of `diff-tree`/`apply --cached`/`reset` with an in-core `merge_incore_nonrecursive` call, fixing the autostash bug where `git reset` triggered a non-re-entrant `save_autostash()` lock on a deleted `MERGE_AUTOSTASH` ref, and adds a `t/t7600-merge.sh` test for fast-forward merge with `--autostash` and `stash.index`.

The v3 changelog covers changing the conflict label for the current index to "Current index", fixing a memory leak of `merge_result`, fixing the order of trees passed to `merge_incore_nonrecursive` to `(merge_base, head, merge)` with a new test (3/5) validating it, and making test 4/5 assert more details of the expected state on failure. Earlier, Phillip Wood had spotted the leak and the confusing label, and Junio C Hamano had diagnosed the transposed tree arguments, which Knoble confirmed came from the differing argument orders of `merge_ort_nonrecursive()` and `merge_incore_nonrecursive()`. Earlier today Phillip Wood proposed a more realistic content-level test for the `--index` merge, and Knoble confirmed it fails on the original wrong tree order and passes on the new code.

## The day in brief

D. Ben Knoble's v3 stash series carries the in-core index merge that fixes the autostash/`stash.index=true` bug, L Naudé reported a new `pack-objects` abort in `tree:0` partial clones, and Andrew Pleeter posted v9 of the four-patch `git var` extension series.

## Notable threads

### pack-objects aborts on missing promised tree during fetch

L Naudé reported that `git pack-objects --exclude-promisor-objects-best-effort` aborts with `BUG: builtin/pack-objects.c:5004: should_include_obj should only be called on existing objects` during `git fetch origin main` in a `tree:0` partial clone of `NousResearch/hermes-agent` on Git 2.55.0. The failure is traced to `is_not_in_promisor_pack_obj()` at `pack-objects.c:5004`, called from `process_tree()` at `list-objects.c:167` for missing tree `d0daf5223f2367bd20fc2deb9d66aed05fcd30e8`.

### dir: skip excluded directory with nested repo on prefix match

René Scharfe posted v2 of a `dir.c` patch that makes an ignored directory containing a nested repository respect pathspec matching, a corner case left open by earlier `fill_directory()` optimizations. The v2 guards the nested-repo `dir_match()` call with `pathspec &&` so it is not invoked when pathspec is NULL, fixing the crash Junio C Hamano had flagged in v1; the series adds a status test in `t/t7061-wtstatus-ignore.sh` and a checkout test in `t/t2021-checkout-overwrite.sh`.

### var: -z output, multiple variables, and broken-out idents

Andrew Pleeter posted v9 of a four-patch series extending `git var`: converting multi-valued variables to a `string_list`, adding a `-z` output mode, accepting more than one variable per invocation, and adding `GIT_AUTHOR_NAME`/`GIT_AUTHOR_EMAIL`/`GIT_AUTHOR_DATE` and their `GIT_COMMITTER_*` counterparts. `GIT_SIGNING_KEY` is dropped because its meaning depends on `gpg.format`. In multi-variable mode, variables with no value are omitted and the command exits 0, reserving non-zero status for real errors, while a single variable still exits 1 when it has no value.

## In brief

- Kristoffer Haugsbakk resent v2 of `format-patch --[no-]range-diff-notes` after the first send lacked a `To` header; the series adds `--range-diff-notes=<ref>` and `--no-range-diff-notes` via a parse-options callback so the range diff can show different notes than the patches.
- Tamir Duberstein posted v5 of a `Documentation/git-log.adoc` patch clarifying `--follow`'s single-file limitation, distinguishing explicit `--follow` errors from `log.follow` no-ops and documenting `--no-follow`.
- Jon Simons fixed repeated runs of `t/perf/p5551-fetch-rescan.sh` by passing `--no-deref` to `update-ref --stdin` when deleting child test refs before each measured fetch.
- Harald Nordgren confirmed his `ci/install-dependencies.sh` patch dropping the no-op `brew link --force gettext` on macOS remains relevant, silencing a warning in GitHub CI macOS jobs.
- Kristoffer Haugsbakk noted his preferred commit email is `code@khaugsbakk.name` and plans a `.mailmap` update, and reviewed Jeff King's regression test in `t/t4201-shortlog.sh` as sensible.
