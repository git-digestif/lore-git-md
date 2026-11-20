## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 73 replies so far from 6 participants over 33 days.*

Julia Evans's six-patch series adds a `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull`. The series progressed from v2 through v5 during October, absorbing Junio C Hamano's corrected diff3 example, requests to list every conflict-causing command explicitly, and formal-construction wording. By late October Evans posted v5 dropping the zdiff3 note and fixing indentation, and Hamano accepted the iteration and marked the topic ready for `next`.

### fetch refmap configuration for shallow repos
*Long thread: 66 replies so far from 5 participants over 42 days.*

Harald Nordgren's series keeps shallow repositories from fetching every branch of a new remote by introducing `remote.<name>.fetchRefmap` (renamed from `refmap` in v12) while `remote.<name>.fetch` still decides what is fetched. The series evolved through v13, adding a missing `get_ref_map()` path and giving `tracking_for_push_dest()` the same fallback so `@{push}` resolves correctly for `--limited-fetch` remotes. Phillip Wood and Junio C Hamano debated how explicit-destination `fetch` entries interact with the refmap, and D. Ben Knoble supplied a second review of v10.

### `git branch --delete-integrated` for squash- and rebase-merged branches
*Long thread: 53 replies so far from 6 participants over 34 days.*

Harald Nordgren's series adds `--delete-integrated <pattern>` to `git branch`, deleting branches whose changes landed upstream via squash or rebase merges and naming the landing commit. The thread explored whether to extend `--delete-merged` instead, with Junio C Hamano objecting to loosening the default after a test showed a private branch wrongly deleted when its changes appeared upstream under a different author. Nordgren conceded that `--delete-merged=squashed` is stopped because the option's value slot is taken, and v6 of the separate `--delete-integrated` series fixed usage lines and honored `.gitattributes` from the index.

### stash: in-core index merge for `stash apply --index`
*Long thread: 53 replies so far from 5 participants over 13 days.*

D. Ben Knoble's series evolved from v4 to v5 during the first week, fixing tri-state handling of `merge_result.clean` that Junio C Hamano had diagnosed. The final patch replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, resolving a non-re-entrant lock bug when autostashing with `stash.index=true`. Phillip Wood confirmed the range-diff and welcomed the removal of subprocesses, and Hamano marked the series for `next` on 10/01.

### CI housekeeping and modernizations
*Long thread: 41 replies so far from 6 participants over 12 days.*

Patrick Steinhardt's eight-patch series fixes smaller CI issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed" and reworking the reftable job. Version 4 reworked t5004 to run under both SHA-1 and SHA-256 by forcing the test repo to SHA-1. Jeff King approved v4 and Junio marked the series for `next`, with a follow-up queued to resurrect a no-Rust CI job by adding `NO_RUST` to a Makefile-based job.

### More `path.*` keys for `git repo info`
*Long thread: 39 replies so far from 5 participants over 109 days.*

K Jayatheerth's GSoC series adds path keys to `git repo info`: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup`, with absolute and relative suffix formatting. On November 1 Junio C Hamano stated that the v7 series has seen no interest or review for a full month and that he will discard the topic from his tree unless volunteers give it a final review in a few weeks; Justin Tobler said he will take a look.

### credentialStore.lockTimeout config option
*Long thread: 38 replies so far from 5 participants over 45 days.*

Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. Ajayi posted v2 looping the two timeout tests over both names, wrapping long lines, and dropping a problematic test. Junio C Hamano reviewed the initial version, suggesting the loop to keep tests in sync and questioning a 0 ms timeout test.

### New beginner Git tutorial
*Long thread: 38 replies so far from 6 participants over 25 days.*

Julia Evans's series replaces the old `gittutorial` with a new 782-line beginner tutorial covering `git init` through `git push`, using only commands available in Git 2.23.0. She posted v4 making instructions Windows-friendly by mentioning Git for Windows' editor prompt and marking SSH steps as Mac/Linux only, and v5 replaced an ambiguous "it" with "GCM". Junio C Hamano reviewed the patch and asked whether the version is in good shape to merge to `next`.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control
*Long thread: 37 replies so far from 3 participants over 96 days.*

Christian Couder's five-patch series introduces the protected `uploadpack.lazyFetchTrusted` configuration variable so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones. On October 6 Couder agreed the v5 series should be marked for `next`, and Junio C Hamano marked the topic ready for `next` on October 13.

