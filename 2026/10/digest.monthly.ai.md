## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 73 replies so far from 6 participants over 33 days.*
Julia Evans's six-patch series adds a `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from the `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` documentation. The series progressed through five versions during October, absorbing Junio C Hamano's corrected diff3 example, requests to list every conflict-causing command explicitly, and formal-construction wording. By late October Evans posted v5 dropping the zdiff3 note and fixing indentation, and Hamano accepted the iteration and marked the topic ready for `next`.

### fetch refmap configuration for shallow repos
*Long thread: 66 replies so far from 5 participants over 42 days.*
Harald Nordgren's series keeps shallow repositories from fetching every branch of a new remote by introducing `remote.<name>.fetchRefmap` (renamed from `refmap` in v12) while `remote.<name>.fetch` still decides what is fetched. The work advanced through thirteen versions, adding a `remote_find_tracking()` fallback so `@{push}` resolves correctly for `--limited-fetch` remotes, with tests locking down negative refspecs and `--delete-merged` behavior. Phillip Wood and D. Ben Knoble supplied second reviews, and Junio C Hamano called v10 fairly well done while requesting additional eyes on the refmap inference logic.

### `git branch --delete-integrated` for squash- and rebase-merged branches
*Long thread: 53 replies so far from 6 participants over 34 days.*
Harald Nordgren's series adds `--delete-integrated <pattern>` to `git branch`, deleting branches whose changes landed upstream via squash or rebase merges and naming the landing commit. The thread explored whether to extend `--delete-merged` instead, with Junio C Hamano objecting to loosening the default after a test showed a private branch wrongly deleted when its changes appeared upstream under a different author. Nordgren conceded that `--delete-merged=squashed` was stopped because the option's value slot is taken, and pivoted to the separate `--delete-integrated` command, which reached v6 with index-aware `.gitattributes` handling.

### stash: in-core index merge for `stash apply --index`
*Long thread: 53 replies so far from 5 participants over 13 days.*
D. Ben Knoble's series evolved from v4 to v5 during the first week, fixing tri-state handling of `merge_result.clean` that Junio C Hamano had diagnosed. The final patch replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, resolving a non-re-entrant lock bug when autostashing with `stash.index=true`. Phillip Wood confirmed the range-diff and welcomed the removal of subprocesses, and Hamano marked the series for `next` on 10/01.

### CI housekeeping and modernizations
*Long thread: 41 replies so far from 6 participants over 12 days.*
Patrick Steinhardt's eight-patch series fixes smaller CI issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed" and reworking the reftable job. Version 4 reworked t5004 to run under both SHA-1 and SHA-256 by forcing the test repo to SHA-1. Jeff King approved v4 and Junio C Hamano marked the series for `next`, with a follow-up queued to resurrect a no-Rust CI job by adding `NO_RUST` to a Makefile-based job.

### More `path.*` keys for `git repo info`
*Long thread: 39 replies so far from 5 participants over 109 days.*
K Jayatheerth's GSoC series adds path keys to `git repo info`: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix` and `path.cdup`, with absolute and relative suffix formatting. On November 1 Junio C Hamano stated that the v7 series had seen no interest or review for a full month and would be discarded from his tree unless volunteers gave it a final review within a few weeks; Justin Tobler said he would take a look.

### New beginner Git tutorial
*Long thread: 38 replies so far from 6 participants over 25 days.*
Julia Evans's series replaces the old `gittutorial` with a new 782-line beginner tutorial covering `git init` through `git push` using only commands available in Git 2.23.0. She posted v4 making instructions Windows-friendly by mentioning Git for Windows' editor prompt and marking SSH steps as Mac/Linux only, and v5 replacing an ambiguous "it" with "GCM". Junio C Hamano reviewed the patch and asked whether the version was in good shape to merge to `next`.

### credentialStore.lockTimeout config option
*Long thread: 38 replies so far from 5 participants over 45 days.*
Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. Ajayi posted v2 looping the two timeout tests over both names, wrapping long lines, and dropping a problematic test. Junio C Hamano reviewed the initial version, suggesting the loop over both variable names to keep tests in sync.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control
*Long thread: 37 replies so far from 3 participants over 96 days.*
Christian Couder's five-patch series introduces the protected `uploadpack.lazyFetchTrusted` configuration variable so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones. On October 5 Junio C Hamano asked whether the topic had reached diminishing returns; Couder agreed v5 should be marked for `next`, and Hamano did so on October 13.

