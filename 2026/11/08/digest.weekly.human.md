## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo
*Long thread: 66 replies so far from 5 participants over 50 days.*
Harald Nordgren's series adds `remote.<name>.fetchRefmap` so shallow repositories fetch only selected branches of a new remote instead of every branch. D. Ben Knoble described his notes workflow and argued a `pushRefmap` would let him separate default push selection from ref mapping for notes pushes.

### repo: add path keys to `git repo info`
*Long thread: 53 replies so far from 6 participants over 115 days.*
K Jayatheerth's GSoC series exposes repository paths via `git repo info` with keys like `path.toplevel`, `path.superproject-root`, and `path.hooks`. After a month without review, Justin Tobler reviewed several patches and Jayatheerth posted v8 dropping `path.grafts` and special-casing `/dev/null`; Phillip Wood then suggested renaming keys. Junio C Hamano had warned the series would be discarded without review.

### reflog: add `%gb` format placeholder
*Long thread: 34 replies so far from 6 participants over 24 days.*
Harald Nordgren's series adds a `%gb` placeholder naming the branch HEAD was on when each reflog entry was recorded, showing "detached at <commit>" when detached. Phillip Wood found edge cases where the detached label showed the wrong commit and rebase checkouts were misparsed; v5 fixes these and handles commit subjects starting with "checkout".

### Removing gittutorial-2 from the documentation
*Long thread: 32 replies so far from 4 participants over 39 days.*
Julia Evans's series deletes `Documentation/gittutorial-2.adoc` and redirects readers to `gitdatamodel` and `gitcore-tutorial`. The removal graduated to `master`, and Tuomas Ahola asked Junio to pick up his rerolled lint topic, though an unanswered review comment from Evans remains.

### Reference-transaction hooks reporting old values
*Long thread: 31 replies so far from 5 participants over 45 days.*
Maciej Ciemborowicz's v6 patch resolves old object IDs in the common transaction hook layer so the reference-transaction hook reports real old values for unconditional deletions. Junio C Hamano said he will discard the topic unless volunteers review it within a few weeks.

### repack: corner cases for cruft-less MIDXs
*Long thread: 31 replies so far from 5 participants over 36 days.*
Taylor Blau's series fixes cases where bitmap generation could fail when geometric repacks omit cruft packs from the MIDX with `repack.midxMustContainCruft=false`. Junio C Hamano re-read the patches and found himself confused by the same points Jeff King had flagged.

### Community voting process proposal
*Long thread: 30 replies so far from 13 participants over 28 days.*
Emily Shaffer's document proposes a community voting process for decisions where consensus cannot be reached. Patrick Steinhardt argued that excluding long-inactive people is not unexpected since an informed vote requires community context, and insisted any human-override process must be fully transparent.

### Git 3.0 lowercase-only hex object IDs
*Long thread: 28 replies so far from 7 participants over 97 days.*
brian m. carlsson's v2 series removes acceptance of uppercase hex in object IDs in breaking-changes mode. Junio C Hamano noted the series had received no reactions and asked for review to confirm alignment with earlier discussion; Patrick Steinhardt added it to his todo list.

### root refs vs pseudorefs documentation
*Long thread: 27 replies so far from 4 participants over 16 days.*
Andy Koppe's series reworks the glossary to distinguish root refs like `ORIG_HEAD` from pseudorefs like `FETCH_HEAD` and `MERGE_HEAD`. Junio C Hamano reviewed v3, calling the pseudoref rewording "very very good" while flagging that non-root paths outside `refs/` are still treated as refs.

### Clarifying ejected topics can return
*Long thread: 23 replies so far from 5 participants over 103 days.*
Junio C Hamano proposed new [Discarded] heading text stating removed topics "can be resubmit if they gained enough interest and showed good fit to the system in the future," agreeing with Phillip Wood that ejection is not a permanent rejection.

### Hint for `<rev>~N` failures in shallow clones
*Long thread: 23 replies so far from 6 participants over 47 days.*
Harald Nordgren's patch adds a hint explaining why `<rev>~N` fails in a shallow clone when the walk stops at a recorded shallow boundary, suggesting the exact `--deepen` needed. Junio C Hamano observed that after a few days nobody had expressed support for the hint.

