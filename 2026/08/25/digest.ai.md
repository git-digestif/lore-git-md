# Git mailing list daily digest for 2026/08/25

## Long threads

### history: add squash subcommand to fold a range

*Long thread (root 2026/06/14/19-25-39): 90 replies so far from 9 participants over 73 days; 2 messages that day.*

This is Harald Nordgren's long-running series adding `git history squash <revision-range>` to fold a range of commits into one, with v14 posted earlier updating commit message trailers and simplifying `rev-parse`/`tr` usage in the tests. Today's exchange concerns the case-sensitivity of matching hex object IDs in `fixup!`/`amend!` subjects: Phillip Wood argued that `starts_with()` is sufficient when the target comes from `git commit --fixup`, but that if uppercase hex remains accepted elsewhere, this new code should not newly reject it. Junio C Hamano countered that the point is not to add new places accepting uppercase hex, and that using `starts_with()` here supports discouraging uppercase hex.

### worktree add: improve message for ambiguous remote branch name

*Long thread (root 2026/08/08/08-21-41): 29 replies so far from 6 participants over 18 days; 2 messages that day.*

Yoichi Nakayama's series improves the error shown when `git worktree add` or `git checkout` hits a branch name matching multiple remote tracking branches, listing the matched remotes in the advice. Today he posted v8, whose new fourth commit changes `git worktree add --guess-remote` to fail when the path basename matches multiple remote-tracking branches instead of falling back to HEAD, with documentation updated accordingly; the earlier commits extract `advice_disambiguating_remotes()` in `builtin/checkout.c`, extend `unique_tracking_name()` to collect matching remote names, and make `git worktree add` die with a descriptive error. Junio C Hamano reviewed patch 4/4, suggesting the advice function could have been placed higher in the series, and concluded "Looking good."

### repo: add path keys to git repo info

*Long thread (root 2026/07/16/01-21-31): 26 replies so far from 4 participants over 41 days; 2 messages that day.*

K Jayatheerth's GSoC series adds `path.*` keys to `git repo info` exposing repository component paths. The keys `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, and `path.grafts` each support absolute and relative suffixes, while `path.git-prefix` and the newly added `path.cdup` are single suffix-less keys. Today's v5 adds `path.cdup`, giving the relative path from the current directory to the working tree root, and drops an unused-header cleanup patch. Junio C Hamano suggested creating a common helper library shared between `builtin/repo.c` and `builtin/rev-parse.c` so only a single implementation of `cdup` exists, noting the same applies to other features duplicating `rev-parse` implementations.

### hex: restrict object IDs to lowercase only (Git 3.0 RFC)

*Long thread (root 2026/07/29/23-32-09): 25 replies so far from 6 participants over 28 days; 10 messages that day.*

brian m. carlson's RFC series proposes removing the ability to use uppercase hex in object IDs in Git 3.0, since Git has always emitted lowercase and accepting uppercase leads to software assuming a unique hex form and to security problems. Today Phillip Wood asked brian to explain the security problems, specifically why `ABCDEF` is a risk when `abcdef^0` is not; brian answered with two cases: policy checks against the lowercase hex form can be bypassed by uppercase input, and regexps like `[0-9a-f]{40}` used to distinguish object IDs from refs fail to match uppercase IDs, and he said he will fix a dropped case-normalization and a comment typo in v2. Junio C Hamano's review flagged the "68-67" comment that should read "68-6f", remaining `HEX_KIND_MIXED` uses in `builtin/index-pack.c` and `notes.c` that should be `HEX_KIND_OID`, a changed `hex_out[]` API contract during the transition, and a question about the t5324 test; Elijah Newren echoed the case-normalization concern and asked about a possible dead `if (val < 0)` check.

## The day in brief

Yoichi Nakayama's v8 reroll of the ambiguous-remote-branch work adds a new fourth commit making `git worktree add --guess-remote` fail on multiple remote-tracking matches, and Junio C Hamano found it looking good. brian m. carlson's lowercase-hex RFC drew concrete security justification and detailed review fixes slated for v2. Derrick Stolee posted a v2 of the trace2 banned-die work that Elijah Newren promptly found bugs in, including a dropped character in `redact_arg()`.

## Notable threads

