## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 66 replies so far from 5 participants over 50 days.*

Harald Nordgren's series adds `remote.<name>.fetchRefmap` so a shallow repository can fetch only selected branches of a new remote instead of every branch, with `--limited-fetch` remotes defaulting to limited fetch. Earlier revisions renamed the config to `fetchRefmap`, added the `get_ref_map()` path storing named branches through the refmap, and fixed `@{push}` resolution. D. Ben Knoble described his notes workflow -- keeping both Junio's amlog and his own patch notes while pushing only his own -- and argued a `pushRefmap` would let him separate default push selection from ref mapping for notes pushes, sharing his config with `core.notesRef`, `notes.rewriteRef`, and a negative refspec excluding amlog.

### repo: add path keys to git repo info

*Long thread: 53 replies so far from 6 participants over 115 days.*

K Jayatheerth's GSoC series exposes repository paths via `git repo info` with keys like `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, and `path.cdup`, and refactors `get_superproject_working_tree()` to take a `struct repository *`. After a month without review, Justin Tobler reviewed several patches and Jayatheerth agreed to his suggestions. Jayatheerth posted v8, dropping `path.grafts`, moving a test to `t/t1500-rev-parse.sh`, and special-casing `/dev/null` in `path.hooks`. Phillip Wood then suggested renaming keys (`path.working-tree.*`, `path.prefix`), questioned whether `path.hooks` is needed, and asked how `path.cdup` differs from `path.working-tree.relative`. Junio C Hamano had warned the series would be discarded without review.

### reflog: add %gb format placeholder

*Long thread: 34 replies so far from 6 participants over 24 days.*

Harald Nordgren's series adds a `%gb` placeholder to `git log -g --format` naming the branch HEAD was on when each reflog entry was recorded, showing "detached at <commit>" when detached. Phillip Wood reported that the "detached at" label shows the commit before switching away rather than where HEAD was detached, that parsing "rebase: checkout <revision>" treats "origin/main" as a branch name and is confused by commit subjects starting with "checkout", and supplied a failing test covering those issues and ':' in a URL in `$GIT_REFLOG_ACTION`. Nordgren posted v5: "detached at" now names the commit HEAD was detached at, `git rebase <upstream> <rev>` counts `<rev>` as a branch only if `refs/heads/<rev>` exists, commit subjects starting with "checkout" are no longer mistaken for rebase checkouts, and tests cover aborted and fast-forwarding detached rebases.

### Removing gittutorial-2 from the documentation

*Long thread: 32 replies so far from 4 participants over 39 days.*

Julia Evans's two-patch series deletes `Documentation/gittutorial-2.adoc` and all references to it, leaving a stub that redirects readers to `gitdatamodel` and `gitcore-tutorial`, to clear obsolete tutorial material. Earlier revisions removed the entry from `command-list.txt` so `git(1)` would not advertise the obsolete guide, and Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out of the series. Tuomas Ahola reported that the removal topic has graduated to `master` and asked Junio to pick up his rerolled `ta/command-list-guides-sync-lint`; after confirming the patch was the result of the rebasing he had planned, Junio noted the thread still has an unanswered review comment from Julia Evans and asked whether it is ready to be picked up.

### Reference-transaction hooks reporting old values

*Long thread: 31 replies so far from 5 participants over 45 days.*

Maciej Ciemborowicz's v6 patch resolves old object IDs in the common transaction hook layer so the reference-transaction hook reports real old values for unconditional deletions, keeping them separate from caller-supplied values. Earlier, Patrick Steinhardt endorsed the common-layer approach. Junio C Hamano said he will discard the topic from his tree unless volunteers review it within a few weeks, and Ciemborowicz acknowledged, thanking him and hoping someone else will review it.

### repack: corner cases for cruft-less MIDXs

*Long thread: 31 replies so far from 5 participants over 36 days.*

With `repack.midxMustContainCruft=false`, geometric repacks can omit cruft packs from the MIDX while relying on the indexed set being closed under reachability; Taylor Blau's series adds tree and tag roots, retains cruft for ordinary incremental repacks, and includes required packs in ordinary and incremental MIDX writes, fixing cases where bitmap generation could fail. Junio C Hamano re-read the patches and their review discussion, found himself confused by the same points Jeff King had flagged, and said he has no concrete improvement ideas right now. Earlier, Taylor had said the series should be ready as-is after Junio questioned the topic's viability, a question Junio now clarified was not meant for the author to answer.

### Community voting process proposal

*Long thread: 30 replies so far from 13 participants over 28 days.*

Emily Shaffer's document proposes a community voting process for decisions where consensus cannot be reached, with eligibility thresholds debated earlier (commit counts, activity windows, human-override exceptions). Patrick Steinhardt argued that excluding long-inactive people is not necessarily unexpected, since an informed vote requires community context, and said that any human-override process (Junio or the PLC adding specific voters) must be fully transparent -- who added whom, with reasoning, published well ahead of time.

