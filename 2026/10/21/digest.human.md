# Git mailing list daily digest for 2026/10/21

## Long threads

### branch: detect squash- and rebase-merged branches in `--delete-merged`

*Long thread: 35 replies so far from 6 participants over 23 days; 3 messages that day.*

Harald Nordgren posted v3 of a series that changes `git branch --delete-merged` so branches merged via GitHub "Squash and merge" or "Rebase and merge" -- whose tips are unreachable from upstream -- are treated as merged. By default a branch now counts as merged when some upstream commit since the fork point has the branch's version of every file it changed, comparing object ids only; an opt-in `--squashed` flag also catches cases where upstream changed the same files, by merging the branch's version into upstream's and counting it landed when upstream is left unchanged. Jasper Mayone tested v3 on generated repositories and found t3200 passes, but the earlier false positive remains: a never-merged branch is deleted because another author made the same change in an unrelated commit, with no option to disable that in v3's default check. Harald Nordgren replied that the case is an extremely rare edge case and the content is already upstream, and rejected a single opt-in plus config-variable design as too much UX mental load.

### connected: incremental connectivity check via rev-list

*Long thread: 29 replies so far from 3 participants over 38 days; 2 messages that day.*

Kristofer Karlsson posted v4 of a series adding an incremental mode to the connectivity check, gated behind `transfer.connectivityCheck=incremental` or `"auto"`, to keep the check from slowing as the number of reachable objects from the boundary grows. v4 adds an internal `--connectivity-check=` rev-list option, an "auto" mode that uses the incremental check up to a configurable cutoff (default 1000 incoming commits), and fallback to the full check for unknown config values, clones, pushes into empty repositories, and deepening fetches; the cover letter cites up to 190x speedup for one commit against a 3.4 GB boundary closure. Junio C Hamano reviewed patch 5/6, questioning whether assuming all non-full modes want to avoid lazy fetching is a trap for future developers and whether `verify_blob` needs explicit flag bits rather than relying on `repo->fetch_if_missing`.

### doc: new beginner Git tutorial

*Long thread: 21 replies so far from 5 participants over 16 days; 2 messages that day.*

Julia Evans's series replaces the old `gittutorial` with a new 739-line `Documentation/gittutorial.adoc` walking beginners from an empty repo through local commits to pushing to a Git host. Today's exchange concerned heading style: Junio C Hamano clarified that he finds mistakes easier to correct with the Atx-style prefix format and has no strong preference, while Julia Evans said she finds the prefix format easier for consistency, but writing `DESCRIPTION` as `== DESCRIPTION` makes `make lint-docs` report the required `DESCRIPTION` section missing, so she prefers keeping the usual Setext heading format rather than mixing formats or updating the linter.

### doc: move BreakingChanges to a manpage

*Long thread: 20 replies so far from 5 participants over 24 days; 3 messages that day.*

Kristoffer Haugsbakk's series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage so the content is reachable via `git help`. Today Junio C Hamano asked what should happen when Git 3.0 arrives -- whether every section would have to be rewritten -- and later questioned whether keeping such a historical document is worth it, asking whether anyone still cares about the Git 1.x-to-2.0 transition issues. Kristoffer Haugsbakk proposed a trailing section for "what happened in 3.0", introduced with text noting the changes are written in the future tense for readers coming from version 2.98, so the rest of the document would not need to change.

### worktree: copy-on-write creation and ephemeral branches

*Long thread: 11 replies so far from 5 participants over 139 days; 9 messages that day.*

Jason Newton posted v3 of a series adding `git worktree add --reflink`, which populates a new worktree by reflinking the source's files and index on copy-on-write filesystems (disabled on Windows because of a ReFS zero-fill bug), plus reaping of the ephemeral branch auto-created by `worktree add` when the worktree is removed. Eric Sunshine objected that the ephemeral-branch patch adds complexity for a use case already covered by `--detach` and would have to be opt-in; Jason Newton argued a detached HEAD leaves status, push, rename and upstream with nothing to name while the worktree exists. Phillip Wood said that reply fell short of the standard of constructive discussion expected on the list.

## The day in brief

v3 of the `--delete-merged` squash-detection series and v4 of the incremental connectivity check landed on the list today, alongside a new beginner tutorial replacing `gittutorial` and a report that a partial-clone merge can fail by requesting a locally created blob from the promisor remote.

## Notable threads

### fetch: commit references before backfilling tags
Karthik Nayak's patch commits the batched ref updates before backfilling tags, so shallow fetches can advertise newly fetched refs as haves instead of making the server resend objects the client already has. Patrick Steinhardt endorsed Justin Tobler's alternative of supplementing the backfill "haves" with the uncommitted refs in the pending transaction, calling it a much better fix if feasible; Karthik Nayak was wary of advertising uncommitted refs since they could fail, while Justin Tobler argued the required objects are already present and `.keep` lockfiles are expected to remain until the transaction commits anyway.

### gpg-interface: fail when the SSH revocation file is missing
Christian Noé Ramos López posted a patch making signature verification fail with an error when `gpg.ssh.revocationFile` names a missing file -- previously Git warned and verified without it -- with a `:(optional)` prefix to skip the check, plus tests, and a companion doc patch on `gpg.ssh.allowedSignersFile` key lifetimes. Junio C Hamano called most of the doc text "empty fluff" and proposed succinct wording, and on the code patch objected that placing the missing-file check in the `else if` block leaves known keys with no signature info on that failure, wondering whether it should fold into the earlier failure block.

### object-file: fix ".keep" lockfile ownership in ODB transactions
Justin Tobler posted v4 of a series fixing a bug where a `git-receive-pack` process deletes a concurrent process's `.keep` lockfile for an identical packfile, leaving the newly written pack vulnerable to housekeeping. The reroll corrects the `write_pack_lockfile()` error-variable handling per Patrick Steinhardt's suggestion; Junio C Hamano had marked the topic "Expecting a reroll".

## In brief
- Junio C Hamano reviewed Phillip Wood's v3 stash series removing redundant index loads and refreshes in `git stash push`/`create`, called it "nicely split and described", and said "Will queue."
- Patrick Steinhardt posted v2 of the `extensions.objectStorage` series, currently supporting only the "files" backend whose payload can relocate `.git/objects`; Karthik Nayak reviewed it and is okay with the series as is.
- Andrew Parmet reported that in a partial clone a merge can fail with "not our ref" because it requests a locally created blob from the promisor remote, with a minimal reproducer script.
- Masaya Kazama reported that since 6e95b07e5f a killed detached maintenance child leaves a stale `objects/maintenance.lock` that makes every later `git maintenance run --auto` return silently; in one large partial clone packs grew to 13.9 GB before a manual run.
- Junio C Hamano marked the refs/files fix keeping root refs out of the loose ref cache for `next` after Patrick Steinhardt's review approval.
- Phillip Wood reviewed Erik Cervin-Edin's v3 allowing `-m`/`-F`/`-c`/`-C` with all `--fixup` variations, found it good, and asked for tests that the editor is or is not invoked.
- tsukasa inoue posted v2 making `git switch -c` and `checkout -b` finish the switch and exit 0 when the tracking config cannot be written; Junio C Hamano said "OK" to the hint fix and called leaving the fully atomic switch for a separate topic sensible.
- Patrick Steinhardt's series restores reloading alternates when an object lookup is retried; Christian Couder noted patch 2/4 reads alternates from the quarantine temporary directory and suggested always reloading the repository's real alternates.
