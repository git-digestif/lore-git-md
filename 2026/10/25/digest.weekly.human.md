## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 69 replies so far from 6 participants over 30 days.*
Julia Evans's six-patch series adds a `gitmergeconflicts` man page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull`. By v4 the series had folded in Junio C Hamano's diff3 example and cleanups; Ben Knoble declared himself happy with v4 and suspected diminishing returns, while Junio flagged only a tab-indentation inconsistency and a missing "from" in one phrase before finding nothing else questionable.

### `git branch --delete-merged` and squash-merged branches
*Long thread: 47 replies so far from 6 participants over 25 days.*
Harald Nordgren's series would let `git branch --delete-merged` treat branches merged via GitHub squash or rebase merges as merged, naming the upstream commit as "landed as ...". Junio C Hamano objected to loosening the default with a test showing a private branch wrongly deleted when its changes appeared upstream under a different author. The thread explored `--delete-integrated`, a subset-superset ladder (`=merged`, `=rebased`, `=squashed`), and a separate `--delete-integrated <pattern>` command, but Nordgren conceded on 10/23 that `--delete-merged=squashed` is stopped because the option's value slot is already taken by the upstream `<pattern>`.

### fetch: avoid fetching every branch of a new remote in a shallow repo
*Long thread: 45 replies so far from 4 participants over 37 days.*
Harald Nordgren's five-patch series keeps shallow repositories from fetching every branch of a newly added remote by adding a `remote.<name>.refmap` config, making refspec-less fetches infer targets from local upstreams, and defaulting `--limited-fetch` for shallow repos. Junio C Hamano called v10 fairly well done but asked for a second reviewer; D. Ben Knoble supplied that read on 10/24, questioning whether the `remote_find_tracking()` fallback could miss mapped refs. Nordgren posted v11 on 10/25 adding the fallback when `remote.<name>.fetch` does not match, with tests locking down negative refspecs and `--delete-merged` behavior.

### CI housekeeping and modernization series
*Long thread: 41 replies so far from 6 participants over 12 days.*
Patrick Steinhardt's eight-patch series fixes CI issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed". Junio C Hamano marked v4 for `next`, and Steinhardt queued a follow-up to resurrect a no-Rust CI job by adding `NO_RUST` to a Makefile-based job.

### connected: incremental connectivity check
*Long thread: 35 replies so far from 3 participants over 40 days.*
Kristofer Karlsson's series adds an incremental connectivity-check mode behind `transfer.connectivityCheck=incremental` or `"auto"` with a configurable cutoff (default 1000 commits). After Junio C Hamano questioned gating `fetch_if_missing` on "not full", Karlsson reworked v5 to use `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`, verified in a partial clone that promised blobs report as missing without fetching, and Junio judged the incremental changes good on 10/23.

### advice: use global config for default branch name
*Long thread: 27 replies so far from 5 participants over 46 days.*
Vsevolod Myalitsin's series makes the `defaultBranchName` advice recommend a global config scope, passing the whole `advice_setting` to `vadvise()` with a scope hint. Junio C Hamano asked whether to mark it for `next` and whether the `vadvise()` API could be cleaned up separately. David Timber warned on 10/24 that a quoted message carried a bogus 2027 timestamp, a spam-filter red flag.

### doc: move BreakingChanges to a manpage
*Long thread: 23 replies so far from 5 participants over 25 days.*
Kristoffer Haugsbakk's series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage. Junio C Hamano questioned whether anyone still cares about 1.x-to-2.0 issues; Haugsbakk proposed a trailing "what happened in 3.0" section written in future tense, and Junio decided to proceed with the manpage.

### doc: add new Git tutorial for beginners
*Long thread: 22 replies so far from 5 participants over 18 days.*
Julia Evans's two-patch series replaces `gittutorial` with a 782-line beginner tutorial covering `git init` through `git push` using only commands available in Git 2.23.0. Junio C Hamano praised the pacing and flagged heading-style nits; Evans preferred Setext headings because Atx-style `== DESCRIPTION` trips `make lint-docs`. Ben Knoble called the v3 range-diff fine with small nits about `vi` descriptions and `:cquit!`.

### reflog: add --operations and --historical-branch
*Long thread: 22 replies so far from 5 participants over 11 days.*
Harald Nordgren's series adds `--operations` (folding rebases) and `--historical-branch` (decorating entries with the branch HEAD was on) to `git reflog` and `git log -g`. Phillip Wood and D. Ben Knoble argued `--operations` cannot truly show one entry per command and suggested `%gb` format specifiers instead. Kristoffer Haugsbakk later doubted his original motivation, finding the per-branch reflog already hides rebase steps, and Nordgren replied the code may not be worth reviewing if the design is unsettled.

