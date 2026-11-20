## Long threads

### history: add squash subcommand to fold a range

*Long thread: 91 replies so far from 9 participants over 87 days.*

Harald Nordgren posted v15 of the series adding `git history squash <revision-range>` to fold a range of commits in `builtin/history.c`, fixing `istarts_with()` to `starts_with()` in `squash_check_autosquash_subject()` so mixed-case hex in `fixup!` subjects is no longer accepted, correcting a `squshed` typo, and adding `squash` to the completion test. Junio C Hamano noted that with the "complete 'git history'" topic now integrated, he no longer needs an evil merge for the `t9902-completion.sh` change. The series remains under active review with the core functionality stabilizing.

### receive-report hook for git-receive-pack

*Long thread: 57 replies so far from 7 participants over 25 days.*

Karthik Nayak's series adds a `receive-report` hook in `builtin/receive-pack.c` that runs after the reference transaction completes but before the report reaches the client; the hook reads the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. The series progressed from v7 through v10 during the week, fixing a `BUG()` when no report was requested, a leak in `override_cmds_error()`, and the `UNKOWN` spelling. Junio C Hamano objected to a `BUG()` in the switch default case and proposed `case REPORT_STATUS_UNKNOWN: break;` instead, while Oswald Buddenhagen suggested trimming verbose commit messages. The series appears close to convergence.

### Convert build-time USE_NSEC to runtime core.useNanosec

*Long thread: 46 replies so far from 6 participants over 36 days.*

D. Ben Knoble's series exposes `USE_NSEC` as the runtime `core.useNanosec` config so distributed builds can use nanosecond index timestamps and avoid racy-git false positives. Knoble resent v7 differing from v6 only in a documentation typofix, and Junio C Hamano queued the three patches, noting they matched what he had already queued from v6 with the typo reworded locally. The resend was mainly a nudge about the stale What's Cooking status. The series is now queued.

### Write object-database alternates at creation time

*Long thread: 36 replies so far from 5 participants over 18 days.*

Patrick Steinhardt's nine-patch series refactors `init_db()` into `create_repository()` and moves alternate writing into `odb_source_create_on_disk()`, removing the ad-hoc `odb_source_write_alternate()` callback. The series advanced to v5 during the week, renaming `collect_one_alternate`/`collect_alternates_data`, dropping an extra parameter, and fixing a "refernce" typo after Karthik Nayak's review. Steinhardt answered review comments on the alternates patch, noting the file-stream error indicator is sticky so per-`fprintf` checks are unnecessary, and declined to formalize backend errors for alternates since nothing would use them yet. Justin Tobler had previously called v4 good.

### Add path keys to git repo info

*Long thread: 34 replies so far from 5 participants over 58 days.*

K Jayatheerth's GSoC series adds seven `path.*` keys to `git repo info`: toplevel, superproject-root, hooks, index, and grafts with absolute and relative suffixes, plus git-prefix and cdup as single keys. He posted v6, refactoring `get_superproject_working_tree()` to take a `struct repository *` instead of the process working directory. Junio C Hamano noted dead null checks after `repo_get_index_file()` in patch 4/7, asked what `path.cdup` returns outside the working tree, and suggested splitting patch 2/7 into a fix with tests plus the new keys.

### sequencer: run auto maintenance once a sequence is done

*Long thread: 27 replies so far from 5 participants over 8 days.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, from `builtin/rebase.c` and `builtin/revert.c`, while passing `maintenance.auto=false` to sequencer-spawned commands via `GIT_CONFIG_PARAMETERS` to avoid lock contention. The series reached v4 with commit messages condensed to about a third at Junio C Hamano's request, who said "Will queue". Patrick Steinhardt objected that the proposed comment for `git_config_append_parameter()` was too specific to `-c key=value`, and disputed the claim that the sequencer lacks a single exit point, arguing cherry-pick, revert, rebase, `--continue` and `--skip` all reach `pick_commits()`; he prefers calling auto maintenance there and on abort.

### last-modified Bloom filter pre-filtering

*Long thread: 26 replies so far from 5 participants over 56 days.*

Toon Claes's series speeds up `git last-modified` by reusing the revision machinery's Bloom filter checks to pre-filter commits, with benchmarks on ziglang/zig showing about 1.95x speedup with `-t` and 3.74x without. Patrick Steinhardt reviewed v4, noting `bloom_filter_contains()` can return -1 for an empty filter while the docs cover only 0 or 1, suggesting renaming `revs_maybe_changed_in_bloom_with_parents()` to `..._with_leading_dirs()`, asking for the performance rationale, and questioning restoring `bloom_filter_settings` after `prepare_revision_walk()` clears them, proposing to store them in `struct last_modified` instead.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 26 replies so far from 6 participants over 41 days.*