### Git 3.0 lowercase-only hex object IDs

*Long thread: 28 replies so far from 7 participants over 97 days.*

brian m. carlsson's v2 series removes acceptance of uppercase hex in object IDs in breaking-changes mode, addressing security issues where downstream software assumes a unique lowercase hex form. Junio C Hamano noted the series had received no reactions and asked for review to confirm it aligns with earlier discussion so it can be marked for `next`; Patrick Steinhardt added it to his todo list for the week.

## The period in brief

The week of 2026/11/02 through 2026/11/08 saw seven active days of heavy traffic dominated by a partial-clone regression that broke Kubernetes CI, a maintainer policy shift toward ejecting unreviewed topics from `seen`, and continued iteration on long-running series including the `git repo info` path keys and the `%gb` reflog placeholder. Readers should not miss Patrick Steinhardt's fix for the stale loose-object cache behind the partial-clone failure, Junio C Hamano's proposal to eject stalled topics, and the splitIndex/skipHash corruption diagnosis and fix.

## Key developments

### Partial clone lazy fetch regression

Patrick Ohly reported that `git merge` in a blobless partial clone fails with "not our ref" while lazily fetching a missing object, breaking Kubernetes CI PR merges, and bisected v2.52.0 (good) to v2.54.0 (bad) identifying 5946a564 as the first bad commit. Kristofer Karlsson reproduced it and traced it to the loose object cache: writing a loose object does not update the cache, so existence checks wrongly report newly written objects missing, triggering a promisor fetch for a locally present blob. Patrick Steinhardt confirmed the root cause predates the bisected commit -- 5946a564cd stopped flushing the cache on misses, which had masked the bug -- and sent a two-patch fix adding `loose_cache_mark_seen()` to update the cache on each loose write, plus a regression test in `t/t5616-partial-clone.sh`. Karlsson verified the test fails without the fix; Derrick Stolee and Junio C Hamano also reviewed it as looking good.

### Ejecting unreviewed topics from `seen`

Junio C Hamano proposed a rule that topics unreviewed for N weeks are ejected from `seen`, announcing notices for every topic lingering unreviewed for more than a month, and sent discard warnings for stalled series including the `git repo info` path keys and the reference-transaction hook fix. Patrick Steinhardt called the rule reasonable, noting evicted topics can be re-added if interest returns, and flagged the confusing "Will merge to `next`?" state; Junio explained the trailing question mark means "I plan to do this unless someone stops me." Phillip Wood suggested clarifying that ejected topics can return to lessen contributor worry, and Junio later proposed new [Discarded] heading text stating removed topics "can be resubmit if they gained enough interest and showed good fit to the system in the future."

### `git describe` distance with skewed commit dates

Linus Torvalds' patch fixes `git describe` miscounting distance when commit dates are unreliable by using commit generation numbers. Patrick Steinhardt and Derrick Stolee agreed `ensure_generations_valid()` should be called in `describe_commit()`, accepting the cost without a commit graph is better than a wrong answer; Junio C Hamano asked whether the cost could be amortized by writing a commit-graph, and Linus Torvalds argued consistently wrong numbers are worse and suggested a flag to force exact computation. Kristofer Karlsson later proposed an exact-depth prototype without generation numbers, and Linus blamed GitLab Marge-bot's --keep-committers, not clock skew, for the kernel case.

### splitIndex and skipHash corruption

Tanga Joris reproduced that with `core.splitIndex` and `index.skipHash` both enabled, index entries are lost or corrupted when the shared index is written, crashing `git status`. He diagnosed the cause: `write_shared_index()` names the file after the zeroed checksum (`sharedindex.000...`), and the null base makes `read_index_from()` treat the shared index as absent. Joris fixed `read-cache.c` so the shared index is never written with a zeroed checksum under `index.skipHash`, adding two tests to `t/t1700-split-index.sh`.

### Reftable HEAD reflog stale value

井上司 reported that with the reftable backend, a transaction that updates a branch and points HEAD at it records the branch's old value in the HEAD reflog, or writes no entry at all when the branch is created in the same transaction, unlike the files backend. He traced the cause to `write_transaction_table()` in `refs/reftable-backend.c`, which resolves the target with pre-transaction refs, and proposed taking the new oid from the same transaction. Karthik Nayak agreed the diagnosis is plausible, noted the files backend avoids the bug only because it lacks atomicity support, and suggested moving reflog logic into the generic `refs.c` layer so backends receive pre-resolved entries.

### Default hash algorithm for Git 3.0