### Contributors' Summit 2026 follow-ups and v3.0 timeline
*Long thread: 20 replies so far from 9 participants over 28 days.*
Following the 2026 Contributors' Summit summary and Junio C Hamano's v3.0 timeline, Christian Couder announced attendance at the Gerrit User Summit and an NVIDIA hackathon on partial clones replacing Git LFS, while Luca Milanesio said he will present his Cassandra object-storage backend prototype for JGit.

### doc: rewrite detached HEAD state explanation
*Long thread: 20 replies so far from 3 participants over 14 days.*
Julia Evans's `gitdetached-head` guide rewrite continued its wording debate. She explained that she shared quotes from experienced users to show the language people actually use, asked for trust in her sorting of comments, and added an XKCD reference; Junio C Hamano had questioned the quotes' purpose and recorded replacement text for the opening paragraphs.

### odb: introduce object storage extension
*Long thread: 14 replies so far from 3 participants over 2 days.*
Patrick Steinhardt's seven-patch series adds `extensions.objectStorage` configured as `<schema>[://<payload>]`, currently only `files` with a payload that can relocate `.git/objects`. He answered Karthik Nayak's questions on relative paths, `gettext.h`, and the `DEFAULT_REPO_FORMAT` protective measure, and argued against a full 30-combination test matrix.

### worktree: copy-on-write creation and ephemeral branches
*Long thread: 11 replies so far from 5 participants over 139 days.*
Jason Newton's v3 adds `git worktree add --reflink` for copy-on-write filesystems (disabled on Windows for a ReFS bug) plus reaping of ephemeral branches. Eric Sunshine objected that the ephemeral-branch patch adds complexity for a case covered by `--detach`; Newton argued a detached HEAD leaves status and push with nothing to name, and Phillip Wood said the reply fell short of the list's standard.

### github-ci: update macOS jobs from macos-14
*Long thread: 10 replies so far from 4 participants over 5 days.*
Junio C Hamano posted v4 updating `osx-*` jobs from `macos-14` to `macos-15` or `macos-26`, then retracted it as bogus and settled on v2 after Patrick Steinhardt suggested mixing versions for coverage and Todd Zullinger asked about `macos-latest`.

## The period in brief
The week of 2026/10/19 through 2026/10/25 saw heavy traffic across seven active days, dominated by long-running design debates on `git branch --delete-merged`, shallow-repo fetch behavior, and incremental connectivity checks, alongside a burst of documentation work from Julia Evans. Readers should not miss the v3.0 timeline follow-ups from the Contributors' Summit, Patrick Steinhardt's `extensions.objectStorage` series reaching v3, and the Git for Windows 2.56.0(3) security release.

## Key developments

### Object storage extension matures
Patrick Steinhardt advanced his seven-patch `extensions.objectStorage` series to v3, splitting the storage URI into an explicit `enum object_storage_format` plus payload, adding `--object-storage-format=` to `git-init` and `git-clone` (with local clones falling back to non-local when formats differ), and propagating the format through `git-submodule`. Karthik Nayak reviewed earlier iterations and was satisfied with v2; the series now supports only the "files" backend whose payload can relocate `.git/objects`.

### Shallow-repo fetch series nears completion
Harald Nordgren's v11 of the five-patch series preventing shallow repos from fetching every branch of a new remote landed after D. Ben Knoble's second review. The series adds `remote.<name>.refmap`, infers refspec-less fetch targets from local upstreams, and defaults `--limited-fetch` for shallow repos; v11 adds the `remote_find_tracking()` fallback to the refmap when the fetch refspec does not match, with tests for negative refspecs and `--delete-merged`.

### Incremental connectivity check reworked
Kristofer Karlsson posted v5 of the incremental connectivity check, removing its dependence on `repo->fetch_if_missing` by using `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`. Junio C Hamano judged the incremental changes good and indicated he would replace the previous version; the series supports `transfer.connectivityCheck=incremental` or `"auto"` with a configurable cutoff defaulting to 1000 commits.

### Documentation push: tutorials and merge conflicts
Julia Evans advanced two major doc series: v4 of the `gitmergeconflicts(7)` man page (69 replies, nearing convergence) and v3 of a new 782-line beginner `gittutorial` replacing the old one. Kristoffer Haugsbakk's `gitbreaking-changes(7)` manpage also progressed after Junio decided to proceed despite questions about retaining old-version content.

### Git for Windows security release
Johannes Schindelin announced Git for Windows 2.56.0(3), addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803 by upgrading to Git LFS v3.8.1, and bundling OpenSSH v10.6.P1 and cURL v8.23.0.

### Fetch negotiator and shallow-fetch fixes
Taylor Blau conceded a related bug in the default fetch negotiator (marking commits SEEN before parsing) and agreed to fix it alongside his skipping-negotiator patch that parses parents before queueing. Daniel Zoghalchali reported a v2.56.0 regression where filtered `--unshallow` of a moved ref aborts in pack-objects, traced to `GIT_SHALLOW_FILE` export; Junio decided the fix belongs in the same series.

## In brief

