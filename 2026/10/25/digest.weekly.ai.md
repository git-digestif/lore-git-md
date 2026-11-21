## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 69 replies so far from 6 participants over 30 days.*
Julia Evans advanced her six-patch series adding a `gitmergeconflicts` man page that explains merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of repeating the explanation in each command. v4 folded in Junio C Hamano's diff3 example and cleanups such as sentence periods and a more accurate cherry-pick commit message. Ben Knoble said he was happy with v4 and suspected the thread was reaching diminishing returns, while Junio flagged a tab-indentation inconsistency and a missing "from" in one phrase but otherwise found nothing questionable in the six patches.

### `git branch --delete-merged` and squash-merged branches
*Long thread: 47 replies so far from 6 participants over 25 days.*
Harald Nordgren's series would let `git branch --delete-merged` treat a branch as merged when an upstream commit since the fork point contains the branch's version of every file it changed, naming that commit as "landed as ...". After Junio C Hamano objected to loosening the default with a test showing a private branch wrongly deleted when another author made the same change upstream, the thread explored `--delete-integrated` and level names like `=merged`, `=rebased`, `=squashed`. Nordgren conceded that `--delete-merged=squashed` is stopped because the option's value slot is already required for the upstream `<pattern>`.

### fetch: avoid fetching every branch of a new remote in a shallow repo
*Long thread: 45 replies so far from 4 participants over 37 days.*
Harald Nordgren posted v11 of his five-patch series changing `git remote add` and fetch refspec handling so adding a remote to an already shallow repository does not commit to fetching every branch. The series adds a `remote.<name>.refmap` config, makes a refspec-less fetch infer what to fetch from local branches whose `@{upstream}` is on that remote, and turns on `--limited-fetch` by default for shallow repos. D. Ben Knoble supplied the requested second review, questioning whether the `remote_find_tracking()` fallback could miss mapped refs for `--delete-merged`; Nordgren agreed to add tests locking that down.

### CI housekeeping and modernization series
*Long thread: 41 replies so far from 6 participants over 12 days.*
Patrick Steinhardt's eight-patch series "ci: some housekeeping and modernizations" fixes CI issues and moves jobs off EOL Docker images, including renaming "linux-exotic" to "linux-feature-testbed". Junio C Hamano had marked v4 for `next`, and Steinhardt queued the resurrection of a no-Rust CI job -- adding NO_RUST to a Makefile-based job such as linux-feature-testbed or linux32 -- into a follow-up series he is preparing rather than the current v4.

### connected: incremental connectivity check
*Long thread: 35 replies so far from 3 participants over 40 days.*
Kristofer Karlsson posted v5 of his six-patch series adding an incremental mode to the connectivity check, gated behind `transfer.connectivityCheck=incremental` or `auto`. The series no longer touches `repo->fetch_if_missing`; it reads blobs and trees with `odb_read_object_info_extended()` and `OBJECT_INFO_SKIP_FETCH_OBJECT` so the verifier never lazily fetches, and reports a corrupt tree as an error instead of dying. Junio C Hamano reviewed v5, said the incremental changes look good, and indicated he will replace the previous version.

### advice: use global config for default branch name
*Long thread: 27 replies so far from 5 participants over 46 days.*
Vsevolod Myalitsin's series makes the `defaultBranchName` advice shown during `git init` recommend a global config scope, since a local-scope setting only affects the repository where the advice was already shown while the advice reappears on the next `git init`. It passes the whole `advice_setting` to `vadvise()` and adds a scope hint. David Timber pointed out that a quoted message in the thread carried a bogus future timestamp dated 2027, warning that future timestamping is a technique spammers use and a "big no no" with modern spam filters.

### doc: move BreakingChanges to a manpage
*Long thread: 23 replies so far from 5 participants over 25 days.*
Kristoffer Haugsbakk's series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage so the content is reachable via `git help`. Junio C Hamano asked what should happen when Git 3.0 arrives and whether keeping such a historical document is worth it; Haugsbakk proposed a trailing section for "what happened in 3.0" written in the future tense for readers coming from version 2.98. Patrick Steinhardt argued the document should stay at least a couple of years because upgrades often jump versions, and Junio decided to proceed with the manpage.

### doc: add new Git tutorial for beginners
*Long thread: 22 replies so far from 5 participants over 18 days.*
Julia Evans posted v3 of her two-patch series replacing the old `gittutorial` with a new 782-line beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0 and explicitly not covering branches or collaboration. Ben Knoble called the v3 range-diff fine with a few small nits: drop "extremely" from the `vi` description, say "bottom-left" for the tilde characters, and ideally recommend `:cquit!` alongside `:q!`. Earlier, Junio C Hamano had called the 739-line v2 well-paced and free of common traps, with nitpicks on heading underlines and SSH key commands.

## The period in brief

The week of 2026/10/19 through 2026/10/25 saw heavy traffic across seven active days, with documentation and fetch behavior dominating the discussion. Readers should not miss Julia Evans's two major documentation series -- the new `gitmergeconflicts(7)` page and the replacement beginner tutorial -- Harald Nordgren's v11 shallow-repo fetch series, and the v2.56.0 regression in filtered `--unshallow` fetches that Daniel Zoghalchali traced to `GIT_SHALLOW_FILE` inheritance.