### trace2: stop allowing die()
Derrick Stolee posted v2 of a series keeping the low-level trace2 API from ever calling `die()`, via a new root-level `banned-die.h` included by all trace2 API sources, banning `xsnprintf`, `xstrdup`, `ALLOC_ARRAY`, `xstrfmt`, `ALLOC_GROW`, and `xcalloc` one patch at a time. Junio C Hamano endorsed the approach and asked whether it could extend to `the_hash_algo` and `the_repository`. Elijah Newren pointed out that `#define die banned(die)` should be `BANNED(die)`, questioned whether a `-1` return in patch 3/7 really avoids `die()`, flagged that patch 5/7's `redact_arg()` memcpy drops the trailing ">" of `:<REDACTED>`, and reported that patch 6/7 alone dies with "no open regions in thread 'main'". Junio C Hamano confirmed the `redact_arg()` observation, agreeing `redact_len + 1` would have worked better.

### branch: protect local upstreams from -d
Harald Nordgren posted a two-patch series making `git branch -d` protect a local branch that is the upstream of another surviving local branch, checking all requested deletions first so multi-branch deletion is order-independent, with `-D` overriding and a complete stack deletable together. Junio C Hamano objected that changing the long-established semantics of `-d` would break end-user workflows, citing the required test change from `-d` to `-D` as evidence that previously safe deletions break, and stated a configuration variable to tweak the behavior is unwelcome.

### am: record blobs of cleanly applied patches with --3way
Nikita Leshenko submitted a patch making `git am --3way` record postimage blobs of cleanly applied patches so a later patch whose preimage is that postimage can be 3-way merged instead of failing with "could not build fake ancestor". Junio C Hamano rejected the eager per-patch overhead and proposed lazily reconstructing the missing preimage inside `fall_back_threeway()` only when `build_fake_ancestor()` fails; Nikita agreed the eager approach was unacceptable but argued the surgical version needs elaborate scanning, proposing instead a simpler lazy alternative that applies all previous patches on their fake ancestors when the build fails and records a marker in `.git/rebase-apply` to avoid repeating work.

### t1401: check symbolic-ref exit codes and --quiet silence
Nikolaus Schuetz submitted a test-only patch asserting that querying a non-symbolic ref exits 128 with the "is not a symbolic ref" message and that `--quiet` exits 1 silently. Junio C Hamano objected that the patch is the kind of "casting wrong behaviour into stone by either documentation or tests" he had advised against earlier.

### packfile: recover when a multi-pack-index names a removed pack
Elijah Newren posted v2 of a series fixing objects reported missing despite being present, due to a race with geometric repacking that deletes subsumed packs while a stale multi-pack-index still names them. The reroll switches `fill_midx_entry()` to a tri-state distinguishing "absent" from "owning pack unavailable", performs the recovery re-read only on `SECOND_READ`, handles multiple objects shared across packs, and adds two patches: plugging a per-tree leak in `git mktree --batch` and fixing QUICK reader recovery while keeping QUICK performance for normal cases. He also suggested teaching geometric repacking about `git multi-pack-index expire` as a complementary mitigation.

## In brief
- Elijah Newren posted v2 of a patch adding `push.shallowExcludeBoundary` (default true) so pushing from a shallow clone does not send the whole toplevel tree, arguing `push.negotiate` defaults would not cover all cases, and said he plans to split further shallow-related work into two series.
- Vladimir Sitnikov's RFC on a configurable stash ref drew discussion: Phillip Wood noted stash messages already record the branch, and Sitnikov proposed one knob with shared, per-branch, or per-worktree scope, noting detached-HEAD worktrees group together under a per-branch filter.
- Andrew Pleeter submitted a new `git whoami` builtin showing resolved author/committer identity and signing configuration; brian m. carlson suggested machine-readable output and a real name in the headers, and Junio C Hamano suggested making the flags additive.
- René Scharfe posted four `worktree add` fixes for `worktree_basename()`, including an out-of-bounds read fix and trimming trailing slashes from derived branch names; Junio C Hamano confirmed the trailing-slash fix and asked for a commit message rewording.
- Patrick Steinhardt posted an 8-patch series moving alternate writing to object database creation time via `odb_source_create_on_disk()`, removing the `odb_source_write_alternate()` callback, and fixing a bug where referencing a worktree failed to resolve alternates.
- Patrick Steinhardt posted a 10-patch series making object database consistency checks pluggable, moving packfile, reverse index, bitmap, multi-pack-index, and loose object verification out of `builtin/fsck.c` into the respective odb sources.
- Patrick Steinhardt reviewed the reftable series avoiding stack reloads when locked, calling the new `locked` bit-field "somewhat fragile" but acceptable, and concluding the series looks good.
- Phillip Wood noted that `git commit` is still spawned when rebasing merges, so large rebases can still trigger gc mid-rebase even without conflicts.
- Kristoffer Haugsbakk clarified that `--no-notes` combined with `--range-diff-notes` yields neither patch notes nor range-diff notes.
- Junio C Hamano solicited reviewers for the v2 patch preserving `wwwauth_headers` across HTTP redirects.
