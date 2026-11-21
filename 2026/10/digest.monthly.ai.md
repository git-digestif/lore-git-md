## Long threads

### New `gitmergeconflicts(7)` documentation page
*Long thread: 73 replies so far from 6 participants over 33 days.*
Julia Evans's six-patch series adds a new manual page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull`. The series went through five iterations, absorbing review from Junio C Hamano, D. Ben Knoble, and Patrick Steinhardt on the "WHAT IS A MERGE CONFLICT?" section, diff3 examples, and OURS/THEIRS terminology. By month's end Junio marked the topic ready for `next` after Evans posted v5 fixing indentation and wording nits.

### fetch refmap configuration for shallow repos
*Long thread: 66 replies so far from 5 participants over 42 days.*
Harald Nordgren's series keeps shallow repositories from fetching every branch of a new remote by adding `remote.<name>.fetchRefmap` (renamed from `refmap` in v12), inferring branches from local upstreams, and defaulting to `--limited-fetch` in shallow clones. The series reached v13, resolving `@{push}` through the refmap and handling interactions with `remote.<name>.fetch`. Phillip Wood and Junio C Hamano debated whether one config should control both what is fetched and where refs are stored, with the final design keeping `fetch` for selection and `fetchRefmap` for remapping.

### `git branch --delete-integrated` for squash- and rebase-merged branches
*Long thread: 53 replies so far from 6 participants over 33 days.*
Harald Nordgren's series adds `--delete-integrated <pattern>` to delete branches whose changes landed upstream via squash or rebase merges, naming the landing commit as "landed as". After objections to loosening `--delete-merged` itself, v4 restored the strict reachability check for `--delete-merged` while the new option handles squash-merged branches. Jasper Mayone tested v4 against master confirming correct behavior across squash merges, renames, and partial landings. The series reached v5 fixing usage text and error messages.

### stash: in-core index merge for `stash apply --index`
*Long thread: 53 replies so far from 5 participants over 13 days.*
D. Ben Knoble's series replaces the subprocess-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call, fixing a bug in the interaction between autostashing, staged entries, and `stash.index=true`. Phillip Wood confirmed the v5 range-diff and Junio C Hamano marked the series for `next` early in the month.

### CI housekeeping and modernization series
*Long thread: 41 replies so far from 6 participants over 12 days.*
Patrick Steinhardt's eight-patch series fixes CI issues and moves jobs off EOL Docker images, renaming "linux-exotic" to "linux-feature-testbed" and reworking the reftable job. The series reached v4, marked for `next` by Junio C Hamano, with Jeff King reviewing the t5004 SHA-256 test rework and the reftable-plus-SHA-256 coverage job.

### New beginner `gittutorial` page
*Long thread: 38 replies so far from 6 participants over 25 days.*
Julia Evans's series replaces the old `gittutorial` with a new 782-line beginner tutorial covering `git init` through `git push` using only commands available in Git 2.23.0. The series reached v5, making instructions Windows-friendly by marking SSH steps as Mac/Linux only and suggesting Git Credential Manager on Windows. Junio C Hamano reviewed the patch and asked whether it was ready for `next`.

### `credentialStore.lockTimeout` config option
*Long thread: 38 replies so far from 5 participants over 45 days.*
Okhuomon Ajayi's Outreachy patch adds `credentialStore.lockTimeout` as the preferred setting for credential lock retries, keeping `lockTimeoutMS` as a deprecated alias. The patch reached v2, looping tests over both variable names and wrapping long lines. Junio C Hamano reviewed the changes and questioned whether the "allows a store" test would fail with a 0 ms timeout.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control
*Long thread: 37 replies so far from 3 participants over 96 days.*
Christian Couder's series introduces the protected `uploadpack.lazyFetchTrusted` config so server operators can mark trusted repositories for lazy fetching during clones. The series reached v5, rebased on Git 2.56 with documentation and test updates. Junio C Hamano marked the topic ready for `next` on October 13 after noting diminishing returns on further review.

### rerere waiting for `MERGE_RR.lock`
*Long thread: 36 replies so far from 5 participants over 43 days.*
Thomas Bachem's series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` and adds a hidden `--skip-locked` option to `git rerere gc` for auto-maintenance. The series reached v8, with Patrick Steinhardt calling it "in a good enough state" and Junio C Hamano asking whether to mark it ready for `next`.

