## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo
*Long thread: 66 replies so far from 5 participants over 50 days.*
Harald Nordgren's series adds `remote.<name>.fetchRefmap` so shallow repositories can fetch only selected branches of a new remote instead of every branch, with `--limited-fetch` remotes defaulting to limited fetch. Earlier revisions renamed the config to `fetchRefmap`, added the `get_ref_map()` path storing named branches through the refmap, and fixed `@{push}` resolution. On November 7, D. Ben Knoble described his notes workflow and argued a `pushRefmap` would let him separate default push selection from ref mapping for notes pushes, sharing his config with `core.notesRef` and a negative refspec excluding amlog.

### repo: add path keys to `git repo info`
*Long thread: 53 replies so far from 6 participants over 115 days.*
K Jayatheerth's GSoC series exposes repository paths via `git repo info` with keys like `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, and `path.cdup`, and refactors `get_superproject_working_tree()` to take a `struct repository *`. After a month without review, Justin Tobler reviewed several patches and Jayatheerth agreed to his suggestions. On November 7, Jayatheerth posted v8, dropping `path.grafts`, moving a test to `t/t1500-rev-parse.sh`, and special-casing `/dev/null` in `path.hooks`. Phillip Wood then suggested renaming keys to `path.working-tree.*`, questioned whether `path.hooks` is needed, and asked how `path.cdup` differs from `path.working-tree.relative`.

### reflog: add format `%gb`
*Long thread: 34 replies so far from 6 participants over 24 days.*
Harald Nordgren's series adds a `%gb` placeholder to `git log -g --format` naming the branch HEAD was on when each reflog entry was recorded, showing "detached at <commit>" when detached. Earlier revisions fixed branch naming under `GIT_REFLOG_ACTION` during rebase. On November 7, he posted v5: "detached at" now names the commit HEAD was detached at, `git rebase <upstream> <rev>` counts `<rev>` as a branch only if `refs/heads/<rev>` exists, commit subjects starting with "checkout" are no longer mistaken for rebase checkouts, and tests cover aborted and fast-forwarding detached rebases.

### Removing gittutorial-2 from the documentation
*Long thread: 32 replies so far from 4 participants over 39 days.*
Julia Evans's two-patch series deletes `Documentation/gittutorial-2.adoc` and all references to it, leaving a stub that redirects readers to `gitdatamodel` and `gitcore-tutorial`. Earlier revisions removed the entry from `command-list.txt` so `git(1)` would not advertise the obsolete guide, and Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out of the series. On November 5, Tuomas Ahola reported that the removal topic has graduated to `master` and asked Junio to pick up his rerolled `ta/command-list-guides-sync-lint`; after confirming the patch was the result of the rebasing he had planned, Junio noted the thread still has an unanswered review comment from Julia Evans and asked whether it is ready to be picked up.

### Reference-transaction hooks reporting old values
*Long thread: 31 replies so far from 5 participants over 45 days.*
Maciej Ciemborowicz's v6 patch resolves old object IDs in the common transaction hook layer so the reference-transaction hook reports real old values for unconditional deletions, keeping them separate from caller-supplied values. Earlier, Patrick Steinhardt endorsed the common-layer approach. On November 2, Junio C Hamano said he will discard the topic from his tree unless volunteers review it within a few weeks, and Maciej Ciemborowicz acknowledged, thanking him and hoping someone else will review it.

### repack: corner cases for cruft-less MIDXs
*Long thread: 31 replies so far from 5 participants over 36 days.*
With `repack.midxMustContainCruft=false`, geometric repacks can omit cruft packs from the MIDX while relying on the indexed set being closed under reachability; Taylor Blau's series adds tree and tag roots, retains cruft for ordinary incremental repacks, and includes required packs in ordinary and incremental MIDX writes, fixing cases where bitmap generation could fail. On November 4, Junio C Hamano re-read the patches and their review discussion, found himself confused by the same points Jeff King had flagged, and said he has no concrete improvement ideas right now.

### Community voting process proposal
*Long thread: 30 replies so far from 13 participants over 28 days.*
Emily Shaffer's document proposes a community voting process for decisions where consensus cannot be reached, with eligibility thresholds debated earlier. On November 2, Patrick Steinhardt argued that excluding long-inactive people is not necessarily unexpected, since an informed vote requires community context, and said that any human-override process must be fully transparent--who added whom, with reasoning, published well ahead of time.

### Git 3.0 lowercase-only hex object IDs
*Long thread: 28 replies so far from 7 participants over 97 days.*
brian m. carlsson's v2 series removes acceptance of uppercase hex in object IDs in breaking-changes mode, addressing security issues where downstream software assumes a unique lowercase hex form. On November 2, Junio C Hamano noted the series had received no reactions and asked for review to confirm it aligns with earlier discussion so it can be marked for `next`; Patrick Steinhardt added it to his todo list for the week.

### root refs vs pseudorefs documentation
*Long thread: 27 replies so far from 4 participants over 16 days.*
Andy Koppe's series reworks the glossary and `gitdatamodel(7)` to distinguish root refs like `ORIG_HEAD` from pseudorefs like `FETCH_HEAD` and `MERGE_HEAD`, and stops calling root refs pseudorefs across docs, refs code and tests. Koppe posted v3 on November 6, explaining pseudorefs' additional semantics in the glossary, clarifying the `FETCH_HEAD` and `MERGE_HEAD` descriptions, and rewording two commit messages. Junio C Hamano reviewed patches 1-4, calling the pseudoref rewording "very very good" and the `core.logAllRefUpdates` correction a fix for a "grave mistake".