### rerere waiting for `MERGE_RR.lock`
*Long thread: 36 replies so far from 5 participants over 43 days.*
Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` before failing, so a rebase hitting a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` option to `git rerere gc`. Version 8 incorporated earlier review; Patrick Steinhardt called v8 good enough and agreed to mark the topic ready for `next`.

### connected: incremental connectivity check
*Long thread: 35 replies so far from 3 participants over 40 days.*
Kristofer Karlsson's series adds an incremental connectivity-check mode behind `transfer.connectivityCheck=incremental` or `"auto"` with a configurable cutoff defaulting to 1000 commits. After Junio C Hamano questioned gating on "not full", Karlsson reworked v5 to use `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`, verified in a partial clone that promised blobs report as missing without fetching, and Hamano judged the incremental changes good on 10/23.

### `http.sslVerifyStatus` for stapled OCSP responses
*Long thread: 33 replies so far from 5 participants over 59 days.*
Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off. After v7 was reverted out of `next` over a t5585 failure traced to a GnuTLS curl backend, Junio C Hamano bisected linux-* CI jobs spinning until their 6-hour timeout to this topic and tentatively ejected it from `seen` on October 7, confirming in isolation on Git 2.56 that it causes the timeouts.

## The period in brief

October 2026 saw heavy, eventful traffic across twenty-eight active days, with thirteen long-running threads active simultaneously at peak. The month was dominated by long-running documentation and infrastructure series absorbing detailed review, alongside major architectural work on object storage and shallow-repo fetching. Readers should not miss Julia Evans's `gitmergeconflicts(7)` page being marked ready for `next`, Patrick Steinhardt's `extensions.objectStorage` series reaching v4, and the protracted debate over squash-merge detection in `git branch`.

## Key developments

### Contributor Summit notes and Git 3.0 planning
Taylor Blau shared notes from the 2026 Git Contributor's Summit covering seven topics: security process, Git 3.0 timing (possible 2.99 and 3.0 in March 2027 with consensus to release 2.99.1 and 3.0 together flipping only `BREAKING_CHANGES` defaults), documentation, Outreachy, pluggable ODB, AI policy, and protocol v2 for pushes. Follow-up discussion covered Gerrit and NVIDIA hackathon attendance, and a Cassandra object-storage backend prototype for JGit.

### AI-assisted contributions policy debate
Scott Chacon's RFC patch to `Documentation/SubmittingPatches` proposed allowing AI-assisted contributions under the existing DCO with human review and an `Assisted-by:` trailer. brian m. carlsson objected that the DCO cannot honestly be asserted for LLM output without knowing all training sources, while Patrick Steinhardt argued the review bottleneck is tightening as AI-generated slop increases. The debate also surfaced in Maciej Ciemborowicz's reference-transaction series, where Steinhardt warned that posting new versions without replying to reviewers is a red flag.

### Object storage extension matures
Patrick Steinhardt advanced his seven-patch `extensions.objectStorage` series to v4, adding the extension modeled on the ref storage extension with `<schema>[://<payload>]` syntax; only the "files" backend exists, whose payload can relocate `.git/objects`. The series adds `GIT_OBJECT_STORAGE_FORMAT`, `git rev-parse --show-object-storage-format`, and `--object-storage-format=` for `git init`, `git clone` and `git submodule`. Karthik Nayak reviewed v4 and said he was overall happy with the version.

### Packfile delta-base cache corruption fix
Patrick Steinhardt's two-patch series fixes corruption from delta base cache entries never evicted when their pack closes, reproducible via recursive submodule merges. Jeff King called the fix obviously correct and characterized the core bug as use-after-free, and on October 7 Junio C Hamano marked the topic for `next`.

### Shallow-repo fetch series nears completion
Harald Nordgren's v13 of the five-patch series preventing shallow repos from fetching every branch of a new remote landed after D. Ben Knoble's second review. The series adds `remote.<name>.fetchRefmap`, infers refspec-less fetch targets from local upstreams, and defaults `--limited-fetch` for shallow repos. Related fixes included Karthik Nayak's v6 advertising fetched refs as "haves" when backfilling tags, and a patch making `upload-pack` advertise all shallow grafts when the client does not request a depth.

### Documentation push: tutorials and merge conflicts
Julia Evans advanced multiple doc series: v5 of the `gitmergeconflicts(7)` man page (marked ready for `next`), v5 of a new 782-line beginner `gittutorial` replacing the old one, v3 of a rewritten detached HEAD guide, and v3 removing `gittutorial-2` from `command-list.txt`. Kristoffer Haugsbakk's `gitbreaking-changes(7)` manpage also progressed after Junio decided to proceed despite questions about retaining old-version content.

