## Long threads

### `git history squash` subcommand for folding a commit range

*Long thread: 90 replies so far from 9 participants over 73 days.*

Harald Nordgren's series adding `git history squash <revision-range>` dominated the month's discussion volume. The series was ejected from `next` on August 3 after Phillip Wood exposed fundamental reachability flaws, sparking a broader debate about review quality and integration policy. Through August, Nordgren posted iterations v11 through v14, resolving message-editing defaults, range validation, and sign-off ordering, while Phillip Wood developed fixups for `UNINTERESTING` commits, root commits, and multi-tip histories. By month's end the series remained technically complete but stalled on procedural friction, with Junio C Hamano diagnosing "bottomless time sink" review cycles and suggesting a possible maintainer takeover if the author could not coordinate with reviewers.

### Moving libgit.a sources into a `lib/` directory

*Long thread: 38 replies so far from 12 participants over 124 days.*

Patrick Steinhardt's RFC to move roughly 700 libgit.a source files from the repository root into a `lib/` directory continued its long-running debate. Proponents argued the move shrinks the top level from over 550 files to about 80, improving discoverability for newcomers. Opponents including SZEDER Gábor and Junio C Hamano countered that the rename fallout is an ongoing burden and that `lib/` is an arbitrary intermediate layer, proposing instead to migrate one subsystem at a time as was done with `builtin/` and `refs/`. Michael Montalbo proposed a script-based grouping of the 231 libgit.a files into 14 directories using commit-history labels and cscope call-graph checks, while Junio suggested carving conceptually clean groups into their own directories without waiting to classify the remainder.

### `git-replay(1)` gains a `--linearize` option

*Long thread: 38 replies so far from 7 participants over 85 days.*

Toon Claes's series adding `--linearize` to `git replay` reached v9 by month's end, restricting the option to single-branch usage after Elijah Newren identified multi-branch ambiguity. The option drops merge commits and replays all reachable commits into a single linear history, updating refs pointing at merges to their last replayed ancestor. Reviewers pressed for clearer error messages distinguishing "multiple branches" from "multiple revision ranges," and the series now rejects `--contained` and multiple branches with simplified documentation. The thread also touched on whether the restriction should extend to `--revert` mode to match `git cherry-pick` behavior.

### `git repack --drop-filtered` for partial clones

*Long thread: 35 replies so far from 4 participants over 30 days.*

Siddharth Shrimali's GSoC series adding `--drop-filtered` and `--dry-run` to `git repack` allows partial clones to reclaim disk space by removing locally cached promisor blobs above a size threshold. The series progressed through v5, addressing a type-mismatch bug in `list_objects_filter__filter_oidset()` where `unsigned long` was passed where `size_t` was expected. By mid-month, Christian Couder and Junio C Hamano confirmed the series was ready for `next`, with remote verification, recency-based culling, and a drop log explicitly deferred as follow-up work rather than blocking requirements.

### Pluggable packfile writes for `git-receive-pack`

*Long thread: 35 replies so far from 3 participants over 16 days.*

Justin Tobler's series introducing `odb_transaction_write_pack()` makes `git receive-pack` backend-agnostic for packfile writes, with the existing `git index-pack`/`git unpack-objects` logic becoming the "files" backend implementation. The series reached v5, adding test assertions for `.keep` file migration to the main ODB and using transaction flags like `ODB_TRANSACTION_RECEIVE` to differentiate callers. Patrick Steinhardt approved the approach, while Junio C Hamano questioned whether reading `receive.unpackLimit` in generic object-layer code constituted a layering violation, suggesting `odb-file.unpackLimit` as a more accurate name.

### Ambiguous remote branch messages in `worktree add` and `checkout`

*Long thread: 32 replies so far from 6 participants over 20 days.*

Yoichi NAKAYAMA's series improving error messages when `git worktree add` or `git checkout` matches multiple remote-tracking branches reached v10. The advice now lists matched remotes ("Branch name '%s' appears in multiple remotes:") and v8 added a fourth commit making `git worktree add --guess-remote` fail on multiple matches instead of falling back to HEAD. Junio C Hamano reviewed v10 favorably and stated he would mark it for `next`, though he noted the multiple-match-but-success case under `--guess-remote` remains a separate bug to fix later.

### Converting build-time `USE_NSEC` into runtime `core.useNanosec`

*Long thread: 32 replies so far from 5 participants over 25 days.*

