# Git mailing list daily digest for 2026/08/25

## Long threads

### history: add squash subcommand to fold a range

*Long thread: 90 replies so far from 9 participants over 73 days; 2 messages that day.*

Harald Nordgren's series adds `git history squash <revision-range>` to fold a range of commits, split into patches for parsing, range validation, branch protection, commit creation, and message editing. Earlier revisions reworked range validation to reject roots and multiple tips while preserving parents when squashing into a merge. Today Phillip Wood argued that `starts_with()` is sufficient when the target comes from `git commit --fixup`, but if uppercase hex remains accepted elsewhere this code should not newly reject it; Junio C Hamano replied that the point is not to add new places accepting uppercase hex, and using `starts_with()` here supports discouraging it.

### worktree add: improve message for ambiguous remote branch name

*Long thread: 29 replies so far from 6 participants over 18 days; 2 messages that day.*

Yoichi Nakayama's series improves the error and advice when a branch name matches multiple remote tracking branches in `git worktree add` and `git checkout`, listing the matched remotes. Earlier revisions split the work into extracting an advice function and improving each command's message, and Junio C Hamano had said the multiple-match case under `--guess-remote` should eventually error out. Today Yoichi Nakayama posted v8, adding a fourth commit making `git worktree add --guess-remote` fail when the path basename matches multiple remote-tracking branches instead of falling back to HEAD. Junio C Hamano reviewed it, suggested the advice function could have been placed higher in the series, and concluded "Looking good."

### add more path keys to git repo info

*Long thread: 26 replies so far from 4 participants over 41 days; 2 messages that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info` exposing the working tree root, superproject root, hooks directory, index, grafts file, git prefix, and cdup, with absolute and relative suffixes. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, and Junio C Hamano had argued that if `path.git-prefix` is offered, `cdup` should be too. Today K Jayatheerth posted v5 adding the `path.cdup` key for the relative path from the current directory to the working tree root. Junio C Hamano suggested a common helper library shared between `builtin/repo.c` and `builtin/rev-parse.c` so only one implementation of cdup exists.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 25 replies so far from 6 participants over 28 days; 10 messages that day.*

brian m. carlson's RFC series proposes removing uppercase hex from object IDs in Git 3.0, since Git has always emitted lowercase and accepting uppercase breaks software assuming a unique hex form. Earlier discussion covered the robustness principle and parallel histories from uppercase parent lines, and Junio C Hamano queued the series while inviting views. Today Phillip Wood asked brian m. carlson to explain the security problems behind rejecting uppercase hex; brian m. carlson gave two cases -- policy checks against the lowercase form can be bypassed by uppercase input, and regexps like `[0-9a-f]{40}` fail to match uppercase IDs -- and said he will fix a dropped case-normalization and a comment typo in v2. Junio C Hamano and Elijah Newren also flagged remaining `HEX_KIND_MIXED` uses and test questions in individual patches.

## The day in brief
Yoichi Nakayama's v8 makes `git worktree add --guess-remote` error on multiple remote matches, and K Jayatheerth's v5 adds `path.cdup` to `git repo info`. brian m. carlson defended restricting object IDs to lowercase hex with concrete security cases, while Derrick Stolee's trace2 `banned-die.h` reroll drew review flags on several patches.

## Notable threads

### trace2: stop allowing die()
Derrick Stolee's v2 introduces a root-level `banned-die.h` included by all trace2 API files, banning die()-capable helpers one at a time so the low-level trace2 API never causes a process-ending failure that could retrigger tracing during die() handling. Junio C Hamano endorsed the approach and asked whether it could extend to `the_hash_algo` and `the_repository`; Elijah Newren flagged a `BANNED(die)` macro mismatch, a `redact_arg()` memcpy dropping the trailing ">", and a bad split between patches 6/7 that dies with "no open regions" when checked out alone. Junio C Hamano confirmed the `redact_arg()` observation.

### packfile: recover when a multi-pack-index names a removed pack
Elijah Newren's v2 fixes objects treated as missing despite being present, due to a race with geometric repacking that deletes packs a multi-pack-index still names. It adopts Jeff King's suggestions: `fill_midx_entry()` returns a tri-state distinguishing "absent from the MIDX" from "present but owning pack unavailable", and the recovery re-read happens only on `SECOND_READ`. The series also plugs a per-tree leak in `git mktree --batch` and fixes QUICK reader recovery. Elijah Newren also suggested teaching geometric repacking about `git multi-pack-index expire` as a complementary mitigation.

### send-pack: avoid sending the whole tree when pushing from a shallow clone
Elijah Newren's v2 adds `push.shallowExcludeBoundary` (default true) so pushing from a shallow clone excludes reachable shallow grafts from the pack instead of sending the entire toplevel tree, which for large repositories could be gigabytes instead of kilobytes. He argued that defaulting `push.negotiate` would not fix all cases -- rebased pull requests, caches pushed to different repos, explicit `push.negotiate=false`, and protocol v2 unavailable over ssh -- and plans to split further shallow work into two series. Tests cover the default optimization, the opt-out, and rejection of rootless snapshots by `receive.shallowUpdate` receivers.

## In brief
- Harald Nordgren proposed protecting local branches from `git branch -d` when a surviving branch depends on them through a local upstream chain; Junio C Hamano objected that changing the long-established `-d` semantics would break workflows and that a configuration variable is unwelcome.
- Patrick Steinhardt posted an 8-patch series writing alternates at object database creation time via `odb_source_create_on_disk()` instead of `odb_source_write_alternate()`, fixing a worktree-alternates bug and removing the unused callback.
- Nikita Leshenko proposed making `git am --3way` record postimage blobs of cleanly applied patches; Junio C Hamano rejected the eager per-patch overhead in favor of lazy reconstruction, and Nikita Leshenko proposed a simpler lazy alternative applying all previous patches when `build_fake_ancestor()` fails.
