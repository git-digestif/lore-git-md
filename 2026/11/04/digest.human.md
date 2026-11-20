# Git mailing list daily digest for 2026/11/04

## Long threads

### reflog: add %gb to name each entry's branch

*Long thread: 32 replies so far from 6 participants over 21 days; 1 message that day.*

Harald Nordgren posted v4 of the series adding a `%gb` placeholder to pretty formats for reflog entries. The reflog of HEAD mixes entries from every branch, so `%gb` names the branch HEAD was on when each entry was recorded, showing "detached at <commit>" when detached. V4 fixes `%gb` for `git rebase <upstream> <branch>` with `GIT_REFLOG_ACTION` set -- a case Phillip Wood had reported showing wrong names -- and rewords the commit message. The first patch defines named constants in `reflog.h` for reflog message strings, used across `builtin/branch.c`, `builtin/checkout.c`, `builtin/rebase.c`, `object-name.c`, `sequencer.c`, and `wt-status.c` with no behavior change. Earlier, Phillip Wood had reviewed the placeholder as "Looks good".

### repack: corner cases for cruft-less MIDXs

*Long thread: 31 replies so far from 5 participants over 36 days; 1 message that day.*

With `repack.midxMustContainCruft=false`, geometric repacks can omit cruft packs from the MIDX while relying on the indexed set being closed under reachability; Taylor Blau's series adds tree and tag roots, retains cruft for ordinary incremental repacks, and includes required packs in ordinary and incremental MIDX writes, fixing cases where bitmap generation could fail. Today Junio C Hamano re-read the patches and their review discussion, found himself confused by the same points Jeff King had flagged, and said he has no concrete improvement ideas right now. Earlier, Taylor had said the series should be ready as-is after Junio questioned the topic's viability, a question Junio now clarified was not meant for the author to answer.

### range-diff: add --matched-only

*Long thread: 20 replies so far from 5 participants over 55 days; 3 messages that day.*

Harald Nordgren's patch adds `git range-diff --matched-only` to show only commits that correspond on both sides, since `--left-only` and `--right-only` cannot be combined. Today Kristoffer Haugsbakk said he uses the `base..old base..new` form so unrelated commits never appear, and Ben Knoble corrected that the three-revision form `git range-diff base old new` selects the same commits without repeating the base, which Kristoffer acknowledged. Earlier, reviewers debated whether running `left_only` and `right_only` together makes sense, with several suggesting better range selection addresses the motivating case.

## The day in brief
Patrick Steinhardt posted a two-patch fix for a stale loose-object cache that made `git merge` in blobless partial clones try to lazy-fetch blobs it had just written locally, breaking Kubernetes CI. Derrick Stolee fixed an fsmonitor/untracked-cache bug that could hide new untracked files, and Harald Nordgren posted v4 of the reflog `%gb` placeholder series.

## Notable threads

### partial clone: lazy fetch fails with "not our ref"
Patrick Ohly reported that `git merge` in a blobless partial clone fails with "not our ref" since 5946a564, breaking Kubernetes CI. Kristofer Karlsson reproduced it and traced it to the loose object cache: writing a loose object does not update the cache, so once a subdirectory is listed, existence checks wrongly report newly written objects missing, triggering a promisor fetch for a locally present blob. Patrick Steinhardt confirmed the root cause predates the bisected commit -- 5946a564cd stopped flushing the cache on misses, which had masked the bug -- and sent a fix series.

### odb/source-loose: fix stale-cache existence checks
Patrick Steinhardt posted a two-patch series fixing the stale loose-object cache behind the partial-clone failure: the first extracts subdirectory bitmap retrieval into `subdir_seen_bitmap()`, the second adds `loose_cache_mark_seen()` to update the cache on each loose write, plus a regression test in `t/t5616-partial-clone.sh`. Kristofer Karlsson verified the test fails without the fix; Derrick Stolee and Junio C Hamano also reviewed it as looking good.

### fsmonitor: invalidate untracked cache after losing history
Derrick Stolee posted a patch for an fsmonitor/untracked-cache bug present since 2017: when fsmonitor fails or returns a trivial response, untracked cache entries could stay marked valid, so `git status` could miss new untracked files. The fix invalidates every untracked cache directory and sets UNTRACKED_CHANGED on a failed token request. Junio C Hamano called it "rather a blunt instrument" but said being without acceleration beats being fast and incorrect, and queued it; he also asked whether it could replace an unreviewed pending series on the same topic.

## In brief
- Derrick Stolee fixed a path-walk regression from Git 2.55.0 that dropped blobs from tagged trees at already-visited paths; Taylor Blau reviewed it as LGTM.
- **describe distance fix**: Kristofer Karlsson proposed an exact-depth prototype without generation numbers; Linus Torvalds blamed GitLab Marge-bot's --keep-committers, not clock skew, for the kernel case.

- Jeff King reviewed the v2 patch preserving WWW-Authenticate headers across HTTP redirects as OK with small suggestions.
- Phillip Wood called the v2 sequencer patch refusing shallow boundary commits "definitely moving in the right direction" while questioning the `--allow-shallow-boundary` option.
- Jeff King proposed skipping alternates matching the commondir in `refs_from_alternate_cb` for Kristofer Karlsson's quarantine alternate-refs fix; Justin Tobler called it reasonable.
- Marc Strapetz revived the send-pack series inspecting the old tip's tree to avoid resending unchanged blobs after a rebase.
- b.murauer reported a `git log --follow` regression bisected to 304812ed33; Patrick Steinhardt noted patches are in review.
