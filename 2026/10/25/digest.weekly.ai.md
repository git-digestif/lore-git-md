## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 69 replies so far from 6 participants over 30 days.*
Julia Evans's six-patch series adding a `gitmergeconflicts` man page reached v4, folding in Junio C Hamano's diff3 example and cleanups such as sentence periods. Ben Knoble signaled the thread was reaching diminishing returns, while Junio flagged a tab-indentation inconsistency and a missing "from" in one phrase but otherwise found nothing questionable. The page will be linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation.

### `git branch --delete-merged` and squash-merged branches
*Long thread: 47 replies so far from 6 participants over 25 days.*
Harald Nordgren's series to make `--delete-merged` detect squash- and rebase-merged branches went through v3 and extensive design debate. Junio C Hamano objected to loosening the default, leading to proposals for `--delete-integrated` and level-based options like `=squashed`. By week's end Nordgren conceded that `--delete-merged=squashed` is stopped because the option's value slot is already taken by the upstream `<pattern>`, leaving the design unresolved.

### fetch: avoid fetching every branch of a new remote in a shallow repo
*Long thread: 45 replies so far from 4 participants over 37 days.*
Harald Nordgren posted v10 and v11 of his five-patch series preventing shallow repositories from fetching every branch of a newly added remote. The series adds `remote.<name>.refmap`, a `--limited-fetch` option to `git remote add`, and infers fetch targets from local branches whose `@{upstream}` is on that remote. D. Ben Knoble supplied the requested second review, questioning `remote_find_tracking()` fallback behavior, which Nordgren addressed in v11 with tests.

### CI housekeeping and modernization series
*Long thread: 41 replies so far from 6 participants over 12 days.*
Patrick Steinhardt's eight-patch CI series fixes issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed". Junio C Hamano marked v4 for `next`, while Steinhardt queued a follow-up to resurrect a no-Rust CI job via `NO_RUST` in a Makefile-based job.

### Incremental connectivity check in `connected.c`
*Long thread: 35 replies so far from 3 participants over 40 days.*
Kristofer Karlsson posted v5 of his six-patch series adding an incremental connectivity check behind `transfer.connectivityCheck=incremental` or `auto`. The series no longer touches `repo->fetch_if_missing`, using `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT` so the verifier never lazily fetches. Junio C Hamano reviewed v5, said the incremental changes look good, and indicated he will replace the previous version.

### advice: use global config for default branch name
*Long thread: 27 replies so far from 5 participants over 46 days.*
Vsevolod Myalitsin's series makes the `defaultBranchName` advice recommend a global config scope rather than local. David Timber pointed out a quoted message carrying a bogus 2027 timestamp, warning that future timestamping is a technique spammers use and a "big no no" with modern spam filters.

### doc: move BreakingChanges to a manpage
*Long thread: 23 replies so far from 5 participants over 25 days.*
Kristoffer Haugsbakk's series moves `BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage. Discussion covered whether to keep historical sections; Haugsbakk proposed a trailing future-tense section for 3.0, and Junio C Hamano decided to proceed with the manpage.

### reflog: add --operations and --historical-branch
*Long thread: 22 replies so far from 5 participants over 11 days.*
Harald Nordgren's series adds `--operations` and `--historical-branch` to `git reflog` and `git log -g`. Phillip Wood and D. Ben Knoble argued `--operations` cannot truly show one entry per command, suggesting a `%gb` format specifier instead. Kristoffer Haugsbakk questioned whether his original motivation was an illusion, and Nordgren acknowledged the series may not proceed.

### New beginner `gittutorial` page
*Long thread: 22 replies so far from 5 participants over 18 days.*
Julia Evans posted v3 of her two-patch series replacing `gittutorial` with a 782-line beginner tutorial covering `git init` through `git push` using only commands available in Git 2.23.0. Ben Knoble called the range-diff fine with small nits about wording.

### doc: rewrite detached HEAD state explanation
*Long thread: 20 replies so far from 3 participants over 14 days.*
Julia Evans's standalone `gitdetached-head` guide rewrite continued its wording debate, with Junio C Hamano questioning the use of quotes from experienced users and Evans defending her curation of user comments.