### Incremental connectivity check in `connected.c`
*Long thread: 35 replies so far from 3 participants over 40 days.*
Kristofer Karlsson's series adds an incremental mode to the connectivity check behind `transfer.connectivityCheck=incremental` or `auto`, using `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT` to avoid lazy fetching. The series reached v5, with Junio C Hamano reviewing and indicating he would replace the previous version after Karlsson removed the dependence on `repo->fetch_if_missing`.

## The period in brief

October 2026 saw heavy traffic across 31 active days, with multiple long-running series reaching maturity and several marked ready for `next`. The month opened the Git 2.98 cycle when Junio C Hamano flushed `next` to `master`, and closed with discussions of the Git 3.0 timeline targeting March 2027. Readers should not miss the new `gitmergeconflicts(7)` documentation page, the shallow-repository fetch improvements, and the ongoing debate over AI-assisted contributions.

## Key developments

### Git 3.0 timeline and Contributors' Summit outcomes
Taylor Blau shared notes from the 2026 Git Contributors' Summit covering security process, Git 3.0 timing, pluggable ODB, and protocol v2 for pushes. The consensus pointed toward releasing 2.99.1 and 3.0 together in March 2027 with only `BREAKING_CHANGES` defaults flipped. Christian Couder announced plans to attend the Gerrit User Summit and an NVIDIA hackathon on partial clones, while Luca Milanesio said he would present his Cassandra object-storage backend prototype for JGit.

### Security releases and signing trust issues
Johannes Schindelin announced Git for Windows 2.56.0(3) addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803, followed by 2.56.0(4) for CVE-2026-111497 fixing corrupt code-signatures in 32-bit MinGit. On the list, Christian Noé Ramos López reported that `valid-before` is checked against signer-controlled dates and that a missing `gpg.ssh.revocationFile` fails open; Patrick Steinhardt and Phillip Wood agreed Git should fail by default on a missing revocation file with a `:(optional)` escape hatch.

### pack-objects BUG on filtered `--unshallow` and shallow boundary handling
Daniel Zoghalchali reported that since v2.56.0, a fetch combining a tree filter with `--unshallow` aborts in `pack-objects` with "BUG: should_include_obj should only be called on existing objects" when the fetched ref has moved. Patrick Steinhardt traced this to `is_not_in_promisor_pack_obj()` calling `BUG()` on missing objects in partial clones and proposed returning 0 instead. Separately, Nakul V S posted v2 refusing to revert, cherry-pick, or rebase shallow boundary commits by default, since reverting such a commit would empty the working tree.

### `git log --follow` performance regression and fix
Patrick Steinhardt reported that commit 304812ed33 slowed `git log --stat --follow` on linux.git from about 28 seconds to 6m43s, causing an incident for GitLab.com. Miklos Vajna submitted a series fixing the regression by recording the path to follow per parent in the existing `log_tree_diff()` loop, reaching v4 by month's end. Junio C Hamano noted the followed-path clue is lost after `log_tree_diff()` flushes its output and suggested inspecting the diff queue instead.

### AI assistance policy debate
Scott Chacon posted an RFC to allow responsible AI-assisted contributions under the DCO with an `Assisted-by:` trailer, drawing objections from brian m. carlsson over legal liability and Kristoffer Haugsbakk over the human/LLM analogy. Patrick Steinhardt argued the review bottleneck is tightening as AI-generated content increases, warning that loosening the policy without solutions would make the problem worse; Junio C Hamano replied "Well articulated." The thread also covered Maciej Ciemborowicz's refs copy/rename series, which Patrick Steinhardt stopped reviewing due to apparently AI-generated content.

### repack `--dry-run` and `.keep` race fixes
Siddharth Shrimali fixed `git repack --drop-filtered --dry-run` writing new packs and deleting old ones despite the documented no-write promise, with Junio C Hamano marking the topic for `next`. Separately, qeesung posted v4 of a fix for `repack -d` losing objects when a concurrent push installs a `.keep` mid-run, passing a startup snapshot of kept packs to `pack-objects` via `--ignore-pack-keep-from=<file>`.

