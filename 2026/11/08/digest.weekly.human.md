## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 66 replies so far from 5 participants over 50 days.*

Harald Nordgren's series adds `remote.<name>.fetchRefmap` so a shallow repository can fetch only selected branches of a new remote instead of every branch, with `--limited-fetch` remotes defaulting to limited fetch. Earlier revisions renamed the config to `fetchRefmap`, added the `get_ref_map()` path storing named branches through the refmap, and fixed `@{push}` resolution. During the week D. Ben Knoble described his notes workflow -- keeping both Junio's amlog and his own patch notes while pushing only his own -- and argued a `pushRefmap` would let him separate default push selection from ref mapping for notes pushes, sharing his config with `core.notesRef`, `notes.rewriteRef`, and a negative refspec excluding amlog.

### repo: add path keys to git repo info

*Long thread: 53 replies so far from 6 participants over 115 days.*

K Jayatheerth's GSoC series exposes repository paths via `git repo info` with keys like `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, and `path.cdup`, and refactors `get_superproject_working_tree()` to take a `struct repository *`. After a month without review, Justin Tobler reviewed several patches and Jayatheerth agreed to his suggestions. On 2026/11/07 Jayatheerth posted v8, dropping `path.grafts`, moving a test to `t/t1500-rev-parse.sh`, and special-casing `/dev/null` in `path.hooks`. Phillip Wood then suggested renaming keys (`path.working-tree.*`, `path.prefix`), questioned whether `path.hooks` is needed, and asked how `path.cdup` differs from `path.working-tree.relative`.

### reflog: add %gb format placeholder

*Long thread: 34 replies so far from 6 participants over 24 days.*

Harald Nordgren's series adds a `%gb` placeholder to `git log -g --format` naming the branch HEAD was on when each reflog entry was recorded, showing "detached at <commit>" when detached. Earlier revisions fixed branch naming under `GIT_REFLOG_ACTION` during rebase. On 2026/11/07 he posted v5: "detached at" now names the commit HEAD was detached at, `git rebase <upstream> <rev>` counts `<rev>` as a branch only if `refs/heads/<rev>` exists, commit subjects starting with "checkout" are no longer mistaken for rebase checkouts, and tests cover aborted and fast-forwarding detached rebases.

### Removing gittutorial-2 from the documentation

*Long thread: 32 replies so far from 4 participants over 39 days.*

Julia Evans's two-patch series deletes `Documentation/gittutorial-2.adoc` and all references to it, leaving a stub that redirects readers to `gitdatamodel` and `gitcore-tutorial`, to clear obsolete tutorial material. Earlier revisions removed the entry from `command-list.txt` so `git(1)` would not advertise the obsolete guide, and Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out of the series. On 2026/11/05 Tuomas Ahola reported that the removal topic has graduated to `master` and asked Junio to pick up his rerolled `ta/command-list-guides-sync-lint`; after confirming the patch was the result of the rebasing he had planned, Junio noted the thread still has an unanswered review comment from Julia Evans and asked whether it is ready to be picked up.

### Reference-transaction hooks reporting old values

*Long thread: 31 replies so far from 5 participants over 45 days.*

Maciej Ciemborowicz's v6 patch resolves old object IDs in the common transaction hook layer so the reference-transaction hook reports real old values for unconditional deletions, keeping them separate from caller-supplied values. Earlier, Patrick Steinhardt endorsed the common-layer approach. On 2026/11/02 Junio C Hamano said he will discard the topic from his tree unless volunteers review it within a few weeks, and Maciej Ciemborowicz acknowledged, thanking him and hoping someone else will review it.

### repack: corner cases for cruft-less MIDXs

*Long thread: 31 replies so far from 5 participants over 36 days.*

With `repack.midxMustContainCruft=false`, geometric repacks can omit cruft packs from the MIDX while relying on the indexed set being closed under reachability; Taylor Blau's series adds tree and tag roots, retains cruft for ordinary incremental repacks, and includes required packs in ordinary and incremental MIDX writes, fixing cases where bitmap generation could fail. On 2026/11/04 Junio C Hamano re-read the patches and their review discussion, found himself confused by the same points Jeff King had flagged, and said he has no concrete improvement ideas right now. Earlier, Taylor had said the series should be ready as-is after Junio questioned the topic's viability, a question Junio now clarified was not meant for the author to answer.

### Community voting process proposal

*Long thread: 30 replies so far from 13 participants over 28 days.*

