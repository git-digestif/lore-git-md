## Long threads

### `git history squash` subcommand for folding a commit range
*Long thread: 90 replies so far from 9 participants over 73 days.*

Harald Nordgren's series adding `git history squash <revision-range>` to `builtin/history.c` reached v14 during August, folding commit ranges into a single commit while preserving the oldest commit's authorship and parents. The series was ejected from `next` early in the month after Phillip Wood's review exposed fundamental reachability flaws with `UNINTERESTING` commits, root commits, and multi-tip histories, sparking a broader discussion about review quality and integration pipeline policy. Phillip Wood developed fixups that were incorporated into later revisions, and the message-editing debate converged on an always-open-editor approach after Phillip conceded his context-sensitive default proposal. By month's end, v14 addressed sign-off ordering and case-sensitivity in `fixup!` subject matching, with Junio C Hamano noting that `starts_with()` should be used rather than `istarts_with()` to avoid accepting uppercase hex object IDs.

### `git-replay(1)` gains a --linearize option
*Long thread: 38 replies so far from 7 participants over 85 days.*

Toon Claes's series adding `--linearize` to `git-replay(1)` reached v9, restricting the option to single-branch usage after Elijah Newren and others identified multi-branch ambiguity. The option drops merge commits and replays all reachable commits into a single linear history, updating refs pointing at merges to their last replayed ancestor. v9 rephrased error messages from "multiple revision ranges" to "multiple branches" and simplified the documentation of the restriction, addressing reviewer requests for clearer wording about the limitation.

### Moving libgit.a sources into a `lib/` directory
*Long thread: 38 replies so far from 12 participants over 124 days.*

Patrick Steinhardt's RFC proposing to move roughly 700 libgit.a source files into a `lib/` directory to declutter the repository root continued through August. The debate centered on discoverability for newcomers versus the ongoing burden of rename fallout, with Junio C Hamano arguing that `lib/` is an arbitrary intermediate layer that fails to deliver meaningful organization, while Johannes Schindelin and brian m. carlson defended the bulk move. Michael Montalbo proposed a script-based grouping of the 231 libgit.a source files into 14 directories using weighted commit-history labels and cscope call-graph checks, placing 163 files into groups and leaving 68 ungrouped. Junio suggested carving conceptually clean groups into their own directories as was done with `builtin/` or `refs/`, without waiting to classify the remainder.

### Pluggable packfile writes for `git-receive-pack`
*Long thread: 35 replies so far from 3 participants over 16 days.*

Justin Tobler's series introducing `odb_transaction_write_pack()` reached v5, making `git-receive-pack`'s packfile writes backend-agnostic through the ODB transaction layer rather than child processes tied to the "files" backend. Patrick Steinhardt approved the v5 series, which uses transaction flags like `ODB_TRANSACTION_RECEIVE` to differentiate callers and adds test assertions verifying that `.keep` files are migrated to the main ODB before removal at finalize. Junio C Hamano questioned reading `receive.unpackLimit` in generic object-layer code as a layering violation, suggesting `odb-file.unpackLimit` as a more accurate name.

### `git repack --drop-filtered` for partial clones
*Long thread: 35 replies so far from 4 participants over 30 days.*

Siddharth Shrimali's GSoC series adding `--drop-filtered` to `git repack` reached v5, allowing partial clones to safely remove locally cached promisor blobs exceeding a size threshold. Christian Couder reviewed v5 positively, and Junio asked whether to mark the topic for `next`. The series concedes that merge/rebase/cherry-pick checks are optional optimizations rather than correctness measures, while Junio's proposed time-based heuristic for avoiding recently fetched objects remains deferred to follow-up work.

### Ambiguous remote branch messages in `worktree add` and `checkout`
*Long thread: 32 replies so far from 6 participants over 20 days.*

Yoichi NAKAYAMA's series improving error messages when `git worktree add` or `git checkout` matches multiple remote-tracking branches reached v10, which Junio marked for `next`. The series lists matched remotes in the advice message ("Branch name '%s' appears in multiple remotes:") and makes `git worktree add --guess-remote` treat multiple matches as an error instead of falling back to HEAD. Earlier revisions split the work into mechanical refactoring and feature commits, with Junio suggesting the advice function could have been placed higher in the series.