brian m. carlsson's series removes Git's acceptance of uppercase hex in object IDs for Git 3.0, since Git has always emitted lowercase and software assuming a unique hex form can break or be exploited by uppercase input. Phillip Wood said the object-ID-versus-ref case made more sense to him and wondered whether refnames whose last component looks like an object ID should also be forbidden. Carlsson posted v2, fixing a hex-range comment, updating commit messages, moving the t5324 fixes to a separate commit, and restoring lowercasing in `parse_oid_prefix`.

### Introduce uploadpack.lazyFetchTrusted

*Long thread: 25 replies so far from 3 participants over 62 days.*

Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config variable (global/system scope only) that sets `GIT_NO_LAZY_FETCH=0` in `builtin/upload-pack.c` for trusted repos, motivated by large-object promisors where a filtered clone can fail because the server cannot provide blobs in the filtered range. v3 reworks `path_allowlist_apply()` around a caller-supplied `allow_path` callback, identifies repos by git dir with absolute paths only, and adds `GIT_INTERNAL_LAZY_FETCH_DEPTH` (max 5) to stop infinite recursion when a promisor remote resolves back to the served repo. The week's exchange concerned the type of the depth counter, with Junio C Hamano noting existing callers mix casts and leaving the choice to Couder.

### ref storage format terminology

*Long thread: 23 replies so far from 4 participants over 6 days.*

Patrick Steinhardt's 13-patch series unifies ref storage terminology: `--ref-format=` becomes `--ref-storage-format=` with old names kept as hidden aliases, alongside matching environment variables and `init.defaultRefStorageFormat`, and `--ref-storage-format=` now accepts `format://payload` URIs. He posted v3 with hidden `OPT_ALIAS()` support, correct environment variables in error messages, and renamed `GIT_TEST_DEFAULT_REF_STORAGE_FORMAT`. Kaartic Sivaraam raised payload-comparison and shared-doc concerns; Steinhardt declined `strbuf_realpath` comparison since a payload need not be a path, and kept the shared `ref-storage-format.adoc` out of the series.

### http: add http.sslVerifyStatus to check stapled OCSP responses

*Long thread: 21 replies so far from 4 participants over 29 days.*

Grayson Gordon's patch adds the `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` in the HTTP transport, defaulting to false so servers without stapled OCSP responses are not blocked. Earlier, Patrick Steinhardt supplied OCSP test infrastructure and Junio C Hamano concluded a combined patch taking the best of both test sets is wanted. Gordon said he would assemble the combined patch and send it over.

### Advice scoping for the defaultBranchName hint

*Long thread: 20 replies so far from 4 participants over 5 days.*

Vsevolod Myalitsin's three-patch series in `advice.c` makes the `defaultBranchName` advice recommend disabling itself in the global configuration, because the current hint writes to the local repository, which is useless since the advice appears while initializing a repository and reappears on the next `git init`. The v4 series passes the whole `advice_setting` to `vadvise()`, adds a scope hint, and sets the global scope for this advice. Junio C Hamano agreed that adding "--global" to the advice hint is acceptable since users can still cut-and-paste and tweak it, but clarified that the original intent was for users to decide the scope themselves, and suggested it may not be necessary to mark each advice individually for scope since the message could simply always say "--global".

## The period in brief

The week of 2026/09/07 through 2026/09/13 saw heavy traffic across all seven days, with Git v2.56.0-rc0 announced mid-week and multiple long-running series reaching convergence or queueing. Readers should not miss the release candidate announcement, the fast-tracking of Rust support in Windows CI to `master`, and the continued refinement of the `receive-report` hook series toward its tenth revision.

## Key developments

### Git v2.56.0-rc0 and the post-2.56 version question

Junio C Hamano announced Git v2.56.0-rc0 for testing on 2026/09/10, comprising 635 non-merge commits since v2.55.0 from 82 people including 32 new contributors. Earlier in the week, Hamano had asked whether the release after 2.56 should be 3.0, 2.99, or business as usual. Patrick Steinhardt noted a Git 2.100 is also possible after 2.99, called GitHub's missing SHA-256 support the biggest question mark for a 3.0 bump, and reported that reftable support is upstreamed in libgit2 with SHA-256 default-enabled. Emily Shaffer said GitHub SHA-256 support leaves her most concerned but noted GitOxide has initial SHA-256 support, leaning toward releasing 3.0.

### Rust integration and Windows CI

