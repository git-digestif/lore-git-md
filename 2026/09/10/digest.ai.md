# Git mailing list daily digest for 2026/09/10

## Long threads

### receive-report hook series reaches v10

*Long thread (root 2026/08/18/07-55-55): 53 replies so far from 6 participants over 24 days; 6 messages that day.*

Karthik Nayak's series introduces a `receive-report` hook in `git-receive-pack` that runs after the reference transaction completes but before the pkt-line status report is sent to the client; the hook's stdout replaces the report and a non-zero exit marks all references rejected. Today he posted v10, which fixes a leak in `override_cmds_error()` by freeing `error_string_owned` with `FREE_AND_NULL` before reassigning the command strings, and adds a test covering a non-zero hook exit alongside a pre-existing `ng` from `proc-receive`. Earlier in the day Junio C Hamano raised the leak concern, Jeff King jokingly noted an interop test could trigger the v9 BUG() case, and Karthik agreed no simple test exists for that path and would add one for the leak instead.

### odb: write alternates at creation time rolls to v5

*Long thread (root 2026/08/25/14-11-49): 34 replies so far from 5 participants over 17 days; 12 messages that day.*

Patrick Steinhardt's nine-patch series refactors the object database so alternates are written by `odb_source_create_on_disk()` at creation time, removing the ad-hoc `write_alternates()` callback and splitting `init_db()` into `create_repository()` plus caller-driven refdb/ODB setup. Today he posted v5, renaming `collect_one_alternate`/`collect_alternates_data`, dropping an extra `collect_alternates()` parameter, clarifying why `commondir` is computed even when unused, and fixing a "refernce" typo, after Karthik Nayak's review of v4 raised those points. Karthik reviewed v5 and said it otherwise looks good.

### last-modified Bloom pre-filtering under review

*Long thread (root 2026/07/17/15-46-58): 26 replies so far from 5 participants over 56 days; 4 messages that day.*

Toon Claes's series makes `git-last-modified(1)` use the pathspec's Bloom key to pre-filter commits, with benchmarks on ziglang/zig showing roughly 1.95x speedup with `-t` and 3.74x without versus master. Today Patrick Steinhardt reviewed the v4 code: he noted `bloom_filter_contains()` can return -1 when the filter length is zero while the documentation only promises 0 or 1, suggested renaming `revs_maybe_changed_in_bloom_with_parents()` to `..._with_leading_dirs()` to avoid confusion with commit parents, asked for the performance rationale, and questioned restoring `bloom_filter_settings` after `prepare_revision_walk()` clears them, proposing instead to store the settings in `struct last_modified`.

### sequencer auto maintenance series at v4

*Long thread (root 2026/09/04/07-53-44): 25 replies so far from 5 participants over 7 days; 1 message that day.*

Thomas Bachem's three-patch series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, from `builtin/rebase.c` and `builtin/revert.c`, while passing `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS`. Today he sent v4 with no code change: commit messages condensed to about a third at Junio C Hamano's request, with 3/3 taking Junio's wording except that the setting goes only to the commit, merge and exec commands the sequencer spawns.

### advice scoping mechanism in v4

*Long thread (root 2027/08/29/00-49-58): 18 replies so far from 4 participants over 2 days; 12 messages that day.*

Vsevolod Myalitsin's series fixes the `defaultBranchName` advice, whose suggested `git config` command writes to the just-initialized repository's local config and so never suppresses the advice on the next `git init`. The v4 series passes the whole `advice_setting` to `vadvise()`, adds a `scope_hint` field, and sets the default-branch advice to recommend the global scope. Today's discussion found that v4 2/3 fails `make DEVELOPER=YesPlease` with `-Werror=switch` over unhandled config scopes, that v4 1/3 has an unbalanced brace, and Vsevolod now favors a `bool is_global_hint` over the enum, a proposal Jeff King endorsed.

## The day in brief
Git v2.56.0-rc0 was announced for testing, comprising 635 non-merge commits since v2.55.0 from 82 contributors. Several long-running series posted new revisions: the receive-report hook (v10, leak fix), alternates-at-creation-time (v5), and the sequencer auto-maintenance change (v4, message-only). Review activity also flagged a build break and a missing brace in the advice-scoping series.

## Notable threads