### Converting build-time `USE_NSEC` into runtime `core.useNanosec`
*Long thread: 32 replies so far from 5 participants over 25 days.*

D. Ben Knoble's series converting the build-time `USE_NSEC` macro into the runtime `core.useNanosec` configuration reached v6, with Patrick Steinhardt answering "Works for me" to whether it is ready for `next`. The series allows distributors to ship a single binary with nanosecond stat support, reducing racy-git problems in index-related commands. Junio flagged an ordering dependency: `repo_config_values()` must first work on repository instances other than `the_repository` before a `--recurse-submodules` option can load submodule indexes in the same process.

### Pluggable packfile generation via the object database
*Long thread: 31 replies so far from 6 participants over 15 days.*

Patrick Steinhardt's series making packfile generation pluggable for `git-upload-pack`, `git-send-pack`, and `git-bundle` reached v4, fixing a use-after-scope bug in `odb_source_files_generate_pack()` that Elijah Newren identified. The series hoists pack-generation machinery into a new `odb_generate_pack()` interface with a "files" backend. Karthik Nayak reviewed v4 positively, and Patrick confirmed that `opts->pack_fd` is consumed by the generator and that a primary source without the callback would die.

### Eagerly loading ODB alternates
*Long thread: 31 replies so far from 5 participants over 12 days.*

Patrick Steinhardt's series eagerly initializing alternates at ODB creation reached v3, adding a first patch that creates the object database after complete repository configuration is written in `init_db()`. Karthik Nayak affirmed the rationale that `core.sharedRepository` must be known when the ODB is created. Junio questioned the `eq()` semantics for filesystems with differing case sensitivity, while Jeff King preferred unconditional case-sensitive matching and flagged a timing hazard in caching `core.ignoreCase` before the repository is fully configured.

### Packfile recovery when a multi-pack-index names a removed pack
*Long thread: 31 replies so far from 5 participants over 14 days.*

Elijah Newren's series fixing objects treated as missing when a geometric repack removes a pack that a stale multi-pack-index still names reached v3. The series renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return, stops `git mktree --batch` from passing `OBJECT_INFO_QUICK`, and makes `find_pack_entry()` scan the MIDX's packs for surviving copies on `OBJECT_INFO_SECOND_READ`. Jeff King endorsed dropping the QUICK-reader patch and approved the rename, while Patrick Steinhardt added a +1 to the review and Jeff agreed with Junio's proposal to mark the series for `next`.

## The period in brief

August 2026 was a heavy, eventful month with 31 active days of traffic on the Git mailing list. The period saw major features land in `next`, including `git branch --delete-merged` after a 25-iteration journey and `git bisect --reset-when-found`, alongside significant performance work such as the `paint_down_to_common()` optimization delivering 100-1000x speedups. The ODB abstraction effort advanced through multiple pluggable series, while policy debates over AI-assisted contributions and Git 3.0's lowercase-hex proposal kept the list engaged. Several regressions in `git rebase`, `git maintenance`, and `git stash` demanded urgent attention alongside routine bug fixes and documentation updates.

## Key developments

### `git branch --delete-merged` lands in `next`
Harald Nordgren's 25-iteration series adding safe automated branch cleanup to `git branch` was merged to `next` after resolving push-detection logic and stacked-branch protection in v24. The command supports `--delete-merged <pattern>` with order-independent protection for branches used as upstreams of surviving branches, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview. Phillip Wood and Junio C Hamano confirmed the v24 fixes resolved all technical blockers, with a follow-up planned to extend stacked-branch protection to `git branch -d`.

### `paint_down_to_common()` optimization delivers 100-1000x speedup
Kristofer Karlsson and Tian Yuchen's series optimizing merge-base queries was queued in Junio's tree after v8. The patch terminates early when one side of an asymmetric query exhausts its commit queue, with benchmarks showing step counts dropping from 81 to 9 and 100-1000x speedups for repositories with import grafts or shallow histories. Elijah Newren and Junio reviewed the Trace2 instrumentation and topological ceiling for v1 commit-graphs, with the series now procedurally unblocked.

