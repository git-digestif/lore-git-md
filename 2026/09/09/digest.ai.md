# Git mailing list daily digest for 2026/09/09

## Long threads

### receive-report hook in `builtin/receive-pack.c`

*Long thread (root 2026/08/18/07-55-55): 48 replies so far from 5 participants over 23 days; 3 messages that day.*

Karthik Nayak posted v9 of the series introducing a `receive-report` hook that runs after the reference transaction completes but before the status report is sent to the client; the hook receives the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. The v9 fix replaces the `BUG("unknown report status version")` in the default case of the new switch with `case REPORT_STATUS_UNKNOWN: break;`, fixing a BUG() failure when a client requests no report -- a case Junio C Hamano had called perfectly valid. Patrick Steinhardt asked whether tests cover the fix; Junio explained that `send-pack` always requests a report when the server advertises `report-status`/`report-status-v2`, so such a test would need a custom client or a way to disable capability advertisement.

### `uploadpack.lazyFetchTrusted` and the lazy-fetch recursion guard

*Long thread (root 2026/07/10/08-51-34): 25 replies so far from 3 participants over 62 days; 2 messages that day.*

Christian Couder's series adds a protected `uploadpack.lazyFetchTrusted` config so server operators can mark repositories whose git dir is trusted for lazy fetching, plus a `GIT_INTERNAL_LAZY_FETCH_DEPTH` guard (max 5) against infinite recursion when a promisor remote resolves back to the served repo. Today's exchange concerned the depth counter's type: Christian proposed casting `git_env_ulong()` to `int` since `git_env_int()` does not exist, and Junio C Hamano noted that `git_env_ulong()` already has 13 callers mixing explicit and implicit casts, leaving the choice to Christian.

### sequencer auto maintenance at the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 24 replies so far from 5 participants over 6 days; 3 messages that day.*

Thomas Bachem posted v4 of the series that runs `git maintenance run --auto` exactly once when a rebase, cherry-pick or revert sequence completes (from `builtin/rebase.c` and `builtin/revert.c`) and passes `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS`; v4 has no code change, only commit messages condensed to about a third at Junio C Hamano's request. Patrick Steinhardt agreed with Junio that proposed log messages have been growing longer with irrelevant detail. Phillip Wood reviewed v4 3/3, suggested rewording the opening of the commit message, questioned one new test assertion in `t/t3418-rebase-continue.sh`, and called the series pretty good.

### unifying ref storage format terminology

*Long thread (root 2026/09/04/10-36-01): 23 replies so far from 4 participants over 6 days; 12 messages that day.*

Patrick Steinhardt posted v3 of the 13-patch series renaming `--ref-format=`, `--show-ref-format`, `GIT_REFERENCE_BACKEND`, `GIT_DEFAULT_REF_FORMAT` and `init.defaultRefFormat` to `ref-storage-format` equivalents across init, clone, refs, submodule, rev-parse and help, keeping old names as hidden aliases via `OPT_ALIAS()`, and letting `--ref-storage-format=` accept a `format://payload` URI. Kaartic Sivaraam raised follow-ups -- comparing payloads with `strbuf_realpath`, error messages naming the wrong environment variable when the fallback is used, and `Documentation/ref-storage-format.adoc` being shared with `git-refs(1)` -- and Patrick agreed to fix the env-var messages and the `GIT_TEST_DEFAULT_REF_FORMAT` rename, rejected the realpath comparison since a payload need not be a path, and kept the shared doc out of the series.

### writing alternates at object database creation time

*Long thread (root 2026/08/25/14-11-49): 23 replies so far from 4 participants over 16 days; 4 messages that day.*

Patrick Steinhardt posted v4 of the 9-patch series that moves alternate writing into `odb_source_create_on_disk()` at creation time -- `create_object_database()` now takes a `const struct strvec *alternates` -- and removes the ad-hoc `odb_source_write_alternate()` callback; v4 adds documentation for the new repository-creation functions. Justin Tobler reviewed v4 and said it looks good to him.

## The day in brief

Karthik Nayak's v9 receive-report hook fixes the BUG() failure when a client requests no report. Patrick Steinhardt's v4 alternates-at-creation series drew positive review, while his v3 ref-storage-format series drew follow-up questions. Thomas Bachem's v4 sequencer auto-maintenance series arrived with condensed commit messages after reviewers pushed back on message length. SZEDER Gábor's precompiled-header Makefile series reports about a 35% build speedup.

## Notable threads

