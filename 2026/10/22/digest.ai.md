# Git mailing list daily digest for 2026/10/22

## Long threads

### branch: detecting squash- and rebase-merged branches for --delete-merged

*Long thread (root 2026/09/29/07-30-30): 46 replies so far from 6 participants over 24 days; 11 messages that day.*

Harald Nordgren's series makes `git branch --delete-merged` treat a branch as merged when an upstream commit since the fork point contains the branch's version of every file it changed, naming that commit in the output as "landed as". Today Junio C Hamano objected that loosening the default would surprise users, attaching a test showing a private branch whose contents were independently added upstream under a different author would be wrongly deleted, and asked that `--delete-merged` keep its current behavior unless an explicit opt-in is given. Harald Nordgren accepted and proposed a separate `--delete-irrelevant` option; D. Ben Knoble countered the "extremely rare" framing with two identical "npm update" PRs rebased onto main. Junio C Hamano then proposed a subset-superset ladder (`=merged`, `=rebased`, `=squashed`) and candidate names including "integrated"; Harald Nordgren leaned toward `--delete-merged[=<level>]` with levels `match-hash`, `match-commits`, `match-content`; Jasper Mayone noted the value slot is already taken by the upstream `<pattern>` and proposed a separate `git branch --delete-integrated <pattern>`, a shape Junio C Hamano said he likes.

### connected: incremental connectivity check via rev-list

*Long thread (root 2026/09/14/09-47-54): 33 replies so far from 3 participants over 39 days; 4 messages that day.*

Kristofer Karlsson's series adds an incremental mode for the connectivity check behind `transfer.connectivityCheck=incremental` or `auto`, running the verifier inside the same rev-list subprocess that `check_connected()` spawns. Today he agreed with Junio C Hamano that gating `fetch_if_missing = 0` on "not full" would trap future modes, then found it doable to use `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT` and stop touching `repo->fetch_if_missing` in the series entirely; he verified manually in a partial clone that `rev-list --connectivity-check=incremental` without `--exclude-promisor-objects` reports promised blobs and trees as missing without fetching them, plans to squash this into v5, and asked whether the commit-log reasoning is still needed. Junio C Hamano had said using `_extended()` in the new code avoids making things worse, that the distilled reasoning belongs in the commit log, and that the broader refactor stays outside this series.

### advice: scoping the defaultBranchName squelch hint

*Long thread (root 2027/08/29/00-49-58): 26 replies so far from 4 participants over 44 days; 1 message that day.*

Vsevolod Myalitsin's series adds a configuration-scope hint to advice settings so the `defaultBranchName` advice recommends the global scope, since the local-scope command only affects the already-initialized repository while the advice reappears on the next `git init`. Earlier, Junio C Hamano had posted a simpler patch adding `--global` to the squelch message. Today he asked whether to mark `jc/advice-config-set-global` for `next` and whether Vsevolod Myalitsin could look into cleaning up the `vadvise()` API by passing the entire `advice_setting` structure as an independent effort.

### doc: move BreakingChanges to a manpage

*Long thread (root 2026/09/28/10-41-24): 23 replies so far from 5 participants over 25 days; 3 messages that day.*

Kristoffer Haugsbakk's series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage for easier visibility. Today he proposed that discussions of old versions be promptly deleted after a major release, or alternatively that only the previous breaking-change discussion be kept; Patrick Steinhardt argued the document should be kept for at least the next couple of years because upgrades often jump multiple versions; Junio C Hamano decided to proceed with the manpage.

## The day in brief
Junio C Hamano pushed back on loosening `git branch --delete-merged` by default, steering squash-merge detection toward an explicit opt-in such as `--delete-integrated`. Kristofer Karlsson reworked the incremental connectivity check to avoid touching `fetch_if_missing` at all. And brian m. carlsson declined both a password-in-URL command and AI-generated review content.

## Notable threads

