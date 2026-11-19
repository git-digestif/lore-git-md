# Git mailing list daily digest for 2026/10/17

## Long threads

### branch: let --delete-merged find squash merged branches

*Long thread: 31 replies so far from 5 participants over 19 days; 1 message that day.*

This series changes `git branch --delete-merged` so that branches merged on GitHub with "Squash and merge" or "Rebase and merge" are treated as merged when some upstream commit contains all of their changes, naming that commit in the output as "landed as". The v2 reroll diffs each upstream commit once and shares the result between all branches, fixes a branch merged with a merge commit being misreported when another branch was searched first, and adds `--no-squashed` to skip the squash search. Earlier, Phillip Wood and Junio C Hamano pressed for a clear account of the detection tradeoffs, and Junio proposed that `--delete-merged` stay a strict reachability check while a separate `--delete-irrelevant` would remove branches made irrelevant by similar upstream work. Today Harald Nordgren argued the feature should focus on the case of his own work being slightly mangled by a squash or merge: he would be very surprised if his branch were deleted because someone else wrote almost the same code in parallel, and he sees no reason to keep a branch once it is picked up by the mainline, regardless of whether it arrived via squash, merge, or preserved commit hash.

## The day in brief

Jun Wu rerolled the series that makes index-writing commands wait briefly for the index lock via a new `core.indexLockTimeout`, Yoichi NAKAYAMA posted v3 of the worktree-repair hardening series, and Harald Nordgren submitted a patch fixing slow protocol v2 fetches from shallow repositories.

## Notable threads

### upload-pack: fix slow fetches from a shallow repository

Harald Nordgren submitted a patch/RFC fixing fetches from a shallow repository hanging over protocol v2: in a clone with 900k commits, fetching one new commit took 7 minutes over v2 versus about a second over v0, because the server walks its whole history to work out which shallow commits to advertise. The patch makes `upload-pack` advertise all shallow grafts when the client does not request a depth, as v0 does; this also makes `git clone --reject-shallow` refuse a shallow repository over v2 in every case and lets `git fetch --unshallow` visit each commit only once.

### Wait for the index lock instead of failing

Jun Wu posted [PATCH v2 0/4], adding `core.indexLockTimeout` (default 1s; 0 disables retry, -1 waits indefinitely) so commands like `git add`, `git checkout` and `git commit` wait briefly for the index lock instead of failing immediately when another process, often a periodic `git status` from an IDE, holds it. The series adds `repo_try_hold_locked_index()` for optional-lock users such as `git status`, and fixes write races in `commit` and `reset` (except `--soft`) by reordering from "read -> lock -> write" to "lock -> read -> write" so they qualify for the wait behavior.

### worktree repair: avoid breaking unrelated .git file and gitdir

Yoichi NAKAYAMA posted [PATCH v3 0/5] hardening `git worktree repair` in `worktree.c`, which previously did not sufficiently validate the cross-references between a linked working tree and its administrative data and could modify the wrong `.git` file or gitdir. V3 makes the separator check in `get_worktree_id()` more lenient; the final patch validates the worktree ID in `repair_gitfile()` to report "unrelated .git file" and skip repair on mismatch, stops `repair_worktree_at_path()` when the gitdir path cannot be inferred, and rewrites `get_worktree_id()` to skip trailing and repeated separators and accept both `/` and `\` before "worktrees".

## In brief

- Andy Koppe's [PATCH v2] keeps root refs such as `HEAD` and `ORIG_HEAD` out of the files backend's loose ref cache, reading them into a temporary cache only when `REFS_FOR_EACH_INCLUDE_ROOT_REFS` is requested, fixing `git for-each-ref --include-root-refs --merged=main` silently omitting merged root refs; it also corrects the `--include-root-refs` documentation, which had wrongly claimed pseudorefs are listed.
- **Beat Bolli posted [PATCH v3] imap-send TLS updates**: Patches 1-2 replace deprecated SSLv23_method() with TLS_method() and use OpenSSL's built-in hostname verification (since 1.1.0); patch 3 adds defines for OpenSSL 4.0's SSL_set1_dnsname().

- Gingrspacecadet reported that `git apply --check` crashes with "Assertion `patch->is_new <= 0' failed" at `apply.c:3856` when checking a patch that creates a new file, on git 2.54.0.
- On the report that `git checkout -b @`, `git switch -c @` and `git branch @` create a branch named `@` despite `check-ref-format` rejecting it, Kristoffer Haugsbakk said deprecating `@` as a `HEAD` synonym would not affect him, and Junio C Hamano agreed such removal would not happen given Git's inertia around long-standing features.
- Junio C Hamano reported that the new test in `t/t1901-repo-structure.sh` is broken with SHA-256 and in Git 3.0 because it hardcodes SHA-1-specific 40-hex-digit constants, suggesting either shared test code across hash algorithms or a SHA1 prerequisite.
- DonQuinleone proposed an RFC extending `git branch --delete-merged` to delete gone-upstream branches only when their tip is reachable from an explicitly named ref, presenting a `--into <ref>` modifier and composable `--upstream-gone`/`--merged=` filters, and asking which shape the list prefers.
- Andy Koppe posted [PATCH v2] fixing `strbuf_add_wrapped_text()` in `utf8.c` losing one column of indentation after a hard line break in `%w()`, with a new test and dedicated `%w` documentation.
- Kristoffer Haugsbakk corrected his earlier claim that `git diff-tree` lacks NUL support for pretty printing, and suggested a `--use-terminator` option for commands using `log-tree.c` so `git diff-tree` could serve piecemeal consumers like `git format-rev --stdin-mode=revs`.