brian m. carlsson posted a patch to `setup.c` defaulting the repository hash algorithm to `GIT_HASH_DEFAULT` when nothing else selects one, so breaking-changes mode and Git 3.0 pick SHA-256, with a new test in `t/t0001-init.sh`. Adam Dinwoodie had reported that `git init` built with `WITH_BREAKING_CHANGES=1` still creates repositories with the sha1 object format. Junio C Hamano said the patch looks good for its stated goal but raised that it would clobber the object format when `git init` reinitializes an existing repository, outlining possible reinit behaviors and asking what happens with the reference storage format.

### fsmonitor and untracked cache invalidation

Derrick Stolee posted a patch for an fsmonitor/untracked-cache bug present since 2017: when fsmonitor fails or returns a trivial response, untracked cache entries could stay marked valid, so `git status` could miss new untracked files. The fix invalidates every untracked cache directory and sets UNTRED_CHANGED on a failed token request. Junio C Hamano called it "rather a blunt instrument" but said being without acceleration beats being fast and incorrect, and queued it; he also asked whether it could replace an unreviewed pending series on the same topic. Stolee later reviewed Ilia K's selective invalidation patch, judged it better than his own full-clear approach, and found no issues worth re-rolling.

### Geometric maintenance and commit-graph

Christoph Weiss reported a bug in the maintenance code: after Forgejo switched Git from 2.52 to 2.54, changing the default maintenance strategy from `gc` to `geometric`, repository health checks started failing because under `geometric` maintenance prunes commits without updating the commit-graph. He provided a reproduction script using `git maintenance run --auto` on a bare repository with `core.commitGraph` enabled.

## In brief

**Connectivity check ignoring replace refs** -- Kristofer Karlsson's patch makes rev-list's `--connectivity-check` call `disable_replace_refs()` and drop `OBJECT_INFO_LOOKUP_REPLACE` from `tree-verify.c`; Junio C Hamano endorsed it as absolutely the right thing to do.

**Alternate refs during quarantine** -- Kristofer Karlsson's v3 fix so `rev-list --alternate-refs` no longer lists the repository's own refs during receive-pack quarantine was marked ready for `next`, with a perf test showing a push into a repository with ~200,000 hidden refs improving from 0.29s to 0.10s.

**Early option scanning** -- Christian Couder's v4 series introducing `early_scan_options()` in parse-options was marked for `next` by Junio C Hamano.

**repo_config_values freeze** -- Junio C Hamano proposed freezing the `repo_config_values` structure against new members in `environment.h`; Patrick Steinhardt approved, saying he is very much in favor of merging it.

**Detached HEAD guide** -- Julia Evans's rewrite of the detached HEAD explanation into a standalone `gitdetached-head` guide approached a finish, with Junio C Hamano answering D. Ben Knoble's closing suggestions and objecting to phrasing that calls it useful for HEAD "to be a commit ID".

**Root refs vs pseudorefs** -- Andy Koppe's v3 series reworking the glossary and `gitdatamodel(7)` to distinguish root refs from pseudorefs was reviewed by Junio C Hamano, who called the pseudoref rewording "very very good" and the `core.logAllRefUpdates` correction a fix for a "grave mistake".

**test-tool genbytes** -- Kristofer Karlsson's v2 series adding `test-tool genbytes <hex-byte> [<count>]` to speed up macOS CI tests settled the discussion, with Johannes Sixt's `printf` alternatives shown to fail on macOS.

**Stash creation optimization** -- Phillip Wood's seven-patch series removing duplicate index loads and refreshes in `git stash push` and `git stash create` was reviewed by Junio C Hamano, who called 7/7 an obvious, very nice simplification.

**Config XDG reading** -- Delilah Ashley Wu's series making `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` received affirmation from Junio C Hamano on the t1300 test patch.

**Shallow-clone hint** -- Harald Nordgren's `<rev>~N` failure hint still lacks any expressed support according to Junio C Hamano, who observed the thread meant to invite supporters had instead been cluttered with messages about the review process.

**loose-object-idx migration** -- Justin Tobler's v3 series fixing an ODB transaction failure on commit when a compatibility hash is configured was agreed by Karthik Nayak to be already in good shape.

**range-diff --matched-only** -- Harald Nordgren's patch adding `git range-diff --matched-only` drew discussion, with Kristoffer Haugsbakk and Ben Knoble noting the three-revision form selects the same commits without repeating the base.

**assume_unchanged configuration** -- Hussain Sitamouwala posted a patch moving the global `assume_unchanged` flag in `environment.c` into repository-specific configuration.

**Valueless config segfaults** -- Thomas Bachem submitted a 15-patch series fixing segfaults caused by settings given without a value (`git -c foo.bar`) across rebase, fetch, grep, blame, merge and more.

**All-or-none switch/checkout** -- 井上司 proposed writing tracking config before the worktree update during `git switch` or `checkout` to avoid leaving the user with unrelated staged changes when the config write fails.