### Contributors' Summit 2026 follow-ups and v3.0 timeline
*Long thread: 20 replies so far from 9 participants over 28 days.*
Christian Couder announced he will attend the Gerrit User Summit and an NVIDIA hackathon on partial clones replacing Git LFS. Luca Milanesio said he will present his Cassandra object-storage backend prototype for JGit at the summit.

### worktree: copy-on-write creation and ephemeral branches
*Long thread: 11 replies so far from 5 participants over 139 days.*
Jason Newton posted v3 adding `git worktree add --reflink` for copy-on-write filesystems. Eric Sunshine objected that the ephemeral-branch reaping adds complexity for a use case covered by `--detach`, and Phillip Wood criticized the exchange's tone. brian m. carlson confirmed macOS `cp -c` support but declined to review further AI-generated content.

### github-ci: update macOS jobs from macos-14
*Long thread: 10 replies so far from 4 participants over 5 days.*
Junio C Hamano posted v4 updating `osx-*` jobs from `macos-14` to `macos-15`/`macos-26`, then retracted it as bogus, having decided to use v2 instead.

## The period in brief
The week of 2026/10/19 through 2026/10/25 saw heavy traffic across seven active days, with multiple long-running series reaching new iterations and several design debates remaining unresolved. Readers should not miss Patrick Steinhardt's object storage extension reaching v3, the Git for Windows 2.56.0(3) security release, and the discovery of a v2.56.0 regression in filtered `--unshallow` fetches.

## Key developments

### Object storage extension (`extensions.objectStorage`)
Patrick Steinhardt advanced his seven-patch series introducing the `extensions.objectStorage` extension through v3, splitting the storage URI into an explicit `enum object_storage_format` plus a separate payload. The series adds `--object-storage-format=` to `git-init` and `git-clone`, propagates the format through `git-submodule` without passing payloads down, and supports only the "files" backend whose payload can relocate `.git/objects`. Karthik Nayak reviewed early patches and approved v2.

### Git for Windows 2.56.0(3) security release
Johannes Schindelin announced Git for Windows 2.56.0(3), addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803 by upgrading to Git LFS v3.8.1, and bundling OpenSSH v10.6.P1 and cURL v8.23.0.

### Filtered `--unshallow` regression in v2.56.0
Daniel Zoghalchali reported that since v2.56.0, a fetch combining a tree filter with `--unshallow` aborts in pack-objects with "BUG: should_include_obj should only be called on existing objects" when the fetched ref has moved past the local tip. He traced the regression to d43b9acb05 exporting `GIT_SHALLOW_FILE` so the pack-objects grandchild inherits the empty shallow file. Taylor Blau conceded a related bug in the default fetch negotiator, and Junio C Hamano decided the fix belongs in the same series.

### Skipping negotiator parent parsing
Taylor Blau's patch parsing each unseen parent before queueing it in `negotiator/skipping.c` progressed through v2. Junio C Hamano objected to marking commits SEEN before parsing succeeds, which Taylor initially declined but later conceded is a related bug in the default negotiator, offering to fix both in the same series with a patch he already has locally.

### `status.compareBranches` default proposal
Harald Nordgren posted v3 of a patch making `git status` compare against both `@{upstream}` and `@{push}` by default. Junio C Hamano argued against flipping the default, noting the original commit made the comparison opt-in pending evidence that people like it, and suggested that whatever encourages `git push -u` could also suggest setting the configuration.

### Fetch refs as haves when backfilling tags
Karthik Nayak posted v4 advertising already-fetched refs as "haves" when backfilling tags, fixing shallow and `--atomic` fetches. Patrick Steinhardt endorsed Justin Tobler's alternative of supplementing the backfill "haves" with uncommitted refs in the pending transaction, calling it a much better fix if feasible.

## In brief

**log --follow speedup** -- Miklos Vajna posted v3 speeding up `git log --follow` across renames in non-linear history by removing `propagate_follow_pathspec_to_parent()` and recording the path per parent in the existing loop.

**read-cache CRLF size check** -- Torsten Bögershausen argued on v5 that a changed file size does not prove filtered content changed, proposing to drop the size shortcut in `ie_modified()` entirely.

