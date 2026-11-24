# Git mailing list daily digest for 2026/11/20

## Long threads

### fetch: limited-fetch refmap for shallow clones

*Long thread: 86 replies so far from 7 participants over 63 days; 1 message that day.*

This series adds `remote.<name>.fetchRefmap` and `git remote add --limited-fetch` so that a shallow single-branch clone can fetch only the branches local branches track instead of every branch of a newly added remote. Today Kristoffer Haugsbakk described his own configuration for tracking all remote refs (including notes) under a local namespace using `fetch = +refs/*:refs/tracking/o/refs/*`, noting negative refspecs cannot exclude branches because they cannot have a destination part. He affirmed that D. Ben Knoble's described notes configuration did not make his setup harder, and stated he has not seen a forcing use case for simplifying Git notes, calling them adequate for single-author or automatic-merge workflows.

### branch: --delete-integrated for squash- and rebase-merged branches

*Long thread: 66 replies so far from 7 participants over 53 days; 7 messages that day.*

This series adds `git branch --delete-integrated <pattern>`, which works like `--delete-merged` but also deletes a branch when some upstream commit contains all of its changes, naming that commit in the output. Today Harald Nordgren posted v9, whose new development fixes `--delete-integrated` dying with "unable to read blob object" when a path was a submodule at the fork point and both sides replaced it with different regular files; such a branch is now kept, with a test. Junio C Hamano reviewed the series, objecting to the helper name `branch_opted_out` as assuming delete-once-merged stays the only destructive operation and proposing an operation-specific name or a generalized key, questioning the default of treating an undeclared branch as not opted out, asking for test coverage of `--dry-run --delete-integrated` and opting out via `deleteMerged=false`, and saying the changes between v8 and v9 looked OK.

### builtin/rerere: convert to subcommands

*Long thread: 44 replies so far from 6 participants over 25 days; 2 messages that day.*

This 13-patch series modernizes `git-rerere(1)` to use `OPT_SUBCOMMAND()` parsing like other Git commands, including a documented backwards-incompatible change to `--skip-locked` placement. Today Patrick Steinhardt said the default-subcommand pattern is acceptable only for bare `git rerere`, and that once a non-default action is requested flags should be refused in favor of an explicit alias, though he would not break the existing interface now. Junio C Hamano considered an optional explicit name for the default action acceptable, proposing `git rerere` or `git rerere [run] [--[no-]rerere-autoupdate]`, while noting uncertainty about whether `OPT_SUBCOMMAND()` supports an optional subcommand.

### doc: move BreakingChanges to a manpage

*Long thread: 24 replies so far from 5 participants over 54 days; 1 message that day.*

This series moves `Documentation/BreakingChanges.adoc` to a `gitbreaking-changes(7)` manpage for visibility via `git help`, replacing message IDs with lore URLs and adding a note that the document is living. Today Kristoffer Haugsbakk asked whether there is any interest in continuing the series, noting it never got included in `seen`/What's Cooking and that there is no actionable feedback on his latest round.

### utf8: bounds-related fixes in utf8.c

*Long thread: 22 replies so far from 3 participants over 5 days; 6 messages that day.*

This series fixes `utf8_strnwidth()` disregarding its `len` argument, corrects its comment, and adds boundary handling around ANSI escape sequence skipping and `utf8_width()`. Today Dirk Gouders posted v3, retracted it after squashing two commits without reflecting, then posted v4 0/5 as an un-squashed resend splitting the boundary fix and a new `display_mode_esc_sequence_len_all()`. Andy Koppe suggested callers could pass `SIZE_MAX` to sidestep length checks, and Junio C Hamano affirmed that approach; Koppe then corrected that `end = s + len` with `SIZE_MAX` would be undefined behavior, so callers must pass a valid length.

## The day in brief
Harald Nordgren's `--delete-integrated` series reached v9 with a fix for a crash on submodule-to-file replacements, while Christian Couder rerolled the promisor-remote fetch fix as an eight-patch refactoring of protocol v2 capability writing. Several threads also surfaced concrete problems: a cold untracked cache holding `index.lock` during scans, and `--cherry-mark` slowing `git status` on diverged branches.

## Notable threads

### connect: promisor-remote reply missing from fetch
This series fixes a bug where the client does not send a `promisor-remote=...` reply with protocol v2 `fetch` commands, which fails with HTTP servers that handle each request separately. Today Christian Couder posted v2 0/8, refactoring client-side capability writing so `write_command_and_capabilities()` is the only function used across `ls-refs`, `bundle-uri`, and `fetch`, with the final patches ensuring fetch sends the reply and documenting it in `gitprotocol-v2.adoc`. He noted the refactoring grew from one patch in v1 to six in v2.

### ls-files: reuse and update the untracked cache
This series lets `git ls-files` reuse the untracked cache populated by `git status` and write pending cache updates back to the index. Today Aniket Gupta reported that with `core.untrackedCache=true` and a cold cache, the patch holds `index.lock` for the whole scan, so a concurrent `git add` fails with "Unable to create '.git/index.lock': File exists", and asked whether the lock needs to be taken before reading.

### status, push: handle a push branch rebased onto a newer upstream
This series makes `git status` and `git push` recognize when a push branch was rebased cleanly onto a newer upstream and suggest `git push --force-with-lease`. Today Aniket Gupta reported that `--cherry-mark` makes `git status` slower on diverged branches because every commit on both sides needs a patch id, and suggested counting without it first. Harald Nordgren acknowledged the regression and raised a concern that the suggested approach might produce a confusing experience if users do not understand why the check disappears.

## In brief
- Andrei Șova fixed `git clone --revision` failing under protocol v2 when passed a valid commit hash in a different hash algorithm, by delaying refspec initialization until after the remote's hash algorithm is known.
- **Maik Riechert proposed invalidating parents of stale cached directories**: After failed fsmonitor queries or trivial responses, `git status` can report both an untracked directory and a directory below it; the patch invalidates ancestor directories.

- **Submodule --jobs handling patch**: Abhishek Agarwal proposed making `git submodule update --jobs=0` use online_cpus() instead of aborting, and rejecting negative values with a clear error.

- Junio C Hamano identified a TOCTOU race and a deadlock scenario in the proposed EXPENSIVE-test lock in test-lib, calling the deadlock far worse.
- Ophir Bleicher made `git rev-parse --output-object-format=sha1` raise an error instead of printing a wrong hash in SHA-1-compatible SHA-256 repositories.
- Patrick Steinhardt posted v2 of the pluggable commit-graph refactoring, adding a commit that verifies trailer hashes across commit graph chains.
- Patrick Steinhardt posted a six-patch packfile lifecycle series, making `pack_max_fds` enforcement process-wide and skipping `do_not_close` packs when closing.