Emily Shaffer's document proposes a community voting process for decisions where consensus cannot be reached, with eligibility thresholds debated earlier (commit counts, activity windows, human-override exceptions). On 2026/11/02 Patrick Steinhardt argued that excluding long-inactive people is not necessarily unexpected, since an informed vote requires community context, and said that any human-override process (Junio or the PLC adding specific voters) must be fully transparent--who added whom, with reasoning, published well ahead of time.

### Git 3.0 lowercase-only hex object IDs

*Long thread: 28 replies so far from 7 participants over 97 days.*

brian m. carlsson's v2 series removes acceptance of uppercase hex in object IDs in breaking-changes mode, addressing security issues where downstream software assumes a unique lowercase hex form. On 2026/11/02 Junio C Hamano noted the series had received no reactions and asked for review to confirm it aligns with earlier discussion so it can be marked for `next`; Patrick Steinhardt added it to his todo list for the week.

## The period in brief

The week of 2026/11/02 through 2026/11/08 saw seven active days of steady traffic, dominated by a partial-clone regression that broke Kubernetes CI and a flurry of long-running series reaching new iterations. Readers should not miss Patrick Steinhardt's fix for the stale loose-object cache behind the partial-clone failure, Junio C Hamano's proposal to eject unreviewed topics from `seen`, and the continued review pressure on K Jayatheerth's `git repo info` path keys series.

## Key developments

### Partial clone lazy fetch regression breaks Kubernetes CI

Patrick Ohly reported that `git merge` in a blobless partial clone fails with "not our ref" while lazily fetching a missing object, breaking Kubernetes CI PR merges. He bisected v2.52.0 (good) to v2.54.0 (bad) and identified 5946a564 ("odb/source: make read_object_info() pluggable") as the first bad commit. Kristofer Karlsson reproduced it and traced it to the loose object cache: writing a loose object does not update the cache, so once a subdirectory is listed, existence checks wrongly report newly written objects missing, triggering a promisor fetch for a locally present blob. Patrick Steinhardt confirmed the root cause predates the bisected commit -- 5946a564cd stopped flushing the cache on misses, which had masked the bug -- and sent a two-patch fix adding `loose_cache_mark_seen()` to update the cache on each loose write, plus a regression test in `t/t5616-partial-clone.sh`. Jeff King argued the failure points to a separate underlying issue: callers using `QUICK` should tolerate false negatives, so `prefetch_cache_entries()` and other prefetch callers may need to omit `QUICK`.

### Ejecting unreviewed topics from `seen`

Junio C Hamano proposed a rule that topics unreviewed for N weeks are ejected from `seen`, announcing notices for every topic lingering unreviewed for more than a month. Patrick Steinhardt called the rule reasonable, noting evicted topics can be re-added if interest returns, and flagged the confusing "Will merge to `next`?" state; Junio explained the trailing question mark means "I plan to do this unless someone stops me," and Phillip Wood suggested clarifying that ejected topics can return to lessen contributor worry. Later in the week Junio proposed new [Discarded] heading text stating removed topics "can be resubmit if they gained enough interest and showed good fit to the system in the future."

### `git describe` distance with skewed commit dates

Linus Torvalds' patch fixes `git describe` miscounting distance when commit dates are unreliable by using commit generation numbers. Patrick Steinhardt and Derrick Stolee agreed `ensure_generations_valid()` should be called in `describe_commit()`, accepting the cost without a commit graph is better than a wrong answer; Junio C Hamano asked whether the cost could be amortized by writing a commit-graph, and Linus Torvalds argued consistently wrong numbers are worse and suggested a flag to force exact computation. Kristofer Karlsson later proposed an exact-depth prototype without generation numbers, and reported the describe date-skew fix makes an expected-failure test pass only with a commit-graph; Junio suggested freezing the test to always run under `GIT_TEST_COMMIT_GRAPH`.

### Connectivity check ignoring replace refs

Kristofer Karlsson's patch makes rev-list's `--connectivity-check` call `disable_replace_refs()` and drop `OBJECT_INFO_LOOKUP_REPLACE` from `tree-verify.c`, so it verifies the same original objects that fsck, prune, pack-objects and upload-pack read. Junio C Hamano endorsed it as absolutely the right thing to do, noting replacement objects being backed by reachable objects could be addressed separately. Patrick Steinhardt and Junio agreed no extra work is needed to check both original and replacement objects.

### Geometric maintenance prunes commits without updating commit-graph