### `size_t` migration for large objects
Elia Pinto posted multiple series switching object and pack size handling from `unsigned long` to `size_t` to avoid truncation of objects 4GiB and larger on LLP64 platforms. The series covered `pack-objects` streamed blob writing, `object-file.c`, `fsck.c`, `commit.c`, `tag.c`, `unpack-objects`, `index-pack`, and `pack-check`, fixing corruption, dies, and false collision errors for large blobs on 32-bit `long` platforms.

### Fetch improvements for shallow repositories
Karthik Nayak's series fixes shallow-clone fetches resending objects during tag backfill by advertising already-fetched refs as haves, reaching v6 with Patrick Steinhardt's approval. Harald Nordgren submitted a patch fixing slow protocol v2 fetches from shallow repositories, where the server walked its whole history to determine shallow grafts; the fix advertises all shallow grafts when the client does not request a depth.

## In brief

**Patrick Steinhardt's reftable reflog-timezone fix** -- v2 storing signed minute offsets per the spec was marked ready for `next`.

**Grant Moyer's filter-branch state-branch fix** -- v3 restores `$from_commit:$to_commit` order in the map initialization.

**Toon Claes's packed-refs fwrite review** -- He listed sanitizations lost by `fwrite()` and proposed a separate preformatted writer function.

**Harald Nordgren's CI failure annotations** -- The series linking GitHub Actions annotations to test scripts was marked ready for `next`.

**Kazumasa Shigeta's `git stash create` options** -- v2 lets the command accept `-u`/`-a`; Junio C Hamano critiqued the commit message framing.

**Phillip Wood's MERGE_LABELS CRLF fix** -- He defended storing conflict labels in `.git/MERGE_LABELS` and fixed parsing with `strbuf_getline()`.

**Submodule merge delta-base cache bug** -- Patrick Steinhardt identified stale delta base cache entries as the cause of corruption in recursive submodule merges, and the fix was marked for `next`.

**merge-ort `--no-overwrite-ignore` bypass** -- Coy Geek reported that directory rename detection can overwrite ignored untracked files despite the guard.

**Kristoffer Haugsbakk's `format-patch --range-diff-notes`** -- The series teaching `git format-patch` to use a different notes ref for range diffs reached v5.

**Andrew Pleeter's `git var -z` series** -- v9 adds NUL-terminated output, multiple variable arguments, and new identity variables.

**"Limbo" object-format state** -- Jeff King proposed letting empty server repositories advertise a limbo state so first pushes from either hash succeed.

**Johannes Schindelin's sha1dc Rust backend** -- The series adds an optional SIMD-based SHA-1 backend, though brian m. carlsson objected to the Rust 1.87 minimum version.

**Colin Hinton's `fetch.followRemoteHEAD` lazy validation** -- v5 defers validation to `do_fetch()`, storing the raw string with a seen flag.

**Julia Evans's section 7 man page cleanup** -- v2 removes required SYNOPSIS sections from section 7 pages.

**Kristoffer Haugsbakk's trailer examples** -- v2 rephrasing `trailer.<key-alias>.cmd` examples was marked for `next`.

**Coy Geek's `repack --dry-run` bug report** -- On Git 2.56.0, `--drop-filtered --dry-run` writes new pack files and deletes existing ones despite the documented no-write promise.

**Domen Kožar's post-worktree hook** -- v3 adds a single lifecycle hook for add, move, and remove events including pruning.

**Scott Chacon's SHA-256 signing RFC** -- The proposal to sign a SHA-256 tree digest drew sharp exchange over whether the SHA-256 default is ready for Git 3.0.

**Ravi Mistry's blame default-ignore-revs** -- The series making `git blame` use `HEAD:.git-blame-ignore-revs` by default reached v6 and was approved as "Very cleanly done".

**Phillip Wood's `checkout -m` conflict labels** -- v4 caches merge config and writes labels to `.git/MERGE_LABELS`; Junio C Hamano said he would replace the previous version.

**Patrick Steinhardt's alternates series** -- The 13-patch series moving alternates into the files ODB backend was reviewed positively by Karthik Nayak.

