## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 73 replies so far from 6 participants over 33 days.*

Julia Evans's six-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of repeating the guidance in each command. The series absorbed extensive review from Junio C Hamano, D. Ben Knoble, and Patrick Steinhardt across five iterations, refining the "WHAT IS A MERGE CONFLICT?" section, the "OURS"/"THEIRS" definitions, and the diff3 example. By month's end, Junio C Hamano accepted the v5 iteration and marked the topic ready for `next`.

### fetch refmap and limited-fetch for shallow repositories

*Long thread: 66 replies so far from 5 participants over 42 days.*

Harald Nordgren's series keeps a shallow repository from fetching every branch of a newly added remote by adding a `remote.<name>.fetchRefmap` config (renamed from `refmap` in v12) that remaps where fetched refs are stored while `remote.<name>.fetch` still decides what is fetched. The series also adds `--limited-fetch` to `git remote add`, defaults to it in shallow repos, and infers branches from local upstreams when only a refmap is configured. It reached v13 by month's end, resolving `@{push}` through the fetchRefmap and adding regression tests for explicit-destination refspecs, with D. Ben Knoble and Phillip Wood providing the requested second reviews.

### `git branch --delete-integrated` for squash- and rebase-merged branches

*Long thread: 53 replies so far from 6 participants over 33 days.*

Harald Nordgren's series adds `--delete-integrated <pattern>` to `git branch`, which deletes branches whose changes landed upstream via squash or rebase merges even when their tips are not reachable, naming the landing commit as "landed as ...". After earlier attempts to loosen `--delete-merged` itself drew objections over false positives, v4 restored `--delete-merged` to strict reachability and moved the looser check to the new option. Jasper Mayone tested v4 against master, confirming it handles squash merges, renames, and partial landings correctly, and v5 fixed usage-line and error-message issues by month's end.

### stash: in-core index merge for `stash apply --index`

*Long thread: 53 replies so far from 5 participants over 13 days.*

D. Ben Knoble's series fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true` by replacing the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. Phillip Wood confirmed the v5 range-diff and Junio C Hamano marked the series for `next` early in the month.

### rerere: wait for `MERGE_RR.lock` before giving up

*Long thread: 36 replies so far from 5 participants over 43 days.*

Thomas Bachem's series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for `MERGE_RR.lock` before failing, so a rebase hitting a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` option to `git rerere gc` for auto-maintenance. The series reached v8 by mid-month, with Patrick Steinhardt calling it "in a good enough state" and Junio C Hamano asking whether to mark it ready for `next`.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control

*Long thread: 37 replies so far from 3 participants over 96 days.*

Christian Couder's series introduces the protected `uploadpack.lazyFetchTrusted` config variable so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones. After v5 was rebased on Git 2.56 with documentation and test updates, Junio C Hamano marked the topic ready for `next` on October 13.

### CI housekeeping and modernizations

*Long thread: 41 replies so far from 6 participants over 12 days.*

Patrick Steinhardt's eight-patch series fixes CI issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed" and reworking the t5004 archive test to work under both SHA-1 and SHA-256. Junio C Hamano marked v4 for `next` on October 15, and later suggested resurrecting a no-Rust build job as a follow-up.

### `credentialStore.lockTimeout` config option

*Long thread: 38 replies so far from 5 participants over 45 days.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, retaining `lockTimeoutMS` as a deprecated alias. After Junio C Hamano's review suggested looping tests over both variable names, Ajayi posted v2 on October 16 with the requested changes.

### New beginner `gittutorial` page

*Long thread: 38 replies so far from 6 participants over 25 days.*

Julia Evans's series replaces the old `gittutorial` with a new 782-line beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0. The series reached v5 by month's end, incorporating Windows-friendly instructions with Git Credential Manager suggestions and addressing review feedback on heading styles and vim references.

### Incremental connectivity check in `connected.c`

*Long thread: 35 replies so far from 3 participants over 40 days.*

Kristofer Karlsson's series adds an incremental mode to the connectivity check behind `transfer.connectivityCheck=incremental` or `auto`, keeping the check from slowing as reachable boundary objects grow. v5 removed the series' dependence on `repo->fetch_if_missing` by using `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`, and Junio C Hamano reviewed it positively on October 28, indicating he would replace the previous version.

## The period in brief

October 2026 was a heavy, eventful month with traffic on all 31 days. The period opened the Git 2.98 cycle when Junio C Hamano flushed `next` down to `master` on October 10, and featured sustained work on documentation, shallow repository handling, and object database infrastructure. Readers should not miss Julia Evans's `gitmergeconflicts(7)` series reaching readiness for `next`, Harald Nordgren's shallow-repo limited-fetch series advancing to v13, and Patrick Steinhardt's object storage extension reaching v4.