### worktree: copy-on-write creation with --reflink
The series adds `git worktree add --reflink`, which populates a new worktree by reflinking files via FICLONE, clonefile(), and ReFS block cloning, with `worktree.reflink` as the default. Today brian m. carlsson confirmed v3 handles macOS `cp -c` correctly so the REFLINK test actually runs there, and stated that substantial portions of Jason Newton's earlier email appeared to be LLM output, citing the project's AI policy and declining to review or respond further to AI-generated content.

### Feature request: update password in remote URL

Jan Waclawek asked for a command that replaces only the password in a remote URL. Today brian m. carlsson said he doesn't think Git will implement or accept such a feature, noting credentials in config files are frequently exposed and that credential helpers, SSH public keys, and deploy keys are better alternatives.

### Documentation: propose community voting process
Emily Shaffer's patch outlines a formal voting process for decisions that cannot be settled by consensus, including a vote on AI contributions. Today Kristoffer Haugsbakk argued the AI vote will pass because monthly contributors are employed by tech companies; Patrick Steinhardt disputed that while agreeing the vote should be secret; Justin Tobler said the proposed eligibility bar (a commit merged to `next` or 2+ code reviews) is too low; brian m. carlsson rebutted the employer-influence claim and endorsed secret votes; Junio C Hamano thanked him for raising the secret-vote point.

### negotiator/skipping.c: parse parents before queueing
Taylor Blau's patch moves `repo_parse_commit()` from `push_parent()` into `rev_list_push()` so the skipping negotiator's priority queue uses actual commit dates rather than zero. Today he posted v2 with a test that recent ancestors are sent before old ref tips; Junio C Hamano judged the test fine but objected that v2 marks a commit SEEN before parsing, which could trigger `BUG("missing parent in priority queue")` on an unparsable commit; Taylor Blau declined the change, noting the default negotiator's `rev_list_push()` also sets SEEN before parsing.

### status: lock the index only to write it
Mihai Cosma's two-patch series makes background `git status` hold `index.lock` only to write. Today he posted it: patch 1 fixes a bug under `index.skipHash` where status could silently revert a concurrent `git add`, and patch 2 adds `repo_lock_and_update_index_if_able()` called after `wt_status_collect()`; benchmarks on a gcc checkout showed `git add` failures dropping from 104/200 to 0/200 with status running in a loop.

## In brief
- tsukasa inoue posted a two-patch series making `git switch -c` and `checkout -b` finish the switch and exit 0 when the tracking config cannot be written, matching `push -u`, `clone` and `fetch`.
- Karthik Nayak posted v4 of "fetch: advertise fetched refs when backfilling tags", adding fetched tips to `negotiation_include_tips` so shallow fetches advertise them as haves during tag backfill.
- Patrick Steinhardt said he is happy with v4 of the series fixing ".keep" lockfile ownership in ODB transactions.
- Miklos Vajna posted v3 of "log: speed up --follow following renames for non-linear history", split into four patches with added t4219 tests.
- Andy Koppe posted a five-patch series adding a "root ref" glossary entry and replacing outdated "pseudoref" uses in docs, code and tests; Junio C Hamano asked for input from Patrick Steinhardt on the pseudoref description.
- Mihai Cosma noted his status-side index-lock series complements Jun Wu's "Wait for the index lock instead of failing" v2, cutting the slowest of 200 `git add` runs from 1020 ms to 160 ms on top of it.
- Justin Tobler submitted a patch merging `loose-object-idx` files into the primary ODB index when migrating a tmp-objdir with a compatibility hash and batch fsync.
- Harald Nordgren posted v3 of the imap-send header-order fix, accepting From/Date/Subject in any order and matching header names case-insensitively per RFC 5322.
- Matt Hunter posted v2 of "init: show absolute path of (re)initialized repository", restoring the absolute path in the init message with a regression test.
- Johannes Schindelin announced Git for Windows 2.56.0(3), a security fix release addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803.