### Incremental connectivity check reworked
Kristofer Karlsson posted v5 of the incremental connectivity check, removing its dependence on `repo->fetch_if_missing` by using `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT`. Junio C Hamano judged the incremental changes good and indicated he would replace the previous version; the series supports `transfer.connectivityCheck=incremental` or `"auto"` with a configurable cutoff defaulting to 1000 commits.

### `repack --dry-run` no longer rebuilds packs
Siddharth Shrimali's patch fixes `git repack --drop-filtered --dry-run`, which was documented to list objects without rebuilding any pack but did both because the `--dry-run` block fell through into the regular repack code. On October 10 he posted v2 returning right after listing the drop candidates; D. Ben Knoble offered his `Reviewed-by:`, and on October 11 Junio marked the topic for `next`.

## In brief

**Blame reads .git-blame-ignore-revs by default** -- Ravi Mistry's v6 teaches `git blame` and `git annotate` to use `.git-blame-ignore-revs` at the working-tree root by default; Junio C Hamano approved v6 as "Very cleanly done".

**`checkout -m` recreating original conflict labels** -- Phillip Wood's v4 teaches the ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` reuses them; Hamano said he would replace the previous version.

**Push `--force-if-includes` checks the pushed ref** -- Tyler Cipriani's v7 makes the check consult the reflog of the ref actually being pushed, rejecting non-branch sources as unverifiable and deferring the check so fast-forward pushes proceed.

**Sequencer refuses shallow boundary commits** -- Nakul V S posted v2 making the sequencer refuse to revert, cherry-pick, or rebase commits at the shallow boundary of a shallow clone, with `--allow-shallow-boundary` overriding for revert and cherry-pick only.

**`git log --follow` performance regression** -- Miklos Vajna posted v4 speeding up `log --follow` for non-linear history, fixing a 2.69x slowdown from commit 304812ed33 that had caused an incident for GitLab.com.

**`git describe` distance counting fix** -- Linus Torvalds submitted a patch fixing `git describe` miscomputing commit distance when commit dates are unreliable, changing priority queue ordering to `compare_commits_by_gen_then_commit_date`.

**`core.indexLockTimeout` for index lock waits** -- Jun Wu posted v2 adding `core.indexLockTimeout` so commands like `git add` wait briefly for the index lock instead of failing when an IDE's `git status` holds it.

**Worktree repair hardening** -- Yoichi NAKAYAMA posted v3 of a five-patch series validating cross-references between linked working trees and administrative data to prevent modification of unrelated `.git` files.

**`git grep --recurse-submodules` fix** -- Patrick Steinhardt posted a three-patch series fixing failures on nested submodule trees since Git 2.56.0; Toon Claes approved all three.

**Notes concurrency fix** -- Farid Zakaria's patch makes commands that modify notes fail with "cannot lock ref" instead of silently dropping notes written by a concurrent process.

**Credential-cache daemon races** -- Farid Zakaria posted a two-patch series fixing races where two `git credential-cache store` commands each start a daemon and the second unlinks the first's socket.

**`git var -z` and multiple variables** -- Andrew Pleeter's v9 adds string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables.

**xdiff `size_t` for `mmfile_t`** -- Jeff King's seven-patch v2 consolidates `mmbuffer_t` into `mmfile_t` and switches buffer sizes to `size_t`.

**Reference copy and rename through transactions** -- Maciej Ciemborowicz's v4 routes reference copy and rename through ordinary transactions so the reference-transaction hook sees the whole operation.

**`git-bisect-rebase` for iterative rebases** -- Alejandro Colman implemented `--abort`, `--continue`, and `--quit` semantics for bisecting the first commit a branch cannot cleanly rebase onto.

**Removing `gittutorial-2`** -- Julia Evans's v3 deletes the stale page and removes it from `command-list.txt` so `git help --guides` does not advertise it.

**Community voting process** -- Emily Shaffer's proposed `CommunityVoting` document drew debate over secret ballots and voter rolls, with open questions on the platform leaning toward CIVS.

**`path.*` keys for `git repo info`** -- K Jayatheerth's GSoC series faces discard from Junio's tree unless Justin Tobler or others provide final review within weeks.

**Reflog branch naming** -- Harald Nordgren posted v3 of a `%gb` placeholder in `--format` naming the branch HEAD was on when each entry was recorded.