**log --follow speedup** -- Miklos Vajna posted v3 speeding up `git log --follow` across renames in non-linear history by removing `propagate_follow_pathspec_to_parent()` and recording the path per parent in the existing diff loop.

**CRLF status fix** -- Torsten Bögershausen argued on v5 that a changed file size does not prove filtered content changed, proposing to drop the size shortcut in `ie_modified()` entirely.

**.keep lockfile ownership** -- Justin Tobler posted v4 of the series migrating `.keep` lockfile creation to ODB transaction commit so concurrent `git receive-pack` processes no longer delete each other's lockfiles.

**status index.lock** -- Mihai Cosma's series makes `git status` take `index.lock` only to write and fixes a `skipHash` race that could silently revert a `git add`; Jun Wu's separate `core.indexLockTimeout` series was noted as relatively independent.

**status.compareBranches default** -- Harald Nordgren posted v3 making `@{upstream} @{push}` the default for `status.compareBranches`; Junio C Hamano argued against flipping the default, saying it would surprise existing users.

**worktree --reflink** -- Jason Newton's v3 adds `git worktree add --reflink` for copy-on-write filesystems; brian m. carlsson confirmed macOS `cp -c` support but declined to review further AI-generated content.

**chainlint/greplint merge** -- Tyler Cipriani's series teaches `t/chainlint.pl` to run greplint checks, cutting `make test-lint` from about 25 seconds to 6.

**submodule ^/ URL syntax** -- Stanislav Aleksandrov's RFC proposes resolving `^/` submodule URLs against the superproject's remote from the server root; Jacob Keller concluded the concept makes sense.

**reflog options** -- Harald Nordgren's `--operations` and `--historical-branch` series faced skepticism that `--operations` cannot truly fold multi-commit commands, with suggestions to use `%gb` format specifiers instead.

**gpg SSH revocation** -- Christian Noé Ramos López posted a patch failing verification when `gpg.ssh.revocationFile` is missing, with a `:(optional)` prefix to skip; Junio objected to the doc text and the placement of the missing-file check.

**partial clone merge failure** -- Andrew Parmet reported that in a partial clone a merge can fail with "not our ref" when requesting a locally created blob from the promisor remote.

**maintenance lock** -- Masaya Kazama reported that a killed detached maintenance child leaves a stale `objects/maintenance.lock`, making later `git maintenance run --auto` return silently.

**advice scoping** -- Vsevolod Myalitsin's series adds a scope hint so the `defaultBranchName` advice suggests global config; Junio asked whether to mark it for `next`.

**hook configuration** -- Thomas Bachem's v2 makes `hook.<event>.enabled=false` suppress hooks-directory hooks and treats missing `.enabled`/`.parallel` values as true; Junio said he will queue it.

**diff-delta safe arithmetic** -- Srujith Poojary posted v5 of the safe-integer-arithmetic patch with expanded commit message bounds after Junio's critique.

**git apply crashes** -- Tanga Joris reported `git apply --check` crashes on patches with blank lines after `diff --git` or with `--- /dev/null` before `new file mode`; Hannes Brandstätter-Müller diagnosed a zero `previous->new_mode` in `check_preimage()` for pure renames under `core.fileMode=false`.

**shallow-fetch protocol v2** -- Harald Nordgren posted v2 of a fix for slow fetches from shallow repositories over protocol v2, writing shallow lines through the packet writer after Junio found the v1 broke sideband-all.

**alternates reload** -- Patrick Steinhardt's series restores reloading alternates when an object lookup is retried, fixing a regression from 5946a564cd; Christian Couder noted patch 2/4 reads from the quarantine directory.

**fetch tag backfill** -- Karthik Nayak posted v4 advertising already-fetched refs as "haves" when backfilling tags, fixing shallow and `--atomic` fetches; Justin Tobler suggested supplementing with uncommitted refs from the pending transaction.

**community voting** -- Emily Shaffer's proposed `CommunityVoting` process drew debate: Kristoffer Haugsbakk worried an AI-contributions vote would drift toward employer positions, while Patrick Steinhardt and brian m. carlsson endorsed secret ballots.

**MyFirstContribution** -- Junio C Hamano posted v4 reordering "respond to reviews" before "send v2" and strengthening the wording.

**ls-files untracked cache** -- Tamir Duberstein posted v3 letting `git ls-files` reuse and update the untracked cache, releasing the index lock before printing paths.

**followRemoteHEAD** -- Junio asked that the resolution in `do_fetch()` be refactored into a `lazy_follow_remote_head()` helper; the deferred validation patch has a textual conflict with Nordgren's fetch-refmap work.

## Looking ahead
The `--delete-merged` squash-detection design remains unresolved after the `--delete-merged=squashed` syntax was conceded stopped; a separate `--delete-integrated` command or a `%gb`-based reflog approach may resurface. Patrick Steinhardt's object storage extension and Harald Nordgren's shallow-fetch series are both queued for further review and likely to land in `next` soon. The incremental connectivity check v5 awaits integration after Junio's approval.