Johannes Schindelin's series enabling Rust in Windows CI jobs reached v4, switching from `CARGO_TARGET`/`--target` to `CARGO_BUILD_TARGET`; Junio C Hamano judged v4 ready and decided to merge it in `next` and fast-track it to `master` immediately, reasoning that Windows has an unusually high ratio of binary users relative to source builders. Mike Hommey posted v4 moving the Rust `gitcore` crate into a dedicated `rust/` subdirectory so the git source can be vendored as a submodule without its Rust crate, though Hamano reported v4 still leaves the tree inconsistent per CI. Schindelin also posted v4 of the 13-patch series upstreaming Git for Windows patches needed for the UCRT64 switch, which Hamano said could be marked for `next`.

### Push safety: --force-if-includes fix

Tyler Cipriani posted v2 fixing `--force-if-includes` in push: `check_if_includes_upstream()` now consults the pushed ref's reflog (`ref->peer_ref`) instead of the destination-named local branch, explicitly rejects detached HEAD, and skips deletions. Patch 2/2 adds `REF_STATUS_REJECT_UNVERIFIABLE` and advice pointing to `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`. Patrick Steinhardt and Junio C Hamano reviewed the patches, questioning deletion handling and the HEAD-only check; Cipriani confirmed the reflog lookup behavior, agreed to resolve symrefs to the target branch and reject unverifiable revisions in a v4, and proposed moving a guard into `apply_push_cas`.

### git var extensions

