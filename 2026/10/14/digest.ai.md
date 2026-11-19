# Git mailing list daily digest for 2026/10/14

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread (root 2026/09/24/14-44-15): 64 replies so far from 6 participants over 21 days; 3 messages that day.*

Julia Evans's six-patch series introduces a new `gitmergeconflicts(7)` man page that explains merge conflicts with examples, and links to it from the git-merge, git-rebase, git-revert, git-cherry-pick, and git-pull documentation instead of repeating the guidance in each command. Today Julia Evans agreed to fix missing full stops in an enumeration, update the "ways to handle a conflict" bullet to cover multi-step operations like `rebase`, replace "added" with "has" in the fruit example, and adopt Junio C Hamano's alternative diff3 example showing the correct resolution. She also made bullet punctuation more consistent, agreed to spell out `--patch` instead of `-p`, proposed deleting the `zdiff3` note because she could not write a compelling example, and declined Patrick Steinhardt's suggestion to add a sentence about always verifying merge results. Junio C Hamano responded that `apply -3` is itself a 3-way merge among the working tree state, the patch preimage, and the patch postimage.

### Rerere lock waiting and credential lock timeout

*Long thread (root 2026/09/02/08-31-37): 36 replies so far from 5 participants over 43 days; 4 messages that day.*

Patrick Steinhardt agreed to mark Thomas Bachem's v8 series "rerere: wait for MERGE_RR.lock, but not in auto maintenance" ready for `next`; earlier, the series had been reworked so `setup_rerere()` waits up to `rerere.lockTimeout` milliseconds for the lock and auto-maintenance passes a hidden `--skip-locked` to `git rerere gc`. Separately, Okhuomon Ajayi submitted an Outreachy patch adding `credentialStore.lockTimeout` as the preferred setting for credential lock retries in `builtin/credential-store.c`, retaining `lockTimeoutMS` as a deprecated alias, after volunteering for the leftover task of giving the old name a synonym without the MS suffix.

### CI housekeeping and modernizations

*Long thread (root 2026/10/08/10-01-18): 31 replies so far from 6 participants over 7 days; 21 messages that day.*

Patrick Steinhardt's eight-patch series fixes CI issues and moves jobs off EOL Docker images. Today he posted v3, adopting "linux-feature-testbed" for the renamed job after earlier discussion of names. Jeff King reviewed several points: whether the t5004 big-file test would still run once SHA-256 becomes the default (suggesting it create its own SHA-1 repo), a loose `*breaking-changes*)` pattern for the CI script, and whether the new linux-reftable-sha256 job adds coverage. Patrick Steinhardt answered that SHA-256 is part of breaking changes but reftables is not, so the new job does add coverage, and agreed to punt on pulling the ref storage format from build options. Junio C Hamano noted the v3 cover letter mislabels its changelog and backported the series onto v2.56.0 for `maint` CI.

### Limited fetch for new remotes in shallow repositories

*Long thread (root 2026/09/19/14-47-26): 31 replies so far from 4 participants over 26 days; 1 message that day.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository. Today he posted v9 of the five-patch series, moving the `do_fetch()` if/else-if/else split into the refmap-inference commit. The series adds a per-remote `remote.<name>.refmap` config variable, extracts a `collect_upstream_from_remote()` helper in remote.c, makes a refspec-less fetch infer what to fetch from local branches whose `@{upstream}` is on that remote, adds a `--limited-fetch` option to `git remote add`, and defaults to it when the repository is already shallow.

### Object database files-backend cleanups for alternates

*Long thread (root 2026/10/13/07-36-24): 14 replies so far from 3 participants over 2 days; 10 messages that day.*

Patrick Steinhardt's series contains cleanups and improvements to the object database "files" backend now possible after migrating alternates into it. Today he posted v2, adding a fifth patch so corrupt objects are not re-fetched from a promisor remote, and agreed to fold Tyrie Vella's regression test into the first patch with a `Helped-by` trailer. Johannes Schindelin noted that patch fixes the same performance bug Microsoft Git had patched independently. Patrick Steinhardt also answered Justin Tobler's review questions about the `OBJECT_INFO_QUICK` fallback, the `local` flag, and `ODB_FOR_EACH_OBJECT_LOCAL_ONLY`, and Justin Tobler accepted the explanations.

