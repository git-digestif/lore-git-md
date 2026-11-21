## Long threads

### history: add squash subcommand to fold a range
*Long thread: 91 replies so far from 9 participants over 87 days.*
Harald Nordgren posted v15 of the series adding `git history squash <revision-range>` to fold commits in `builtin/history.c`, fixing `istarts_with()` to `starts_with()` so mixed-case hex in `fixup!` subjects is no longer accepted, correcting a `squshed` typo, and adding subcommand completion tests. Junio C Hamano noted that with the "complete 'git history'" topic now merged, he no longer needs an evil merge for the `t9902-completion.sh` change. The series remains under active review with the core functionality stabilizing.

### receive-report hook for git-receive-pack
*Long thread: 57 replies so far from 7 participants over 25 days.*
Karthik Nayak's series adds a `receive-report` hook in `builtin/receive-pack.c` that runs after the reference transaction completes but before the report reaches the client; the hook reads the pkt-line report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. The series progressed through v10, fixing a `BUG()` when no report was requested, a leak in `override_cmds_error()`, and the `UNKOWN` spelling. Junio C Hamano objected to the `BUG()` in the switch's default case and proposed `case REPORT_STATUS_UNKNOWN: break;` instead, while Patrick Steinhardt expressed satisfaction with the direction.

### Convert build-time USE_NSEC to runtime core.useNanosec
*Long thread: 46 replies so far from 6 participants over 36 days.*
D. Ben Knoble's series exposes `USE_NSEC` as the runtime `core.useNanosec` config so distributed builds can use nanosecond index timestamps and avoid racy-git false positives. Knoble resent v7 differing from v6 only in a documentation typofix, and Junio C Hamano queued the three patches, noting they matched what he had already queued from v6 with the typo reworded locally. The resend served mainly as a nudge about the stale What's Cooking status.

### Write object-database alternates at creation time
*Long thread: 36 replies so far from 5 participants over 18 days.*
Patrick Steinhardt's nine-patch series refactors `init_db()` into `create_repository()` and moves alternate writing into `odb_source_create_on_disk()`, removing the ad-hoc `odb_source_write_alternate()` callback. The series reached v5, renaming `collect_one_alternate`/`collect_alternates_data`, dropping an extra parameter, and fixing a "refernce" typo. Justin Tobler reviewed v4 as looking good, and Steinhardt addressed review comments about sticky file-stream error indicators and declined to formalize backend errors for alternates since nothing would use them yet.

### Add path keys to git repo info
*Long thread: 34 replies so far from 5 participants over 58 days.*
K Jayatheerth's GSoC series adds seven `path.*` keys to `git repo info`: toplevel, superproject-root, hooks, index, and grafts with absolute and relative suffixes, plus git-prefix and cdup as single keys. He posted v6 refactoring `get_superproject_working_tree()` to take a `struct repository *` instead of the process working directory. Junio C Hamano noted dead null checks after `repo_get_index_file()` in patch 4/7, asked what `path.cdup` returns outside the working tree, and suggested splitting patch 2/7 into a fix with tests plus the new keys.

### sequencer: run auto maintenance once a sequence is done
*Long thread: 27 replies so far from 5 participants over 8 days.*
Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, from `builtin/rebase.c` and `builtin/revert.c`, while passing `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS`. The series reached v4 with commit messages condensed to about a third at Junio C Hamano's request; earlier he had said "Will queue". Patrick Steinhardt objected that the proposed comment for `git_config_append_parameter()` was too specific to `-c key=value` and disputed the claim that the sequencer lacks a single exit point, preferring to call auto maintenance in `pick_commits()` and on abort.

### Git 3.0: restrict hex object IDs to lowercase only
*Long thread: 26 replies so far from 6 participants over 41 days.*
brian m. carlsson's series removes Git's acceptance of uppercase hex in object IDs for Git 3.0, since Git has always emitted lowercase and software assuming a unique hex form can break or be exploited by uppercase input. He posted v2 fixing a hex-range comment, updating commit messages, moving the t5324 fixes to a separate commit, and restoring lowercasing in `parse_oid_prefix`. Phillip Wood said the object-ID-versus-ref case made more sense to him and wondered whether refnames whose last component looks like an object ID should also be forbidden.