Christoph Weiss reported a bug in the maintenance code: after Forgejo switched Git from 2.52 to 2.54, changing the default maintenance strategy from `gc` to `geometric`, repository health checks started failing because under `geometric` maintenance prunes commits without updating the commit-graph. He provided a reproduction script using `git maintenance run --auto` on a bare repository with `core.commitGraph` enabled.

### splitIndex with skipHash corrupts index and crashes status

With `core.splitIndex` and `index.skipHash` both enabled, index entries are lost or corrupted when the shared index is written, crashing `git status`. Tanga Joris reproduced both cases on master and diagnosed the cause: `write_shared_index()` names the file after the zeroed checksum (`sharedindex.000...`), and the null base makes `read_index_from()` treat the shared index as absent. He posted a fix to `read-cache.c` so the shared index is never written with a zeroed checksum under `index.skipHash`, adding two tests to `t/t1700-split-index.sh`.

### Reftable HEAD reflog records a stale value

井上司 reported that with the reftable backend, a transaction that updates a branch and points HEAD at it records the branch's old value in the HEAD reflog, or writes no entry at all when the branch is created in the same transaction, unlike the files backend. He traced the cause to `write_transaction_table()` in `refs/reftable-backend.c`, which resolves the target with pre-transaction refs, and proposed taking the new oid from the same transaction. Karthik Nayak agreed the diagnosis is plausible, noted the files backend avoids the bug only because it lacks atomicity support, and suggested moving reflog logic into the generic `refs.c` layer so backends receive pre-resolved entries.

### Defaulting the hash algorithm to `GIT_HASH_DEFAULT`

brian m. carlsson posted a patch to `setup.c` defaulting the repository hash algorithm to `GIT_HASH_DEFAULT` when nothing else selects one, so breaking-changes mode and Git 3.0 pick SHA-256, with a new test in `t/t0001-init.sh`. This addresses Adam Dinwoodie's report that `git init` built with `WITH_BREAKING_CHANGES=1` still creates repositories with the sha1 object format. Junio C Hamano said the patch looks good for its stated goal but asked whether `HASH_UNKNOWN` should instead mark a not-yet-initialized limbo state, and later raised that the patch would clobber the object format when `git init` reinitializes an existing repository, outlining possible reinit behaviors and asking what happens with the reference storage format.

### Alternate refs during receive-pack quarantine

Kristofer Karlsson's patch fixes the object database code so that `rev-list --alternate-refs` no longer lists the repository's own refs--including hidden ones--while receive-pack's quarantine connectivity check runs, where the quarantine directory is swapped in as the primary object directory. The v3 patch reverts to skipping the repository's own object directory directly in the alternate-refs code, with a perf test showing a push into a repository with ~200,000 hidden refs improving from 0.29s to 0.10s. Jeff King reviewed the v3 patch and said it looks good, and Junio C Hamano marked the topic ready for `next`.

## In brief

**Standardizing early option scanning** -- Christian Couder's v4 series introduces an "early-scan" sub-API in parse-options so commands can safely skip option values during hand-rolled early scans; Kaartic Sivaraam reviewed it and Junio C Hamano marked the topic for `next`.

**Repository config handling cleanup** -- Patrick Steinhardt approved Junio C Hamano's patch freezing `struct repo_config_values` with a comment warning against new members until it can support arbitrary repository instances, saying he is very much in favor of merging it so newcomers do not waste time.

**Bare --force-with-lease ignored on remote groups** -- Patrick Peroša reported a bare `--force-with-lease` is ignored on remote groups: the parent process does not forward the flag to child pushes.

**`git log --graph` indentation fix** -- herr.kaste posted a patch stopping visual roots with columns to their right from being indented in `git log --graph`; Pablo Sabater reviewed the resent patch and said it looks good.

**HTTP exponential-backoff and retry** -- Ravi Mistry posted v3 of the series, defaulting `http.retryAfter` to 10 seconds; Junio C Hamano questioned the topic structure of first implementing 0-second waits.

**size_t fix for large blobs** -- Junio C Hamano queued Elia Pinto's v3 using `size_t` for streamed large blobs in `pack-objects`, fixing size truncation on platforms where `long` is 32 bits but `size_t` is 64 bits; Kristofer Karlsson also changed `stream_blob()` in `builtin/unpack-objects.c` to take the blob size as `size_t`.