**Kristofer Karlsson's commit-graph optimization** -- v3 of writing the commit-graph from updated refs only showed fetches dropping from 357 ms to under 10 ms.

**Karthik Nayak's packed-refs optimization** -- v4 of the `fwrite()` optimization showed ~20% faster deletions.

**Jens Röcker's push resends history bug** -- A pre-push hook repacking the object database causes pushes to resend common history.

**Maciej Ciemborowicz's refs copy/rename transactions** -- v4 routes copy and rename through ordinary transactions so hooks see the whole operation.

**Souma's history signing series** -- v5 adds completion coverage for `-S`/`--gpg-sign` on history subcommands; marked ready for `next`.

**Marc Becker's fast-import fix** -- v2 fixes a crash when the `from` command references a commit created in the same session.

**Torsten Bögershausen's read-cache fix** -- v5 stops `git status` from trusting a size change when CRLF conversion is active.

**Beat Bolli's imap-send TLS updates** -- v3 replaces deprecated SSL methods and uses OpenSSL hostname verification.

**Yoichi NAKAYAMA's worktree repair hardening** -- v3 validates cross-references so repair cannot break an unrelated `.git` file.

**Jon Simons's SubmittingPatches update** -- Documents the preference for submitting a bug fix's test in the same commit as the code change.

**Christian Couder's parse-options early-scan** -- v3 adds `early_scan_options()` and the `PARSE_OPT_EARLY` flag.

**Igor Wiedler's connectivity check fix** -- v2 negates `--alternate-refs` in deepening fetches and ignores refs of shallow alternates.

**Patrick Steinhardt's fsck fix** -- Fixes a bogus corruption report when a packed blob's size equals `core.bigFileThreshold`.

**Andy Koppe's sequencer fix** -- Fixes clobbering the HEAD ref name in worktree lookup by using `refs_resolve_refdup()`.

**Junio C Hamano's Git 2.98 cycle start** -- Flushed topics waiting in `next` down to `master` and rewound `next`.

**Venkat Ragavan's loose-ref seeking fix** -- Fixes seeking across directory boundaries in `cache_ref_iterator_seek()`; marked ready for `next`.

**Justin Tobler's ".keep" lockfile fix** -- v4 migrates lockfile creation to ODB transaction commit so concurrent processes do not delete each other's lockfiles.

**Ian Hu's sigchain race fix** -- Fixes a race where concurrent signal handlers leave a process spinning at 100% CPU.

**Olof Fairbrook's bundle fix** -- Fixes freeing uninitialized memory on bad arguments in `git bundle create`.

**Andy Koppe's root refs fix** -- v3 keeps root refs out of the loose ref cache via a temporary cache.

**Patrick Steinhardt's object storage extension** -- v4 of the `extensions.objectStorage` series was reviewed positively by Karthik Nayak.

**Taylor Blau's negotiator fix** -- v3 sets SEEN only after a commit is successfully parsed in both default and skipping negotiators.

**Mihai Cosma's status index.lock fix** -- Makes `git status` take `index.lock` only to write and fixes a `skipHash` race.

**Jun Wu's `core.indexLockTimeout`** -- v2 makes index-writing commands wait briefly for the lock instead of failing.

**Johannes Schindelin's gitsecurityboundary** -- Posted `Documentation/gitsecurityboundary.adoc` defining Git's security boundary.

**Nakul V S's shallow boundary refusal** -- v2 refuses to pick or revert shallow boundary commits by default.

**Linus Torvalds's `git describe` fix** -- Fixes distance counting with skewed commit dates by using `compare_commits_by_gen_then_commit_date`.

**Ravi Mistry's HTTP retry series** -- 4-patch series fixing retry counts and adding exponential backoff.

**Thomas Bachem's merge-ort config fix** -- Accepts `merge.renames` and friends given without a value.

**Harald Nordgren's `@{push}` resolution** -- v2 resolves to the push remote's same-named branch under `push.default=simple`.

**Jasmeet Bhatia's stash -p optimization** -- Passes prepared untracked file list to skip redundant index work, 16.7% faster.

**Jeff King's t5702 fix** -- Replaces `env`-redirect-and-grep with direct `echo` due to coreutils 9.12 quoting.