## The day in brief

Patrick Steinhardt agreed to mark the rerere lock-wait series ready for `next`, and Jon Simons posted a series cutting push refspec-matching time for 100 deletes against 100k advertised refs from seconds to ~0.15s. Documentation work also advanced, with Julia Evans accepting final review points on the new `gitmergeconflicts(7)` page, while Junio C Hamano said he will queue several small fixes.

## Notable threads

### Faster client-side refspec matching in push
Jon Simons posted v2 of a 12-patch series speeding up client-side refspec matching in push: pushing 100 delete refspecs to a server advertising 100k refs drops from ~2.5s from an empty client and ~6.6s from a mirror to ~0.15s-0.19s. The series reuses `match_parse_rule()` in `refname_match()`, converts linear ref traversals in remote.c to strmap lookups, plugs a send-pack memory leak, and adds tests and a p5516 benchmark.

### `core.indexLockTimeout` for waiting on the index lock
Jun Wu's two-patch series adds `core.indexLockTimeout` so commands like `add` can wait for the index lock instead of failing, motivated by IDEs running `git status` periodically in the background. Today Jun Wu explained why `--no-optional-locks status` slows over time, Jeff King identified a pre-existing race in `commit` where the index is read before the lock is taken, and Jun Wu answered Junio C Hamano's questions about the read-before-lock skip rule and the default timeout.

### Fewer index loads in stash creation

Phillip Wood posted v2 of a seven-patch series removing redundant index loads and refreshes in `git stash push` and `git stash create`, cutting lstat() calls per changed path by eight and four respectively. Junio C Hamano reviewed, noting a commit message likely meant `do_push_stash()` rather than `do_stash_push()`, and asked for clearer commit-message structure in one patch.

### Safer `git worktree repair`
Yoichi NAKAYAMA posted v2 of a five-patch series making `git worktree repair` validate the cross-references between a linked working tree and its administrative data before repairing them, so it no longer modifies an unrelated `.git` file or gitdir. The patches align `.git` file handling with `repair_gitfile()`, centralize inferred backlink selection, and reduce `.git` file reads. Junio C Hamano reviewed the cover letter as a sensible progression of smaller pieces.

### Connectivity check with alternates in deepening fetches
Igor Wiedler posted v2 fixing the connectivity check when alternates are present: `--alternate-refs` is now negated in both deepening and non-deepening fetches so the alternate's refs act as a boundary, and `odb_for_each_alternate_ref()` ignores refs of shallow and grafted alternates. This fixes a check that took 227s with a 7M-object alternate after a fetch receiving an empty pack.

## In brief
- Marc Becker posted v4 letting `fast-import`'s `from` reference new commits and tags by OID; Junio C Hamano said he will queue it.
- **pack-check threshold fix**: Patrick Steinhardt proposed fixing a bogus `git-fsck` report when a packed blob equals `core.bigFileThreshold`; Junio C Hamano said "Will queue."

- Andy Koppe fixed `add_decorations_to_list()` in sequencer.c clobbering the HEAD branch name when another branch on the same commit is checked out in a different worktree; Junio C Hamano will queue it.
- Johannes Schindelin posted a five-patch series addressing Coverity overflow reports in archive-tar.c, http.c, wrapper.c, git-compat-util.h, and date.c.
- Miklos Vajna discussed the `git log --follow` performance regression, saying the slowdown is partly expected from the new rename-following design.
- Matthew Suozzo bounded an object record's offset count by its remaining length in `reftable_obj_record_decode()`.
- Patrick Steinhardt reviewed v5 of the `repo structure` revision-filtering patch and is happy aside from a few nits.
- Julia Evans agreed to Junio C Hamano's fixes for the detached-HEAD documentation rewrite.
- Jeff King confirmed `make test-greplint` is slow and proposed an early-return optimization in t/greplint.pl.
- Andy Koppe fixed root refs being omitted from `git for-each-ref --include-root-refs --merged=main` in the files backend.
- Andy Koppe fixed lost indentation after a newline in the `%w()` pretty format.
- Jeff King gave the "hostname:" includeIf series a cursory read and saw nothing to complain about.