### precompiling `git-compat-util.h` in the Makefile
SZEDER Gábor posted a 4-patch series making the Makefile precompile `git-compat-util.h`, reporting about a 35% build speedup; it reintroduces `REFTABLE_OBJS` so reftable sources (which mostly do not include the header) are excluded, and adds a `NO_PRECOMPILED_HEADER` knob. He also benchmarked an optional follow-up covering the remaining compat/reftable files at only ~1% extra gain and judged it not worth the churn. Junio C Hamano suggested leading patch 3/4's commit message with the series' endgame rather than the historical `REFTABLE_OBJS` detail.

### `git var` broken-down idents, signing key, multiple args, `-z`
Andrew Pleeter reposted v6 of the patch teaching `git var` to expose `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, the committer equivalents, `GIT_SIGNING_KEY`, multiple variable arguments and `-z`. Phillip Wood objected to returning non-zero when one of several requested variables has no value, since that prevents distinguishing fatal errors by exit code, and again asked for the use case for `GIT_SIGNING_KEY`. Junio C Hamano argued the extra delimiter after multi-valued variables makes the wrong trade-off and stated his preferred design: drop the empty-record convention and model multi-variable output as `var=value` pairs like `git var -l`.

### moving the Rust gitcore crate into `rust/`
Mike Hommey reposted v3, moving `Cargo.toml`, `cargo-meson.sh`, `meson.build` and the `src/*.rs` files into a dedicated `rust/` subdirectory so a top-level `Cargo.toml` no longer implies a bare `cargo build` works and the source can be vendored as a submodule. Junio C Hamano flagged that `rust/src/lib.rs` and `rust/src/varint.rs` are each listed twice in `RUST_SOURCES` and that the `clean` target still removes the top-level `Cargo.lock`; brian m. carlson questioned the rationale, asking how vendoring without the Rust components would work once Rust becomes mandatory.

### upstreaming more Git for Windows patches
Johannes Schindelin posted v3 of the 12-patch series, moving the CMake/meson/`MINGW_PREFIX` hunks out of 08/12 into 12/12 and dropping a fly-by cleanup from `t0060`. Johannes Sixt found 08/12 good but flagged a stale "Note:" paragraph about CMake/Meson support; Johannes agreed to remove it and add a `Helped-by:`. Junio C Hamano reviewed 12/12, called the movement logically correct, and asked whether the topic should be marked ready for `next`.

### `format-patch --[no-]range-diff-notes`
Kristoffer Haugsbakk clarified that his design means `--no-range-diff-notes` disables range-diff notes entirely, and proposed a redesign offering only `--no-RD-notes` and `--RD-notes=<ref>` via a parse-opts callback, dropping the arg-less form and any revert-to-patch-notes behavior. Junio C Hamano confirmed agreement on the interpretation, agreed the "turn off this override" docs section is inaccurate, and noted the `warning: notes ref refs/notes/ is invalid` output from an empty `--notes=` is a UI bug that can be fixed.

## In brief
- Jeff King said the v3 "advice: use global config for default branch name" patch looks OK with nits; Junio C Hamano wants a small final reroll split into three patches (vadvise() API, scope mechanism, then defaultBranchName global) with the motivation in the commit message.
- Aleksei Sviridkin reported benchmarks for the `--force-if-includes` fix when the remote-tracking ref has no reflog: the zero cutoff adds about 0.26s only on repositories still holding reflog entries older than 90 days where gc has not run.
- Jeff King proposed NULL as the advice_setting for ad-hoc `vadvise()` calls; Junio C Hamano accepted the proposal.
- Christian Couder submitted Git's Outreachy December 2026 community application offering to mentor and sponsor 2 interns with the two previously proposed projects, and suggested an "Outreachy sponsoring" topic for the Contributor's Summit.
- Junio C Hamano noted the `CARGO_BUILD_TARGET` patch is based on a stale codebase and suggested `${CARGO_BUILD_TARGET+$CARGO_BUILD_TARGET/}` to avoid double slashes in `src/cargo-meson.sh`.
- Tuomas Ahola posted a two-patch series adding `gitformat-loose(5)` and `gitpacking(7)` to `command-list.txt` and extending `Documentation/lint-manpages.sh` to check the guide list; Junio C Hamano praised the additions and pointed out quoting fixes.
- Patrick Steinhardt queued a one-word commit-message reword in "submodule-config: stop registering submodule sources" but did not resend for that alone.
- Kristoffer Haugsbakk noted a Sep 7 maintenance-doc topic was missing from the "What's cooking" listing; Junio C Hamano replied that resending a patch shows broader interest and that he had left the topic for a final reroll.
- Patrick Steinhardt posted v3 01/13 introducing `OPT_ALIAS_F()`, which accepts flags so aliases can be hidden with `PARSE_OPT_HIDDEN`.