**diff-delta safe arithmetic** -- Srujith A Poojary posted v9 replacing manual size arithmetic in `create_delta_index()` with overflow-checked helpers.

**loose-object-idx migration** -- Justin Tobler's v3 fixes ODB transaction commit failures when a compatibility hash is configured by reading the source mapping file before acquiring the lock.

**`.keep` lockfile ownership** -- Justin Tobler's v4 migrates `.keep` lockfile creation to ODB transaction commit so concurrent `git receive-pack` processes no longer delete each other's lockfiles.

**Reftable reflog timezone fix** -- Patrick Steinhardt's three-patch series fixes `refs/reftable-backend.c` storing timezones as signed HHMM instead of minute offsets; marked for `next`.

**HTTP transport fix for curl 8.23.0** -- Jeff King posted a patch because upcoming curl strips credentials from `CURLINFO_EFFECTIVE_URL`, using `curl_url()`-based `match_effective_url()`.

**filter-branch state-branch fix** -- Grant Moyer posted v3 fixing `git filter-branch --state-branch` map initialization that interpreted object pairs in reverse since 2.50.0.

**Git for Windows security releases** -- Johannes Schindelin announced 2.56.0(3) addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803, and 2.56.0(4) addressing CVE-2026-111497.

**SSH signing trust limits** -- Christian Ramos reported `valid-before` is checked against a signer-written date and missing `gpg.ssh.revocationFile` fails open; Steinhardt suggested failing by default with a `:(optional)` escape hatch.

**`git status` suggests `git merge --continue`** -- Julia Evans's v3 changes merge-conflict advice from `git commit` to `git merge --continue`; marked for `next`.

**Signing commits created by `git history` rewrites** -- A two-patch series exposes `-S`/`--gpg-sign` on history subcommands; marked ready for `next` on October 13.

**Faster client-side refspec matching** -- Jon Simons posted v2 of a 12-patch series converting linear ref traversals to strmap lookups, cutting a 100-delete push against 100k refs from ~2.5s to ~0.15s.

**Parse-options early-scan API** -- Christian Couder posted v3 adding `early_scan_options()` so `git fast-import` can safely skip option values during early scans.

**Fetch commit-graph using updated refs** -- Kristofer Karlsson posted v3 with benchmarks showing updated-ref fetches dropping from about 357 ms to under 10 ms.

**Packed-refs `fwrite()` optimization** -- Karthik Nayak posted v4 with a ~20% deletion win; Patrick Steinhardt called it welcome.

**Alternates moved into files ODB backend** -- Patrick Steinhardt's 13-patch series drops the `read_alternates` callback from `struct odb_source`; Karthik Nayak said the whole series looks good.

**CI hard-fail on optional dependency failures** -- Patrick Steinhardt suggested CI should fail outright when perforce, git-lfs or JGit fail to install; Harald Nordgren agreed.

**`git -C` alias worktree environment bug** -- Maya Barriault reported `git -C other_repo` via an alias from a worktree sets `GIT_DIR` to the worktree's values.

**Default reflog expiry periods swapped back** -- Pushkar Singh's v3 swaps back the default reflog expiry periods; Hamano accepted testing at 88 days.

**Sparse-checkout clobbering untracked files** -- Webstrand reported `git checkout` with sparse-checkout clobbers untracked in-cone files instead of aborting.

**merge-ort `--no-overwrite-ignore` bypass** -- Coy Geek reported the flag fails to protect ignored files when directory rename detection moves a committed file onto that path.

**Branch hook gaps** -- Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook.

**`fetch.followRemoteHEAD` deferred validation** -- Colin Hinton posted v5 deferring validation to `do_fetch()` so `git push` stops warning.

**Zsh completion deduplication** -- Fionn Fitzmaurice patched the Zsh completion to exclude files already present in the argument list.

**`interpret-trailers` documentation examples** -- Kristoffer Haugsbakk posted v2 rephrasing `trailer.<key-alias>.cmd` examples; marked for `next`.

**`@{p}` shorthand for `@{push}`** -- Harald Nordgren's patch makes `@{p}` short for `@{push}`; marked queued.

**`git checkout -m` conflict labels** -- Phillip Wood's series makes `git checkout -m` reuse original conflict labels via `.git/MERGE_LABELS`.

**`git repo structure` ref-type filters** -- Patrick Steinhardt objected to Mark Chu-Carroll's ref-type filter flags, preferring revision-based filtering.