### Git 3.0 lowercase-hex RFC sparks security debate
brian m. carlson's RFC to restrict hex object IDs to lowercase in Git 3.0 drew philosophical debate throughout the month. brian cited security cases where uppercase input bypasses policy checks against lowercase forms and regexps like `[0-9a-f]{40}` fail to match uppercase IDs, while Junio invoked the Robustness Principle. Phillip Wood asked for concrete security problems, and the thread remains unresolved as Junio queued the series while inviting views, with discussion touching on parallel histories created by uppercase hex in object payloads.

### AI-assisted contribution policy tested
Multiple patches disclosed AI assistance, triggering policy debates that tested Git's stance. Arijit Banerjee's promisor-pack performance patch stalled when brian m. carlson objected on DCO grounds, with Junio suggesting a case-by-case approach rather than an outright ban. A treewide `utime.h` migration generated by Gemini Flash was refused review by brian m. carlsson, who argued AI-generated code cannot satisfy the Developer's Certificate of Origin. Tim Wiederhake's `gitk` patch faced scrutiny over Claude-generated code, with Johannes Sixt asking for community input on whether iterative reworking meets ownership requirements.

### `git add --resolved` and `git bisect --reset-when-found` advance
Junio's `git add --resolved` v3 stages only paths with removed conflict markers, sparking debate with Peff over false completeness and Michael Montalbo's `--skip-conflict-markers` alternative for incremental staging. The series is technically sound and on track for `next`. Harald Nordgren's `git bisect --reset-when-found[=<where>]` v6 adds auto-reset after finding the first bad commit, with Junio's final clean-ups reading `refs/bisect/bad` directly and using a sentinel value making it ready for `next`.

## In brief

**Packfile URI race-condition fix** -- Ted Nyman's v6 fixing concurrent HTTP packfile downloads was cleared for merge by Junio after Peff's final approval, using deterministic staging paths to prevent corruption.

**`git cat-file --batch-command` `%(objecttype)`** -- Pablo Sabater's GSoC series reached v6, feature-complete with a dedicated `struct fetch_object_info_results` replacing `struct object_info`, ready for integration.

**Trailer parsing fix** -- Kristoffer Haugsbakk's patch stopping URLs from being parsed as trailers was marked for `next` in v3, excluding `://` separators.

**`git fast-import` libification** -- Christian Couder's 12-patch series modernizing parse-options usage neared completion, with Elijah Newren reviewing minor documentation and consistency issues.

**MinGW build updates** -- Johannes Schindelin's 12-patch v2 series upstreaming Git for Windows build fixes addressed Python support, MSYS2 locale handling, and dashed-form executable removal.

**`git history`/`git replay` notes preservation** -- D. Ben Knoble reported that both commands silently drop notes during history rewriting; Phillip Wood proposed copying notes before `exec`/`edit` stops to fix the OID-mapping issue.

**`git push --porcelain` inconsistency** -- Xavier Morel reported human-readable errors for non-existent refs instead of machine-readable format; Junio proposed using `!` as an error prefix.

**ODB streaming unification** -- Patrick Steinhardt's v2 fixed a critical bug where the in-memory backend computed object names as `OBJ_BLOB` for all types, unifying `odb_write_stream` and `odb_read_stream` into `odb_stream`.

**`git maintenance` geometric repack bugs** -- Stefan Haller reported concurrency failures and an auto-condition bug triggering 256x more aggressively than documented; Patrick Steinhardt proposed raising the threshold to 6700.

**`git rebase` submodule regression** -- A fatal error with commit graphs when submodules are involved was bisected to `bb5da75d61`, with Patrick Steinhardt asking for confirmation on `master`.

**`gitk` security flaw** -- Tim Wiederhake's custom commands patch allowed command injection via unquoted placeholders; the author is evaluating fixes.

**`git worktree` fixes** -- Multiple patches addressed out-of-bounds reads in `worktree_basename()`, relative path handling in `worktree repair`, and ambiguous remote branch messages listing matched remotes.

