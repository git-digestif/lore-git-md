# Git mailing list daily digest for 2026/11/20

## Long threads

### fetch: limited-fetch and refmap-only remotes

*Long thread (root 2026/09/19/14-47-26): 86 replies so far from 7 participants over 63 days; 1 message that day.*

This series changes `remote.c` and the fetch machinery so that a remote can fetch only the branches that local branches track, adding `remote.<name>.fetchRefmap` as a configured default for `--refmap` and `git remote add --limited-fetch`, defaulting to limited fetch in shallow repositories. Today Kristoffer Haugsbakk described his own configuration for tracking all remote refs, including notes, under a local namespace using `fetch = +refs/*:refs/tracking/o/refs/*`, noting that negative refspecs cannot exclude branches because they cannot have a destination part; he affirmed that D. Ben Knoble's described notes configuration did not make his setup harder, and stated that he has not seen a forcing use case for simplifying Git notes, comparing collaborative commit metadata to "history of history". Earlier, Phillip Wood and Junio C Hamano had debated whether a command-line `src:dst` refspec should ignore `.fetchRefmap`, with Phillip Wood proposing a preliminary patch and Junio C Hamano restating the agreement that the refmap applies only to refspec entries lacking a destination.

### branch: --delete-integrated for squash- and rebase-merged branches

*Long thread (root 2026/09/29/07-30-30): 66 replies so far from 7 participants over 53 days; 7 messages that day.*

This series adds `git branch --delete-integrated <pattern>` to `builtin/branch.c`, which works like `--delete-merged` but also deletes a branch when some upstream commit contains all of its changes, as after a squash or rebase merge, naming that commit in the output. Today Harald Nordgren posted v9 0/4, whose new development fixes `--delete-integrated` dying with "unable to read blob object" when a path was a submodule at the fork point and both the branch and upstream replaced it with different regular files; such a branch is now kept, with a test added. Junio C Hamano reviewed the series, objecting to the helper name `branch_opted_out` in patch 1/4 as assuming "delete once merged" will remain the only destructive operation, questioning why a missing or true `branch.<name>.deleteMerged` should mean "not opted out", suggesting a message-template array for the dry-run/landed/remote messages, and asking for test coverage of `--dry-run --delete-integrated` and of opting out via `deleteMerged=false`; he concluded that the changes between v8 and v9 looked OK.

### builtin/rerere: convert to subcommands

*Long thread (root 2026/10/27/10-46-43): 44 replies so far from 6 participants over 25 days; 2 messages that day.*

This 13-patch series modernizes `builtin/rerere.c` to use `OPT_SUBCOMMAND()` parsing like other Git commands, including a documented backwards-incompatible change to option placement. Today Patrick Steinhardt said the default-subcommand pattern is acceptable only for a bare `git rerere`; once a non-default action is requested, flags should be refused in favor of an explicit alias such as `git rerere perform --rerere-autoupdate`, though he would not break the existing interface now. Junio C Hamano considered an optional explicit name for the default action acceptable, proposing `git rerere` or `git rerere [run] [--[no-]rerere-autoupdate]`, while noting uncertainty about whether `OPT_SUBCOMMAND()` supports an optional subcommand.

### doc: move BreakingChanges to a manpage

*Long thread (root 2026/09/28/10-41-24): 24 replies so far from 5 participants over 54 days; 1 message that day.*

This series transforms `Documentation/BreakingChanges.adoc` into a `gitbreaking-changes(7)` manpage for easier visibility via `git help`, leaving a pointer file behind and replacing message IDs with lore URLs. Today Kristoffer Haugsbakk asked whether there is any interest in continuing the series, noting that the topic never got included in `seen`/What's Cooking and that there is no actionable feedback on his latest round, and asking what is next.

### utf8: bounds-related fixes in utf8.c

*Long thread (root 2026/11/16/18-56-59): 22 replies so far from 3 participants over 5 days; 6 messages that day.*

This series fixes `utf8_strnwidth()` in `utf8.c` disregarding its `len` argument and corrects the function's comment, which wrongly stated it returns `strlen()`. Today Dirk Gouders posted v3 and then retracted it after squashing two commits without reflecting, then posted v4 0/5 as an un-squashed resend: an optional end pointer for `display_mode_esc_sequence_len()` so it never reads past a counted buffer's end, the corrected `utf8_strnwidth()` comment, boundary checks after skipping ANSI escape sequences, a boundary argument for `utf8_width()`, and a new `display_mode_esc_sequence_len_all()`. Andy Koppe suggested callers could pass `SIZE_MAX` to the counted variant, then corrected that `end = s + len` with `SIZE_MAX` would be undefined behavior; Junio C Hamano affirmed he liked the simplicity of that approach.

## The day in brief

