## Long threads

### history: add squash subcommand to fold a range
*Long thread: 91 replies so far from 9 participants over 87 days.*
Harald Nordgren posted v15 of the series adding `git history squash <revision-range>` to fold commits in `builtin/history.c`, fixing `istarts_with()` to `starts_with()` so mixed-case hex in `fixup!` subjects is rejected, correcting a `squshed` typo, and updating completion tests. Junio C Hamano noted that with the "complete 'git history'" topic merged, an evil merge for the completion test change is no longer needed. The series remains the most-discussed topic of the week.

### receive-report hook for git-receive-pack
*Long thread: 57 replies so far from 7 participants over 25 days.*
Karthik Nayak's series adds a `receive-report` hook running after the reference transaction but before the client report, reading the pkt-line report on stdin with stdout replacing it and non-zero exit rejecting all refs. The week saw v8 through v10, fixing the `UNKOWN` spelling, a `BUG()` when no report is requested, and a leak in `override_cmds_error()`. Junio C Hamano objected to the `BUG()` in the switch default case, proposing `case REPORT_STATUS_UNKNOWN: break;` instead, and later supplied a coccinelle rule for the `FREE_AND_NULL()` guard. Patrick Steinhardt expressed satisfaction with the direction.

### Convert build-time USE_NSEC to runtime core.useNanosec
*Long thread: 46 replies so far from 6 participants over 36 days.*
D. Ben Knoble's series exposes `USE_NSEC` as the runtime `core.useNanosec` config, letting distributed builds use nanosecond index timestamps to avoid racy-git false positives. After a v7 resend differing only in a documentation typofix, Junio C Hamano queued the three patches, noting they matched what he had already queued from v6.

### Write object-database alternates at creation time
*Long thread: 36 replies so far from 5 participants over 18 days.*
Patrick Steinhardt's nine-patch series refactors `init_db()` into `create_repository()` and moves alternate writing into `odb_source_create_on_disk()`, removing the ad-hoc `odb_source_write_alternate()` callback. The week saw v4 and v5, with Justin Tobler reviewing v4 as good and Steinhardt addressing comments on sticky file-stream error indicators and declining to formalize backend errors for alternates.

### Add path keys to git repo info
*Long thread: 34 replies so far from 5 participants over 58 days.*
K Jayatheerth's GSoC series adds seven `path.*` keys to `git repo info` covering toplevel, superproject-root, hooks, index, grafts, git-prefix and cdup. v6 refactored `get_superproject_working_tree()` to take a `struct repository *`. Junio C Hamano noted dead null checks after `repo_get_index_file()` and suggested splitting patch 2/7 into a fix with tests plus the new keys.

### sequencer: run auto maintenance once a sequence is done
*Long thread: 27 replies so far from 5 participants over 8 days.*
Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, from `builtin/rebase.c` and `builtin/revert.c`, while passing `maintenance.auto=false` to spawned commands. v3 and v4 condensed commit messages at Junio's request; Phillip Wood called the series pretty good. Patrick Steinhardt later objected to the proposed comment for `git_config_append_parameter()` and argued for calling auto maintenance in `pick_commits()` instead.

### Git 3.0: restrict hex object IDs to lowercase only
*Long thread: 26 replies so far from 6 participants over 41 days.*
brian m. carlsson's series removes Git's acceptance of uppercase hex in object IDs for Git 3.0. v2 fixed a hex-range comment, updated commit messages, moved t5324 fixes to a separate commit, and restored lowercasing in `parse_oid_prefix`. Phillip Wood questioned whether refnames ending in object-ID-like strings should also be forbidden.

### last-modified Bloom filter pre-filtering
*Long thread: 26 replies so far from 5 participants over 56 days.*
Toon Claes's series speeds up `git last-modified` by reusing the revision machinery's Bloom filter checks, showing about 1.95x speedup with `-t` and 3.74x without on ziglang/zig. Patrick Steinhardt reviewed v4, noting `bloom_filter_contains()` can return -1 for empty filters, suggesting a rename to `..._with_leading_dirs()`, and proposing to store `bloom_filter_settings` in `struct last_modified` since `prepare_revision_walk()` clears them.

### Introduce uploadpack.lazyFetchTrusted
*Long thread: 25 replies so far from 3 participants over 62 days.*
Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config (global/system scope only) setting `GIT_NO_LAZY_FETCH=0` for trusted repos, motivated by large-object promisors. v3 reworks `path_allowlist_apply()` around a callback and adds `GIT_INTERNAL_LAZY_FETCH_DEPTH` (max 5) to stop infinite recursion. Junio C Hamano preferred a plain `int` depth counter and questioned the config overriding `GIT_NO_LAZY_FETCH`.