## Key developments

### Object storage extension in the object database

Patrick Steinhardt advanced his seven-patch series introducing the `extensions.objectStorage` repository extension, modeled on the ref storage extension with `<schema>[://<payload>]` syntax. Only the "files" backend exists, whose payload can relocate `.git/objects`. The series adds `GIT_OBJECT_STORAGE_FORMAT`, `git rev-parse --show-object-storage-format`, and `--object-storage-format=` for `git init`, `git clone`, and `git submodule`. It reached v4 by month's end, with Karthik Nayak reviewing it as overall happy with the version.

### Shallow repository fetch and refmap handling

Harald Nordgren's long-running series to avoid fetching every branch of a new remote in a shallow repository reached v13. The series adds `remote.<name>.fetchRefmap`, a `--limited-fetch` option to `git remote add`, and defaults to limited fetch in shallow repos. v13 added `@{push}` resolution through the fetchRefmap and regression tests for explicit-destination refspecs. Phillip Wood and Junio C Hamano debated how `remote.<name>.fetch` and the refmap interact when both are configured, with the series implementing two-location storage for explicitly mapped refs.

### Documentation overhaul by Julia Evans

Julia Evans drove three major documentation series. The `gitmergeconflicts(7)` page was marked ready for `next` after five iterations. Her replacement beginner `gittutorial` reached v5 with Windows-friendly instructions. She also advanced a rewritten detached HEAD guide and a series removing the stale `gittutorial-2`, with Junio C Hamano proposing to queue the reroll on a synthetic base.

### Packfile, repack, and MIDX fixes

Several pack-related fixes landed or advanced. Patrick Steinhardt's two-patch fix for packfile corruption from stale delta base cache entries was marked for `next` on October 7. Siddharth Shrimali's fix for `git repack --drop-filtered --dry-run` writing packs despite the documented no-write promise was marked for `next` on October 11. Taylor Blau's eight-patch series fixing cruft-less MIDX corner cases reached a state where he declared it ready as-is, though Junio C Hamano questioned whether it was past diminishing returns. Pia Park's fix for `git multi-pack-index write` publishing zero-object MIDXs was queued by Junio C Hamano.

### Git 2.98 cycle and Contributors' Summit 2026

Junio C Hamano started the Git 2.98 cycle on October 10 by flushing topics from `next` to `master`. Taylor Blau shared notes from the 2026 Git Contributors' Summit covering seven topics including Git 3.0 timing (possible 2.99 and 3.0 in March 2027), pluggable ODB, and protocol v2 for pushes. Christian Couder announced plans to attend the Gerrit User Summit and an NVIDIA hackathon on partial clones.

### Security releases and signing policy

Johannes Schindelin announced Git for Windows 2.56.0(3) on October 22 addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803, followed by 2.56.0(4) on October 26 for CVE-2026-111497. Christian Noé Ramos López posted a patch making SSH signature verification fail when `gpg.ssh.revocationFile` names a missing file, fixing a fail-open issue. Scott Chacon's RFC proposing to allow responsible AI-assisted contributions under the DCO drew objections from brian m. carlsson and critique from Junio C Hamano, while Patrick Steinhardt argued the review bottleneck was tightening as AI-generated content increased.

### Fetch connectivity and tag backfill

Karthik Nayak's series fixing shallow-clone fetches resending objects during tag backfill reached v6, appending fetched tips to the transport's `negotiation_include_tips` rather than committing the first transaction. Patrick Steinhardt reviewed v6 as happy with the changes. Kristofer Karlsson's incremental connectivity check reached v5, and Justin Tobler's fix for ".keep" lockfile ownership in ODB transactions reached v4.

### Branch management and status improvements

Harald Nordgren's `--delete-integrated` option reached v5, fixing usage lines and error messages. His proposal to make `git status` compare against both `@{upstream}` and `@{push}` by default drew debate, with Junio C Hamano arguing against flipping the default. Phillip Wood's series teaching `git checkout -m` to recreate original conflict labels via `.git/MERGE_LABELS` reached v4.

## In brief

**Stash index merge** -- D. Ben Knoble's in-core index merge for `stash apply --index` was marked for `next` early in the month.

**Rerere lock handling** -- Thomas Bachem's series making rerere wait for `MERGE_RR.lock` reached v8, with Patrick Steinhardt calling it good enough.

**Refs transaction hooks** -- Maciej Ciemborowicz's series routing copy and rename through reference transactions reached v4, fixing test failures from earlier versions, though Patrick Steinhardt warned about posting new versions without engaging reviewers.