### migrating `loose-object-idx` with batch fsync
*Long thread: 23 replies so far from 4 participants over 13 days.*
Justin Tobler's series fixes an ODB transaction failure on commit when a compatibility hash is configured, because the temporary-directory migration code did not handle the loose object index file. Karthik Nayak agreed the series is already in good shape.

### worktree remove refuses rebasing/bisecting worktrees
*Long thread: 22 replies so far from 7 participants over 13 days.*
Jasper Mayone's v4 treats a branch being rebased or bisected as checked out: `worktree remove` refuses such worktrees without `--force`, `%(worktreepath)` names the rebasing worktree, and `git branch` drops the `+` marker there.

### Detached HEAD guide rewrite in the documentation
*Long thread: 22 replies so far from 4 participants over 33 days.*
Julia Evans's series rewrites the detached HEAD explanation into a standalone `gitdetached-head` guide wired into `advice.c` and `command-list.txt`. D. Ben Knoble named two blockers -- a reroll with the SQUASH??? commit applied and resolution of the HEAD-description discussion -- and concluded the thread is approaching a finish; Junio C Hamano objected to phrasing calling it useful for HEAD "to be a commit ID".

### Alternate refs during receive-pack quarantine
*Long thread: 21 replies so far from 5 participants over 9 days.*
Kristofer Karlsson's v3 patch fixes `rev-list --alternate-refs` so it no longer lists the repository's own refs during receive-pack's quarantine connectivity check, with a perf test showing a push into a repo with ~200,000 hidden refs improving from 0.29s to 0.10s. Jeff King reviewed it as looking good and Junio marked the topic ready for `next`.

### range-diff: add `--matched-only`
*Long thread: 20 replies so far from 5 participants over 55 days.*
Harald Nordgren's patch adds `git range-diff --matched-only` to show only commits that correspond on both sides. Kristoffer Haugsbakk and Ben Knoble noted the three-revision form `git range-diff base old new` already selects the same commits.

### standardizing early option scanning
*Long thread: 20 replies so far from 4 participants over 63 days.*
Christian Couder's v4 series introduces an "early-scan" sub-API in parse-options so commands can safely skip option values during hand-rolled early scans. Kaartic Sivaraam reviewed it as looking good and Junio C Hamano marked the topic for `next`.

### config: read both home and XDG files for `--global`
*Long thread: 20 replies so far from 5 participants over 393 days.*
Delilah Ashley Wu's series makes `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` via `do_git_config_sequence()`. Junio C Hamano affirmed the t1300 test patch and asked about XDG path assumptions.

### stash creation: avoid unnecessary index loads and refreshes
*Long thread: 20 replies so far from 2 participants over 33 days.*
Phillip Wood's seven-patch series removes duplicate index loads in `git stash push` and `git stash create`, cutting `lstat()` calls per changed path by four for create and eight for push. Junio C Hamano called the last two patches sensible and an obvious simplification.

### `test-tool genbytes` to speed up macOS CI tests
*Long thread: 15 replies so far from 3 participants over 2 days.*
Kristofer Karlsson's series adds a `test-tool genbytes` helper to replace `genzeros | tr` in expensive tests like t4205. Johannes Sixt's `printf` alternatives failed on macOS; Karlsson posted v2 adding `genbytes` and converting remaining callers.

### HTTP exponential-backoff and retry
*Long thread: 8 replies so far from 2 participants over 4 days.*
Ravi Mistry posted v3 of the HTTP retry series defaulting `http.retryAfter` to 10 seconds. Junio C Hamano questioned the topic structure of first implementing 0-second waits.

### Moving `assume_unchanged` into repository configuration
*Long thread: 6 replies so far from 2 participants over 1 day.*
Hussain Sitamouwala posted a patch moving the global `assume_unchanged` flag in `environment.c` into repository-specific configuration with a `repo_assume_unchanged()` getter.

## The period in brief
The week of 2026/11/02 through 2026/11/08 saw heavy traffic across seven active days, dominated by a partial-clone regression that broke Kubernetes CI and a maintainer push to eject unreviewed topics from `seen`. Readers should not miss Patrick Steinhardt's fix for the stale loose-object cache, Andy Koppe's root-refs documentation rework reaching v3, and Julia Evans's detached HEAD guide approaching completion.

## Key developments