**fsmonitor/untracked-cache invalidation** -- Derrick Stolee posted a patch for an fsmonitor/untracked-cache bug present since 2017: when fsmonitor fails or returns a trivial response, untracked cache entries could stay marked valid, so `git status` could miss new untracked files. Junio C Hamano called it "rather a blunt instrument" but queued it, and Derrick Stolee later reviewed Ilia K's selective invalidation approach as better than his own full-clear.

**Path-walk regression** -- Derrick Stolee fixed a path-walk regression from Git 2.55.0 that dropped blobs from tagged trees at already-visited paths; Taylor Blau reviewed it as LGTM.

**WWW-Authenticate headers** -- Jeff King reviewed the v2 patch preserving WWW-Authenticate headers across HTTP redirects as OK with small suggestions.

**Sequencer shallow boundary** -- Phillip Wood called the v2 sequencer patch refusing shallow boundary commits "definitely moving in the right direction" while questioning the `--allow-shallow-boundary` option.

**Alternates matching commondir** -- Jeff King proposed skipping alternates matching the commondir in `refs_from_alternate_cb` for Kristofer Karlsson's quarantine alternate-refs fix; Justin Tobler called it reasonable.

**send-pack tree inspection** -- Marc Strapetz revived the send-pack series inspecting the old tip's tree to avoid resending unchanged blobs after a rebase.

**`git log --follow` regression** -- b.murauer reported a `git log --follow` regression bisected to 304812ed33; Patrick Steinhardt noted patches are in review, and separately questioned whether the `--follow` speedup series regresses sub-tree merges and objected that its fix fails for copied files.

**Rebase --autosquash --empty** -- Phillip Wood reviewed Farid Zakaria's v5 patch making `git rebase --autosquash` honor `--empty` when a fixup empties its target, calling it pretty good but requesting comment restructuring and a rebase onto master.

**Worktree remove for rebasing branches** -- Jasper Mayone posted v4 treating a branch being rebased or bisected as checked out: `worktree remove` refuses such worktrees without `--force`, `%(worktreepath)` names the rebasing worktree, and `git branch` drops the `+` marker there.

**`git repo structure` concern reporting** -- Mark C. Chu-Carroll posted v5 adding degree-of-concern reporting to `git repo structure`, with thresholds configurable via `repo.concern.*` keys.

**CI test-variable consolidation** -- Patrick Steinhardt posted v2 of the series; Phillip Wood reviewed it as ready for `next`.

**Governance documentation** -- Antonin Delpeuch submitted a patch adding `Documentation/Governance.adoc` describing current project governance; Junio C Hamano asked for line-wrapping and clarifications on several points.

**`git apply` rename+edit** -- Aniket Gupta reproduced the `git apply` failure on a pure rename followed by an edit with `core.fileMode=false` and proposed `S_IFREG | (patch->old_mode & 07777)`; Junio C Hamano questioned whether the input is valid and whether the fix works in reverse.

**`get_follow_remote_head()` context** -- Junio C Hamano endorsed Matt Hunter's approach of passing more context into `get_follow_remote_head()` for lazy `remote.<name>.followRemoteHEAD` resolution, calling it "very cleanly done"; Colin Hinton will adopt it in his next re-roll.

**Bundle-URI relative URIs** -- Derrick Stolee gave an LGTM to Aniket Gupta's bundle-URI series skipping unresolvable relative URIs instead of dying; Gupta conceded the credentials patch covers only one message and offered a corrected v4.

**Notes tree repository storage** -- Okhuomon Ajayi posted a replacement six-patch series storing the repository in `struct notes_tree` so `notes-utils.c` can drop `USE_THE_REPOSITORY_VARIABLE`.

**Test default object storage format** -- Patrick Steinhardt posted a five-patch series introducing `GIT_TEST_DEFAULT_OBJECT_STORAGE_FORMAT=` to run tests with a non-default object storage backend.

**`pack.packSizeLimit` repo config** -- Tanga Joris tested v3 of moving `pack.packSizeLimit` into repo config and noted patch 2/2 conflicts with the freeze on new `repo_config_values` members.

**`promisor.fetchOrder`** -- Yopa Nelly posted v2 with the parsed list per-repository in `struct promisor_remote_config`; Junio C Hamano asked how it meshes with server-side promisor ordering work.

**Rev-list early option scan** -- Azuama Odinaka Maximus's patch makes the early option scan in `cmd_rev_list()` skip separate arguments of options like `--grep`; Junio C Hamano criticized the hardcoded option-name array as fragile and incomplete.