**`backfill --dry-run` RFC** -- Pablo Sabater's RFC adds `--dry-run` to `git backfill`; Derrick Stolee proposed `--info=(count|size)`.

**`early_scan_options` review** -- Kaartic Sivaraam reviewed Christian Couder's series, flagging abbreviation handling.

**Optional sha1dc Rust backend** -- Johannes Schindelin proposed a compile-time option for the Rust `sha1dc` crate; brian m. carlsson objected to the Rust 1.87 requirement.

**merge-ll tempfile cleanup** -- Jeff King's v3 cleans up `.merge_file_*` temporaries on Ctrl+C; marked for `next`.

**Incremental connectivity check via rev-list** -- Kristofer Karlsson posted v2 of the opt-in mode reporting up to 190x speedup for single-commit pushes.

**RFC: sign SHA-256 digest of tree contents** -- Scott Chacon posted an RFC; brian m. carlsson rejected it in favor of moving to SHA-256.

**`history` sign rewritten commits** -- Souma posted v5 adding Git completion coverage for `-S`/`--gpg-sign` on history subcommands.

**Documentation synopsis cleanup** -- Julia Evans published v2 removing required SYNOPSIS sections from section 7 man pages.

**Precompiled header `__FILE__` expansion** -- SZEDER Gábor noted the precompiled header changes `__FILE__` expansion, which he judged acceptable.

**`repack --dry-run` writes packs bug** -- Coy Geek reported the command writes new pack files contradicting the documented no-write promise.

**fsmonitor untracked cache invalidation** -- Ilia K posted a patch invalidating stale untracked-cache directories after trivial fsmonitor responses.

**post-worktree hook with prune notification** -- Domen Kožar posted v3 adding a single `post-worktree` lifecycle hook extended to pruning.

**`git stash create` options** -- Kazumasa Shigeta's v2 lets `git stash create` accept `-u`/`-a`.

**`git apply --check` crash** -- Gingrspacecadet reported a crash on patches creating new files; Hannes Brandstätter-Müller diagnosed a zero `previous->new_mode`.

**`git init` absolute path** -- Matt Hunter posted v3 restoring the absolute path in `git init`'s output message; marked ready for `next`.

**`submodule foreach` options** -- William Hatfield posted v2 adding `--reverse-traversal`, `--include-superproject`, and `--reversive`.

**`worktree remove` mid-rebase** -- Jasper Mayone posted a series refusing removal of worktrees mid-rebase or bisect without `--force`.

**`gitsecurityboundary.adoc`** -- Johannes Schindelin posted documentation defining Git's security boundary around unexpected execution of attacker-controlled code.

**HTTP retry handling** -- Ravi Mistry posted a 4-patch series adding exponential backoff, `http.retryStatusCodes`, and POST retry support.

**`merge-ort` config without value** -- Thomas Bachem submitted a patch accepting `merge.renames` etc. given without a value.

**`stash -p` performance** -- Jasmeet Bhatia submitted a patch passing the untracked file list to `do_create_stash()`, reporting 16.7% faster `stash -p`.

**`size_t` for large objects** -- Elia Pinto posted multiple series switching size handling to `size_t` to avoid truncation of objects 4GiB and larger.

**`@{push}` simple resolution** -- Harald Nordgren posted v2 making `@{push}` resolve to the push remote's same-named branch under `push.default=simple`.

**`t5702` env fix** -- Jeff King posted a patch replacing `env`-redirect-and-grep checks with direct `echo` output compared via `test_cmp`.

**root-refs-vs-pseudorefs docs** -- Andy Koppe posted v2 adding both to the ref types in `gitdatamodel(7)`.

**`imap-send` TLS updates** -- Beat Bolli posted v3 replacing deprecated `SSLv23_method()` with `TLS_method()`.

**`archive-zip` UT field** -- René Scharfe's patch omits the UT extra field when commit time exceeds signed 32-bit range.

**`fast-import` crash fix** -- Marc Becker posted v2 fixing a crash when querying commits created in the same session.

**`fsmonitor` daemon renamed subdirs** -- Erik Cervin-Edin reported the Linux daemon misses edits in subdirectories of renamed directories.

**`fetch --all --depth` fix** -- Rong Bao fixed `--all --depth=` and `--all --unshallow` not deepening shallow clones.

**`connectivity check` with alternates** -- Igor Wiedler posted v2 negating `--alternate-refs` in deepening fetches; ejected from `seen` after breaking Windows CI.

**`fsck` bigFileThreshold fix** -- Patrick Steinhardt fixed a bogus corruption report when a packed blob's size equals `core.bigFileThreshold`.