D. Ben Knoble's series converting the build-time `USE_NSEC` macro into the runtime `core.useNanosec` configuration reached v6, allowing distributors to ship a single binary with nanosecond stat support. The series addresses racy-git detection in `read-cache.c` and `statinfo.c`, with the `NO_NSEC` strategy ignoring the config while using the `use_nanosec` member unconditionally. Patrick Steinhardt confirmed readiness for `next` with "Works for me," while Junio C Hamano flagged an ordering dependency: `repo_config_values()` must first work on repository instances other than `the_repository` before submodule recursion can safely use the config.

### Packfile recovery when a multi-pack-index names a removed pack

*Long thread: 31 replies so far from 5 participants over 14 days.*

Elijah Newren's series fixes objects being treated as missing when a concurrent geometric repack removes a pack that a stale multi-pack-index still names as owner. The v3 series renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return, stops `git mktree --batch` from passing `OBJECT_INFO_QUICK`, and teaches `find_pack_entry()` to scan the MIDX's covered packs for surviving copies on `OBJECT_INFO_SECOND_READ`. Jeff King endorsed dropping the QUICK-reader patch, arguing QUICK is designed to accept false negatives, while Patrick Steinhardt identified the real bug: a stale MIDX hides secondary copies from non-MIDX lookups. By month's end, Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`.

### Eagerly loading ODB alternates

*Long thread: 31 replies so far from 5 participants over 12 days.*

Patrick Steinhardt's series making the object database eagerly initialize alternates instead of scattering deferred `odb_prepare_alternates()` calls reached v3. The series adds a first patch creating the ODB after complete repository configuration is written in `init_db()`, addressing a timing hazard where `core.ignoreCase` was cached before the repository was fully configured. Jeff King preferred unconditional case-sensitive matching for deduplication, while Junio C Hamano proposed a best-effort case-insensitive match where the platform hints at one, concluding the code can stay as-is since inconsistent casing only causes a redundant duplicate search.

### Pluggable packfile generation in the object database

*Long thread: 31 replies so far from 6 participants over 15 days.*

Patrick Steinhardt's series making packfile generation pluggable for `git upload-pack`, `git send-pack`, and `git bundle` reached v4, converting these commands to use a new `odb_generate_pack()` interface with a "files" backend. The series fixes a use-after-scope bug in `odb_source_files_generate_pack()` where a stale pointer to a function-local struct could be dereferenced on abnormal exit. Karthik Nayak reviewed v4 as looking good, while Elijah Newren proposed an alternative for `the_repository` cleanup in `bundle.c`, which Steinhardt defended. Junio C Hamano asked whether `opts->pack_fd` should be closed when the primary source does not support `generate_pack`.

## The period in brief

August 2026 was a heavy, eventful month on the Git mailing list with traffic on all 31 days, dominated by the maturation of the ODB abstraction effort, the dramatic ejection and revival of `git history squash`, and the landing of long-awaited branch-management features. Readers should not miss the ODB pluggability series reaching implementation, the 25-iteration `git branch --delete-merged` finally merging to `next`, the policy debate over AI-assisted contributions crystallizing around specific patches, and the Git 3.0 lowercase-hex RFC sparking philosophical debate about robustness versus security.

## Key developments

### ODB abstraction reaches implementation phase

Patrick Steinhardt, Justin Tobler, and others advanced the object database abstraction from design to code across multiple series. On-disk structure pluggability graduated to `master`, making backend-specific directory creation delegated via a `create_on_disk` callback. Tobler's `odb_transaction_write_pack()` series for `git receive-pack` and Steinhardt's `odb_generate_pack()` series for upload-pack/send-pack/bundle both reached advanced review stages, making packfile writes and generation backend-agnostic. The eagerly-loading alternates series also progressed to v3, removing scattered `odb_prepare_alternates()` calls. These efforts collectively move Git toward supporting non-filesystem object storage backends.

### `git history squash` ejected from `next` then revived

Harald Nordgren's `git history squash` series became the month's central drama. After reaching v11 and appearing ready for integration, Phillip Wood's review on August 3 exposed fundamental reachability flaws, leading Junio C Hamano to eject the topic from `next` and propose a temporary moratorium on new topics in `seen` without substantive review. Nordgren posted v12 through v14 addressing the flaws, but the series stalled on procedural friction, with Junio diagnosing that rapid rerolls invalidated nearly-complete reviews. The episode sparked a broader discussion about review quality and integration pipeline policy, with Matt Hunter reflecting on the difficulty of providing substantive positive reviews.

### `git branch --delete-merged` lands after 25 iterations