**Merge --no-overwrite-ignore** -- Aniket Gupta's patch adds a `preserve_ignored` bit to `struct merge_options` so `git merge --no-overwrite-ignore` aborts instead of overwriting ignored files during real merges.

**Config NULL values segfaults** -- Thomas Bachem's 15-patch series fixes segfaults from valueless config settings across rebase, fetch, grep, blame, merge and more; Junio C Hamano reviewed the sideband fix skipping unrecognized `allowControlCharacters` keywords and marked the topic ready for `next`.

**`git apply` crashes** -- Tanga Joris posted two Outreachy patches fixing `git apply` crashes: freeing names from unused `diff --git` lines, and rejecting new/deleted files that carry both names.

**Unpack-trees cleanups** -- Junio C Hamano posted two cleanups: dropping `USE_THE_REPOSITORY` via an `unpack_repo()` helper, and renaming an `index` parameter to `istate` for consistency.

**`test-tool genbytes`** -- Kristofer Karlsson posted v2 of the series adding a `test-tool genbytes <hex-byte> [<count>]` helper and converting tests away from `genzeros | tr` to speed up macOS CI tests.

**`assume_unchanged` repository config** -- Hussain Sitamouwala posted a patch moving the global `assume_unchanged` flag in `environment.c` into repository-specific configuration.

**Detached HEAD guide** -- Julia Evans's series rewrites the detached HEAD explanation into a standalone `gitdetached-head` guide; D. Ben Knoble reiterated interest in landing it, naming two blockers, and Junio C Hamano answered his closing suggestions, saying he hopes the thread is approaching a finish.

**Root refs vs pseudorefs** -- Andy Koppe's v3 series reworks the glossary and `gitdatamodel(7)` to distinguish root refs like `ORIG_HEAD` from pseudorefs like `FETCH_HEAD` and `MERGE_HEAD`; Junio C Hamano reviewed patches 1-4, calling the pseudoref rewording "very very good".

**Shallow-clone `<rev>~N` hint** -- Harald Nordgren's patch adds a hint explaining why `<rev>~N` fails in a shallow clone; Junio C Hamano observed that after a few days nobody had expressed support for the hint.

**Range-diff --matched-only** -- Harald Nordgren's patch adds `git range-diff --matched-only` to show only commits that correspond on both sides; Kristoffer Haugsbakk and Ben Knoble discussed alternative range selection forms.

**Loose-object-idx migration** -- Justin Tobler's v3 series fixes an ODB transaction failure on commit when a compatibility hash is configured; Karthik Nayak agreed the series is already in good shape.

**Config home and XDG files** -- Delilah Ashley Wu's series makes `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`; Junio C Hamano affirmed the t1300 test patch and asked several clarifying questions.

**Stash index loads** -- Phillip Wood's seven-patch series removes duplicate index loads and refreshes in `git stash push` and `git stash create`; Junio C Hamano reviewed the last two patches, calling 6/7 sensible and 7/7 an obvious, very nice simplification.

**All-or-none switch/checkout** -- 井上司 reported that when writing tracking config fails during `git switch` or `checkout` of a new branch, the worktree and index are already updated but HEAD still points at the old branch; he proposes writing the tracking config before the worktree update and removing it if a later step fails.

**Credential OPT_SUBCOMMAND** -- Tanga Joris reviewed the `git credential` OPT_SUBCOMMAND conversion, suggesting a helper for the repeated `parse_options()` calls and a test that subcommands reject extra arguments with exit code 129.

**Diff EXAMPLES anchors** -- Jeff King flagged that `[[name_only]]` anchors in the rewritten `git diff` EXAMPLES section work with Asciidoctor but not python AsciiDoc, and questioned the placement of the `[[ignore_all_space]]` anchor.

**Credential-cache daemon race** -- Farid Zakaria's fix drew questions from Junio C Hamano about which existing arrangements could be ignored; Zakaria replied that he was unsure how to respond and had scoped his contribution to what he felt comfortable with.

**t0027 optimization** -- Torsten Bögershausen reported an alternative t0027 optimization cutting runtime from 71 to 54 seconds on an older macOS.

**Reflog expire loop** -- Laveesh Bansal posted a `builtin/reflog` patch expiring `--all` and named refs in a single loop, keeping the name lookup inside the loop so output and exit status are unchanged.

**Outreachy proposal** -- Cynthia Uche posted Outreachy proposal draft v3 targeting `git bisect` for parse-options conversion, with an updated candidate inventory, methodology section, and week-by-week timeline.