**`git stash show` use-after-free** -- Nicolas Le Cam reported freed heap memory printed in diff headers since 2.52.0, traced to `OPT_STRING_F` storing pointers into freed argv elements.

**`http.sslVerifyStatus`** -- Grayson Gordon's v6 patch adding OCSP stapling validation was reviewed, with tests moved to t5551.

**Reftable stack reload avoidance** -- Karthik Nayak's series avoiding stack reloads inside locked transactions showed 15-20% improvement and constant `newfstatat()` counts.

**`receive-report` hook** -- Karthik Nayak's v4 series for `git-receive-pack` allows hooks to modify the client report, with non-zero exit rejecting all refs.

**`git repo info` path keys** -- K Jayatheerth's GSoC series adding `path.*` keys advanced to v5 with `path.cdup`, though SZEDER Gábor flagged a build break from `repo->prefix` dependency.

**`git organize` RFC** -- Michael Montalbo posted a 14-patch RFC for recording and applying file placement guidelines via `.gitorganize`.

**`git config --global` XDG fix** -- Delilah Ashley Wu's series making `--global` read both `$HOME/.gitconfig` and XDG config files advanced through multiple revisions.

**`stash.ref` configuration** -- Vladimir Sitnikov proposed a configurable stash ref for per-worktree isolation, with Kristoffer Haugsbakk endorsing `refs/worktree/stash`.

**Negative pathspecs fix** -- Diogo Castro's patch fixing `git ls-files` and `git add` mishandling negative pathspecs shorter than the common prefix drew discussion about prefix computation.

**`die_for_incompatible_opts()`** -- Junio's varargs proposal for unbounded option checks was discarded after Jeff King noted the lack of `LAST_ARG_MUST_BE_NULL` safety.

**`you_still_use_that()` rewording** -- Junio's v3 changed deprecation messages to "'%s' will be removed soon" and "If you need a replacement:".

**`last-modified` Bloom filter** -- Toon Claes's v3 series was ejected from `seen` after CI failed reproducibly.

**`fetch-pack` parallel packfile URIs** -- Patrick Steinhardt's series adding `fetch.packfileURIThreads` was dropped temporarily pending Justin Tobler's ODB transaction series.

**`git send-email` subject fix** -- Harald Nordgren's patch improving missing subject error messages was approved for `next`.

**`git filter-branch` bugfix** -- Grant Moyer fixed a long-standing inversion in commit mapping with `--state-branch`.

**Git Rev News edition 137** -- Christian Couder published the monthly newsletter on August 3.

**`git whatchanged` deprecation** -- Junio reminded users to find replacements before reporting continued use.

**`git pull --hard` proposal** -- Artur Bieniek's proposal shifted toward Phillip Wood's `git pull --reset` alternative.

**`git am --3way` postimage** -- Nikita Leshenko proposed recording postimage blobs, but Junio rejected eager overhead in favor of lazy reconstruction.

**`git symbolic-ref` exit codes** -- Reviewers debated pinning exit codes 1 vs 128.

**`git subtree` rebase bug** -- Gabriel Ford reported conflicts between main repo and subtree files with identical relative paths after squash merging.

**`credential/libsecret` fix** -- Daniel Martí's patch explicitly loading secrets fixed assertion failures when `secret_service_search_sync()` returned unloaded items.

**`git-svn` updates** -- Wesley Schwengle's v2 silenced v1-layout migration noise and added `NO_GIT_SVN` build knob.

**`ident` command** -- Andrew Pleeter's v2 renamed the new builtin from `whoami` and added `--porcelain` output.

**Typo fixes** -- Hardik Kumar corrected spelling in `versioncmp.c`, `git-gui`, and test scripts.

**`bitmapPseudoMerge.*` docs** -- kapicharly documented the experimental pseudo-merge bitmap configuration options.

**`git repo structure` Unicode opt-out** -- Oswald Buddenhagen proposed an opt-out for box-drawing output; Junio agreed it was needed.

**`gitdatamodel(7)` advertising** -- Kristoffer Haugsbakk's v2 series cross-referenced the data model documentation.