Harald Nordgren's `--delete-merged` series for `git branch` finally merged to `next` on August 5 after 25 iterations spanning six months. The feature provides safe automated local branch cleanup with order-independent stacked-branch protection, per-branch opt-out via `branch.<name>.deleteMerged=false`, and `--dry-run` preview. Phillip Wood's final review confirmed v24 fixes resolved push-detection logic and stacked-branch protection blockers. Junio C Hamano queued the series with only mechanical test-formatting nits remaining, and a follow-up to extend protection to `git branch -d` was planned.

### Partial clone tooling matures

Several series improved partial clone workflows. Ted Nyman's packfile URI race-condition fix received final approval from Peff and was cleared for merge, using deterministic staging paths to prevent corruption during concurrent HTTP fetches. Siddharth Shrimali's `git repack --drop-filtered` reached feature-complete status at v4/v5, allowing safe reclamation of disk space by dropping oversized promisor blobs. Pablo Sabater's GSoC series adding `%(objecttype)` to `git cat-file --batch-command` remote-object-info queries also reached v6, completing the protocol's core metadata support.

### Performance optimizations land

Tian Yuchen and Kristofer Karlsson's `paint_down_to_common()` optimization series was queued in Junio's tree, delivering 100-1000x speedups for asymmetric merge-base queries by terminating early when one side exhausts its commit queue. Johannes Schindelin's fix for the O(N^2) packfile-loading regression was marked for `next`, adding a `skip_dup_check` parameter to `packfile_list_append()` to avoid linear scans when loading many packs. Patrick Steinhardt's parallel packfile URI fetch series introduced `fetch.packfileURIThreads` with benchmarked speedups up to 7.58x, though it was temporarily dropped pending Tobler's ODB transaction series.

### Git 3.0 lowercase-hex RFC sparks debate

brian m. carlson's RFC to restrict hex object IDs to lowercase in Git 3.0 generated significant philosophical discussion. Jeff King identified an edge case where uppercase hex in object payloads creates parallel histories, while Junio invoked the Robustness Principle to argue for liberal acceptance. brian m. carlson countered with security cases: policy checks against lowercase forms can be bypassed by uppercase input, and regexps like `[0-9a-f]{40}` fail to match uppercase IDs. The series was queued while inviting views, with Phillip Wood asking for concrete security problems and brian providing two specific cases.

### AI-assisted contributions policy crystallizes

The month saw the project's stance on AI-generated code harden through specific cases. brian m. carlsson declined to review a treewide `utimensat()` migration series generated by Gemini Flash, arguing it cannot satisfy the Developer's Certificate of Origin requirement, a stance Junio endorsed as prudent. Arijit Banerjee's promisor-pack performance patch stalled on similar policy grounds, with brian m. carlsson explicitly ruling out acceptance of nontrivial LLM-generated patches. Tim Wiederhake's `gitk` custom commands patch faced scrutiny over AI assistance disclosure alongside a critical security flaw: unquoted shell substitution enabling command injection via malicious commit titles.

### Worktree and branch management UX improvements

Yoichi NAKAYAMA's series improving ambiguous remote branch messages in `git worktree add` and `git checkout` reached v10 and was marked for `next`, listing matched remotes and erroring on multiple `--guess-remote` matches. Matthias Aßhauer's fix for an out-of-bounds read in `worktree_basename()` that could cause data loss with `git worktree add ""` was marked for `next`. Harald Nordgren's `git bisect --reset-when-found` also reached ready-for-`next` status after v6 incorporated Junio's final clean-ups.

## In brief

**`git cat-file --batch-command` `%(objecttype)` support** -- Pablo Sabater's GSoC series reached v6, feature-complete with a dedicated `struct fetch_object_info_results` replacing `struct object_info` for remote queries, enabling tools to query object types without fetching full objects.

**Packfile URI race-condition fix** -- Ted Nyman's v6 series received final approval from Peff and Junio, using deterministic staging paths and read-write mode to prevent corruption during concurrent HTTP packfile downloads.

**`gitk` color-preference dialog overhaul** -- Johannes Sixt's six-patch series making color swatches clickable and labels translatable reached v2, while a separate `gitk` custom commands patch from Tim Wiederhake was blocked on a command injection vulnerability and AI-assistance policy questions.

**`git rebase` regression with submodules** -- Philippe Blain reported a fatal error when submodule pointers change, bisected to `bb5da75d61`, with Patrick Steinhardt asking for confirmation whether the issue persists on `master`.

**`git maintenance` geometric repacking bugs** -- Stefan Haller reported concurrency failures in `prune_packed_objects()` and an auto-condition triggering 256x more aggressively than documented; Patrick Steinhardt proposed raising the threshold from 100 to 6700.

