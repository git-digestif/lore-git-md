## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 69 replies so far from 6 participants over 30 days.*

Julia Evans posted v4 of her six-patch series adding a `gitmergeconflicts` man page that explains merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation instead of repeating the explanation in each command. The v4 changelog folds in Junio C Hamano's diff3 example and cleanups such as sentence periods and a more accurate cherry-pick commit message. Ben Knoble said he is happy with v4 and suspects the thread is reaching diminishing returns. Junio C Hamano flagged a tab-indentation inconsistency in the diff3 example and the phrase "each removed one fruit to the list" needing "from", retracted an earlier `SYNOPSIS` lint concern, and otherwise found nothing questionable in the six patches.

### `git branch --delete-merged` and squash-merged branches

*Long thread: 47 replies so far from 6 participants over 25 days.*

Harald Nordgren's series would let `git branch --delete-merged` treat a branch as merged when an upstream commit since the fork point has the branch's version of every file it changed, naming that commit in the output as "landed as ...". After Junio C Hamano objected to loosening the default and the thread explored `--delete-integrated` and level names, Harald Nordgren conceded that the `--delete-merged=squashed` proposal is stopped: the option's value slot is already required for the upstream `<pattern>` and cannot also carry a level. Earlier iterations had drawn testing from Jasper Mayone, who found a false positive where a never-merged branch is deleted because another author made the same change upstream.

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 45 replies so far from 4 participants over 37 days.*

Harald Nordgren's series changes `git remote add` and fetch refspec handling so that adding a remote to an already shallow repository does not commit to fetching every branch the remote has. The series adds a `remote.<name>.refmap` config, makes a refspec-less fetch infer what to fetch from local branches whose `@{upstream}` is on that remote, and turns on `--limited-fetch` by default for shallow repos. v11 makes `remote_find_tracking()` fall back to `remote.<name>.refmap` when `remote.<name>.fetch` does not match, with tests that negative refspecs still exclude refs. D. Ben Knoble supplied a requested second review, questioning whether the fallback could miss mapped refs for `--delete-merged`, which Nordgren answered by adding tests locking that down.

### CI housekeeping and modernization series

*Long thread: 41 replies so far from 6 participants over 12 days.*

Patrick Steinhardt's eight-patch series "ci: some housekeeping and modernizations" fixes CI issues and moves jobs off EOL Docker images, including renaming "linux-exotic" to "linux-feature-testbed". Junio C Hamano marked v4 for `next`. Steinhardt queued the resurrection of a no-Rust CI job -- adding NO_RUST to a Makefile-based job such as linux-feature-testbed or linux32 -- into a follow-up series he is preparing, not the current v4.

### connected: incremental connectivity check

*Long thread: 35 replies so far from 3 participants over 40 days.*

Kristofer Karlsson posted v5 of the six-patch series adding an incremental mode to the connectivity check, gated behind `transfer.connectivityCheck=incremental` or `auto`. The series no longer touches `repo->fetch_if_missing`; it reads blobs and trees with `odb_read_object_info_extended()` and `OBJECT_INFO_SKIP_FETCH_OBJECT` so the verifier never lazily fetches, and reports a corrupt tree as an error instead of dying. Junio C Hamano reviewed v5, said the incremental changes look good, and indicated he will replace the previous version. Earlier, Patrick Steinhardt had agreed that a 1000-commit cutoff is a good first iteration for the "auto" mode.

### advice: use global config for default branch name

*Long thread: 27 replies so far from 5 participants over 46 days.*

Vsevolod Myalitsin's series makes the `defaultBranchName` advice shown during `git init` recommend a global config scope, since a local-scope setting only affects the repository where the advice was already shown while the advice reappears on the next `git init`; it passes the whole `advice_setting` to `vadvise()` and adds a scope hint. Earlier discussion had converged on adding `--global` to the squelch hint. David Timber pointed out that a quoted message in the thread carries a bogus future timestamp (dated 2027), warning that future timestamping is a technique spammers use and a "big no no" with modern spam filters.

### doc: move BreakingChanges to a manpage

*Long thread: 23 replies so far from 5 participants over 25 days.*

Kristoffer Haugsbakk's series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage so the content is reachable via `git help`. Junio C Hamano asked what should happen when Git 3.0 arrives and questioned whether keeping such a historical document is worth it. Haugsbakk proposed a trailing section for "what happened in 3.0", introduced with text noting the changes are written in the future tense for readers coming from version 2.98. Patrick Steinhardt argued the document should stay at least a couple of years because upgrades often jump versions; Junio C Hamano decided to proceed with the manpage.