**`git branch --recurse-submodules` fix** -- Volodymyr Vriukalo fixed a BUG abort when the start point names no ref.

**`push.shallowExcludeBoundary`** -- Elijah Newren's v2 adds config to exclude shallow grafts when pushing from shallow clones, avoiding sending entire toplevel trees.

**`OBJECT_INFO_DIE_IF_CORRUPT`** -- Patrick Steinhardt's v2 made corrupt-object handling backend-agnostic with a proper status enum.

**Connectivity check optimization** -- Kristofer Karlsson proposed making `check_connected()` cost proportional to incoming set rather than repository size.

**`git commit --amend` during conflicts** -- Elijah Newren's v3 extends refusal to `am`, `revert`, and rebase stops, plus partial commits.

**`checkout/switch/restore` refactoring** -- Junio's draft series repartitioning `builtin/checkout.c` was posted as an early rough draft not for merging.

**`worktree lifecycle hooks`** -- Domen Kožar's v2 series adding `post-worktree-add/remove/move` hooks argued wrappers cannot reproduce `git worktree prune` semantics.

**`http.sslVerifyHost` debate** -- Patrick Steinhardt and brian m. carlson opposed the option as vulnerable to MITM.

**`reflink` support** -- Vlad Petric added reflink support for local clones with fallback order hardlink -> reflink -> copy.

**Git for Windows 2.55.0(4)** -- Johannes Schindelin announced a security update fixing CVE-2026-62960 in bundle URI handling.

**`proxy authentication`** -- Tom Noonan II reported `git-remote-https` failing to extract credentials from proxy URLs.

**`git diff` completion** -- Junio's v6 series fixed tracked-path-only completion for `git checkout` and `git diff` with `git -C`.

**`zsh completion`** -- Lutz Lengemann's v2 fixed `git -C <path>` support by skipping global options.

**`fetch_if_missing` libification** -- Tian Yuchen's v6 moving the global into `struct repository` was reviewed favorably by Junio.

**`writev()` wrapper** -- Patrick Steinhardt's series reviving the wrapper showed 1.37x latency improvement in upload-pack.

**`git maintenance` "none" strategy** -- David Lin's bugfix restoring the strategy was ready to merge.

**`git worktree add` DWIM** -- Yoichi NAKAYAMA's v4 fixing `-b`/`-B` DWIM regression was queued.

**`git worktree repair`** -- Yoichi NAKAYAMA's v4 relative path fix was marked for `next`.

**`git worktree add` basename fixes** -- The four-patch series fixing out-of-bounds reads was marked for `next`.

**`git branch -d` upstream protection** -- Elijah Newren reported a hole in Harald Nordgren's protection logic.

**`sequencer --empty` fixup** -- Farid Zakaria's v4 making `--empty=drop` honor fixup chains was reviewed by Phillip Wood.

**`git checkout -m` autostash** -- Phillip Wood reviewed Harald Nordgren's refinement.

**`diff provider` retraction** -- Michael Montalbo retracted the v7 abstraction in favor of a narrower pkt-line interface.

**`git repo info` path.git-prefix** -- Lucas Seiki Oshiro questioned whether it belongs in repo info; Junio argued it is repository information.

**`git config --global` test** -- Delilah Ashley Wu noted the need to test both `--get` and `list`.

**`Signed-off-by` critique** -- Дилян Палаузов questioned the trailer's legal validity; no responses yet.

**`git rebase -X` inconsistency** -- zrzut01 reported `git rebase -X <invalid>` silently ignores while `git merge -X` errors.

**`git rebase` commentChar bug** -- Markus Geiger reported hardcoded `#` conflicting with `core.commentChar` in worktrees.

**`terminal output corruption`** -- Junio asked Hugo Osvaldo Barrera to confirm the sideband.c fix resolves truncation.

**`trace2 ancestry` prerequisite** -- Jamie Magee's patch refining `TRACE2_ANCESTRY` for QEMU was under review.

**`csh` test fix** -- Kenneth Lorber's patch fixing `ssh-agent` invocation under `tcsh` was queued.

