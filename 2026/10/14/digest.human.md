# Git mailing list daily digest for 2026/10/14

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 64 replies so far from 6 participants over 21 days; 3 messages that day.*

Julia Evans's six-patch series adds a new `gitmergeconflicts(7)` man page explaining merge conflicts with examples, linked from the git-merge, git-rebase, git-revert, git-cherry-pick and git-pull docs. Today she accepted Junio C Hamano's alternative diff3 example showing the correct resolution, agreed to replace "added" with "has" in the fruit example and to fix missing full stops, and confirmed the third bullet now covers multi-step operations like `rebase`. She also tried consistent bullet punctuation, agreed to spell out `--patch`, proposed deleting the `zdiff3` note, and declined Patrick Steinhardt's suggestion to add a sentence about always verifying merge results. Earlier, Junio had reviewed v3 and Patrick had raised wording nits.

### rerere lock wait and `credentialStore.lockTimeout`

*Long thread: 36 replies so far from 5 participants over 43 days; 4 messages that day.*

Thomas Bachem's series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` and adds a hidden `--skip-locked` to `git rerere gc` for auto maintenance. Today Patrick Steinhardt agreed to mark the v8 topic ready for `next`. Separately, Okhuomon Ajayi volunteered for the leftover task of giving `credentialStore.lockTimeoutMS` a synonym without the MS suffix, and submitted an Outreachy patch adding `credentialStore.lockTimeout` as the preferred setting with `lockTimeoutMS` kept as a deprecated alias.

### CI housekeeping and modernizations

*Long thread: 31 replies so far from 6 participants over 7 days; 21 messages that day.*

Patrick Steinhardt's eight-patch series fixes CI issues and moves EOL Docker images to supported ones. Today he posted v3, adopting "linux-feature-testbed" for the renamed job after Junio C Hamano suggested it. Jeff King questioned whether the t5004 big-file test would run once SHA-256 becomes default and suggested giving it its own SHA-1 repo, and whether the new linux-reftable-sha256 job adds coverage; Patrick answered that SHA-256 is a breaking change but reftables is not, so the job does add coverage, and that Debian installs `locales-all`. Junio noted the v3 cover letter mislabels its changelog and backported the series for `maint` CI.

### Limited fetch for new remotes in shallow repositories

*Long thread: 31 replies so far from 4 participants over 26 days; 1 message that day.*

Harald Nordgren's five-patch series avoids fetching every branch of a new remote in a shallow repo. Today he posted v9, moving the `do_fetch()` if/else split into the refmap-inference commit. The series adds `remote.<name>.refmap`, a `--limited-fetch` option to `git remote add`, and defaults to it in shallow repositories. Earlier reviews had shaped the helper extraction and refmap inference.

### odb/files alternates cleanups

*Long thread: 14 replies so far from 3 participants over 2 days; 10 messages that day.*

Patrick Steinhardt's six-patch v2 series cleans up the files object database backend after alternates migrated into it, adding a patch so corrupt objects are not re-fetched from a promisor remote. Today Johannes Schindelin noted patch 1/6 fixes the same performance bug Microsoft Git patched independently and asked for Tyrie Vella's regression test; Patrick agreed to fold it in with a `Helped-by` trailer. Justin Tobler accepted the explanations and agreed the corrupt-object change is good.

## The day in brief

Patrick Steinhardt agreed to mark the rerere lock-wait topic ready for `next`, while Okhuomon Ajayi added a `credentialStore.lockTimeout` synonym. Jon Simons posted a series cutting push refspec-matching time against a 100k-ref server from seconds to ~0.15s. Julia Evans's merge-conflicts documentation series absorbed final review fixes, and several small correctness patches in fsck and the sequencer were queued.

## Notable threads

### Faster client-side refspec matching in push
Jon Simons posted v2 of a 12-patch series speeding up client-side refspec matching in `git push`, cutting a 100-delete push against 100k advertised refs from ~2.5s/~6.6s to ~0.15s/~0.19s by converting linear ref traversals in remote.c to strmap lookups; v2 also plugs a send-pack memory leak.

### `git log --follow` performance regression
Patrick Steinhardt reported that `git log --stat --follow` on linux.git takes ~8 minutes with Git 2.56 versus ~33 seconds with 2.55, bisected to 304812ed33. Today Miklos Vajna said the slowdown is partly expected because the new design finds more commits in non-linear history, and proposed keeping the path-storing design while avoiding unneeded work.

### stash creation index loads

Phillip Wood posted v2 of a seven-patch series avoiding unnecessary index loads and refreshes in `git stash push`/`create`, cutting lstat() calls per changed path by eight for push and four for create. Junio Hamano noted a commit-message mix-up and asked for clearer problem/solution wording in patch 5.

## In brief
- Yoichi NAKAYAMA posted v2 of a five-patch series making `git worktree repair` validate cross-references so it cannot break an unrelated `.git` file or gitdir.
- Igor Wiedler posted v2 fixing the connectivity check with alternates in deepening fetches by negating `--alternate-refs` in both cases and ignoring refs of shallow/grafted alternates.
- Patrick Steinhardt fixed a bogus fsck corruption report when a packed blob's size equals `core.bigFileThreshold`; Junio C Hamano queued it.
- Andy Koppe fixed the sequencer clobbering the HEAD ref name in worktree lookup by using `refs_resolve_refdup()`; queued.
- **fast-import: 'from' by oid v4**: Marc Becker posted v4 allowing 'from' to reference new commits/tags by oid; Junio said "Will queue."
