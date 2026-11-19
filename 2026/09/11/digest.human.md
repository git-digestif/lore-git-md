# Git mailing list daily digest for 2026/09/11

## Long threads

### receive-report hook for git-receive-pack

*Long thread: 57 replies so far from 7 participants over 25 days; 4 messages that day.*

Karthik Nayak's series adds a `receive-report` hook that runs after the reference transaction completes but before the status report reaches the client; the hook reads the pkt-line report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. Earlier revisions fixed a BUG() when no report was requested and a leak when reassigning command strings. Today Oswald Buddenhagen suggested trimming the verbose commit message, which Nayak agreed had grown clunky but left as-is. Junio C Hamano noted `FREE_AND_NULL()` is safe on an already-NULL pointer, so the guard in `override_cmds_error()` could be dropped, and supplied a follow-up diff plus a coccinelle rule; Nayak will skip a reroll and may send the coccinelle fixup separately.

### Convert build-time USE_NSEC to runtime core.useNanosec

*Long thread: 46 replies so far from 6 participants over 36 days; 3 messages that day.*

D. Ben Knoble's series exposes `USE_NSEC` as the runtime `core.useNanosec` config so distributed builds can use nanosecond index timestamps and avoid racy-git false positives. Today Knoble resent v7, differing from v6 only in a documentation typofix, and Junio C Hamano queued the three patches, noting they matched what he had already queued from v6 with the typo reworded locally. Knoble explained the resend was mainly a nudge about the stale What's Cooking status.

### Write object-database alternates at creation time

*Long thread: 36 replies so far from 5 participants over 18 days; 2 messages that day.*

Patrick Steinhardt's nine-patch series refactors `init_db()` into `create_repository()` and moves alternate writing into `odb_source_create_on_disk()`, removing the `odb_source_write_alternate()` callback. Today Steinhardt answered review comments on the alternates patch: the file-stream error indicator is sticky, so per-`fprintf` checks are unnecessary, and he declined to formalize backend errors for alternates since nothing would use them yet. He also acknowledged overlooking a comment on the v5 cover letter and said it needs no further changes.

### Add path keys to git repo info

*Long thread: 34 replies so far from 5 participants over 58 days; 4 messages that day.*

K Jayatheerth's GSoC series adds seven `path.*` keys to `git repo info`: toplevel, superproject-root, hooks, index, and grafts with absolute and relative suffixes, plus git-prefix and cdup as single keys. Today he posted v6, refactoring `get_superproject_working_tree()` to take a `struct repository *` instead of the process working directory. Junio C Hamano noted dead null checks after `repo_get_index_file()` in patch 4/7, asked what `path.cdup` returns outside the working tree, and suggested splitting patch 2/7 into a fix with tests plus the new keys.

### Run auto maintenance once at the end of a rebase/cherry-pick/revert sequence

*Long thread: 27 replies so far from 5 participants over 8 days; 2 messages that day.*

Thomas Bachem's series makes a sequence run `git maintenance run --auto` exactly once when done, from `builtin/rebase.c` and `builtin/revert.c`, while passing `maintenance.auto=false` to sequencer-spawned commands. Today Patrick Steinhardt objected that the proposed comment for `git_config_append_parameter()` was too specific to `-c key=value`, and disputed the claim that the sequencer lacks a single exit point, arguing cherry-pick, revert, rebase, `--continue` and `--skip` all reach `pick_commits()`; he prefers calling auto maintenance there and on abort.

## The day in brief
Junio C Hamano queued D. Ben Knoble's `core.useNanosec` runtime-config series after a typofix-only v7 resend, and decided to queue Jeff King's "ci: use system asciidoctor" patch once it was found never to have been picked up. Patrick Steinhardt posted v3 of the pluggable object-database consistency-check series, and Tyler Cipriani committed to a v4 of the `--force-if-includes` fix that generalizes the reflog check beyond HEAD.

## Notable threads

### Use the system asciidoctor in CI documentation builds
Jeff King's patch drops the pinned asciidoctor 1.5.8 gem install in `ci/install-dependencies.sh` in favor of the Ubuntu image's package. Today King explained the patch was missed when its topic branch was picked up; Tuomas Ahola corrected the view that it had been merged, showing the third patch of the series was never applied, and Junio C Hamano decided to queue it on top.

### Fix --force-if-includes consulting the wrong ref
Tyler Cipriani's series makes `git push --force-if-includes` check the reflog of the ref actually being pushed rather than a same-named local branch, fixing false rejections and potential data loss. Today Patrick Steinhardt and Junio C Hamano reviewed the patches, questioning deletion handling, the HEAD-only check, and missing test coverage; Cipriani confirmed the reflog lookup behavior, agreed to resolve symrefs to the target branch and reject unverifiable revisions in a v4, and proposed moving a guard into `apply_push_cas`.

### Make object-database consistency checks pluggable

*Long thread: 21 replies so far from 3 participants over 18 days; 8 messages that day.*

Patrick Steinhardt's ten-patch series moves fsck checks into the ODB sources, including loose-object verification in the loose source and reverse-index verification in the packed source. Today Toon Claes reviewed, suggesting each source decide on `ODB_FSCK_FULL`/`--full` itself and use early returns instead of a `ret` variable; Steinhardt posted v3 agreeing to move the check into the "files" backend, prepare repo settings, and drop the variable.

## In brief
- Jeff King posted v2 cleaning up merge-ll external driver temporaries with the tempfile API; Elijah Newren flagged an LLP64 narrowing hazard when a result exceeds LONG_MAX and proposed a fail-fast die().
- Ravi Mistry proposed making git-blame default to a `.git-blame-ignore-revs` file at the working-tree root, with opt-outs via config or `--no-ignore-revs-file`.
- Johannes Schindelin posted v2 of using Rust in Windows CI jobs, fixing the UCRT64 target to `x86_64-pc-windows-gnu`; Junio C Hamano suggested `CARGO_BUILD_TARGET` over `CARGO_TARGET`.
- Patrick Steinhardt reviewed Ariel Keselman's patch skipping the packed-refs lock for root-ref deletions, suggesting stronger test assertions.
- Patrick Steinhardt argued against retaining NULL checks in repository config getters, proposing a redesign before further iteration.
