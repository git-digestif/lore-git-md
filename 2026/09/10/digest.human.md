# Git mailing list daily digest for 2026/09/10

## Long threads

### receive-report hook in git-receive-pack

*Long thread: 53 replies so far from 6 participants over 24 days; 6 messages that day.*

Karthik Nayak's series adds a `receive-report` hook that runs after the reference transaction completes but before the report reaches the client; it reads the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. Today he posted v10, fixing a leak in `override_cmds_error()` -- `error_string_owned` is now freed with `FREE_AND_NULL` before the new error is set -- plus a test for a non-zero hook exit with a pre-existing `ng` from proc-receive. Earlier, Junio C Hamano questioned the leak and Jeff King showed an interop test could trigger the v9 BUG() when no report is requested; Karthik agreed no simple test exists for that case.

### writing alternates at object database creation time

*Long thread: 34 replies so far from 5 participants over 17 days; 12 messages that day.*

Patrick Steinhardt's series refactors the object database so alternates are written by `odb_source_create_on_disk()` at creation time, removing the ad-hoc `odb_source_write_alternate()` callback. Today he posted v5 of the 9-patch series, renaming `collect_one_alternate`/`collect_alternates_data`, dropping an extra `collect_alternates()` parameter, clarifying why `commondir` is computed even when unused, and fixing a "refernce" typo after Karthik Nayak's review of v4. Earlier, Justin Tobler had called v4 good.

### last-modified Bloom filter pre-filtering

*Long thread: 26 replies so far from 5 participants over 56 days; 4 messages that day.*

Toon Claes's series speeds up `git last-modified` by reusing the revision machinery's Bloom filter checks to pre-filter commits, with benchmarks on ziglang/zig showing about 1.95x speedup with `-t` and 3.74x without. Today Patrick Steinhardt reviewed v4: he noted `bloom_filter_contains()` can return -1 for an empty filter while the docs cover only 0 or 1, suggested renaming `revs_maybe_changed_in_bloom_with_parents()` to `..._with_leading_dirs()`, asked for the performance rationale, and questioned restoring `bloom_filter_settings` after `prepare_revision_walk()` clears them, proposing to store them in `struct last_modified` instead. Earlier, v4 had fixed CI by overriding `GIT_TEST_COMMIT_GRAPH` in the t8020 tests.

### sequencer auto maintenance at end of sequence

*Long thread: 25 replies so far from 5 participants over 7 days; 1 message that day.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, from `builtin/rebase.c` and `builtin/revert.c`, while passing `maintenance.auto=false` to sequencer-spawned commands to avoid lock contention. Today he sent v4 with unchanged code, condensing the commit messages to about a third at Junio C Hamano's request. Earlier, Phillip Wood had called the series pretty good.

### global scope for defaultBranchName advice

*Long thread: 18 replies so far from 4 participants over 2 days; 12 messages that day.*

Vsevolod Myalitsin's series fixes the `advice.defaultBranchName` turn-off hint, which suggests a local-scope `git config set` that only affects the already-initialized repository while the advice reappears on the next `git init`. Today he posted v4, a three-patch series passing the whole `advice_setting` to `vadvise()`, adding a `scope_hint` field, and setting the defaultBranchName hint to `CONFIG_SCOPE_GLOBAL`. Review found a `-Werror=switch` build failure and an unbalanced brace in 1/3, and Vsevolod later proposed a `bool is_global_hint` instead, which Jeff King endorsed.

## The day in brief

Git v2.56.0-rc0 was announced for testing with 635 non-merge commits since v2.55.0. Karthik Nayak posted v10 of the receive-report hook series fixing a leak, and Patrick Steinhardt posted v5 of the alternates-at-creation-time refactor. Junio C Hamano said Johannes Schindelin's Git for Windows series could be marked for `next`.

## Notable threads

### Git v2.56.0-rc0 release candidate
Junio C Hamano announced Git v2.56.0-rc0 for testing, comprising 635 non-merge commits since v2.55.0 from 82 people including 32 new contributors, with tarballs at kernel.org and the `v2.56.0-rc0` tag in the listed public repositories.

### upstreaming more Git for Windows patches

Johannes Schindelin posted v4 of the 13-patch series upstreaming Git for Windows patches needed for the UCRT64 switch, reinstating a fly-by style fix as a separate commit and adding Helped-by trailers. Junio C Hamano acknowledged the reroll and said that, with Johannes Sixt's earlier blessing, the series could be marked for `next`.

### moving the rust gitcore crate into rust/
Mike Hommey posted v4 moving `Cargo.toml`, `build.rs`, `cargo-meson.sh` and the Rust sources into a `rust/` subdirectory so the git source can be vendored as a submodule without its Rust crate. Junio C Hamano reported that v4 still leaves the tree inconsistent per CI and does not know whether `ci/run-rust-checks.sh` needs updating.

## In brief
- Kaartic Sivaraam posted v3 of a fix in `builtin/history.c` adding the missing `repo_unuse_commit_buffer()` call that fixes an AddressSanitizer leak; Jeff King reviewed it as good.
- Ariel Keselman posted v2 skipping the packed-refs lock when deleting root refs in the files backend, fixing post-commit cleanup errors in linked worktrees with read-only shared metadata.
- Michal Koutný submitted a `merge-ll.c` patch cleaning up `.merge_file_*` temporaries after Ctrl+C; Jeff King suggested using the tempfile cleanup handler instead.
- Delilah Ashley Wu proposed dropping the forward-slash patch in her global-config series and instead setting a forward-slash `XDG_CONFIG_HOME` in the tests.
- Andrew Pleeter posted v7 of `git var` extensions adding broken-down ident components, `GIT_SIGNING_KEY`, multiple arguments and `-z`.
- James Le Cuirot posted v3 making the Rust build respect `CARGO_BUILD_TARGET` when locating `libgitcore.a`.
