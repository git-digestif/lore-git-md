# Git mailing list daily digest for 2026/10/17

## Long threads

### branch: let --delete-merged find squash merged branches

*Long thread (root 2026/09/29/07-30-30): 31 replies so far from 5 participants over 19 days; 1 message that day.*

Harald Nordgren's series makes `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all of its changes, naming that commit in the output as "landed as", and adds `--no-squashed` to skip the slower squash search. Branches merged on GitHub with "Squash and merge" or "Rebase and merge" have tips that are not reachable from the upstream, so they were never deleted. Earlier in the thread, Phillip Wood and Junio C Hamano pressed for a clear account of the detection tradeoffs -- patch IDs versus tree comparison, rename detection, and learning from range-diff's commit matching -- and Junio C Hamano proposed that `--delete-merged` stay a strict check while a separate `--delete-irrelevant` would remove branches made irrelevant by similar upstream work. Today Harald Nordgren argued the feature should focus on the case of one's own work being slightly mangled by a squash or merge: he would be very surprised if his branch were deleted because someone else wrote almost the same code in parallel, and he sees no reason to keep a branch once its work is picked up by the mainline, whether via squash, merge, or preserved commit hash.

## The day in brief

Jun Wu rerolled the index-lock waiting series as v2 with `core.indexLockTimeout` and write-race fixes in `commit` and `reset`, Harald Nordgren posted a protocol v2 fix for slow fetches from shallow repositories, and Yoichi NAKAYAMA sent v3 of the worktree repair hardening.

## Notable threads

### Wait for the index lock instead of failing

Jun Wu posted v2 of a four-patch series that lets index-writing commands wait briefly for the index lock instead of failing immediately when another process (often a periodic `git status` from an IDE) holds it. The series adds `core.indexLockTimeout` (default 1000 ms; 0 disables retry, -1 waits indefinitely), adds `repo_try_hold_locked_index()` for optional-lock users such as `status` and `describe --dirty`, and fixes write races in `commit` and `reset` (except `--soft`) by reordering from read-lock-write to lock-read-write so those commands can use the new wait behavior.

### upload-pack: fix slow fetches from a shallow repository

Harald Nordgren submitted a patch/RFC for `upload-pack` fixing fetches from a shallow repository with a long history hanging over protocol v2 -- in a homebrew-core clone with 900k commits, fetching one new commit took 7 minutes versus about a second over v0. The patch makes the server advertise all shallow grafts when the client does not request a depth, as v0 does, instead of walking the full history; this also makes `git clone --reject-shallow` refuse a shallow repository over v2 in every case and lets `git fetch --unshallow` visit each commit only once.

### worktree repair: avoid breaking unrelated .git file and gitdir

Yoichi NAKAYAMA posted v3 of a five-patch series hardening `git worktree repair`, which previously did not sufficiently validate cross-references between a linked working tree and its administrative data and could modify the wrong `.git` file or gitdir. V3 makes the separator check in `get_worktree_id()` more lenient: the final patch validates the worktree ID in `repair_gitfile()` to report "unrelated .git file" and skip repair on mismatch, stops `repair_worktree_at_path()` when the gitdir path cannot be inferred, and rewrites `get_worktree_id()` to skip trailing and repeated separators and accept both `/` and `\` before "worktrees", with Yoichi NAKAYAMA committing to handle duplicated separators on Windows.

### imap-send: prepare for OpenSSL 4.0

Beat Bolli posted v3 of a three-patch `imap-send` series: replacing the deprecated `SSLv23_method()` with `TLS_method()`, dropping the custom hostname verification functions in favor of OpenSSL's `SSL_set1_host()` and `SSL_get_verify_result()` (gaining support for IP addresses in subject alternative names, as suggested by brian m. carlsson), and adding compatibility defines so the code calls `SSL_set1_dnsname()` on OpenSSL 4.0+ and falls back to `SSL_set1_host()` earlier.

### refs/files: keep root refs out of loose ref cache

Andy Koppe submitted v2 fixing the files backend so root refs such as `HEAD` and `ORIG_HEAD` are kept out of the loose ref cache and instead read into a separate temporary cache overlaid only when an iterator requests `REFS_FOR_EACH_INCLUDE_ROOT_REFS`. Previously, once the cache was populated by an earlier iterator, root refs were silently omitted or leaked into later iterators -- for example `git for-each-ref --include-root-refs --merged=main` omitted merged root refs. The patch also corrects the documentation, help text, and `refs.h` comment for `--include-root-refs`, which had wrongly claimed it lists pseudorefs.

## In brief

- Gingrspacecadet reported that `git apply --check` crashes with "Assertion `patch->is_new <= 0' failed" at apply.c:3856 when checking a patch that creates a new file, on git 2.54.0.
- DonQuinleone proposed an RFC extending `git branch --delete-merged` to delete gone-upstream branches only when their tip is reachable from an explicitly named ref, leaning toward a `--into <ref>` modifier and asking whether "selection is not proof" is an acceptable basis.
- Andy Koppe posted v2 fixing `strbuf_add_wrapped_text()` losing one column of indentation after a hard line break in `%w()`, with a new test, updated t4205 expectations, and dedicated `%w` documentation.
- Junio C Hamano reported that the new test in t/t1901-repo-structure.sh is broken with SHA-256 and in Git 3.0 because it hardcodes 40-hex-digit SHA-1 constants, suggesting shared test code or a SHA1 prerequisite.
- **`@` branch bug**: `git branch @` creates branch `@` despite `check-ref-format` rejecting it. Haugsbakk doubts removing `@` as the `HEAD` synonym would happen; Hamano agrees, citing inertia.

- Kristoffer Haugsbakk corrected his earlier claim that `git diff-tree` lacks NUL support for pretty printing, and suggested a `--use-terminator` option for commands using tree-log.c so `git diff-tree` could serve piecemeal consumers like `git format-rev --stdin-mode=revs`.
- Simon Richter suggested `off_t` instead of `size_t` for the byte position in `fwrite_sha1_file`; Junio C Hamano argued `size_t` is correct because the position offsets into a contiguous in-core buffer.
- Harald Nordgren stated that with `branch.autoSetupMerge=inherit`, tracking should occur only when the start point is a remote branch, and called proposed `inherit-or-*` synonyms overkill.
- Ammaar Bakshi submitted a patch replacing `N_()` with `_()` in the `die()` for the "combined diff formats" error in `builtin/difftool.c`, so the already-translated message is actually shown.
- brian m. carlson endorsed updating the GitHub CI macOS jobs to macos-15, preferring it over macos-26 to avoid newer features older versions lack.
- Oswald Buddenhagen questioned why debug messages in `describe_commit()` are internationalized; Junio C Hamano noted others had raised the same point.
- Siddharth Shrimali agreed with Junio C Hamano's nitpick that the v3 `repack --dry-run` commit message redundantly restated the bug's history.