### last-modified Bloom filter pre-filtering
*Long thread: 26 replies so far from 5 participants over 56 days.*
Toon Claes's series speeds up `git last-modified` by reusing the revision machinery's Bloom filter checks to pre-filter commits, with benchmarks on ziglang/zig showing about 1.95x speedup with `-t` and 3.74x without. Patrick Steinhardt reviewed v4, noting `bloom_filter_contains()` can return -1 for an empty filter while the docs cover only 0 or 1, suggesting renaming `revs_maybe_changed_in_bloom_with_parents()` to `..._with_leading_dirs()`, and questioning restoring `bloom_filter_settings` after `prepare_revision_walk()` clears them, proposing to store them in `struct last_modified` instead.

## The period in brief

The week of 2026/09/07 through 2026/09/13 saw seven active days of heavy traffic, with major series iterating rapidly and the v2.56.0-rc0 release candidate announced mid-week. Readers should not miss the v2.56.0-rc0 announcement with 635 non-merge commits, the receive-report hook series reaching v10 with a leak fix, and the post-2.56 version discussion weighing whether the next release should be 3.0.

## Key developments

### Git v2.56.0-rc0 and the road to 3.0
Junio C Hamano announced Git v2.56.0-rc0 for testing, comprising 635 non-merge commits since v2.55.0 from 82 people including 32 new contributors. The accompanying "What will come after Git 2.56?" thread drew concrete positions on whether the next release should be 3.0, 2.99, or business as usual. Patrick Steinhardt called GitHub's missing SHA-256 support the biggest question mark for a 3.0 bump, while Emily Shaffer noted Google would disable SHA-256 for new repos via system config, making her lean toward releasing 3.0.

### git bundle create with bitmaps omits required trees
Peter Elmers reported that `git bundle create` with bitmaps omits a tree required by an advertised ref when the included commit shares a root tree with an excluded sibling. Taylor Blau reproduced the issue, diagnosed that the non-bitmap traversal overcounts objects past the boundary while the bitmap walk builds an exact answer, and proposed "bundle: restrict pack haves to recorded prerequisites" with a new bitmap test.

### push --force-if-includes consulting the wrong ref
Tyler Cipriani posted v2 fixing `--force-if-includes` in push: `check_if_includes_upstream()` now consults the pushed ref's reflog (`ref->peer_ref`) instead of the destination-named local branch, explicitly rejects detached HEAD, and skips deletions. He later committed to a v4 generalizing the reflog check beyond HEAD after review from Patrick Steinhardt and Junio C Hamano questioned deletion handling and missing test coverage.

### Git for Windows upstreaming and Windows CI Rust
Johannes Schindelin posted v4 of the 13-patch series upstreaming Git for Windows patches needed for the UCRT64 switch, which Junio C Hamano said could be marked for `next`. Separately, Schindelin's series enabling Rust in Windows CI jobs reached v4, and Junio judged it ready, deciding to merge it in `next` and fast-track it to `master` immediately, reasoning that Windows has an unusually high ratio of binary users relative to source builders.

### ODB transaction packfile flush ordering
Justin Tobler posted a two-patch series fixing a commit failure in `object-file.c` when an ODB transaction contains both a loose object and a "large" blob with `core.fsyncMethod` set to `batch`: the transaction's temporary directory was migrated to the main ODB before the packfile was flushed, producing a rename error. The fix moves the packfile flush ahead of the object migration in `odb_transaction_files_commit()` and adds a `t/t1050-large.sh` test.

### Validating git worktree repair targets
Yoichi NAKAYAMA submitted a two-patch series fixing `git worktree repair`: the command did not sufficiently validate cross-references between a linked working tree and its administrative data, so it could modify the wrong `.git` file or gitdir. The patches refactor `repair_worktree_at_path()` to read the `.git` file once via `read_gitfile_raw()` and verify the recorded worktree ID matches the worktree being repaired.