### http: add http.sslVerifyStatus to check stapled OCSP responses
*Long thread: 21 replies so far from 4 participants over 29 days.*
Grayson Gordon's patch adds `http.sslVerifyStatus` mapping to `CURLOPT_SSL_VERIFYSTATUS`, defaulting false. Patrick Steinhardt supplied OCSP test infrastructure, and Junio C Hamano concluded a combined patch taking the best of both test sets is wanted; Gordon agreed to assemble it.

### Advice scoping for the defaultBranchName hint
*Long thread: 20 replies so far from 4 participants over 5 days.*
Vsevolod Myalitsin's three-patch series makes the `defaultBranchName` advice recommend global configuration, since the local-scope hint is useless during `git init`. v4 passes the whole `advice_setting` to `vadvise()` with a scope hint set to `CONFIG_SCOPE_GLOBAL`. Junio C Hamano agreed adding "--global" is acceptable but suggested the message could simply always say "--global" without marking each advice individually.

## The period in brief
The week of 2026/09/07 through 2026/09/13 saw heavy traffic across all seven days, anchored by the Git v2.56.0-rc0 release candidate and an active debate over whether the next version should be 3.0. Long-running series dominated the discussion, with the `git history squash` command, the `receive-report` hook, and the runtime `core.useNanosec` config all drawing sustained review attention. Readers should not miss the v2.56.0-rc0 announcement, the post-2.56 versioning discussion centered on GitHub SHA-256 support, and the fix for `git push --force-if-includes` consulting the wrong ref.

## Key developments

### Git v2.56.0-rc0 and the road to 3.0
Junio C Hamano announced Git v2.56.0-rc0 on 2026/09/10, comprising 635 non-merge commits since v2.55.0 from 82 people including 32 new contributors. The same week, Hamano asked whether the release after 2.56 should be 3.0, 2.99, or business as usual. Patrick Steinhardt called GitHub's missing SHA-256 support the biggest question mark for a 3.0 bump, noting reftable is upstreamed in libgit2 with SHA-256 default-enabled, so both blockers should clear with the next libgit2 release. Emily Shaffer leaned toward 3.0, saying Google would disable SHA-256 for new repos via system config.

### push --force-if-includes consulting the wrong ref
Tyler Cipriani posted v2 of a fix making `git push --force-if-includes` consult the reflog of the ref actually being pushed (`ref->peer_ref`) rather than a same-named local branch, explicitly rejecting detached HEAD and skipping deletions. Patch 2/2 adds `REF_STATUS_REJECT_UNVERIFIABLE` with advice pointing to `--force-with-lease` or `--no-force-if-includes`. Review on 2026/09/11 questioned deletion handling and missing tests; Cipriani committed to a v4 generalizing the reflog check beyond HEAD and resolving symrefs.