## Key developments

### Object storage extension in the object database
Patrick Steinhardt posted v3 of his seven-patch series introducing the `extensions.objectStorage` extension, configured via `extensions.objectStorage` with the same syntax as the ref storage extension. V3 splits the storage URI into an explicit `enum object_storage_format` plus a separate payload, adds `--object-storage-format=` to `git-init` and `git-clone` (local clones fall back to a non-local clone with a warning when formats differ), and propagates the format through `git-submodule` without passing payloads down. Karthik Nayak reviewed earlier iterations and was okay with the series as is.

### Git for Windows 2.56.0(3) security release
Johannes Schindelin announced Git for Windows 2.56.0(3), addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803 by upgrading to Git LFS v3.8.1, and bundling OpenSSH v10.6.P1 and cURL v8.23.0.

### Filtered `--unshallow` of a moved ref aborts in pack-objects
Daniel Zoghalchali reported that since v2.56.0, a fetch combining a tree filter with `--unshallow` aborts in a non-partial shallow clone with "BUG: should_include_obj should only be called on existing objects" when the fetched ref has moved past the local tip. He traced the regression to d43b9acb05, which exports `GIT_SHALLOW_FILE` so the pack-objects grandchild inherits the empty shallow file and walks to a promisor commit whose root tree was never fetched. Taylor Blau conceded a related bug in the default fetch negotiator, and Junio C Hamano decided the fix belongs in the same series.

### negotiator/skipping.c: parse parents before queueing
Taylor Blau's patch makes the skipping fetch negotiator parse each unseen parent before pushing it onto the date-sorted queue, so old ref tips no longer sort ahead of unparsed parents with zero dates. After Junio C Hamano objected to marking commits SEEN before parsing, Taylor conceded the default negotiator has a related bug, accepted a hybrid approach, and offered to fix the default negotiator alongside this series. Junio decided the default negotiator fix should be done as part of this series.

### `git status` comparing against both upstream and push branch by default
Harald Nordgren posted v3 of a patch making `git status` compare the current branch against both `@{upstream}` and `@{push}` by default, building on `status.compareBranches`. Junio C Hamano argued against flipping the default, saying the original commit made the comparison opt-in pending evidence that people like it, and suggested that whatever encourages `git push -u` could also suggest setting the configuration. D. Ben Knoble supported making `@{upstream} @{push}` the default but agreed a stronger case was needed.

### Test linting: greplint inside chainlint
Tyler Cipriani posted a four-patch series teaching `t/chainlint.pl` to run the greplint checks while processing its own tokens, so `make test-lint` parses each of the roughly 1100 test scripts once, in parallel, instead of twice with greplint running serially. The cover letter reports five runs dropping from 25.158s to 5.972s.

## In brief

- **Shallow-fetch protocol v2 fix**: Harald Nordgren posted v2 of a fix for slow fetches from shallow repositories over protocol v2, writing shallow lines through the packet writer after Junio C Hamano found the earlier fix broke protocol v2 with sideband-all.
- **`git log --follow` speedup**: Miklos Vajna submitted v3 fixing `git log --follow` slowdowns on linux.git by removing `propagate_follow_pathspec_to_parent()` and recording the path to follow per parent in the existing `log_tree_diff()` loop.
- **ODB alternates reload**: Patrick Steinhardt posted a four-patch series restoring alternates reloading when an object is not found, a regression from 5946a564cd, bringing 20k missing-object lookups from 309ms back to 197ms.
- **`.keep` lockfile ownership**: Justin Tobler posted v4 of a series fixing a bug where a `git-receive-pack` process deletes a concurrent process's `.keep` lockfile for an identical packfile; Junio C Hamano had marked the topic "Expecting a reroll".
- **`git worktree add --reflink`**: Jason Newton posted v3 adding copy-on-write worktree creation via reflinking, disabled on Windows because of a ReFS zero-fill bug; brian m. carlson confirmed v3 handles macOS `cp -c` but declined to review further AI-generated content.
- **Submodule `^/` URL syntax**: Stanislav Aleksandrov's RFC proposes resolving a submodule URL starting with `^/` against the superproject's remote from the server root, so forks at a different depth do not break relative `../` URLs; Jacob Keller concluded the concept makes sense.
- **Reflog options**: Harald Nordgren's series adding `--operations` and `--historical-branch` to `git reflog` drew skepticism from Phillip Wood and D. Ben Knoble, who argued `--operations` cannot truly show one entry per command and suggested `%gb` format specifier instead.
- **CRLF status bug**: Torsten Bögershausen argued on the v5 patch that a changed file size does not prove filtered content changed, proposing to drop the size shortcut in `ie_modified()` entirely.
- **Community voting process**: On Emily Shaffer's proposed `CommunityVoting` process, Kristoffer Haugsbakk argued an AI-contributions vote would drift toward tech-employer positions, while Patrick Steinhardt and brian m. carlson endorsed secret ballots.
- **Hook configuration**: Thomas Bachem posted v2 of two hook configuration changes -- `hook.<event>.enabled=false` now also suppresses the hooks-directory hook, and `.enabled`/`.parallel` accept a missing value as true -- and Junio C Hamano said he will queue the series.