### rerere waiting for `MERGE_RR.lock`
*Long thread: 36 replies so far from 5 participants over 43 days.*

Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` before failing, and adds a hidden `--skip-locked` option to `git rerere gc`. Version 8 incorporated earlier review; Patrick Steinhardt retracted a suggestion for a separate flags enum and called v8 good enough, agreeing to mark the topic ready for `next`.

### connected: incremental connectivity check
*Long thread: 35 replies so far from 3 participants over 40 days.*

Kristofer Karlsson's series adds an incremental connectivity-check mode behind `transfer.connectivityCheck=incremental` or `"auto"` with a configurable cutoff defaulting to 1000 commits. After Junio C Hamano questioned gating `fetch_if_missing`, Karlsson reworked v5 to use `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`, and Hamano judged the incremental changes good on 10/23.

### sequencer: leave auto maintenance to the end of a sequence
*Long thread: 33 replies so far from 5 participants over 25 days.*

Thomas Bachem's v5 series makes merge-backend rebase sequences run auto maintenance once at completion, matching the apply backend. Patrick Steinhardt noted on 9/28 that the series is already cooking in `next` and agreed it looks ready, effectively concluding the thread's active development phase.

### `http.sslVerifyStatus` for stapled OCSP responses
*Long thread: 33 replies so far from 5 participants over 59 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`. After v7 was reverted out of `next` over a t5585 failure, Junio C Hamano bisected linux-* CI jobs spinning until their 6-hour timeout to this topic and tentatively ejected it from `seen` on October 7, confirming in isolation on Git 2.56 that it causes the timeouts.

### reflog: name the branch of each reflog entry
*Long thread: 31 replies so far from 6 participants over 13 days.*

Harald Nordgren posted v3 of the reflog series, dropping v2's `--operations` and `--historical-branch` for a `%gb` placeholder in `--format` that names the branch HEAD was on when each entry was recorded. Phillip Wood preferred a format specifier and D. Ben Knoble agreed the reflog is not a true operations log.

### repack corner cases for cruft-less MIDXs
*Long thread: 30 replies so far from 5 participants over 30 days.*

Taylor Blau's eight-patch series fixes omissions in geometric repacks with `repack.midxMustContainCruft=false` that can make bitmap generation fail. Junio C Hamano questioned the topic's viability after nearly a month of mixed feedback, asking whether it was still improving; Taylor Blau stated the topic should be ready to go as-is.

### refs: report old values to reference-transaction hooks
*Long thread: 29 replies so far from 5 participants over 13 days.*

Maciej Ciemborowicz's v6 single-patch series resolves old object IDs in the common transaction layer before the "preparing" hook, fixing batched deletions reporting all-zero old OIDs. On 10/01 Ciemborowicz lowered the priority of the underlying bug after resolving his branch and tag deletion issues.

### Documentation: community voting process
*Long thread: 29 replies so far from 13 participants over 25 days.*

Discussion of Emily Shaffer's proposed voting-process document continued. Toon Claes argued eligible voters should be able to abstain, and Shaffer summarized reroll changes including secret-vote requirement and votes referencing a proposal's Message-ID. Junio C Hamano asked to drop the word "code" and tally non-patch contributions.

### Removing `gittutorial-2`
*Long thread: 28 replies so far from 4 participants over 29 days.*

Julia Evans's two-patch series deletes the stale `gittutorial-2`, leaving a stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`. She posted v3 removing it from `command-list.txt` so `git help --guides` does not advertise the obsolete tutorial, while Junio C Hamano asked that changes to `Documentation/gittutorial.adoc` be left out since the main tutorial is being rewritten.

### advice: use global config for default branch name
*Long thread: 27 replies so far from 5 participants over 46 days.*

Vsevolod Myalitsin's series makes the `defaultBranchName` advice recommend a global config scope, passing the whole `advice_setting` to `vadvise()` with a scope hint. Junio C Hamano asked whether to mark it for `next` and whether the `vadvise()` API could be cleaned up separately.

### odb: introduce object storage extension
*Long thread: 27 replies so far from 4 participants over 10 days.*

Patrick Steinhardt posted v4 of the series adding the `extensions.objectStorage` extension with `<schema>[://<payload>]` syntax; only the "files" backend exists, whose payload can relocate `.git/objects`. The series adds `GIT_OBJECT_STORAGE_FORMAT`, `git rev-parse --show-object-storage-format`, and `--object-storage-format=` for `git init`, `git clone` and `git submodule`. Karthik Nayak reviewed v4 and said he is overall happy.