### Partial-clone lazy fetch regression breaks Kubernetes CI
Patrick Ohly reported that `git merge` in blobless partial clones fails with "not our ref" since commit 5946a564, breaking Kubernetes CI PR merges. Kristofer Karlsson traced the root cause to a stale loose-object cache: writing a loose object does not update the cache, so existence checks wrongly report newly written objects missing, triggering a spurious promisor fetch. Patrick Steinhardt posted a two-patch fix adding `loose_cache_mark_seen()` to update the cache on each loose write, with a regression test verified by Karlsson; Jeff King noted the fix leaves related stale-cache cases such as objects moved into a pack.

### Ejecting unreviewed topics from `seen`
Junio C Hamano proposed ejecting topics unreviewed for N weeks from `seen`, with notices for those lingering over a month, and sent discard warnings for stalled series including the `git repo info` path keys and the reference-transaction hook fix. Patrick Steinhardt called the rule reasonable, noting evicted topics can be re-added, and suggested the bar for `seen` be at least one somewhat favorable human review. Phillip Wood proposed clarifying that ejected topics can return, and Junio agreed to new [Discarded] heading text stating removed topics can be resubmitted if they gained interest.

### Root refs vs pseudorefs documentation
Andy Koppe's series reworks the glossary and `gitdatamodel(7)` to distinguish root refs like `ORIG_HEAD` from pseudorefs like `FETCH_HEAD` and `MERGE_HEAD`, replacing outdated "pseudoref" uses across docs, code and tests. Patrick Steinhardt proposed an alternative pseudoref definition emphasizing additional semantics, which Junio C Hamano preferred with minor wording tweaks; Koppe posted v3 incorporating the feedback, which Hamano reviewed as "very very good" while flagging that non-root paths outside `refs/` are still treated as refs.

### Detached HEAD guide rewrite
Julia Evans's series rewrites the detached HEAD explanation into a standalone `gitdetached-head` guide wired into `advice.c` and `command-list.txt`. D. Ben Knoble identified two blockers -- applying the SQUASH??? commit and resolving the HEAD-description discussion -- and proposed merging paragraphs; Junio C Hamano objected to phrasing calling it useful for HEAD "to be a commit ID", suggesting instead "directly referring to a commit", and said the thread is approaching a finish.

### Geometric maintenance prunes commits without updating commit-graph
Christoph Weiss reported that after Forgejo switched to Git 2.54, changing the default maintenance strategy to `geometric` caused repository health checks to fail because geometric maintenance prunes commits without updating the commit-graph. He provided a reproduction script using `git maintenance run --auto` on a bare repository with `core.commitGraph` enabled.

### Reftable HEAD reflog records stale value
井上司 reported that with the reftable backend, a transaction updating a branch and pointing HEAD at it records the branch's old value in the HEAD reflog, or writes no entry when the branch is created in the same transaction. Karthik Nayak agreed the diagnosis is plausible and suggested moving reflog logic into the generic `refs.c` layer, while welcoming the specific fix.

### Default hash algorithm for Git 3.0
brian m. carlsson posted a patch defaulting the repository hash algorithm to `GIT_HASH_DEFAULT` so breaking-changes mode picks SHA-256, fixing `git init` with `WITH_BREAKING_CHANGES=1` which still defaulted to sha1. Junio C Hamano raised a concern that the patch would clobber the object format when `git init` reinitializes an existing repository, asking what happens with the reference storage format.

### splitIndex with skipHash corrupts index
Tanga Joris diagnosed that with `core.splitIndex` and `index.skipHash` both enabled, `write_shared_index()` names the file after the zeroed checksum, causing `read_index_from()` to treat the shared index as absent and crash `git status`. He posted a fix ensuring the shared index is never written with a zeroed checksum, adding tests to `t/t1700-split-index.sh`.

