# Git mailing list daily digest for 2026/11/04

## Long threads

### reflog: add %gb format placeholder naming each entry's branch

*Long thread (root 2026/10/15/13-13-55): 32 replies so far from 6 participants over 21 days; 1 message that day.*

Harald Nordgren posted v4 of the series that adds a `%gb` placeholder to pretty formats so `git log -g --format='%h %gd: (%gb) %gs'` names the branch `HEAD` was on when each reflog entry was recorded, following branch switches, renames and rebases and showing "detached at <commit>" for a detached `HEAD`. The reroll fixes `%gb` to name the correct branch when `git rebase <upstream> <branch>` runs with `GIT_REFLOG_ACTION` set -- the bug Phillip Wood reported after reviewing v3 -- and rewords the commit message. The first patch defines named constants in `reflog.h` for the reflog message strings and uses them across `builtin/branch.c`, `builtin/checkout.c`, `builtin/rebase.c`, `object-name.c`, `sequencer.c` and `wt-status.c` with no behavior change; the second implements `%gb` by reading those entries.

### repack: corner cases for cruft-less MIDXs

*Long thread (root 2026/09/30/01-28-34): 31 replies so far from 5 participants over 36 days; 1 message that day.*

Taylor Blau's eight-patch series fixes omissions that can make bitmap generation fail with `repack.midxMustContainCruft=false` even though all reachable objects remain available: it adds directly enumerated tree and tag roots, retains cruft for ordinary incremental repacks, and includes required packs when writing ordinary or incremental MIDXs. Earlier, Taylor had said the series should be ready to go as-is after Junio C Hamano questioned whether the topic was still viable. Today Junio re-read the patches and the existing review discussion, found himself confused by the same points Jeff King had previously called confusing, and said he has no concrete improvement ideas to offer right now.

### range-diff: add --matched-only

*Long thread (root 2026/09/11/16-41-17): 20 replies so far from 5 participants over 55 days; 3 messages that day.*

Harald Nordgren's patch adds `git range-diff --matched-only` to show only commits that correspond on both sides of the two ranges, since `--left-only` and `--right-only` cannot be combined. Earlier discussion settled on keeping the code and fixing the documentation, and Ben Knoble had asked for the motivation, which Harald first gave as scrolling past unrelated commits after a rebase and then retracted. Today Kristoffer Haugsbakk said he uses the `base..old base..new` form to keep unrelated commits out, and Ben Knoble corrected that the three-revision form `git range-diff base old new` selects the same commits without repeating the base, which Kristoffer acknowledged.

## The day in brief
Patrick Steinhardt posted a fix for loose-object existence checks failing from a stale cache, which also explains the partial-clone "not our ref" merge failures reported today. Harald Nordgren posted v4 of the reflog `%gb` series fixing the `GIT_REFLOG_ACTION` rebase case Phillip Wood found. Derrick Stolee posted fixes for a path-walk regression dropping tagged-tree blobs and an fsmonitor/untracked-cache invalidation bug.

## Notable threads

### odb/source-loose: fix existence checks for newly written objects
Patrick Steinhardt posted a two-patch series fixing a bug where `git merge` in a partial clone writes merged blobs but the loose object cache is not updated, so existence checks think the blobs are missing and a promisor remote is asked to backfill objects it does not have. The second patch adds `loose_cache_mark_seen()` to update the cache whenever a new loose object is written, plus a regression test in `t/t5616-partial-clone.sh`. Kristofer Karlsson verified the test fails without the fix, Derrick Stolee called the test good, and Junio C Hamano reviewed the patch as looking very good.

### describe: count commit distance correctly with skewed dates
Linus Torvalds posted a patch fixing `git describe` miscomputing commit distance when commit dates are unreliable, because the walk goes purely by date order past the tagged release; the fix takes commit generation numbers into account. Kristofer Karlsson proposed an alternative prototype computing exact depth without generation numbers via visited-commit counters, yielding the kernel's expected `v7.3-19059` with and without a commit-graph, and Junio C Hamano called it interesting. Linus then identified the kernel's skew as GitLab Marge-bot's `--keep-committers` rebase copying original committer dates, not clock skew.

### partial clone lazy fetch fails with "not our ref"
Patrick Ohly reported that `git merge` in a blobless partial clone fails with "not our ref" since 5946a564, breaking Kubernetes CI. Kristofer Karlsson reproduced it and traced it to the loose object cache: a newly written object in an already-listed subdirectory is wrongly reported missing, so the promisor remote is asked for an object that exists only locally. Patrick Steinhardt confirmed the root cause predates the bisected commit -- the cache is never updated on writes and the old flush only masked it -- and pointed to his fix series.

### sequencer: refuse to pick or revert shallow boundary commits
Nakul V S's v2 patch makes `revert`, `cherry-pick` and `rebase` refuse shallow boundary commits, which look like root commits and would empty or refill the working tree, with `--allow-shallow-boundary` for revert and cherry-pick. Phillip Wood affirmed the changelog, questioned the need for the override option, suggested moving the check to `shallow.c` since nothing is sequencer-specific, proposed restructuring the apply-backend check into a single revision walk, and concluded the reroll is definitely moving in the right direction.

### http: preserve wwwauth_headers across redirects
The v2 patch fixes credential helpers losing the server's WWW-Authenticate challenges when cURL follows a redirect, because `credential_from_url()` clears them on the URL update. Jeff King reviewed it as OK with small suggestions, noting the headers are response-derived and `fwrite_wwwauth()` resets the list on each new status line; Matthew John Cheetham confirmed the intended behavior and the awkwardness of carrying challenges in the credential struct. Michal Suchánek asked for clearer wording in the patch description, and Junio C Hamano said he would wait to see how Aaron Plattner responds.

## In brief
- Derrick Stolee fixed a path-walk regression from Git 2.55.0 that dropped blobs from tagged trees at already-visited paths; Taylor Blau reviewed it as LGTM.
- Derrick Stolee fixed an fsmonitor/untracked-cache bug where a failed fsmonitor response left untracked entries valid; Junio C Hamano called the blunt invalidation better than fast and incorrect and queued it.
- Tyler Cipriani posted v2 of a fix for `--force-with-lease` forwarding to remote groups in `push_multiple()`, correcting the Reported-by name and commit message wording.
- Kristofer Karlsson's quarantine alternate-refs fix drew Jeff King's proposal to skip alternates matching the commondir in `refs_from_alternate_cb`; Justin Tobler called it reasonable and Junio approved the reworded commit message.
- Farid Zakaria posted v2 of a three-patch series fixing credential-cache daemon startup races, so a second daemon no longer takes over a running daemon's socket.
- Marc Strapetz revived the send-pack series that inspects the old tip's tree to avoid resending unchanged blobs after a rebase, asking whether the default-path optimization is a reasonable tradeoff.
- Patrick Steinhardt posted a ten-patch commit-graph refactoring series passing all write options via `struct commit_graph_write_options`; Derrick Stolee gave an LGTM.