### Keeping `die()` out of the trace2 API
*Long thread: 26 replies so far from 6 participants over 84 days.*

This series introduces a root-level `banned-die.h` header included by the trace2 API sources, banning die-capable helpers one at a time. On October 6 Derrick Stolee said the topic has sat idle because the `banned-die.h` protections are incomplete.

### CI annotations naming the failing test script
*Long thread: 25 replies so far from 4 participants over 19 days.*

Harald Nordgren's series links failure and leak annotations in CI output to the test script, showing the file and line as plain text after reverting direct file/line linking that failed for files unchanged in a PR. Junio C Hamano marked the topic ready for `next` on October 13.

### format-patch `--range-diff-notes` options
*Long thread: 25 replies so far from 4 participants over 42 days.*

Kristoffer Haugsbakk's series teaches `git format-patch` `--[no-]range-diff-notes=<ref>` so the range diff can use a different notes ref than the patches. The series saw v4 trim expository text and v5 adopt Junio C Hamano's condensed documentation wording.

### Blame reads .git-blame-ignore-revs by default
*Long thread: 24 replies so far from 4 participants over 35 days.*

Ravi Mistry's series teaches `git blame` and `git annotate` to use `.git-blame-ignore-revs` at the working-tree root by default. Version 6 fixed Junio C Hamano's review points, and Hamano approved v6 as "Very cleanly done".

### advertise fetched refs when backfilling tags
*Long thread: 24 replies so far from 4 participants over 22 days.*

Karthik Nayak's series fixes a shallow-clone regression where refs fetched before tag backfill were not advertised as haves, by appending fetched tips to the transport's `negotiation_include_tips`. He posted v6 reordering the upstream-info test commit, and Patrick Steinhardt reviewed v6 and stated he is happy with the changes.

### push: check pushed ref for `--force-if-includes`
*Long thread: 20 replies so far from 5 participants over 45 days.*

Tyler Cipriani's series fixes `--force-if-includes` to consult the reflog of the ref actually being pushed rather than the local branch named after the destination. Version 7 resolves the pushed ref via `ref->peer_ref`, rejects non-branch sources as unverifiable, and defers the reflog check until after fast-forward rules.

### git var `-z`, multiple variables, broken-out idents
*Long thread: 22 replies so far from 6 participants over 39 days.*

Andrew Pleeter's v9 series splits into four patches adding string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, support for multiple variable arguments, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables. Phillip Wood reviewed the series as good with only minor test-style nits.

### `git-bisect-rebase` for iterative rebases
*Long thread: 22 replies so far from 6 participants over 8 days.*

Alejandro Colman's `git-bisect-rebase` bisects for the first commit a branch cannot cleanly rebase onto. On October 8 he implemented `--abort`, `--continue`, and `--quit` with semantics unlike `git-rebase(1)` and corrected the tree algorithm.

### Reference copy and rename through transactions
*Long thread: 22 replies so far from 6 participants over 22 days.*

Maciej Ciemborowicz's four-patch series routes reference copy and rename through ordinary transactions so the reference-transaction hook sees the whole operation. Patrick Steinhardt warned that posting new versions without replying to reviewers is a red flag, and the thread debated AI-assisted workflows.

### `checkout -m` recreating original conflict labels
*Long thread: 22 replies so far from 3 participants over 14 days.*

Phillip Wood's three-patch series teaches the ort machinery to write conflict labels to `.git/MERGE_LABELS` when switching to a conflicted merge result so `git checkout -m` reuses them. Version 4 added a patch caching merge config settings and labels once per invocation, and Junio C Hamano reviewed v4 and said he would replace the previous version.

### diff-delta: use safe integer arithmetic
*Long thread: 22 replies so far from 5 participants over 5 days.*

Srujith A Poojary posted v9 replacing manual size arithmetic in `create_delta_index()` with the overflow-checked `st_mult`/`st_add`/`st_add3` helpers, framing the change as defensive against future padding changes.

### loose-object-idx migration with batch fsync
*Long thread: 22 replies so far from 4 participants over 9 days.*

Justin Tobler's series fixes ODB transaction commit failures when a compatibility hash is configured, because tmp-objdir migration did not merge the "loose-object-idx" mapping file. He posted v3 reading the source mapping file before acquiring the lock, and Patrick Steinhardt reviewed v3 and said he is happy with it.

### xdiff: use `size_t` for `mmfile_t`
*Long thread: 21 replies so far from 4 participants over 3 days.*