## In brief
- **Connectivity check ignoring replace refs**: Kristofer Karlsson's patch makes rev-list's `--connectivity-check` call `disable_replace_refs()`, endorsed by Junio C Hamano as absolutely the right thing to do.
- **`git describe` distance with skewed commit dates**: Linus Torvalds' patch uses commit generation numbers to fix miscounting; Patrick Steinhardt and Derrick Stolee agreed `ensure_generations_valid()` should be called in `describe_commit()`.
- **fsmonitor/untracked-cache bug**: Derrick Stolee fixed a bug since 2017 where failed fsmonitor responses could leave untracked cache entries marked valid, causing `git status` to miss new files; Junio queued it as a "blunt instrument" but correct.
- **Path-walk regression**: Derrick Stolee fixed a regression from Git 2.55.0 that dropped blobs from tagged trees at already-visited paths.
- **Config NULL values segfaults**: Thomas Bachem's 15-patch series fixes segfaults from valueless config settings across rebase, fetch, grep, blame, merge and more.
- **Merge honoring --no-overwrite-ignore**: Aniket Gupta's patch adds a `preserve_ignored` bit so `git merge --no-overwrite-ignore` aborts instead of overwriting ignored files during real merges.
- **Replaying 2-parent merges**: Philippe Blain reported that replaying a merge whose first parent was amended incorrectly maps the second parent, causing conflicts, while calling the series a massive time saver.
- **Freezing `repo_config_values`**: Junio C Hamano proposed freezing the structure against new members; Patrick Steinhardt approved, wanting to prevent newcomers from wasting time.
- **CI test-variable consolidation**: Patrick Steinhardt's v2 was reviewed by Phillip Wood as ready for `next`.
- **`git repo structure` degree-of-concern**: Mark C. Chu-Carroll posted v5 adding concern reporting with configurable thresholds.
- **HTTP retry series**: Ravi Mistry posted v3 defaulting `http.retryAfter` to 10 seconds.
- **Stash index optimization**: Phillip Wood's series cuts `lstat()` calls in `git stash push` and `create`.
- **Config --global XDG**: Delilah Ashley Wu's series reads both home and XDG config files.
- **Documentation/Governance.adoc**: Antonin Delpeuch submitted a patch describing project governance; Junio asked for line-wrapping and clarifications.
- **Bare --force-with-lease**: Patrick Peroša reported the flag is ignored on remote groups.
- **`git log --graph` indentation**: herr.kaste fixed visual roots with columns to their right being indented.
- **size_t for large blobs**: Elia Pinto's v3 fixes truncation of blobs over 4GB on 32-bit `long` platforms.
- **WWW-Authenticate headers**: Jeff King reviewed the v2 patch preserving headers across HTTP redirects as OK.
- **Sequencer shallow boundary**: Phillip Wood called the v2 patch refusing shallow boundary commits "definitely moving in the right direction".
- **send-pack series**: Marc Strapetz revived the series inspecting the old tip's tree to avoid resending unchanged blobs.
- **`git log --follow` regression**: b.murauer reported a regression bisected to 304812ed33; Patrick Steinhardt noted patches are in review.
- **notes_tree repository storage**: Okhuomon Ajayi posted a series storing the repository in `struct notes_tree`.
- **GIT_TEST_DEFAULT_OBJECT_STORAGE_FORMAT**: Patrick Steinhardt posted a series to run tests with non-default object storage.
- **promisor.fetchOrder**: Yopa Nelly posted v2 with parsed list per-repository; Junio asked how it meshes with server-side work.
- **unpack-trees cleanups**: Junio C Hamano posted patches dropping `USE_THE_REPOSITORY` and renaming parameters.
- **Sideband fix**: Junio marked Thomas Bachem's fix skipping unrecognized `allowControlCharacters` keywords ready for `next`.
- **Outreachy patches**: Tanga Joris posted fixes for `git apply` crashes; Cynthia Uche posted a proposal draft targeting `git bisect`.
- **credential-cache daemon race**: Farid Zakaria's fix drew questions from Junio about existing arrangements.
- **t0027 optimization**: Torsten Bögershausen reported cutting runtime from 71 to 54 seconds on older macOS.
- **builtin/reflog expire**: Laveesh Bansal posted a patch expiring `--all` and named refs in a single loop.
- **diff EXAMPLES anchors**: Jeff King flagged that `[[name_only]]` anchors work with Asciidoctor but not python AsciiDoc.
- **followRemoteHEAD validation**: Colin Hinton will adopt Matt Hunter's cleaner approach in his next re-roll.
- **All-or-none switch/checkout**: 井上司 proposed writing tracking config before worktree update and removing it on failure.
- **assume_unchanged repo config**: Hussain Sitamouwala posted a patch moving the flag into repository configuration.

## Looking ahead
The `git repo info` path keys series remains under review pressure after Junio C Hamano's repeated warnings, with v8 posted but Phillip Wood's renaming suggestions still unaddressed. The shallow-clone `<rev>~N` hint still lacks expressed support and may be discarded, as may the reference-transaction hooks topic unless volunteers review it soon.