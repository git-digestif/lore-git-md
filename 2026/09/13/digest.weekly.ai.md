## Long threads

### history: add squash subcommand to fold a range

*Long thread: 91 replies so far from 9 participants over 87 days.*

Harald Nordgren posted v15 of the series adding `git history squash <revision-range>` to fold a range of commits in `builtin/history.c`, fixing `istarts_with()` to `starts_with()` in `squash_check_autosquash_subject()` so `fixup! ABCdef` mixed-case hex is no longer accepted, fixing a `squshed` typo, and adding `squash` to the `git history` subcommand completion test. Junio C Hamano replied that with the "complete 'git history'" topic now in, he no longer needs an evil merge for the `t9902-completion.sh` change. The series remains the most-discussed thread of the week, standing at 91 replies over 87 days.

### receive-report hook for git-receive-pack

*Long thread: 57 replies so far from 7 participants over 25 days.*

Karthik Nayak's series reached v10, fixing a leak in `override_cmds_error()` where `error_string_owned` is now freed with `FREE_AND_NULL` before the new error is set, plus a test for a non-zero hook exit with a pre-existing `ng` from proc-receive. Earlier v9 fixed a `BUG()` when the client requested no report, after Junio C Hamano wrote that skipping the report is perfectly valid. Junio objected to the `BUG()` in the switch's default case and proposed `case REPORT_STATUS_UNKNOWN: break;` instead, and later noted `FREE_AND_NULL()` is safe on an already-NULL pointer so a guard could be dropped. The hook reads the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected.

### Convert build-time USE_NSEC to runtime core.useNanosec

*Long thread: 46 replies so far from 6 participants over 36 days.*

D. Ben Knoble resent v7 of the series exposing `USE_NSEC` as the runtime `core.useNanosec` config so distributed builds can use nanosecond index timestamps and avoid racy-git false positives; v7 differed from v6 only in a documentation typofix. Junio C Hamano queued the three patches, noting they matched what he had already queued from v6 with the typo reworded locally. Knoble explained the resend was mainly a nudge about the stale What's Cooking status.

### Write object-database alternates at creation time

*Long thread: 36 replies so far from 5 participants over 18 days.*

Patrick Steinhardt's nine-patch series reached v5, renaming `collect_one_alternate`/`collect_alternates_data`, dropping an extra `collect_alternates()` parameter, clarifying why `commondir` is computed even when unused, and fixing a "refernce" typo after Karthik Nayak's review of v4. The series refactors `init_db()` into `create_repository()` and moves alternate writing into `odb_source_create_on_disk()`, removing the ad-hoc `odb_source_write_alternate()` callback. Justin Tobler had called v4 good, and Steinhardt answered review comments noting the file-stream error indicator is sticky so per-`fprintf` checks are unnecessary.

### Add path keys to git repo info

*Long thread: 34 replies so far from 5 participants over 58 days.*

K Jayatheerth posted v6 of the GSoC series adding seven `path.*` keys to `git repo info` -- toplevel, superproject-root, hooks, index, and grafts with absolute and relative suffixes, plus git-prefix and cdup as single keys -- refactoring `get_superproject_working_tree()` to take a `struct repository *` instead of the process working directory. Junio C Hamano noted dead null checks after `repo_get_index_file()` in patch 4/7, asked what `path.cdup` returns outside the working tree, and suggested splitting patch 2/7 into a fix with tests plus the new keys.

### sequencer: run auto maintenance once a sequence is done

*Long thread: 27 replies so far from 5 participants over 8 days.*

Thomas Bachem's series reached v4 with commit messages condensed to about a third at Junio C Hamano's request and no code change; the series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, from `builtin/rebase.c` and `builtin/revert.c`, while passing `maintenance.auto=false` to sequencer-spawned commands. Patrick Steinhardt objected that the proposed comment for `git_config_append_parameter()` was too specific to `-c key=value`, and disputed the claim that the sequencer lacks a single exit point, arguing cherry-pick, revert, rebase, `--continue` and `--skip` all reach `pick_commits()` and preferring to call auto maintenance there and on abort. Phillip Wood had reviewed v4 3/3 and called the series pretty good.

### last-modified Bloom filter pre-filtering

*Long thread: 26 replies so far from 5 participants over 56 days.*

Toon Claes's series speeds up `git last-modified` by reusing the revision machinery's Bloom filter checks to pre-filter commits, with benchmarks on ziglang/zig showing about 1.95x speedup with `-t` and 3.74x without. Patrick Steinhardt reviewed v4, noting `bloom_filter_contains()` can return -1 for an empty filter while the docs cover only 0 or 1, suggesting renaming `revs_maybe_changed_in_bloom_with_parents()` to `..._with_leading_dirs()`, asking for the performance rationale, and questioning restoring `bloom_filter_settings` after `prepare_revision_walk()` clears them, proposing to store them in `struct last_modified` instead.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 26 replies so far from 6 participants over 41 days.*

brian m. carlsson posted v2 of the series removing Git's acceptance of uppercase hex in object IDs for Git 3.0, fixing a hex-range comment, updating commit messages, moving the t5324 fixes to a separate commit, and restoring lowercasing in `parse_oid_prefix()`. Phillip Wood said the object-ID-versus-ref case made more sense to him and wondered whether refnames whose last component looks like an object ID should also be forbidden.

## The period in brief

The week of 2026/09/07 through 2026/09/13 saw heavy traffic across seven active days, anchored by the Git v2.56.0-rc0 release candidate and a flurry of long-running series reaching new revisions. Readers should not miss the Windows CI Rust series being fast-tracked to `master`, Taylor Blau's diagnosis of a `git bundle create` bitmap bug, and the ongoing debate over whether the next release should be Git 3.0.