Jeff King's seven-patch v2 consolidates `mmbuffer_t` into `mmfile_t`, switches buffer sizes to `size_t`, NUL-terminates buffers read by `read_mmfile()`, and makes `merge-ll` report errors when reading external merge results fails.

### Contributors' Summit 2026 follow-ups and v3.0 timeline
*Long thread: 20 replies so far from 9 participants over 28 days.*

Following the 2026 Contributors' Summit summary and Junio C Hamano's v3.0 timeline, Christian Couder announced attendance at the Gerrit User Summit and an NVIDIA hackathon on partial clones, while Luca Milanesio said he will present his Cassandra object-storage backend prototype for JGit.

### doc: rewrite detached HEAD state explanation
*Long thread: 20 replies so far from 3 participants over 14 days.*

Julia Evans's `gitdetached-head` guide rewrite continued its wording debate, with Hamano questioning the purpose of quotes from experienced users and recording replacement text for the opening paragraphs.

### doc: add more AsciiDoc cross-references
*Long thread: 20 replies so far from 5 participants over 27 days.*

Julia Evans's patch updates `Documentation/` to use the verbose `<<EXAMPLES,EXAMPLES>>` cross-reference form. Jean-Noël Avila argued for the double-reference form because translated manual pages can require changes incompatible with simply inserting the title.

### Teaching documentation navigation in `man git`
*Long thread: 15 replies so far from 5 participants over 10 days.*

Julia Evans's patch rewrites `man git` to explain that `git push -h` gives short help and `git push --help` the full documentation. On October 7 the thread covered why git.github.io was replaced by git-scm.com, and Hamano asked the next iteration to justify dropping the user-manual references.

### odb/files alternates cleanups
*Long thread: 14 replies so far from 3 participants over 2 days.*

Patrick Steinhardt's six-patch v2 series cleans up the files object database backend after alternates migrated into it, adding a patch so corrupt objects are not re-fetched from a promisor remote. Johannes Schindelin noted patch 1/6 fixes the same performance bug Microsoft Git patched independently, and Junio marked the topic ready for `next` on October 16.

### Faster SHA-1 collision detection via sha1dc-accel
*Long thread: 14 replies so far from 6 participants over 10 days.*

Scott Chacon's four-patch series ports Sam Reis's sha1dc Rust crate to C in `sha1dc-accel/`, claiming about 2.7x faster hashing. D. Ben Knoble raised license compatibility, Reis clarified the UBC tables are autogenerated, and Junio confirmed Git's GPL-2 can incorporate MIT-licensed code.

### worktree: copy-on-write creation and ephemeral branches
*Long thread: 11 replies so far from 5 participants over 139 days.*

Jason Newton's v3 adds `git worktree add --reflink` for copy-on-write filesystems plus reaping of ephemeral branches. Eric Sunshine objected that the ephemeral-branch patch adds complexity for a case covered by `--detach`.

### read-cache: do not trust a size change when conversion is active
*Long thread: 10 replies so far from 7 participants over 276 days.*

Curtis Smith and Torsten Bögershausen's patch fixes `ie_modified()` so `git status` no longer reports a file as modified when a CRLF rewrite converts different working-tree bytes to the same blob. Junio C Hamano corrected the commit message and warned the new tests may hit racy-git paths.

### github-ci: update macOS jobs from macos-14
*Long thread: 10 replies so far from 4 participants over 5 days.*

Junio C Hamano posted v4 updating `osx-*` jobs from `macos-14` to `macos-15` or `macos-26`, then retracted it as bogus and settled on v2 after Patrick Steinhardt suggested mixing versions for coverage.

### ".keep" lockfile ownership in ODB transactions
*Long thread: 6 replies so far from 2 participants over 1 day.*

Justin Tobler's series fixes a bug in `object-file.c` where a `git-receive-pack` process could delete another concurrent process's ".keep" lockfile for an identical packfile. Version 2 splits the hash-agnostic comment change and renames to `write_pack_lockfile()`.

## The period in brief

October 2026 saw heavy, eventful traffic across all twenty-eight days, with thirteen long-running threads active simultaneously at peak. The month was dominated by long-running documentation and infrastructure series reaching resolution: Julia Evans's `gitmergeconflicts(7)` page was marked ready for `next`, Harald Nordgren advanced both the shallow-fetch refmap work to v13 and the `--delete-integrated` branch option to v6, and Patrick Steinhardt's object storage extension reached v4 with a positive review. A heated debate over AI-assisted contributions and Contributor Summit follow-ups on the Git 3.0 timeline rounded out a month in which several topics landed in `next` while others stalled on design questions.

