# Git mailing list daily digest for 2026/10/22

## Long threads

### branch: let --delete-merged find squash merged branches

*Long thread: 46 replies so far from 6 participants over 24 days; 11 messages that day.*

This series makes `git branch --delete-merged` treat branches merged via GitHub "Squash and merge" or "Rebase and merge" as merged, naming the upstream commit that landed them. Today Junio C Hamano objected to loosening the default, attaching a test showing a private branch whose contents were independently added upstream under a different author would be wrongly deleted, and asked for an explicit opt-in. Harald Nordgren accepted and proposed a separate `--delete-irrelevant` option; D. Ben Knoble countered the "extremely rare" claim with two identical "npm update" PRs. Junio then proposed a subset-superset ladder (`=merged`, `=rebased`, `=squashed`) and candidate names including "integrated"; Nordgren endorsed `--delete-merged[=<level>]` and "integrated", while Jasper Mayone noted the value slot is taken by the upstream `<pattern>` and proposed a separate `git branch --delete-integrated <pattern>`, a shape Junio said he likes.

### connected: add incremental connectivity check via rev-list

*Long thread: 33 replies so far from 3 participants over 39 days; 4 messages that day.*

This series adds an incremental connectivity check behind `transfer.connectivityCheck=incremental` or `"auto"` so the check does not slow as reachable boundary objects grow. Today Kristofer Karlsson agreed with Junio C Hamano that gating `fetch_if_missing = 0` on "not full" would trap future modes, then reported he could use `odb_read_object_info_extended()` with `OBJECT_INFO_SKIP_FETCH_OBJECT` and stop touching `repo->fetch_if_missing` in the series entirely; he verified manually in a partial clone that `rev-list --connectivity-check=incremental` reports promised blobs and trees as missing without fetching them, and plans to squash this into v5. Junio agreed the broader refactor stays outside the series.

### advice: introduce advice scoping mechanism

*Long thread: 26 replies so far from 4 participants over 44 days; 1 message that day.*

The `defaultBranchName` advice tells users to run `git config set advice.defaultBranchName false`, which writes locally where the advice already appeared; the series adds a scope hint so the advice suggests the global scope. Today Junio C Hamano asked whether to mark `jc/advice-config-set-global` for `next` and whether Vsevolod Myalitsin could clean up the `vadvise()` API to take the whole `advice_setting` as an independent effort.

### doc: move BreakingChanges to a manpage

*Long thread: 23 replies so far from 5 participants over 25 days; 3 messages that day.*

This series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage for visibility. Today Kristoffer Haugsbakk proposed promptly deleting old-version discussions, or keeping only the previous one; Patrick Steinhardt argued the document should stay at least a couple of years because upgrades often jump versions; Junio C Hamano decided to proceed with the manpage.

## The day in brief

Git for Windows 2.56.0(3) shipped as a security fix release. The `--delete-merged` thread weighed several opt-in shapes for squash- and rebase-merged branches without converging on a final design, while the incremental connectivity check was reworked to avoid touching `fetch_if_missing` entirely.

## Notable threads

### Git for Windows 2.56.0(3) security release

Johannes Schindelin announced Git for Windows 2.56.0(3), addressing the eMudhra Intermediate CA Compromise and Git LFS CVE-2026-105803 by upgrading to Git LFS v3.8.1, and bundling OpenSSH v10.6.P1 and cURL v8.23.0.

### worktree: add --reflink for copy-on-write worktree creation

brian m. carlson confirmed v3 handles macOS `cp -c` (clonefile(2)) so the REFLINK test runs there, and declined to review further AI-generated content from Jason Newton, citing the project's AI policy.

### Documentation: propose community voting process

On Emily Shaffer's proposed `CommunityVoting` process, Kristoffer Haugsbakk argued an AI-contributions vote would drift toward tech-employer positions; Patrick Steinhardt and brian m. carlson endorsed secret ballots so voters can differ from their employers; Justin Tobler argued the eligibility bar (a commit in `next` or two reviews) is too low.

## In brief

- Mihai Cosma posted a series making `git status` take `index.lock` only to write, plus a fix for a `skipHash` race that could silently revert a `git add`; benchmarks showed `git add` failures dropping from 104/200 to 0/200.
- brian m. carlson said Git will not add a command to update passwords in remote URLs, pointing to credential helpers, SSH keys, and deploy keys.
- Taylor Blau posted v2 parsing parents before queueing in `negotiator/skipping.c`; Junio C Hamano objected to marking commits SEEN before parsing, which Taylor declined as consistent with the default negotiator.
- Karthik Nayak posted v4 advertising already-fetched refs as "haves" when backfilling tags, fixing shallow and `--atomic` fetches.
- Miklos Vajna posted v3 speeding up `log --follow` across renames in non-linear history.
- Junio C Hamano posted v4 of `MyFirstContribution` reordering "respond to reviews" before "send v2" and strengthening the wording.
- Jun Wu's series adds `core.indexLockTimeout` so index-writing commands wait briefly for the lock; Mihai Cosma noted it conflicts with his status series only on one lock call.