### Ejecting unreviewed topics from `seen`
*Long thread: 23 replies so far from 5 participants over 103 days.*
Junio C Hamano proposed a rule that topics unreviewed for N weeks are ejected from `seen`, announcing notices for every topic lingering unreviewed for more than a month. Patrick Steinhardt called the rule reasonable, noting evicted topics can be re-added if interest returns. On November 6, agreeing with Phillip Wood that ejection is not a permanent rejection, Junio proposed new [Discarded] heading text stating removed topics "can be resubmit if they gained enough interest and showed good fit to the system in the future."

### migrating `loose-object-idx` with batch fsync
*Long thread: 23 replies so far from 4 participants over 13 days.*
Justin Tobler's series fixes an ODB transaction failure on commit when a compatibility hash is configured, because the temporary-directory migration code did not handle the loose object index file; v3 reads the source mapping before acquiring the lock and adds a comment to `loose_object_idx_merge()`. On November 3, Karthik Nayak agreed the series is already in good shape after earlier naming suggestions were declined as not worth a reroll.

### Hint for `<rev>~N` failures in shallow clones
*Long thread: 23 replies so far from 6 participants over 47 days.*
Harald Nordgren's patch adds a hint in the `object-name` code explaining why `<rev>~N` fails in a shallow clone when the walk runs out of parents exactly at a recorded shallow boundary, suggesting the exact `--deepen` needed. On November 5, Junio C Hamano observed that after a few days nobody had expressed support for the hint, and that the thread meant to invite supporters had instead been cluttered with messages about the review process.

### Detached HEAD guide rewrite in the documentation
*Long thread: 22 replies so far from 4 participants over 33 days.*
Julia Evans's series rewrites the explanation of detached HEAD state into a standalone `gitdetached-head` guide, wired into `advice.c`, `command-list.txt`, and the build files so the advice printed on detaching can point users to `git help detached-head`. On November 8, Junio C Hamano answered D. Ben Knoble's closing suggestions, explaining that the "parent of the next commit" mental model matches HEAD being "where you started from", but objected to phrasing that calls it useful for HEAD "to be a commit ID", suggesting instead "directly referring to a commit (instead of indirectly referring to a commit by naming the current branch)".

### Alternate refs during receive-pack quarantine
*Long thread: 21 replies so far from 5 participants over 9 days.*
Kristofer Karlsson's patch fixes the object database code so that `rev-list --alternate-refs` no longer lists the repository's own refs--including hidden ones--while receive-pack's quarantine connectivity check runs. On November 5, v3 reverts to skipping the repository's own object directory directly in the alternate-refs code, with a perf test showing a push into a repository with ~200,000 hidden refs improving from 0.29s to 0.10s. Jeff King reviewed the v3 patch and said it looks good, and Junio C Hamano marked the topic ready for `next`.

### range-diff: add `--matched-only`
*Long thread: 20 replies so far from 5 participants over 55 days.*
Harald Nordgren's patch adds `git range-diff --matched-only` to show only commits that correspond on both sides, since `--left-only` and `--right-only` cannot be combined. On November 4, Kristoffer Haugsbakk said he uses the `base..old base..new` form so unrelated commits never appear, and Ben Knoble corrected that the three-revision form `git range-diff base old new` selects the same commits without repeating the base.

### config: read both home and XDG files for `--global`
*Long thread: 20 replies so far from 5 participants over 393 days.*
Delilah Ashley Wu's series makes `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config` via `do_git_config_sequence()`. On November 6, Junio C Hamano affirmed the t1300 test patch, asked whether XDG config is always `~/.config/git/config`, asked Wu to shorten the config_sequence commit message, and wondered about adding a `BUG(...)` guard in `git_config_system()` against NULL callers.

### stash creation: avoid unnecessary index loads and refreshes
*Long thread: 20 replies so far from 2 participants over 33 days.*
Phillip Wood's seven-patch series removes duplicate index loads and refreshes in `git stash push` and `git stash create`, cutting `lstat()` calls per changed path by four for create and eight for push. On November 6, Junio C Hamano reviewed the last two patches, calling 6/7 sensible since `create_index_from_tree()` is file-scope static and 7/7 an obvious, very nice simplification of `reset_tree()`'s signature.

### `test-tool genbytes` to speed up macOS CI tests
*Long thread: 15 replies so far from 3 participants over 2 days.*
Kristofer Karlsson's series adds a `test-tool genbytes <hex-byte> [<count>]` helper and converts tests away from `genzeros | tr`, because t4205's EXPENSIVE tests spend most of their time in `tr(1)` on macOS. On November 8, Karlsson posted v2 as two patches: patch 1 adds `genbytes` taking the byte as two lowercase hex digits and renames test-genzeros.c into test-genbytes.c, and patch 2 converts the remaining `genzeros` callers and removes it.

### Moving `assume_unchanged` into repository configuration
*Long thread: 6 replies so far from 2 participants over 1 day.*
Hussain Sitamouwala posted a patch moving the global `assume_unchanged` flag in `environment.c` into repository-specific configuration, adding a `repo_assume_unchanged()` getter and updating the callers in `builtin/update-index.c` and `read-cache.c`, so the setting is no longer shared across repositories.

## The period in brief
The week of 2026/11/02 through 2026/11/08 saw heavy traffic across all seven days, with several long-running series reaching new iterations and a significant partial-clone regression fixed. Readers should not miss Patrick Steinhardt's fix for the stale loose-object cache that broke `git merge` in blobless partial clones, K Jayatheerth's v8 of the `git repo info` path keys, and Junio C Hamano's proposal to e