## Key developments

### Contributor Summit notes and Git 3.0 planning
Taylor Blau shared notes from the 2026 Git Contributor's Summit covering seven topics: security process, Git 3.0 (possible 2.99 and 3.0 in March 2027 with consensus to release 2.99.1 and 3.0 together with only `BREAKING_CHANGES` defaults flipped), documentation, Outreachy, pluggable ODB, AI policy, and protocol v2 for pushes. Follow-up discussion covered attendance at the Gerrit User Summit and an NVIDIA hackathon on partial clones replacing Git LFS.

### AI-assisted contributions policy debate
Scott Chacon's RFC patch to `Documentation/SubmittingPatches` proposed allowing AI-assisted contributions under the existing DCO with human review and an `Assisted-by:` trailer. brian m. carlsson objected that the DCO cannot honestly be asserted for LLM output without knowing all training sources, while Patrick Steinhardt argued the review bottleneck is tightening as AI-generated slop increases. The debate also surfaced in Maciej Ciemborowicz's reference-transaction series, where Steinhardt warned that posting new versions without replying to reviewers is a red flag.

### Object storage extension matures
Patrick Steinhardt advanced his seven-patch `extensions.objectStorage` series to v4, splitting the storage URI into an explicit `enum object_storage_format` plus payload, adding `--object-storage-format=` to `git-init` and `git-clone`, and propagating the format through `git-submodule`. Karthik Nayak reviewed v4 and said he is overall happy with the version; the series now supports only the "files" backend whose payload can relocate `.git/objects`.

### Shallow-repo fetch series nears completion
Harald Nordgren's v13 of the five-patch series preventing shallow repos from fetching every branch of a new remote landed after D. Ben Knoble's second review. The series adds `remote.<name>.fetchRefmap`, infers refspec-less fetch targets from local upstreams, and defaults `--limited-fetch` for shallow repos; v13 adds the `remote_find_tracking()` fallback when the fetch refspec does not match.

### Incremental connectivity check reworked
Kristofer Karlsson posted v5 of the incremental connectivity check, removing its dependence on `repo->fetch_if_missing` by using `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`. Junio C Hamano judged the incremental changes good and indicated he would replace the previous version; the series supports `transfer.connectivityCheck=incremental` or `"auto"` with a configurable cutoff defaulting to 1000 commits.

### Documentation push: tutorials and merge conflicts
Julia Evans advanced two major doc series: v5 of the `gitmergeconflicts(7)` man page (marked ready for `next`) and v5 of a new 782-line beginner `gittutorial` replacing the old one. Kristoffer Haugsbakk's `gitbreaking-changes(7)` manpage also progressed after Junio decided to proceed despite questions about retaining old-version content.

### Packfile corruption fix lands in `next`
Patrick Steinhardt's two-patch series fixes corruption from delta base cache entries never evicted when their pack closes, reproducible via recursive submodule merges. Jeff King called the fix obviously correct, and on October 7 Junio C Hamano marked the topic for `next`.

### `git log --follow` performance regression
Miklos Vajna posted v4 speeding up `log --follow` for non-linear history, fixing a regression from commit 304812ed33 that had slowed `git log --stat --follow` on linux.git from about 28 seconds to 6m43s and caused an incident for GitLab.com.

## In brief

**Git for Windows security releases** -- Johannes Schindelin announced Git for Windows 2.56.0(3) addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803, and 2.56.0(4) addressing CVE-2026-111497 with corrupt code-signatures in 32-bit MinGit.

**`repack --dry-run` no longer rebuilds packs** -- Siddharth Shrimali's v3 fixes `git repack --drop-filtered --dry-run` writing a new promisor pack instead of just listing candidates; Junio marked the topic for `next`.

**Blame defaults to `HEAD:.git-blame-ignore-revs`** -- Ravi Mistry's v6 was approved as "Very cleanly done" and marked ready for `next`.

**`git status` suggests `git merge --continue`** -- Julia Evans's v3 changes the merge-conflict advice in `wt-status.c` from `git commit` to `git merge --continue`; Junio will apply it with Phillip Wood's `Acked-by:`.

**SSH signing trust limits** -- Christian Ramos reported that `valid-before` is checked against a signer-written date and a missing `gpg.ssh.revocationFile` fails open; Patrick Steinhardt suggested failing by default on a missing revocation file with a `:(optional)` escape hatch.