Harald Nordgren's v9 of `git branch --delete-integrated` fixes a crash when a submodule path is replaced by regular files on both sides, and Junio C Hamano's review found the v8-to-v9 changes acceptable. Christian Couder rerolled the promisor-remote fix as a v2 0/8 series that refactors protocol v2 capability writing so `fetch` sends the missing `promisor-remote` reply. Andrei Șova posted v2 of a fix for `git clone --revision` failing with a valid commit hash under a differing hash algorithm.

## Notable threads

### connect: promisor-remote reply missing from fetch

Christian Couder posted v2 0/8 "Fix promisor-remote reply missing from fetch", refactoring client-side protocol v2 capability writing in `connect.c` so `write_command_and_capabilities()` is the only function used, and fixing the bug where the client omits the "promisor-remote=..." reply with `fetch` commands--failing with HTTP servers that handle each request separately and when fetching only object IDs. The series adds tests in t5710 and a new t5711 HTTP script, and documents that the reply should be sent with each `fetch` request asking for a pack.

### builtin/clone: fix --revision when remote and local hash algos differ
Andrei Șova posted v2 fixing `git clone --revision` failing under protocol v2 with "fatal: Remote revision ... not found in upstream origin" when passed a valid commit hash using a hash algorithm different from the default, by delaying refspec item initialization until after the remote repository's hash algorithm is gathered so `parse_refspec` parses the hash correctly, leaving the prefix list empty when `--revision` fetches all advertised refs, with a new test cloning with a raw SHA-256 commit hash.

### status, push: handle a push branch rebased onto a newer upstream
Aniket Gupta reported that using `--cherry-mark` makes `git status` slower when branches have diverged because every commit on both sides needs a patch id, giving examples of 3000 commits per side going from 25ms to 150ms and 10000 from 60ms to 500ms, and suggested counting without `--cherry-mark` first and only running the patch-id pass when the counts are small. Harald Nordgren acknowledged the performance regression and raised a concern that the suggested approach might produce a weird user experience if people do not understand why the check is suddenly gone.

### ls-files: reuse and update the untracked cache
Aniket Gupta reported that with `core.untrackedCache=true` and a cold cache, the patch holds `index.lock` for the whole scan, causing a concurrent `git add` to fail with "fatal: Unable to create '.git/index.lock': File exists" (reproduced with 60k untracked files), noted that master does not do this and that `cmd_status()` locks only after collecting, and asked whether the lock needs to be taken before reading.

### test-lib: run EXPENSIVE tests one at a time
Junio C Hamano identified a TOCTOU race in `acquire_expensive_lock_` where a waiter that read the previous owner's PID just before the lock was released could delete a new holder's active lock, allowing concurrent EXPENSIVE tests, and a deadlock scenario where a process killed between creating the lock directory and writing its PID leaves a stale lock with no identifiable owner, causing all waiters to block indefinitely; he stated that between the two, the deadlock is far worse.

## In brief

- Patrick Steinhardt posted v2 of "Introduce pluggable commit graphs (pt.1)", adding a commit that fixes verification of trailer hashes with commit graph chains and dropping the second half of the series.
- Patrick Steinhardt posted a six-patch "packfile: some lifecycle improvements" series, renaming lifecycle functions to `packfile_` prefixes and making `pack_max_fds` enforcement process-wide via a global `all_packfiles` list.
- Maik Riechert fixed `git status` reporting both an untracked directory and a directory below it after failed fsmonitor queries or trivial responses by invalidating ancestor directories too.
- Junio C Hamano reviewed the `clone --revision` SHA-256 fix and proposed swapping the initialization sequence; Andrei Șova replied that a circular dependency prevents that and that leaving ref prefixes uninitialized under `--revision` is acceptable.
- Okhuomon Ajayi posted v4 documenting a maximum 72-column wrap for emailed text in CodingGuidelines, SubmittingPatches, and ReviewingGuidelines; Junio C Hamano said "Will queue". *Long thread (root 2026/11/19/10-53-00): 7 replies so far from 3 participants over 2 days; 8 messages that day.*
- Abhishek Agarwal made `git submodule update --jobs=0` use `online_cpus()` instead of aborting with a BUG, and negative `--jobs` values die with "'--jobs' cannot be negative".
- Ophir Bleicher fixed `git rev-parse --output-object-format=sha1` to raise an error instead of outputting the wrong SHA-256 hash in SHA-1-compatible SHA-256 repositories.
- Andy Koppe added a `%(body:trailers=<bool>)` pretty placeholder that omits the trailer block with `trailers=no` while keeping content after the trailers.
- Junio C Hamano reviewed the compatObjectFormat v2 series, agreeing that returning an error for a misconfigured repository is appropriate and endorsing the rev-parse error-message approach.
- Phillip Wood argued the config continued-lines patch aligns section copy/rename parsing with config-reading parsing as a bug fix, but reported it mishandles carriage returns in values and subsection names.