### doc: new beginner Git tutorial

*Long thread: 22 replies so far from 5 participants over 18 days.*

Julia Evans posted v3 of her two-patch series replacing the old `gittutorial` with a new 782-line beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0 and explicitly not covering branches or collaboration. Ben Knoble called the v3 range-diff fine with a few small nits: drop "extremely" from the `vi` description, say "bottom-left" for the tilde characters, and ideally recommend `:cquit!` alongside `:q!`. Earlier, Junio C Hamano had called the 739-line v2 well-paced and free of common traps, with nitpicks on heading underlines and SSH key commands.

## The period in brief

The week of 2026/10/19 through 2026/10/25 saw heavy traffic across seven active days, with documentation and fetch behavior dominating the discussion. Readers should not miss Julia Evans's advancing documentation series, Harald Nordgren's v11 shallow-repo fetch series, and the v2.56.0 regression in filtered `--unshallow` fetches that Taylor Blau and Junio C Hamano traced to the skipping negotiator.

## Key developments

### odb: introduce object storage extension

Patrick Steinhardt posted v3 of the seven-patch series introducing the `extensions.objectStorage` extension, configured via `extensions.objectStorage` with the same syntax as the ref storage extension. V3 splits the storage URI into an explicit `enum object_storage_format` plus a separate payload, adds `--object-storage-format=` to `git-init` and `git-clone` (local clones fall back to a non-local clone with a warning when formats differ), and propagates the format through `git-submodule` without passing payloads down. Karthik Nayak reviewed earlier versions and is okay with the series as is.

### negotiator/skipping.c: parse parents before queueing

Taylor Blau's patch makes the skipping fetch negotiator parse each unseen parent before pushing it onto the date-sorted queue, so old ref tips no longer sort ahead of unparsed parents with zero dates. Today Taylor conceded that Junio C Hamano is right that the default negotiator has a related bug (marking commits SEEN before parsing succeeds), accepted a hybrid approach, and offered to fix the default negotiator alongside this series. Junio C Hamano decided that, given a clear grasp of the existing bug and a straightforward fix, the default negotiator fix should be done as part of this series.

### Git for Windows 2.56.0(3) security release

Johannes Schindelin announced Git for Windows 2.56.0(3), addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803 by upgrading to Git LFS v3.8.1, and bundling OpenSSH v10.6.P1 and cURL v8.23.0.

### filtered --unshallow of a moved ref aborts in pack-objects

Daniel Zoghalchali reported that since v2.56.0, a fetch combining a tree filter with `--unshallow` (or `--deepen`) aborts in a non-partial shallow clone with "BUG: should_include_obj should only be called on existing objects" when the fetched ref has moved past the local tip. He traced the regression to d43b9acb05 ("setup: unify setup of shallow file"), which exports `GIT_SHALLOW_FILE` so the pack-objects grandchild inherits the empty shallow file. Taylor Blau conceded a related bug in the default fetch negotiator, and Junio C Hamano decided the fix belongs in the same series.

### `git status` comparing against both upstream and push branch by default

Harald Nordgren posted v3 of a patch making `git status` compare the current branch against both `@{upstream}` and `@{push}` by default, building on `status.compareBranches`. The strengthened commit message argues the push-branch comparison reminds a returning user how the branch differs from what was last pushed, reveals local rewrites needing a forced push, and validates what is about to be pushed. Junio C Hamano argued against flipping the default, saying the original commit made the comparison opt-in pending evidence that people like it, and that the change would surprise existing users.

### t: run greplint checks inside chainlint

Tyler Cipriani posted a four-patch series teaching `t/chainlint.pl` to run the greplint checks while processing its own tokens, so `make test-lint` parses each of the roughly 1100 test scripts once, in parallel, instead of twice with greplint running serially. The cover letter reports five runs dropping from 25.158s to 5.972s.

## In brief

- **log: speed up --follow for non-linear history**: Miklos Vajna submitted v3 speeding up `git log --follow` across renames in non-linear history by removing `propagate_follow_pathspec_to_parent()` and recording the path to follow per parent in the existing `log_tree_diff()` loop.