**`format-patch --range-diff-notes`** -- Kristoffer Haugsbakk's series teaching `git format-patch` the `--[no-]range-diff-notes` options reached v5 with condensed documentation.

**`git var` extensions** -- Andrew Pleeter's v9 series adding `-z`, multiple variables, and broken-out idents to `git var` drew minor test-style nits from Phillip Wood.

**CI annotations** -- Harald Nordgren's series linking GitHub Actions failure and leak annotations to test scripts was marked ready for `next` on October 13 after reverting direct file/line linking in favor of plain text.

**Blame default ignore-revs** -- Ravi Mistry's v6 series making `git blame` use `HEAD:.git-blame-ignore-revs` by default was approved by Junio C Hamano as "Very cleanly done".

**`checkout -m` conflict labels** -- Phillip Wood's v4 series writing conflict labels to `.git/MERGE_LABELS` was reviewed positively by Junio C Hamano.

**Worktree `--reflink`** -- Jason Newton's v3 series adding `git worktree add --reflink` for copy-on-write filesystems drew objections from Eric Sunshine and brian m. carlsson about the ephemeral-branch reaping patch.

**Loose-object-idx migration** -- Justin Tobler's v3 series fixing ODB transaction commit failures with compatibility hashes drew a happy review from Patrick Steinhardt.

**Diff-delta arithmetic** -- Srujith A Poojary's v9 series replacing manual size arithmetic with overflow-checked helpers in `create_delta_index()` was framed as defensive against future padding changes.

**`git describe` fix** -- Linus Torvalds submitted a patch fixing commit distance miscounting with skewed dates by changing the priority queue ordering to `compare_commits_by_gen_then_commit_date`.

**Shallow boundary refusal** -- Nakul V S posted v2 refusing to revert, cherry-pick, or rebase shallow boundary commits by default, with `--allow-shallow-boundary` for revert and cherry-pick.

**Size_t for large objects** -- Elia Pinto posted multiple series switching object and pack size handling from `unsigned long` to `size_t` to fix truncation of objects 4GiB and larger on LLP64 platforms.

**HTTP retry handling** -- Ravi Mistry posted a four-patch series for HTTP retry handling including exponential backoff and `http.retryStatusCodes`.

**`core.indexLockTimeout`** -- Jun Wu's series adding `core.indexLockTimeout` so index-writing commands wait briefly for the lock drew questions from D. Ben Knoble and Junio C Hamano about trade-offs.

**`git log --follow` performance** -- Miklos Vajna's v4 series speeding up `log --follow` on non-linear history addressed a regression from commit 304812ed33 that caused an incident for GitLab.com.

**AI policy discussion** -- Scott Chacon's RFC on AI-assisted contributions and Patrick Steinhardt's response about the review bottleneck generated significant debate, with brian m. carlsson objecting on DCO grounds.

**Community voting process** -- Emily Shaffer's proposed `CommunityVoting` document drew discussion about secret ballots, voter eligibility, and platform choice, with Kristoffer Haugsbakk and Justin Tobler raising concerns.

**`extensions.objectStorage`** -- Patrick Steinhardt's v4 series was reviewed by Karthik Nayak as overall happy, with a commit-message typo fixed locally.

**Alternates reloading** -- Patrick Steinhardt's v3 series restoring alternates reloading on object lookup retry was agreed to be ready to go by month's end.

**`git grep --recurse-submodules`** -- Patrick Steinhardt posted a three-patch series fixing "unable to read tree" failures on nested submodule trees since commit 93cd344e34.

**Notes concurrency** -- Farid Zakaria's patch making concurrent `git notes add` fail with "cannot lock ref" instead of silently dropping notes was judged a pure improvement by Kristoffer Haugsbakk.

**`git apply --check` crash** -- Tanga Joris reported an assertion crash on patches with blank lines after the `diff --git` line, confirmed reproducible on master.

**`fast-import` fixes** -- Marc Becker posted v4 allowing `from` to reference commits created in the same session, and v2 of a wincred refactor fixing CRLF handling.

**Test modernization** -- Multiple contributors posted patches replacing legacy test patterns with `test_path_is_file` and `test_path_is_missing`, and Tyler Cipriani's series merging greplint into chainlint cut `make test-lint` from 25 seconds to 6.

**Translation and docs** -- Jean-Noël Avila argued for verbose AsciiDoc cross-references in translated manuals, and Kristoffer Haugsbakk's series moving `BreakingChanges` to a `gitbreaking-changes(7)` manpage proceeded with a trailing section for Git 3.0.