### history: sign rewritten commits
Souma posted v3 of a two-patch series teaching the `git history` rewrite commands to honor commit signing. The series adds a `sign_commit` field to `replay_revisions_options` and exposes `-S`/`--gpg-sign[=<key-id>]` and `--no-gpg-sign` across `git history drop`, `fixup`, `reword`, and `split`, reading `commit.gpgSign` as the default and applying the selected key to every new commit including both halves of a split.

### git var extensions
Andrew Pleeter progressed through v5 to v7 of the `git var` extension adding `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, committer equivalents, `GIT_SIGNING_KEY`, multiple variable arguments, and `-z`. Phillip Wood pressed for the use case for `GIT_SIGNING_KEY` and objected to a non-zero exit when one requested variable is unset, while Junio C Hamano proposed modeling multi-variable output on `git var -l` var=value pairs.

## In brief

- **git pull overwrites a local directory with a symlink** -- AIKSXD ax reported that with a `.gitignore` pattern like `dataset/` and a committed symlink of the same name, `git pull` silently replaces the local directory with the symlink; brian m. carlsson explained the trailing-slash pattern semantics and recommended not keeping valuable data in untracked files.
- **worktree lifecycle hooks** -- Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks, while Kristoffer Haugsbakk proposed a `--id=<string>` ownership option instead.
- **precompiling git-compat-util.h** -- SZEDER Gábor posted a 4-patch Makefile series reporting about a 35% build speedup by precompiling the header.
- **moving the Rust gitcore crate into rust/** -- Mike Hommey posted v4 moving `Cargo.toml` and Rust sources into a `rust/` subdirectory so Git can be vendored as a submodule; Junio C Hamano reported v4 still leaves the tree inconsistent per CI.
- **system asciidoctor in CI** -- Jeff King's patch drops the pinned asciidoctor 1.5.8 gem install in favor of the Ubuntu image's package; Junio C Hamano decided to queue it after discovering it was never picked up.
- **pull crash on invalid merge head** -- Jiri Kuncar submitted a patch adding NULL guards for `lookup_commit_reference()` in `builtin/pull.c` so invalid merge-head references from parallel fetches or gc races fall back to a normal merge instead of segfaulting.
- **refs/files packed-refs lock for root ref deletion** -- Ariel Keselman posted v3 skipping the packed-refs lock when deleting root refs like `AUTO_MERGE`, fixing post-commit cleanup errors in linked worktrees with read-only shared metadata.
- **uploadpack.lazyFetchTrusted** -- Christian Couder's v3 adds the protected config variable for trusted repos with a `GIT_INTERNAL_LAZY_FETCH_DEPTH` recursion guard; Junio C Hamano questioned a config variable overriding `GIT_NO_LAZY_FETCH`.
- **http.sslVerifyStatus** -- Grayson Gordon's patch adds the boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` for stapled OCSP responses, defaulting to false.
- **ref storage format terminology** -- Patrick Steinhardt's v3 unifies `--ref-format=` into `--ref-storage-format=` with hidden aliases and URI payload support.
- **pluggable ODB consistency checks** -- Patrick Steinhardt's v3 moves fsck checks into the ODB sources, with Toon Claes suggesting each source decide on `ODB_FSCK_FULL` itself.
- **advice.defaultBranchName global scope** -- Vsevolod Myalitsin's series makes the hint recommend `--global` configuration; Junio C Hamano agreed this is acceptable but suggested the message could simply always say "--global".
- **imap-send OpenSSL 4.1 preparation** -- Beat Bolli posted a three-patch series covering OpenSSL 4.1 preparation, ASN1_STRING NUL-termination, and RFC 6125 common-name checking.
- **stash.index redundant entry** -- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry.
- **worktree add "" on Windows** -- Paul DE TEMMERMAN reported that `git worktree add "" HEAD` on Windows deletes the entire workspace including `.git`.
- **bash completion for git worktree repair** -- Yoichi NAKAYAMA added completion for paths to linked worktrees or worktrees with broken links.
- **documentation lint and quoting** -- Todd Zullinger posted fixes for inconsistent backtick-quoting in `git-refs` and `git-pack-refs` docs; Jean-Noël Avila argued against extending the lint regex to commands.