**odb alternates reload** -- Patrick Steinhardt posted a four-patch series restoring alternates reloading on missing-object retry, bringing 20k lookups from 309ms back to 197ms via a hashmap-first check.

**.keep lockfile ownership** -- Justin Tobler posted v4 fixing a bug where a `git receive-pack` process deletes a concurrent process's `.keep` lockfile, migrating creation to ODB transaction commit.

**gpg SSH revocation file** -- Christian Noé Ramos López posted a patch making verification fail when `gpg.ssh.revocationFile` names a missing file, with a `:(optional)` prefix to skip the check.

**stash index loads** -- Phillip Wood's v3 removing duplicate index loads in `git stash push`/`create` was marked "Will queue" by Junio C Hamano.

**switch/checkout tracking config** -- tsukasa inoue posted v2 making `git switch -c` and `checkout -b` finish the switch and exit 0 when the tracking config cannot be written.

**worktree list --branch** -- Jasper Mayone posted v2 of `git worktree list --branch=<branch>`, dropping earlier reachability code in favor of a repeatable branch selector.

**diff-delta safe arithmetic** -- Srujith Poojary posted v5 of the safe integer arithmetic patch with expanded commit message bounds analysis after Junio's critique.

**chainlint/greplint merge** -- Tyler Cipriani posted a four-patch series teaching `t/chainlint.pl` to run greplint checks, cutting `make test-lint` from about 25 seconds to 6.

**submodule `^/` URL syntax** -- Stanislav Aleksandrov's RFC proposes resolving a submodule URL starting with `^/` against the superproject's remote from the server root, keeping protocol and host so forks at different depths do not break relative `../` URLs.

**git apply rename bug** -- Hannes Brandstätter-Müller reported `git apply` fails with "error: renamed: wrong type" on a pure rename followed by an edit under `core.fileMode=false`, with a tentative fix and regression test.

**hook configuration** -- Thomas Bachem posted v2 making `hook.<event>.enabled=false` also suppress the hooks-directory hook and accepting missing values as true; Junio C Hamano said he will queue the series.

**MyFirstContribution** -- Junio C Hamano posted v4 reordering "respond to reviews" before "send v2" and strengthening the wording.

**core.indexLockTimeout** -- Jun Wu's series adds `core.indexLockTimeout` so index-writing commands wait briefly for the lock instead of failing.

**Community voting process** -- Emily Shaffer proposed a `CommunityVoting` process; Kristoffer Haugsbakk, Patrick Steinhardt, and brian m. carlson debated AI-contribution votes, secret ballots, and eligibility bars.

**partial clone merge failure** -- Andrew Parmet reported that in a partial clone a merge can fail with "not our ref" because it requests a locally created blob from the promisor remote.

**maintenance lock stale** -- Masaya Kazama reported that since 6e95b07e5f a killed detached maintenance child leaves a stale `objects/maintenance.lock` that makes every later `git maintenance run --auto` return silently.

**ls-files untracked cache** -- Tamir Duberstein posted v3 letting `git ls-files` reuse and update the untracked cache, releasing the index lock before printing paths.

**reflog expire dry-run** -- Laveesh Bansal's fix making `git reflog expire --dry-run` say "would prune" for named refs drew a suggestion to consolidate the two expiration loops.

**t1450 macOS fix** -- Torsten Bögershausen posted v3 using `mv -f` when overwriting a read-only loose object on pre-12.3 macOS.

**GitHub Actions pinning** -- Taylor Blau defended pinning GitHub Actions to full commit IDs as a security measure; Junio C Hamano agreed a reworded commit message would be a good way forward.

**NO_PARSEOPT removal** -- René Scharfe endorsed dropping `NO_PARSEOPT` from four builtins but asked whether completion still works given `git fast-import --git-completion-helper` waits for input.

## Looking ahead
The object storage extension series will likely see further review and possible integration in the coming week, while Harald Nordgren's shallow-repo fetch series awaits final integration after v11. The reflog `--operations` series appears stalled after criticism of its design, and the submodule `^/` URL RFC will need broader consensus before proceeding.