**`.mailmap` updates** -- Christian Couder and D. Ben Knoble updated their entries.

**`git fetch` refspec mapping** -- Junio proposed extending `--refmap` for Douglas Puchalski's feature request.

**`interactive patch selection`** -- Junio's proposal to unify `git add -p`/`checkout -p` sparked discussion about triage actions.

**`git maintenance` promisor packs** -- Taylor Blau's two-patch bugfix for geometric repack with promisor packs was reviewed.

**`git fsmonitor` macOS** -- Tamir Duberstein's cookie flush fix was reviewed by Patrick Steinhardt.

**`git regex` memory leak** -- Chungmin Lee's macOS TRE leak fix was reviewed.

**`git maintenance` flaky test** -- Patrick Steinhardt's v2 fixed a race in `t7900-maintenance.sh`.

**`gitk` color dialog** -- Johannes Sixt's v2 six-patch series overhauling the color-preference dialog was ready for final review.

**`git add -e` direct call** -- Junio marked for `next` replacing `run_command()` with direct `apply_all_patches()`.

**`git add --interactive` removal** -- Junio rejected the patch for lack of evidence of harm.

**`git interactive` command** -- A feature request drew skepticism from Michael Montalbo.

**`blackgit` monorepo helper** -- Zhonghua Zhu proposed a proof-of-concept remote helper for narrow clones.

**`git history` completion** -- Vincent Mailhol's v3 series for bash completion was backed by Patrick Steinhardt.

**`git repo` GSoC** -- K Jayatheerth's weekly updates continued via blog posts.

**`large object promisors`** -- Srayan Jana asked about LOP feature status; no maintainer response yet.

**`git rebase` regression fixed** -- Markus Geiger confirmed the worktree advisory comment regression was fixed in 2.48.0.

**`git rebase -x` notes** -- D. Ben Knoble reported notes loss; Phillip Wood diagnosed the OID mapping issue.

**`git history` notes** -- D. Ben Knoble reported silent dropping; pre-exec copy approach converged.

**`git maintenance` geometric threshold** -- Patrick Steinhardt's fix raising threshold to 6700 was endorsed by Justin Tobler.

**`git branch --delete-merged`** -- Harald Nordgren's v25 merged to `next`.

**`git bisect --reset-when-found`** -- Harald Nordgren's v6 ready for `next`.

**`git add --resolved`** -- Junio's v3 ready for `next` despite usability debate.

**`packfile URI race`** -- Ted Nyman's v6 cleared for merge.

**`cat-file --batch-command`** -- Pablo Sabater's v6 ready for integration.

**`trailer parsing`** -- Kristoffer Haugsbakk's v3 marked for `next`.

**`HTTP daemon test helpers`** -- Michael Montalbo's v2/v3 ready for integration.

**`gitk` custom commands` -- Tim Wiederhake's patch stalled on security and AI policy.

**`paint_down_to_common`** -- Kristofer Karlsson's v8 queued in Junio's tree.

**`ODB pluggable creation`** -- Patrick Steinhardt's series graduated to `master`.

**`ODB streaming`** -- Patrick Steinhardt's v2 fixed the in-memory backend bug.

**`ODB alternates`** -- Patrick Steinhardt's v3 eager loading series.

**`ODB transactions`** -- Justin Tobler's v5 for receive-pack.

**`ODB packfile generation`** -- Patrick Steinhardt's v4 pluggable generation.

**`MIDX removed pack race`** -- Elijah Newren's v3 recovery series.

**`USE_NSEC runtime`** -- D. Ben Knoble's v6 ready for `next`.

**`worktree ambiguous remote`** -- Yoichi NAKAYAMA's v10 marked for `next`.

**`git-replay --linearize`** -- Toon Claes's v9 with single-branch restriction.

**`receive-report hook`** -- Karthik Nayak's v4 renamed series.

**`trace2 banned-die`** -- Derrick Stolee's v3 making trace2 die-free.

**`lib/ directory`** -- Patrick Steinhardt's RFC for root decluttering.

**`git history squash`** -- Harald Nordgren's v14 with case-sensitivity fixes.