Andrew Pleeter posted successive revisions (v5 through v7) teaching `git var` to expose `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, committer equivalents, `GIT_SIGNING_KEY`, multiple variables per call, and `-z`. Phillip Wood pressed for the use case for `GIT_SIGNING_KEY` and objected to a non-zero exit when one requested variable is unset; Junio C Hamano proposed modeling multi-variable output on `git var -l` var=value pairs rather than the empty-record convention and asked whether the delimiter also appears without `-z`.

### Pluggable object-database consistency checks

Patrick Steinhardt posted v3 of a ten-patch series moving fsck checks into the ODB sources, including loose-object verification in the loose source and reverse-index verification in the packed source. Toon Claes reviewed, suggesting each source decide on `ODB_FSCK_FULL`/`--full` itself and use early returns instead of a `ret` variable; Steinhardt agreed to move the check into the "files" backend, prepare repo settings, and drop the variable.

### Bug fixes: pull crash, bundle bitmaps, and worktree repair

Jiri Kuncar submitted a patch adding NULL guards for `lookup_commit_reference()` in `builtin/pull.c` to avoid a segfault when invalid merge-head references arise from parallel fetches or gc races. Taylor Blau diagnosed a bitmap bug where `git bundle create` omits a tree required by an advertised ref when the included commit shares a root tree with an excluded sibling, proposing "bundle: restrict pack haves to recorded prerequisites". Yoichi NAKAYAMA submitted a two-patch series fixing `git worktree repair` to validate cross-references between a linked working tree and its administrative data, preventing corruption when worktree directories are swapped.

### Build system: precompiling git-compat-util.h

SZEDER Gabor posted a 4-patch series making the Makefile precompile `git-compat-util.h`, reporting about a 35% build speedup; patch 3/4 reintroduces `REFTABLE_OBJS` so reftable sources, which mostly do not include that header, can be excluded. He supplied an optional follow-up covering the remaining files but judged its ~1% gain not worth it, and Junio C Hamano suggested reorganizing patch 3/4's commit message.

### history: sign rewritten commits

Souma posted v3 of a two-patch series teaching the `git history` rewrite commands to honor commit signing. The series adds a `sign_commit` field to `replay_revisions_options` in the replay API and exposes `-S`/`--gpg-sign[=<key-id>]` and `--no-gpg-sign` across `git history drop`, `fixup`, `reword`, and `split`, reading `commit.gpgSign` as the default and applying the selected key to every new commit including both halves of a split and replayed descendants.

## In brief

**git pull overwrites a local directory with a symlink** -- AIKSXD ax reported that with a `.gitignore` pattern like `dataset/` and a committed symlink of the same name, `git pull` silently replaces the local directory with the symlink; brian m. carlsson explained that a trailing-slash pattern ignores everything under the directory rather than the entry itself, and recommended not keeping valuable data in untracked or ignored files.

**worktree lifecycle hooks** -- Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks so tools like devenv can provision per-worktree environments; Kristoffer Haugsbakk proposed instead a `--id=<string>` option on `git worktree add` recording ownership.

**git worktree add "" HEAD on Windows** -- Paul DE TEMMERMAN reported that this command deletes the entire workspace including `.git` instead of failing safely.

**stash.index=true redundant stash** -- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry and deleting `MERGE_AUTOSTASH` fails.

**imap-send series** -- Beat Bolli posted a three-patch series covering OpenSSL 4.1 preparation, ASN1_STRING NUL-termination, and RFC 6125 common-name checking; Patrick Steinhardt questioned the `xmemdupz()` copy in the ASN1_STRING fix.

**midx-write: skip writes with no object entries** -- Pia Park posted v2 moving the empty-entries check so all write modes return silent success instead of publishing a zero-object MIDX; Taylor Blau found silent exit 0 acceptable.

**fetch-pack: trace packfile URI downloads** -- Junio C Hamano decided to queue Ted Nyman's patch adding a Trace2 region around the packfile URI download loop in `fetch-pack.c`.

**git init --ref-format deprecation** -- Kaartic Sivaraam asked whether the old `--ref-format=` should be deprecated or aliased to `--ref-storage-format`, and noted the "unknown ref storage format" error now prints the whole backend URI.

**range-diff-notes redesign** -- Kristoffer Haugsbakk proposed only `--no-RD-notes` and `--RD-notes=<ref>`; Junio C Hamano accepted the design.

**Outreachy Community Application** -- Christian Couder submitted Git's application for the December 2026 cohort, offering to mentor two projects.

**history reword memory leak** -- Patrick Steinhardt proposed a fix for the `history reword` memory leak in `builtin/history.c`; Kaartic Sivaraam posted v3 adding the missing `repo_unuse_commit_buffer()` call, which Jeff King reviewed as good.

**merge-ll temporaries** -- Michal Koutný submitted a patch cleaning up `.merge_file_*` temporaries after Ctrl+C; Jeff King suggested using the tempfile cleanup handler instead, and posted v2 doing so, with Elijah Newren flagging an LLP64 narrowing hazard.

**git-blame default ignore-revs file** -- Ravi Mistry proposed making git-blame default to a `.git-blame-ignore-revs` file at the working-tree root, with opt-outs via config or `--no-ignore-revs-file`.

**NULL checks in repository config getters** -- Patrick Steinhardt argued against retaining NULL checks in repository config getters, proposing a redesign before further iteration.

**packed-refs lock for root ref deletion** -- Ariel Keselman posted v3 skipping the packed-refs lock when deleting root refs such as `AUTO_MERGE` in the files backend, fixing post-commit cleanup errors in linked worktrees with read-only shared metadata; Patrick Steinhardt suggested stronger test assertions.

**global-config forward-slash** -- Delilah Ashley Wu proposed dropping the forward-slash patch in her global-config series and instead setting a forward-slash `XDG_CONFIG_HOME` in the tests.

**bash completion for worktree repair** -- Yoichi NAKAYAMA added bash completion for the `git worktree repair` subcommand, completing paths to linked worktrees or worktrees with broken links.

**documentation backtick-quoting** -- Todd Zullinger posted a three-patch series fixing inconsistent backtick-quoting in the `git-refs` and `git-pack-refs` documentation; Jean-Noël AVILA argued against extending the lint regex to commands, citing false positives, and Zullinger said he would drop the lint patch.

**macOS CI gettext link** -- Harald Nordgren submitted a patch to remove `brew link --force gettext` from macOS CI, which only produced a spurious "Already linked" warning, and scoped perforce, git-lfs, and JGit presence warnings to the platforms that actually install them.

**CMake UCRT64** -- Johannes Schindelin changed `contrib/buildsystems/CMakeLists.txt` so the CMake Windows x64 build defines `ENSURE_MSYSTEM_IS_SET="UCRT64"` and `MINGW_PREFIX="ucrt64"`.

**Perl Windows test fixes** -- Johannes Schindelin fixed two Perl-related Windows test failures: `t9700` now checks `$ENV{MSYSTEM}` because MSYS2 Perl reports `$^O` as `cygwin`, and `t9129` skips its UTF-8 cases on `MINGW`.

**coccinelle free(E) rule** -- René Scharfe questioned a coccinelle rule converting `if (!E) free(E);` into an unconditional `free(E)`, arguing it turns a no-op into a potentially damaging call; Junio C Hamano asked whether rewriting it as an explicit no-op comment would trip `-Werror`, and Scharfe noted GCC accepts `if (!E);` while Clang warns.

**fetch --prune-tags tests** -- Orgad Shaneh pinged Junio C Hamano about tag content verification in the `fetch --prune-tags` tests; Hamano replied he needs to swap the old discussion back into context.

## Looking ahead

The `receive-report` hook series appears poised to land soon after ten revisions, with remaining discussion focused on commit message verbosity and a coccinelle cleanup. The sequencer auto-maintenance series was queued by Junio C Hamano but Patrick Steinhardt's objection about the call site may prompt a follow-up. The post-2.56 version discussion will likely continue as GitHub's SHA-256 support and libgit2's next release clarify the path toward Git 3.0.