## Key developments

### Git v2.56.0-rc0 released

Junio C Hamano announced Git v2.56.0-rc0 for testing, comprising 635 non-merge commits since v2.55.0 from 82 people including 32 new contributors, with tarballs available at kernel.org and the `v2.56.0-rc0` tag in the listed public repositories.

### Windows CI builds with Rust

Johannes Schindelin's series lets the Windows CI jobs build with Rust, which Git requires by default since v2.55.0; v4 switched from `CARGO_TARGET`/`--target` to `CARGO_BUILD_TARGET` to pick a GCC-compatible Cargo target under MSYS2/MinGW. Junio C Hamano judged v4 ready and decided to merge it in `next` and fast-track it to `master` immediately, reasoning that Windows has an unusually high ratio of binary users relative to source builders.

### Git for Windows upstreaming series marked for next

Johannes Schindelin posted v4 of the 13-patch series upstreaming Git for Windows patches needed for the UCRT64 switch, reinstating a fly-by style fix as a separate commit and adding Helped-by trailers. Junio C Hamano acknowledged the reroll and said that, with Johannes Sixt's earlier blessing, the series could be marked for `next`.

### git bundle create with bitmaps omits required tree

Peter Elmers reported that `git bundle create` with bitmaps omits a tree required by an advertised ref when the included commit shares a root tree with an excluded sibling; `-c pack.useBitmaps=false` works around it. Taylor Blau reproduced the issue, diagnosed that the non-bitmap traversal overcounts objects past the boundary while the bitmap walk builds an exact answer, and proposed "bundle: restrict pack haves to recorded prerequisites" with a new bitmap test.

### push --force-if-includes consults wrong ref

Tyler Cipriani posted v2 fixing `--force-if-includes` in push: `check_if_includes_upstream()` now consults the pushed ref's reflog (`ref->peer_ref`) instead of the destination-named local branch, explicitly rejects detached HEAD, and skips deletions. Cipriani confirmed the reflog lookup behavior, agreed to resolve symrefs to the target branch and reject unverifiable revisions in a v4, and proposed moving a guard into `apply_push_cas`.

### What will come after Git 2.56?

Junio C Hamano's question of whether the release after 2.56 should be 3.0, 2.99, or business as usual drew concrete positions. Patrick Steinhardt noted a Git 2.100 is also possible after 2.99, called GitHub's missing SHA-256 support the biggest question mark for a 3.0 bump, and reported that reftable support is upstreamed in libgit2 with SHA-256 default-enabled. Emily Shaffer said GitHub SHA-256 support leaves her most concerned and leaned toward releasing 3.0.

## In brief

- **git pull overwrites local directory with symlink** -- AIKSXD ax reported that with a `.gitignore` pattern like `dataset/` and a committed symlink of the same name, `git pull` silently replaces the local directory with the symlink; brian m. carlsson explained the trailing-slash pattern ignores everything under the directory rather than the entry itself.

- **worktree lifecycle hooks** -- Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks so tools like devenv can provision per-worktree environments; Kristoffer Haugsbakk proposed instead a `--id=<string>` option on `git worktree add` recording ownership.

- **git var extensions** -- Andrew Pleeter posted v7 teaching `git var` to expose `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, committer equivalents, `GIT_SIGNING_KEY`, multiple variables per call, and `-z`; Junio C Hamano proposed modeling multi-variable output on `git var -l` var=value pairs.

- **moving the Rust gitcore crate into rust/** -- Mike Hommey posted v4 moving `Cargo.toml`, `build.rs`, `cargo-meson.sh` and the Rust sources into a `rust/` subdirectory; Junio C Hamano reported that v4 still leaves the tree inconsistent per CI.

- **pluggable ODB consistency checks** -- Patrick Steinhardt posted v3 of the ten-patch series moving fsck checks into the ODB sources, agreeing to move the check into the "files" backend and drop a `ret` variable after Toon Claes's review.

- **history: sign rewritten commits** -- Souma posted v3 of a two-patch series teaching `git history` rewrite commands to honor `commit.gpgSign` and explicit signing requests via `-S`/`--gpg-sign` across `drop`, `fixup`, `reword`, and `split`.

- **pull: avoid crash of invalid merge head** -- Jiri Kuncar submitted a patch adding NULL guards for `lookup_commit_reference()` in `builtin/pull.c` so invalid merge-head references from parallel fetches or gc races fall back to a normal merge instead of segfaulting.

- **refs/files: avoid packed-refs lock for root ref deletion** -- Ariel Keselman posted v3 skipping the packed-ref transaction for root-ref deletions like `AUTO_MERGE` and `CHERRY_PICK_HEAD`, fixing post-commit cleanup errors in linked worktrees with read-only shared metadata.

- **advice.defaultBranchName global scope** -- Vsevolod Myalitsin's series makes the `defaultBranchName` advice recommend disabling itself in global configuration; Junio C Hamano agreed adding "--global" to the hint is acceptable but suggested the message could simply always say "--global".

- **http.sslVerifyStatus** -- Grayson Gordon's patch adds the `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`, defaulting to false; he said he would assemble a combined patch taking the best of both test sets.

- **uploadpack.lazyFetchTrusted** -- Christian Couder posted v3 of the series adding the protected config variable for trusted repos, reworking `path_allowlist_apply()` around a caller-supplied callback and adding `GIT_INTERNAL_LAZY_FETCH_DEPTH` to stop infinite recursion.

- **ref storage format terminology** -- Patrick Steinhardt posted v3 of the 13-patch series unifying ref storage terminology with `--ref-storage-format=` accepting `format://payload` URIs and hidden `OPT_ALIAS()` support for old names.