**sequencer HEAD ref fix** -- Andy Koppe fixed the sequencer clobbering the HEAD ref name in worktree lookup.

**`stash` index loads** -- Phillip Wood posted v2 avoiding unnecessary index loads in `git stash push`/`create`.

**sigchain race** -- Ian Hu fixed a race where concurrent signal handlers could spin a process at 100% CPU.

**HTTP push token-only URL** -- Dan reported `git push` over smart HTTP fails with token-only userinfo after libcurl 8.23.0.

**cache-tree index lock** -- Szymon Wilczek dropped `LOCK_DIE_ON_ERROR` from `write_index_as_tree()` so `git write-tree` proceeds while `index.lock` is held.

**detached HEAD guide** -- Julia Evans posted v3 and v4 of a rewritten detached HEAD guide.

**root refs in files backend** -- Andy Koppe posted v2 keeping root refs out of the files backend's loose ref cache.

**branch named `@`** -- Kristoffer Haugsbakk and Junio C Hamano agreed deprecating `@` as a `HEAD` synonym would not happen.

**`t1901` SHA-256 test** -- Junio C Hamano reported the new test is broken with SHA-256 because it hardcodes 40-hex-digit constants.

**RFC `--delete-merged --into`** -- DonQuinleone proposed extending `--delete-merged` with a `--into <ref>` modifier.

**`strbuf_add_wrapped_text` fix** -- Andy Koppe posted v2 fixing lost indentation after a hard line break in `%w()`.

**`config_buf_fgetc`** -- Olof Fairbrook made `config_buf_fgetc()` return blob bytes as `unsigned char`.

**bundle uninitialized memory** -- Olof Fairbrook fixed `git bundle create` freeing uninitialized stack memory on bad arguments.

**`followRemoteHEAD`** -- Colin Hinton deferred validation to `do_fetch()`; Hamano asked for a `lazy_follow_remote_head()` helper.

**cargo target-dir** -- Andy Koppe pinned `--target-dir target` in Makefile cargo recipes.

**`prune-packed` repository variable** -- Shashank S removed `USE_THE_REPOSITORY_VARIABLE` from `prune-packed.c`.

**test modernization** -- Srujith Poojary posted v3 replacing old-style `test -f` checks in `t/t2030-unresolve-info.sh`.

**`GIT_DEPTH` for CI** -- Patrick Steinhardt proposed setting `GIT_DEPTH: 0` for GitLab CI whitespace and style jobs.

**`status.compareBranches` default** -- Harald Nordgren posted v3 making `@{upstream} @{push}` the default; Hamano argued against flipping the default.

**chainlint/greplint merge** -- Tyler Cipriani's series teaches `t/chainlint.pl` to run greplint checks, cutting `make test-lint` from 25s to 6s.

**submodule `^/` URL syntax** -- Stanislav Aleksandrov's RFC proposes resolving `^/` against the superproject's remote; Jacob Keller concluded the concept makes sense.

**hook configuration** -- Thomas Bachem's v2 makes `hook.<event>.enabled=false` suppress hooks-directory hooks; Hamano said he will queue it.

**partial clone merge failure** -- Andrew Parmet reported a merge can fail with "not our ref" when requesting a locally created blob from the promisor remote.

**maintenance lock** -- Masaya Kazama reported a killed detached maintenance child leaves a stale `objects/maintenance.lock`.

**`ls-files` untracked cache** -- Tamir Duberstein posted v3 letting `git ls-files` reuse and update the untracked cache.

**`MyFirstContribution`** -- Junio C Hamano posted v4 reordering "respond to reviews" before "send v2".

**`git repo info` path keys** -- K Jayatheerth's series faces discard unless reviewed soon.

**community voting** -- Emily Shaffer's document remains under discussion with open questions on platform and voter roll.

## Looking ahead

K Jayatheerth's GSoC series adding `path.*` keys to `git repo info` faces discard from Junio C Hamano's tree unless Justin Tobler or other volunteers provide a final review within a few weeks. Emily Shaffer's community voting process document remains under discussion and may see a reroll addressing requests to drop the word "code" and use Message-IDs. The `--delete-integrated` design for squash-merged branch detection may resurface after the `--delete-merged=squashed` syntax was conceded stopped. Patrick Steinhardt's object storage extension and Harald Nordgren's shallow-fetch refmap series are both queued for further review and likely to land in `next` soon, as is the incremental connectivity check v5 after Junio's approval.