**`core.indexLockTimeout` for index lock waits** -- Jun Wu posted v2 adding `core.indexLockTimeout` (default 1s) so commands wait briefly for the index lock instead of failing when an IDE's `git status` holds it.

**Worktree repair hardening** -- Yoichi NAKAYAMA posted v3 of a five-patch series validating cross-references between a linked working tree and its administrative data.

**Upload-pack slow fetches from shallow repositories** -- Harald Nordgren submitted a patch fixing protocol v2 fetches from shallow repositories hanging because the server walks its whole history; the patch makes `upload-pack` advertise all shallow grafts when the client does not request a depth.

**Parse-options early-scan API** -- Christian Couder posted v3 adding `early_scan_options()` and the `PARSE_OPT_EARLY` flag; `git fast-import` now uses it for `--allow-unsafe-features`.

**Signing commits created by `git history` rewrites** -- A two-patch series exposes `-S`/`--gpg-sign` on `history` subcommands; Junio marked the topic ready for `next` on October 13.

**Faster client-side refspec matching in push** -- Jon Simons posted v2 of a 12-patch series converting linear ref traversals in `remote.c` to strmap lookups, cutting a 100-delete push against 100k advertised refs from ~2.5s to ~0.15s.

**Sequencer refuses shallow boundary commits by default** -- Nakul V S posted v2 making the sequencer refuse to revert, cherry-pick, or rebase commits at the shallow boundary of a shallow clone.

**`git describe` distance counting with skewed commit dates** -- Linus Torvalds submitted a patch fixing `git describe` miscomputing commit distance when commit dates are unreliable, changing the priority queue ordering to `compare_commits_by_gen_then_commit_date`.

**`git grep --recurse-submodules <tree>` fix** -- Patrick Steinhardt posted a three-patch series fixing failures on nested submodule trees since Git 2.56.0.

**Notes concurrency fix** -- Farid Zakaria's patch makes commands that modify notes fail with "cannot lock ref" instead of silently dropping notes written by a concurrent process.

**Credential-cache daemon startup races** -- Farid Zakaria posted a two-patch series fixing races when two `git credential-cache store` commands run at once and each starts a daemon.

**`pack-objects` BUG on filtered `--unshallow` fetches** -- Patrick Steinhardt traced an abort to `is_not_in_promisor_pack_obj()` calling `BUG()` when an object read fails, and fixed it by skipping missing objects.

**Negotiator fix v3** -- Taylor Blau posted v3 setting SEEN only after a commit is successfully parsed and queued in both negotiators, preventing a "missing parent in priority queue" BUG.

**CI actions pinning** -- Taylor Blau posted v2 pinning CI actions to full commit IDs so a moved tag cannot substitute a malicious revision; Junio queued the patch.

**`git init` absolute path** -- Matt Hunter posted v3 restoring the absolute path in `git init`'s output message; Junio marked it ready for `next`.

**`@{push}` simple resolution** -- Harald Nordgren posted v2 making `@{push}` resolve to the push remote's same-named branch under `push.default=simple`; Junio will queue it.

**HTTP retry handling** -- Ravi Mistry posted a 4-patch series adding exponential backoff, `http.retryStatusCodes` config, and retry support for POST requests.

**`size_t` for large objects** -- Elia Pinto posted multiple series switching object, pack, grep, diff, archive and apply size handling from `unsigned long` to `size_t` to avoid truncation of objects 4GiB and larger.

**`git apply --check` crashes** -- Tanga Joris reported crashes on patches with blank lines after `diff --git`; Hannes Brandstätter-Müller diagnosed a zero `previous->new_mode` in `check_preimage()` for pure renames under `core.fileMode=false`.

**`fetch --all --depth` fix** -- Rong Bao fixed `git fetch --all --depth=` not deepening shallow clones by passing `--depth` to the fetcher subprocess.

**`fsck bigFileThreshold` fix** -- Patrick Steinhardt fixed a bogus fsck corruption report when a packed blob's size equals `core.bigFileThreshold`.

**`sigchain` race** -- Ian Hu fixed a race where concurrent signal handlers could spin a process at 100% CPU.

**`stash -p` performance** -- Jasmeet (Jazz) Bhatia submitted a patch passing the already prepared untracked file list to `do_create_stash()`, reporting 16.7% faster `stash -p`.

**`worktree remove` mid-rebase** -- Jasper Mayone posted a series making `git worktree remove` refuse a worktree mid-rebase or bisect without `--force`.

**`submodule foreach` options** -- William Hatfield posted v2 adding `--reverse-traversal`, `--include-superproject`, and the `--reversive` shorthand.

