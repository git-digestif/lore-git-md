# Git mailing list daily digest for 2026/11/07

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 66 replies so far from 5 participants over 50 days; 1 message that day.*

Harald Nordgren's series adds `remote.<name>.fetchRefmap` so a shallow repository can fetch only selected branches of a new remote instead of every branch, with `--limited-fetch` remotes defaulting to limited fetch. Earlier revisions renamed the config to `fetchRefmap`, added the `get_ref_map()` path storing named branches through the refmap, and fixed `@{push}` resolution. Today D. Ben Knoble described his notes workflow -- keeping both Junio's amlog and his own patch notes while pushing only his own -- and argued a `pushRefmap` would let him separate default push selection from ref mapping for notes pushes, sharing his config with `core.notesRef`, `notes.rewriteRef`, and a negative refspec excluding amlog.

### repo: add path keys to git repo info

*Long thread: 53 replies so far from 6 participants over 115 days; 2 messages that day.*

K Jayatheerth's GSoC series exposes repository paths via `git repo info` with keys like `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, and `path.cdup`, and refactors `get_superproject_working_tree()` to take a `struct repository *`. After a month without review, Justin Tobler reviewed several patches and Jayatheerth agreed to his suggestions. Today Jayatheerth posted v8, dropping `path.grafts`, moving a test to `t/t1500-rev-parse.sh`, and special-casing `/dev/null` in `path.hooks`. Phillip Wood then suggested renaming keys (`path.working-tree.*`, `path.prefix`), questioned whether `path.hooks` is needed, and asked how `path.cdup` differs from `path.working-tree.relative`.

### reflog: add format %gb

*Long thread: 34 replies so far from 6 participants over 24 days; 2 messages that day.*

Harald Nordgren's series adds a `%gb` placeholder to `git log -g --format` naming the branch HEAD was on when each reflog entry was recorded, showing "detached at <commit>" when detached. Earlier revisions fixed branch naming under `GIT_REFLOG_ACTION` during rebase. Today he posted v5: "detached at" now names the commit HEAD was detached at, `git rebase <upstream> <rev>` counts `<rev>` as a branch only if `refs/heads/<rev>` exists, commit subjects starting with "checkout" are no longer mistaken for rebase checkouts, and tests cover aborted and fast-forwarding detached rebases.

### doc: checkout: rewrite detached HEAD state explanation

*Long thread: 21 replies so far from 4 participants over 32 days; 1 message that day.*

Julia Evans's patch rewrites the detached HEAD explanation into a standalone `git help detached-head` guide wired into `advice.c` and `command-list.txt`. Earlier rounds debated how to describe HEAD, with Evans citing user research. Today D. Ben Knoble reiterated interest in landing the manual, naming two blockers -- a reroll with the SQUASH??? commit applied and resolution of the HEAD-description discussion -- proposed merging v4's second paragraph with Junio C Hamano's points about not clobbering branches, and concluded the thread is approaching a finish.

## The day in brief
Thomas Bachem's 15-patch series fixes segfaults from valueless config settings across rebase, fetch, grep, blame, merge and more. Harald Nordgren posted v5 of the `%gb` reflog format, and K Jayatheerth posted v8 of the `git repo info` path keys. Tanga Joris diagnosed a splitIndex/skipHash index corruption crash.

## Notable threads

### config: handle NULL values when listing colors and expiry dates
Thomas Bachem submitted a 15-patch series fixing segfaults caused by settings given without a value (`git -c foo.bar`), covering `rebase.missingCommitsCheck`, `core.alternateRefs*`, interactive patch color slots, `fetch.bundleCreationToken`, `grep.patternType`, `remote.*.tagOpt`, `color.blame.*`, `branch.*.mergeOptions`, `remotes.*`, `safe.bareRepository`, bitmap pseudo-merge patterns, `scalar.repo`, `hasconfig` includes, bundle list keys, and `git config list --type=color`/`--type=expiry-date` (a regression from 2.54.0).

### splitIndex with skipHash corrupts index and crashes status
With `core.splitIndex` and `index.skipHash` both enabled, index entries are lost or corrupted when the shared index is written, crashing `git status`. Tanga Joris reproduced both cases on master and diagnosed the cause: `write_shared_index()` names the file after the zeroed checksum (`sharedindex.000...`), and the null base makes `read_index_from()` treat the shared index as absent. He is working on a fix; Andy Koppe confirmed he had not started one.

### merge: honor --no-overwrite-ignore for real merges
Aniket Gupta's patch adds a `preserve_ignored` bit to `struct merge_options` so `git merge --no-overwrite-ignore` aborts instead of overwriting ignored files during real merges -- previously honored only for fast-forwards -- wiring the option through the trivial-merge path in `builtin/merge.c` and merge-ort's checkout, with tests for a plain merge and a directory rename.

## In brief
- Tanga Joris posted two Outreachy patches fixing `git apply` crashes: freeing names from unused `diff --git` lines, and rejecting new/deleted files that carry both names.
- Aniket Gupta clarified that Hannes's pure-rename-then-edit `format-patch` series is valid input for a single `git apply`, which broke only with `core.fileMode=false`; Junio C Hamano questioned whether one invocation applying two patches is intended.
- Junio C Hamano posted two `unpack-trees` cleanups: dropping `USE_THE_REPOSITORY` via an `unpack_repo()` helper, and renaming an `index` parameter to `istate` for consistency.
- Junio C Hamano reviewed Thomas Bachem's sideband fix skipping unrecognized `allowControlCharacters` keywords, called the change reasonable, and marked the topic ready for `next`.
- Ravi Mistry posted v3 of the HTTP retry series, defaulting `http.retryAfter` to 10 seconds; Junio C Hamano questioned the topic structure of first implementing 0-second waits.
