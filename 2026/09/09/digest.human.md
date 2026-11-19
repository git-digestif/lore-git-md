# Git mailing list daily digest for 2026/09/09

## Long threads

### receive-report hook in `builtin/receive-pack.c`

*Long thread: 48 replies so far from 5 participants over 23 days; 3 messages that day.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the status report goes to the client; the hook reads the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. Today he posted v9, fixing a bug where receive-pack hit a `BUG()` when the client requested no report, after Junio C Hamano wrote that skipping the report is perfectly valid. Patrick Steinhardt asked whether tests cover the fix; Junio explained the send-pack client always requests a report when the server advertises report-status, so such a test would need a custom client. Earlier, Junio had objected to the `BUG()` in the default case of the new switch and proposed `case REPORT_STATUS_UNKNOWN: break;` instead.

### `uploadpack.lazyFetchTrusted` and lazy-fetch recursion guard

*Long thread: 25 replies so far from 3 participants over 62 days; 2 messages that day.*

Christian Couder's series adds protected config `uploadpack.lazyFetchTrusted` letting server operators mark repos whose git dir is trusted for lazy fetching. The motivation is large-object promisors: when a server moves blobs to a promisor remote, a filtered clone can fail because the server cannot provide blobs in the filtered range. Today's exchange concerned the type of the `GIT_INTERNAL_LAZY_FETCH_DEPTH` counter that stops infinite recursion when a promisor remote resolves back to the served repo: Christian proposed casting `git_env_ulong()` to `int` since no `git_env_int()` exists, and Junio noted existing callers mix casts and left the choice to him.

### sequencer auto maintenance

*Long thread: 24 replies so far from 5 participants over 6 days; 3 messages that day.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, and disables auto maintenance in sequencer-spawned commands via `maintenance.auto=false` in `GIT_CONFIG_PARAMETERS`, avoiding lock contention on `MERGE_RR.lock` and repacks deleting open packs. Today he posted v4 with commit messages condensed to about a third at Junio's request, no code change. Phillip Wood reviewed v4 3/3, suggesting a reworded opening, questioning a new test assertion, and calling the series pretty good.

### ref storage format terminology

*Long thread: 23 replies so far from 4 participants over 6 days; 12 messages that day.*

Patrick Steinhardt's 13-patch series unifies ref storage terminology: `--ref-format=` becomes `--ref-storage-format=` with old names kept as hidden aliases, alongside matching environment variables and `init.defaultRefStorageFormat`, and `--ref-storage-format=` now accepts `format://payload` URIs. Today he posted v3 with hidden `OPT_ALIAS()` support, correct environment variables in error messages, and renamed `GIT_TEST_DEFAULT_REF_STORAGE_FORMAT`. Kaartic Sivaraam raised payload-comparison and shared-doc concerns; Patrick declined `strbuf_realpath` comparison since a payload need not be a path, and kept the shared `ref-storage-format.adoc` out of the series.

### writing alternates at object database creation

*Long thread: 23 replies so far from 4 participants over 16 days; 4 messages that day.*

Patrick Steinhardt's series refactors alternates handling so `create_object_database()` takes the alternates list and writes `info/alternates` at creation time via a lockfile, removing the ad-hoc `odb_source_write_alternate()` callback. Today he posted v4 adding documentation for the new repository-creation functions, and Justin Tobler reviewed it as looking good.

## The day in brief
Karthik Nayak posted v9 of the receive-report hook fixing the no-report `BUG()`, Patrick Steinhardt posted v3 of the ref-storage-format terminology series, and SZEDER Gábor posted a Makefile series precompiling `git-compat-util.h` for about a 35% build speedup.

## Notable threads

### moving the Rust gitcore crate into `rust/`
Mike Hommey reposted v3 moving all Rust-specific files (`Cargo.toml`, `src/*.rs`, build glue) into a dedicated `rust/` subdirectory, since a top-level `Cargo.toml` implies `cargo build` works there and breaks vendoring Git as a submodule. Junio C Hamano reviewed it, flagging duplicate `RUST_SOURCES` entries and a stale `clean` target; brian m. carlsson questioned the rationale given Rust components become mandatory in Git 3.0.

### `git var` broken-down idents, signing key, multiple args
Andrew Pleeter reposted v6 teaching `git var` to expose `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, committer equivalents, `GIT_SIGNING_KEY`, multiple variables per call, and `-z`. Phillip Wood pressed again for the use case for `GIT_SIGNING_KEY` and objected to a non-zero exit when one requested variable is unset; Junio C Hamano proposed modeling multi-variable output on `git var -l` var=value pairs rather than the empty-record convention.

### precompiling `git-compat-util.h` in the Makefile
SZEDER Gábor posted a 4-patch series making the Makefile precompile `git-compat-util.h`, reporting about a 35% build speedup; patch 3/4 reintroduces `REFTABLE_OBJS` so reftable sources, which mostly do not include that header, can be excluded. He supplied an optional follow-up covering the remaining files but judged its ~1% gain not worth it, and Junio suggested reorganizing patch 3/4's commit message.

## In brief
- **Kristoffer Haugsbakk's range-diff-notes redesign**: Interpretation (2) was his own design, accepted by Junio C Hamano; he proposed only `--no-RD-notes` and `--RD-notes=<ref>`.

- Johannes Schindelin posted v3 of the 12-patch Git for Windows upstreaming series, moving CMake/meson hunks from 08/12 to 12/12; Johannes Sixt found 08/12 good apart from a stale "Note:" paragraph.
- Christian Couder submitted Git's Outreachy Community Application for the December 2026 cohort, offering to mentor two projects and proposing a sponsoring topic for the Contributor's Summit.
- Reviewers of the `advice.defaultBranchName` global-config patch agreed on a single-placeholder message and a three-patch split; Vsevolod Myalitsin raised a concern about `CONFIG_SCOPE_UNKNOWN` defaulting.
- Aleksei Sviridkin clarified that the `--force-if-includes` fallback triggers when the remote-tracking ref has no reflog at all, and benchmarked the zero-cutoff cost.