**`cat-file remote-object-info` limit** -- Pablo Sabater proposed removing the 10k-OID limit and batching internally.

**`imap-send` header reordering** -- Harald Nordgren posted v5 accepting From, Date and Subject headers in any order and case-insensitively.

**`rebase --continue` cleanup** -- Phillip Wood posted a 3-patch series so `git rebase --continue` removes `REBASE_HEAD` and `AUTO_MERGE`.

**`hook.<event>.enabled=false`** -- Thomas Bachem's v2 makes the config suppress hooks-directory hooks; Junio said he will queue it.

**`merge-ort` config without value** -- Thomas Bachem submitted a patch making merge-ort accept `merge.renames` and friends given without a value.

**`ls-files` untracked cache** -- Tamir Duberstein posted v3 letting `git ls-files` reuse and update the untracked cache.

**`git config --global` XDG support** -- Delilah Ashley Wu posted v3 making `git config --global` read both `$HOME/.gitconfig` and the XDG config file.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll posted v4 adding a `[<revs>...]` argument to limit objects processed.

**`fast-import` same-session lookup** -- Marc Becker posted v2 fixing a crash when querying commits created in the same session.

**`bundle` uninitialized memory** -- Olof Fairbrook fixed `git bundle create` freeing uninitialized stack memory on bad arguments.

**`config_buf_fgetc`** -- Olof Fairbrook made `config_buf_fgetc()` return blob bytes as `unsigned char` so `0xff` is not treated as EOF.

**`strbuf_add_wrapped_text` fix** -- Andy Koppe posted v2 fixing lost indentation after a hard line break in `%w()`.

**`root refs` in files backend** -- Andy Koppe posted v2 keeping root refs out of the files backend's loose ref cache.

**`cache-tree` index lock** -- Szymon Wilczek dropped `LOCK_DIE_ON_ERROR` from `write_index_as_tree()` so `git write-tree` proceeds while `index.lock` is held.

**`sequencer` HEAD ref fix** -- Andy Koppe fixed the sequencer clobbering the HEAD ref name in worktree lookup.

**`prune-packed` repository variable** -- Shashank S removed `USE_THE_REPOSITORY_VARIABLE` from `prune-packed.c`.

**`t1901` SHA-256 test** -- Junio C Hamano reported the new test in `t/t1901-repo-structure.sh` is broken with SHA-256 because it hardcodes 40-hex-digit constants.

**`branch named @`** -- Kristoffer Haugsbakk and Junio C Hamano agreed that deprecating `@` as a `HEAD` synonym would not happen.

**`Zsh completion` deduplication** -- Fionn Fitzmaurice patched the Zsh completion to exclude files already present in the argument list.

**`interpret-trailers` documentation** -- Kristoffer Haugsbakk posted v2 rephrasing the `trailer.<key-alias>.cmd` examples; Junio marked the topic for `next` on 10/04.

**`packed-refs` `fwrite()` optimization** -- Karthik Nayak posted v4 with a ~20% deletion win; Patrick Steinhardt called it welcome.

**`fetch commit-graph` from updated refs** -- Kristofer Karlsson posted v3 with benchmarks showing updated-ref fetches dropping from about 357 ms to under 10 ms.

**`alternates` moved into files ODB backend** -- Patrick Steinhardt posted a 13-patch series dropping the `read_alternates` callback from `struct odb_source`.

**`fetch.followRemoteHEAD` deferred validation** -- Colin Hinton posted v5 deferring validation to `do_fetch()`; Junio endorsed dying rather than warning on bad values.

**`post-worktree` hook** -- Domen Kožar posted v3 adding a single `post-worktree` lifecycle hook for add, move, and remove, extended to pruning.