- **read-cache: do not trust a size change when conversion is active**: Torsten Bögershausen argued on the v5 patch that a changed file size does not prove the filtered content changed, since `ce_modified_check_fs()` verifies content; he proposed dropping the size shortcut in `ie_modified()` entirely.

- **object-file: fix ".keep" lockfile ownership in ODB transactions**: Justin Tobler posted v4 of a series fixing a bug where a `git-receive-pack` process deletes a concurrent process's `.keep` lockfile for an identical packfile; the reroll corrects the `write_pack_lockfile()` error-variable handling per Patrick Steinhardt's suggestion.

- **odb/source-files: reload alternates on retry**: Patrick Steinhardt posted a four-patch series restoring alternates reloading when an object is not found, a regression from 5946a564cd, with the last two patches bringing 20k missing-object lookups from 309ms back to 197ms.

- **fetch: commit references before backfilling tags**: Karthik Nayak posted v4 advertising already-fetched refs as "haves" when backfilling tags, fixing shallow and `--atomic` fetches; Patrick Steinhardt endorsed Justin Tobler's alternative of supplementing the backfill "haves" with the uncommitted refs in the pending transaction.

- **gpg-interface: fail when the SSH revocation file is missing**: Christian Noé Ramos López posted a patch making signature verification fail with an error when `gpg.ssh.revocationFile` names a missing file, with a `:(optional)` prefix to skip the check.

- **worktree: add --reflink for copy-on-write worktree creation**: brian m. carlson confirmed v3 handles macOS `cp -c` (clonefile(2)) so the REFLINK test runs there, and declined to review further AI-generated content from Jason Newton, citing the project's AI policy.

- **submodule: make '^/' work like '../' but with an absolute path**: Stanislav Aleksandrov's RFC proposes resolving a submodule URL starting with `^/` against the superproject's remote from the server root; Jacob Keller concluded the concept makes sense.

- **reflog: add --operations and --historical-branch**: Harald Nordgren's three-patch series adds `--operations` and `--historical-branch` to `git reflog` and `git log -g`; Kristoffer Haugsbakk expressed doubt about the motivation after testing showed per-branch reflog lists only rebase-finish entries, and criticized the `--historical-branch` implementation.

- **Documentation: propose community voting process**: On Emily Shaffer's proposed `CommunityVoting` process, Kristoffer Haugsbakk argued an AI-contributions vote would drift toward tech-employer positions; Patrick Steinhardt and brian m. carlson endorsed secret ballots; Justin Tobler argued the eligibility bar is too low.

- **Contributors' Summit 2026 follow-ups**: Christian Couder announced he will attend the Gerrit User Summit on November 9-10 and an NVIDIA hackathon on November 11-13 on partial clones replacing Git LFS; Luca Milanesio said he will present his Cassandra object-storage backend prototype for JGit at the summit.

- **github-ci: update macOS jobs from macos-14**: Junio C Hamano posted v4 of the patch updating the four `osx-*` jobs from `macos-14` to `macos-15` or `macos-26`, then retracted it as bogus, having decided to use v2 instead.

- **advice: introduce advice scoping mechanism**: Vsevolod Myalitsin's series adds a scope hint so the `defaultBranchName` advice suggests the global scope; Junio C Hamano asked whether to mark `jc/advice-config-set-global` for `next`.

- **doc: rewrite detached HEAD state explanation**: Julia Evans's standalone `gitdetached-head` guide rewrite continued its wording debate, with Junio C Hamano questioning what she aimed to illustrate with quotes from experienced users.

- **Various small fixes**: Junio C Hamano found Harald Nordgren's shallow-fetch fix breaks protocol v2 with sideband-all; Andy Koppe posted v3 keeping root refs out of the loose ref cache; Phillip Wood posted v3 of the stash series removing duplicate index loads; Mihai Cosma posted a series making `git status` take `index.lock` only to write; Jun Wu's series adds `core.indexLockTimeout`; Tamir Duberstein posted v3 letting `git ls-files` reuse the untracked cache; Hannes Brandstätter-Müller reported `git apply` fails on a pure rename followed by an edit under `core.fileMode=false`; Thomas Bachem posted v2 of hook configuration changes; Srujith Poojary posted v5 of the `diff-delta.c` safe-integer-arithmetic patch; Tanga Joris reported `git apply --check` crashes on a patch with a blank line after the `diff --git` line; René Scharfe endorsed dropping `NO_PARSEOPT` from four builtins.