# Git mailing list daily digest for 2026/09/18

## The day in brief

qeesung posted v2 of a five-patch series fixing a race in which `git repack -d` can delete a pack whose objects were never copied when a `.keep` file appears mid-run. Derrick Stolee opened an RFC for a "safe" strbuf API that cannot reach `die()`, aimed at recursive die loops in trace2 under allocation failure. Junio C Hamano confirmed a bug in `git rerere remaining` that skips consecutive conflicted paths and supplied a fix.

## Notable threads

### repack: don't lose objects to a ".keep" that appears mid-run

This is a bugfix for the repack and pack-objects code. A concurrent push or fetch can make `git repack -d` delete a pack whose objects were never copied anywhere and still exit 0: repack scans for `.keep` files and decides which packs to delete, then spawns pack-objects with `--honor-pack-keep`, which scans again; if an `index-pack --keep` installs a `.keep` next to a pack repack already decided to delete, pack-objects leaves that pack's objects out and repack deletes the pack anyway. qeesung posted v2 of the series, dropping the former patch 1/6 because Justin Tobler is going to fix the receive-pack side by having the ODB transaction create the `.keep` itself at commit time, and renumbering the remaining five patches. The core change has repack write its startup snapshot of `.keep` packs to a tempfile and pass it to pack-objects via a new `--keep-pack-from-file=<file>` option instead of `--honor-pack-keep`, so both processes work from one snapshot; pack names containing newlines are refused. Supporting patches make a `--keep-pack` pack behave as kept-open under `--stdin-packs=follow`, invalidate the kept-pack cache after re-marking fresh packs for the cruft walk, and sort the `--keep-pack` list once for binary lookup, the file option existing because command-line length limits can be exceeded with many kept packs.

### RFC: a "safe" strbuf API that cannot reach die()

Derrick Stolee posted an RFC series proposing a `strbuf-safe` API in new `strbuf-safe.c` and `strbuf-safe.h` files whose methods cannot transitively call `die()` or `exit()`. The motivation is that the trace2 API uses json-writer, which uses the strbuf API, so an allocation failure there can trigger recursive die() loops; Stolee asked whether the direction is worthwhile or should be abandoned. The series moves the strbuf struct definition and `STRBUF_INIT` into `strbuf-safe.h`, adds `sstrbuf_grow()`, `sstrbuf_init()` and `sstrbuf_release()` under an `s` prefix naming convention, adds `safe_memory_limit_check()` which returns an error code without dying while `memory_limit_check()` dies only outside gentle mode, reads `GIT_ALLOC_LIMIT` proactively during `setup_environment()` and documents it in `Documentation/git.adoc`, and adapts `json-writer.c` to use `sstrbuf_release()` and return error codes from `jw_release()`, `fmt_double()` and `append_sub_jw()`, while noting that callers of those json-writer methods are not yet adapted to the new return values.

### `rerere remaining` skips consecutive conflicted paths

Junio C Hamano confirmed Mikko Rantalainen's diagnosis that `check_one_conflict()` in `rerere.c` skips consecutive conflicted paths when each has only a stage-1 index entry, because its stage-1 skipping loop lacks a same-path check. He posted a patch adding `ce_same_name(e, istate->cache[i])` to the while-loop condition so only stage-1 entries for the same path are skipped, which also fixes `git mergetool` exiting successfully while conflicts remain.

## In brief

- Mike Hommey's v5 patch moving the Rust `gitcore` crate into a `rust/` subdirectory drew review questions from Junio C Hamano about `--manifest-path` usage and the meson `subdir()` choice; Hommey answered that `--manifest-path` is needed because `Cargo.toml` is no longer in `$PWD` and explained the meson path change.
- Haokai Ding fixed `diff --no-index -R` for file/directory conflicts, where `queue_diff()` queues the file before recursing and bypasses reverse handling; the fix swaps the filespecs of the early queue entry when `reverse_diff` is set, with regression tests in `t/t4053-diff-no-index.sh`.
- René Scharfe posted a `dir.c` fix so an excluded directory containing a nested repository is pathspec-matched on prefix match, adding a `dir_match()` helper and a test in `t/t7061-wtstatus-ignore.sh`.
- Junio C Hamano noted a GitLab CI failure with "cargo: command not found" in the Windows CI Rust thread.
- Johannes Schindelin defended keeping the `slen` parameter in `check_signature()` in `gpg-interface.c`, arguing that falling back to `strlen()` would truncate signatures with embedded NULs before verification; Junio C Hamano concluded the existing code is not worth changing.
- Mark C. Chu-Carroll accepted Junio C Hamano's review suggestions on modernizing `t/t4001-diff-rename.sh` and said he would send an update.
- Mark C. Chu-Carroll posted v4 of the three-patch "t40*: modernize" series, using a `test_expect_success 'setup'` block, consistent capitalization, improved rename test names, and tab indentation.
- Junio C Hamano grumbled that GitHub's web cache serves stale content for the git/git Actions workflow page.