**`git stash create` options** -- Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`; Junio critiqued the commit message's user-visible framing.

**`sparse-checkout` clobbering untracked files** -- Webstrand reported that `git checkout` with sparse-checkout clobbers an untracked in-cone file instead of aborting.

**`merge-ort --no-overwrite-ignore` bypass** -- Coy Geek reported that `git merge --no-overwrite-ignore` fails to protect an ignored file when directory rename detection moves a committed file onto that path.

**`repack` `.keep` race** -- Qin ShiCheng's five-patch series fixes a race where a concurrent push installs a `.keep` after `git repack -d` decided which packs to delete; Junio suggested waiting for the `ps/odb-files-alternates` topic.

**`backfill --dry-run` RFC** -- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee questioned the size metric.

**`early_scan_options` review** -- Kaartic Sivaraam reviewed Christian Couder's `early_scan_options()` series, flagging abbreviation handling.

**`fsmonitor` untracked cache invalidation** -- Ilia K posted a patch adding `untracked_cache_invalidate_stale_dirs()` after the fsmonitor daemon sends a trivial response.

**`merge-ll` tempfile cleanup** -- Jeff King's v3 cleans up `.merge_file_*` temporaries on Ctrl+C; Junio said he would mark it for `next`.

**`default reflog expiry` periods** -- Pushkar Singh's v3 swaps back the default reflog expiry periods in `reflog.h`.

**`optional sha1dc` Rust backend** -- Johannes Schindelin proposed a compile-time option to use the Rust `sha1dc` crate; brian m. carlsson objected that the crate requires Rust 1.87.

**`CI hard-fail` on optional dependency install failures** -- Patrick Steinhardt suggested the CI script should fail outright when perforce, git-lfs or JGit fail to install.

**`git -C` alias worktree environment bug** -- Maya Barriault reported that `git -C other_repo` run via an alias from a worktree sets `GIT_DIR` to the worktree's values.

**`ZIP timestamp` bugs** -- René Scharfe explained DOS timestamps cover only 1980 onward and recommended rejecting non-representable timestamps in `git archive --format=zip`.

**`branch hook` gaps** -- Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook.

**`refs copy/rename` transaction hook impasse** -- Maciej Ciemborowicz's v2 hit a review impasse when Patrick Steinhardt declared it not reviewable due to apparently AI-generated content.

**`RFC: sign SHA-256` digest of tree contents** -- Scott Chacon posted an RFC; brian m. carlsson rejected it in favor of moving to SHA-256.

**`shallow clone` boundary commit warning** -- Patrick Steinhardt agreed Git should warn about or refuse editing a shallow boundary commit by default.

**`incremental connectivity` check via rev-list** -- Kristofer Karlsson posted v2 of the opt-in `transfer.connectivityCheck=incremental` mode.

**`status/diff/add` conversion disagreement** -- Curtis Allen Smith's series fixes `git status` disagreeing with `git diff` and `git add` when a clean filter is active.

**`Wincred` refactor** -- Marc Becker posted v2 splitting credential blob dissection into separate methods.

**`imap-send.c` ASN1 handling** -- Xiangfei Ding's patch uses `ASN1_STRING_length` and `strncasecmp` instead of assuming null-terminated data.

**`archive-zip` UT field** -- René Scharfe's patch makes `archive-zip.c` omit the UT extra field when the commit time exceeds the signed 32-bit range.

**`worktree repair` error behavior** -- Yoichi NAKAYAMA explained adjusted error-case behavior and said he will split the commits.

**`SubmittingPatches` test preference** -- Jon Simons documented the preference for submitting a bug fix's test in the same commit as the code change.

**`GSoC fsck` microproject** -- Aniket Gupta introduced himself as a GSoC applicant and proposed an fsck plural-message microproject.

**`chainlint/greplint` merge** -- Tyler Cipriani's series teaches `t/chainlint.pl` to run greplint checks, cutting `make test-lint` from about 25 seconds to 6.

**`submodule ^/` URL syntax** -- Stanislav Aleksandrov's RFC proposes resolving `^/` submodule URLs against the superproject's remote; Jacob Keller concluded the concept makes sense.

**`partial clone` merge failure** -- Andrew Parmet reported that in a partial clone a merge can fail with "not our ref" when requesting a locally created blob from the promisor remote.

**`maintenance lock`** -- Masaya Kazama reported that a killed detached maintenance child leaves a stale `objects/maintenance.lock`.

**`diff-delta` safe arithmetic** -- Srujith Poojary posted v5 of the safe-integer-arithmetic patch with expanded commit message bounds.

**`community voting`** -- Emily Shaffer's proposed `CommunityVoting` process drew debate on voter eligibility and platform choice.

**`MyFirstContribution`** -- Junio C Hamano posted v4 reordering "respond to reviews" before "send v2".

**`gitsecurityboundary.adoc`** -- Johannes Schindelin posted a document defining Git's security boundary around unexpected execution of attacker-controlled code.

**`quarantine alternate-refs` perf** -- Kristofer Karlsson proposed skipping the repository's own object directory when listing alternate refs in the receive-pack quarantine.

**`not a git repository` errors** -- Kaartic Sivaraam posted v3 improving these errors to name the inspected directory and its source.