### Git v2.56.0-rc0 announced
Junio C Hamano announced the availability of Git v2.56.0-rc0 for testing, with tarballs at kernel.org and the `v2.56.0-rc0` tag in the listed public repositories; the release comprises 635 non-merge commits since v2.55.0 contributed by 82 people, 32 of them new contributors.

### Git for Windows patch upstreaming at v4
Johannes Schindelin posted v4 of the 13-patch series upstreaming more Git for Windows patches ahead of the UCRT64 switch; v4 reinstates a fly-by style fix as a separate commit, adds Helped-by trailers, and moves a paragraph between commit messages. Junio C Hamano acknowledged the reroll and said he will replace the previous version and mark the series for `next`.

### builtin/history: unuse the commit buffer after use
Kaartic Sivaraam posted v3 of the fix for a leak in `git history reword` when the commit was parsed from the commit-graph: `commit_tree_ext()` in `builtin/history.c` now calls `repo_unuse_commit_buffer()` on the buffer, which a plain `free` would double-free on the cached path. Jeff King reviewed v3 as good with an accurate message, and Junio C Hamano agreed further message polishing has diminishing returns.

### refs/files: avoid packed-refs lock for root ref deletion
Ariel Keselman posted v2 skipping the packed-refs transaction when deleting root refs such as `AUTO_MERGE`, which cannot be packed, fixing post-commit cleanup errors in linked worktrees with read-only shared metadata. The v2 addresses Patrick Steinhardt's review: a focused comment, one existing root ref in the test, dropped timeout overrides, and a new test that a transaction deleting a root ref and a packed branch together still fails under a held packed-refs lock.

### Move rust gitcore crate to a rust/ subdirectory
Mike Hommey posted v4 moving `Cargo.toml`, `build.rs`, `cargo-meson.sh`, meson files and the Rust sources into `rust/`, fixing the meson `subdir()` error and the `.gitignore`/clean-target paths that Tuomas Ahola and Junio C Hamano caught. Junio reported that v4 still leaves the tree inconsistent per CI and does not know whether `ci/run-rust-checks.sh` or the file moves need tweaking.

## In brief
- Michal Koutný patched `merge-ll.c` to clean up `.merge_file_*` temporaries after Ctrl+C by sigchaining SIGINT/SIGQUIT around the external merge driver; Jeff King suggested using `xmks_tempfile()`/`delete_tempfile()` instead.
- Andrew Pleeter posted v7 of `git var` extensions (broken-down idents, `GIT_SIGNING_KEY`, multiple args, `-z`), with doc grouping, BUG() checks, and cleaner `-z` tests.
- Junio C Hamano posted a coccinelle-based rewrite introducing `advice_if_enabled()` across seven files; Jeff King noted it misses complex conditionals and questioned whether the extra turn-off lines would feel overwhelming.
- Aleksei Sviridkin clarified that the 20,000-entry reflog benchmark was a worst case and argued the `--force-if-includes` cutoff only helps when it wrongly rejects a valid push.
- James Le Cuirot posted v3 respecting `CARGO_BUILD_TARGET` in the Rust build output path in the `Makefile` and `src/cargo-meson.sh`, avoiding double slashes when unset.
- Kristoffer Haugsbakk confirmed the commit graph was the triggering factor for the `git history reword` memory leak.
- Tyler Cipriani posted v3 of the `--force-if-includes` fix with corrected patch threading; both patches are unchanged from v2.
- Brigham Campbell's v2 fixing conjoined maintenance-strategy bullets in `git-config(1)` was approved by Patrick Steinhardt; Jeff King explained the rendering difference between asciidoc and asciidoctor.
- Mark C. Chu-Carroll posted v2 modernizing t4001, t4009 and t4010; Junio C Hamano pointed him to `Documentation/SubmittingPatches` for commit-message style.
- Tuomas Ahola posted v2 keeping `Documentation/Makefile` and `command-list.txt` in sync, adding `gitformat-loose(5)` and `gitpacking(7)` and a lint-manpages check.
- Mark C. Chu-Carroll's v2 3/3 updates `t/t4010-diff-pathspec.sh` to modern style with inline heredocs.
- Delilah Ashley Wu proposed dropping the forward-slash patch in favor of setting a forward-slash `XDG_CONFIG_HOME` in the tests, asking Junio C Hamano for his view.