**Diff provider RFC retracted** -- Michael Montalbo retracted the v7 "diff provider" abstraction, deciding to return to the v6 line with a narrower pkt-line xdiff-interface, and separately posted a 14-patch RFC for a new `git organize` command.

**`receive-report` hook for `git-receive-pack`** -- Karthik Nayak's v4 series introduces a hook running after the reference transaction but before the client report, renamed from "report" to avoid ambiguity, with non-zero exit marking all references rejected.

**Trace2 banned-die header** -- Derrick Stolee's v3 series introduces `banned-die.h` to make the low-level trace2 API die()-free, banning helpers like `xsnprintf` and `ALLOC_GROW` one patch at a time.

**`git repo info` path keys** -- K Jayatheerth's GSoC series adding `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup` reached v5, though SZEDER Gábor flagged a build break due to dependency on `repo->prefix`.

**Trailer parsing fix** -- Kristoffer Haugsbakk's patch stopping Git from misinterpreting URLs as trailers was marked for `next` at v3, modifying `find_separator()` to reject lines containing `://`.

**`git worktree repair` relative paths** -- Yoichi NAKAYAMA's v4 fix for converting relative `.git` file cross-references to absolute paths was judged "very good" by Junio and marked for `next`.

**`git push --porcelain` inconsistency** -- Xavier Morel reported that `git push --delete --porcelain` outputs human-readable errors for non-existent refs instead of machine-readable format; Junio proposed using `!` as an error prefix.

**`git add --resolved` design debate** -- Junio's v3 series staging only paths with removed conflict markers was ready for integration, though Peff raised usability concerns about false completeness and Michael Montalbo proposed incremental staging alternatives.

**Lowercase-hex edge cases** -- Beyond the main RFC, Jeff King flagged that uppercase hex in object payloads creates parallel histories since Git does not normalize case when hashing contents.

**`git send-email` subject error** -- Harald Nordgren's patch improving the error message for missing "Subject:" headers was approved for `next` after Junio requested rewording to avoid implying the file must start with "Subject:".

**Sparse-index use-after-free** -- Shlok Kulshreshtha's v3 fix for `get_oid_with_context_1()` collapsing exit paths to free memory after last use was queued by Junio.

**`git fast-import` libification** -- Christian Couder's 12-patch series modernizing parse-options usage neared completion, with Elijah Newren reviewing minor documentation and consistency issues.

**Coccinelle performance regression** -- SZEDER Gábor's benchmarks showed Coccinelle 1.3.1 is 4.5x slower than 1.1.1, challenging Jeff King's patch to bump the Ubuntu image for static-analysis jobs.

**`git pull --hard` proposal** -- Artur Bieniek's proposal drew mild opposition from Junio, who noted `git fetch` followed by `git reset --hard @{u}` achieves the behavior safely; discussion shifted toward Phillip Wood's `git pull --reset` alternative.

**`http.sslVerifyStatus` for OCSP** -- Grayson Gordon's v6 patch adding stapled OCSP response validation was reviewed favorably, with tests moved to t5551 and documentation rephrased.

**`git config --global` XDG support** -- Delilah Ashley Wu's series making `--global` read both `$HOME/.gitconfig` and XDG config files progressed through v2, addressing the inconsistency where plain `git config list` read both but `--global` read only the former.

**Negative pathspec bug** -- Diogo Castro's patch fixing `git ls-files` and `git add` mishandling negative pathspecs shorter than the common positive prefix drew discussion from Junio about whether the prefix should be computed across both kinds.

**`git stash show --src-prefix` use-after-free** -- Nicolas Le Cam reported freed heap memory printed in diff headers since 2.52.0, traced to `OPT_STRING_F` storing pointers into freed argv elements.

**Worktree lifecycle hooks** -- Domen Kožar's v2 series adding `post-worktree-add/remove/move` hooks argued that wrappers cannot reliably detect pruned entries, proposing consolidation into a single `post-worktree` hook.

**`git replay --linearize` restrictions** -- Toon Claes's v9 restricted the option to single branches and rejected `--contained`, with Elijah Newren suggesting the restriction extend to `--revert` mode.

**`die_for_incompatible_opts()` varargs** -- Junio C Hamano's proposal for an unbounded option-check API was discarded after Jeff King noted varargs lack the safety of `LAST_ARG_MUST_BE_NULL`.

**Checkout/switch/restore refactoring** -- Junio posted an eight-patch draft repartitioning `builtin/checkout.c` so `cmd_switch()` and `cmd_restore()` dispatch directly to helpers, described as an early rough draft not intended for `seen`.