### git var extensions for idents, signing key, and multiple args
Andrew Pleeter advanced his series teaching `git var` to expose `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, committer equivalents, `GIT_SIGNING_KEY`, multiple variables per call, and `-z`, reaching v7 by 2026/09/10. Phillip Wood pressed for the use case for `GIT_SIGNING_KEY` and objected to non-zero exit when one variable is unset; Junio C Hamano proposed modeling multi-variable output on `git var -l` var=value pairs rather than empty records.

### Rust integration: gitcore crate move and Windows CI
Mike Hommey posted v4 moving the Rust `gitcore` crate into a `rust/` subdirectory so Git can be vendored as a submodule without a top-level `Cargo.toml`; Junio C Hamano reported CI still showed inconsistency. Separately, Johannes Schindelin's series enabling Rust in Windows CI jobs reached v4 using `CARGO_BUILD_TARGET`, and Hamano judged it ready, merging it to `next` and fast-tracking to `master` on 2026/09/13.

### Git for Windows upstreaming series
Johannes Schindelin posted v4 of a 13-patch series upstreaming Git for Windows patches needed for the UCRT64 switch, reinstating a fly-by fix as a separate commit. Hamano acknowledged the reroll and, with Johannes Sixt's earlier blessing, marked the series for `next`.

### Bundle bitmap bug and ODB consistency checks
Taylor Blau diagnosed that `git bundle create` with bitmaps omits a tree required by an advertised ref when the included commit shares a root tree with an excluded sibling, proposing "bundle: restrict pack haves to recorded prerequisites". Patrick Steinhardt posted v3 of a ten-patch series making object-database consistency checks pluggable, moving fsck checks into ODB sources; Toon Claes suggested each source decide on `ODB_FSCK_FULL` itself.

### Worktree repair validation and pull crash fix
Yoichi NAKAYAMA submitted a two-patch series validating `git worktree repair` targets by reading the `.git` file once and verifying the worktree ID matches, preventing corruption when directories are swapped. Jiri Kuncar added NULL guards in `builtin/pull.c` for `lookup_commit_reference()` so invalid merge heads from racing fetches or gc fall back to a normal merge instead of segfaulting.

### Transaction packfile flush before migration
Justin Tobler posted a two-patch series fixing a commit failure when an ODB transaction contains both a loose object and a large blob with `core.fsyncMethod=batch`, moving the packfile flush ahead of object migration in `odb_transaction_files_commit()`.

## In brief
**git pull overwrites local directory with symlink** -- AIKSXD ax reported that `git pull` silently replaces a local directory with a committed symlink of the same name when a `.gitignore` pattern like `dataset/` is present; brian m. carlsson explained the trailing-slash pattern ignores contents rather than the entry itself.

**worktree lifecycle hooks** -- Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks; Kristoffer Haugsbakk proposed a `--id=<string>` ownership alternative, which Kožar argued does not cover tools observing the lifecycle regardless of which tool invokes Git.

**precompiling git-compat-util.h** -- SZEDER Gábor posted a four-patch Makefile series precompiling `git-compat-util.h` for about a 35% build speedup, with patch 3/4 reintroducing `REFTABLE_OBJS` to exclude reftable sources.

**history: sign rewritten commits** -- Souma posted v3 teaching `git history` rewrite commands to honor `commit.gpgSign` and `-S`/`--gpg-sign`, adding a `sign_commit` field to `replay_revisions_options`.

**refs/files: avoid packed-refs lock for root ref deletion** -- Ariel Keselman posted v3 skipping the packed-refs lock when deleting root refs like `AUTO_MERGE`, fixing post-commit cleanup errors in linked worktrees with read-only shared metadata.

**Use the system asciidoctor in CI** -- Jeff King's patch drops the pinned asciidoctor 1.5.8 gem in favor of the Ubuntu package; Hamano decided to queue it after discovering the third patch of the series was never applied.

**midx-write: skip writes with no object entries** -- Pia Park posted v2 moving the empty-entries check so all write modes return silent success instead of publishing a zero-object MIDX.

**imap-send OpenSSL 4.1 preparation** -- Beat Bolli posted a three-patch series covering OpenSSL 4.1 preparation, ASN1_STRING NUL-termination, and RFC 6125 common-name checking.

**Outreachy application** -- Christian Couder submitted Git's Outreachy Community Application for the December 2026 cohort, offering to mentor two projects.

**git worktree add "" on Windows** -- Paul DE TEMMERMAN reported that `git worktree add "" HEAD` on Windows deletes the entire workspace including `.git` instead of failing safely.

**stash.index redundant entry** -- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry.

**range-diff-notes redesign** -- Kristoffer Haugsbakk proposed only `--no-RD-notes` and `--RD-notes=<ref>` for his redesign, accepted by Hamano.

**bash completion for worktree repair** -- Yoichi NAKAYAMA added completion for `git worktree repair`, completing paths to linked worktrees or those with broken links.

**documentation lint and quoting** -- Todd Zullinger posted a series fixing backtick-quoting in `git-refs` and `git-pack-refs` docs; Jean-Noël AVILA argued against extending the lint regex to commands due to false positives in definition lists.

**merge-ll temporary cleanup** -- Michal Koutný submitted a patch cleaning up `.merge_file_*` temporaries after Ctrl+C; Jeff King suggested using the tempfile cleanup handler and flagged an LLP64 narrowing hazard.

**git-blame ignore-revs default** -- Ravi Mistry proposed making `git blame` default to a `.git-blame-ignore-revs` file at the working-tree root.

**CARGO_BUILD_TARGET for Rust build** -- James Le Cuirot posted v3 making the Rust build respect `CARGO_BUILD_TARGET` when locating `libgitcore.a`.

**fetch-pack trace2 region** -- Hamano decided to queue Ted Nyman's "fetch-pack: trace packfile URI downloads" adding a Trace2 region around the download loop.

**history reword memory leak** -- Kaartic Sivaraam posted v3 adding the missing `repo_unuse_commit_buffer()` call fixing an AddressSanitizer leak in `builtin/history.c`.

## Looking ahead
The `receive-report` hook series reached v10 with the leak fix and coccinelle rule discussion; expect a v11 or graduation to `next` once the `REPORT_STATUS_UNKNOWN` handling is settled. The `git history squash` series at v15 and the `uploadpack.lazyFetchTrusted` series at v3 both remain in active review. The post-2.56 versioning discussion will likely continue as libgit2's next release approaches, with GitHub SHA-256 support as the central open question. Tyler Cipriani's promised v4 of the `--force-if-includes` fix and Andrew Pleeter's `git var` series at v7 will see further review in the coming week.