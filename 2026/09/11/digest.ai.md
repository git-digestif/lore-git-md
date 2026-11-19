# Git mailing list daily digest for 2026/09/11

## Long threads

### receive-report hook for git-receive-pack

*Long thread (root 2026/08/18/07-55-55): 57 replies so far from 7 participants over 25 days; 4 messages that day.*

Karthik Nayak's series adds a `receive-report` hook that runs after the reference transaction completes but before the status report reaches the client: it reads the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references rejected. Today Oswald Buddenhagen suggested trimming the hook patch's commit message to what is not redundant with the documentation, and Karthik Nayak agreed it had grown clunky but said he will leave it as is while keeping the feedback in mind. Junio C Hamano noted that `FREE_AND_NULL()` is safe on an already-NULL pointer, so the NULL guard in `override_cmds_error()` could be dropped, and supplied a follow-up diff plus a new coccinelle rule to apply later; Karthik Nayak said he will avoid a re-roll for it and offered to send the coccinelle fixup separately if Junio C Hamano is okay with that.

### Convert build-time USE_NSEC into runtime core.useNanosec

*Long thread (root 2026/08/07/11-56-23): 46 replies so far from 6 participants over 36 days; 3 messages that day.*

D. Ben Knoble's series turns the build-time `USE_NSEC` knob into the runtime `core.useNanosec` configuration so distributed builds can use nanosecond index timestamps and avoid racy-git problems on supported filesystems. Today he resent [PATCH v7 0/3], which differs from v6 only in a documentation typofix ("comparisions" to "comparisons"), and Junio C Hamano queued the three patches, noting they matched what he had already queued from v6 after locally rewording the typo. Ben Knoble explained the resend was primarily a nudge about the topic's stale What's Cooking status.

### odb: write alternates at creation time

*Long thread (root 2026/08/25/14-11-49): 36 replies so far from 5 participants over 18 days; 2 messages that day.*

Patrick Steinhardt's nine-patch series writes alternates when an object database source is created, instead of via a `write_alternates()` callback that is never used after creation and is about to become an implementation detail of the backend. Today he answered v4 review comments on the patch adding alternates support to `odb_source_create_on_disk()`: the stream error indicator is sticky, so checking it on every `fprintf` is unnecessary, and he declined to formalize backend errors for alternates handling since nothing would use them yet. He also acknowledged overlooking a comment on the v5 cover letter and replied that it requires no further changes.

### git repo info path keys

*Long thread (root 2026/07/16/01-21-31): 34 replies so far from 5 participants over 58 days; 4 messages that day.*

K Jayatheerth's GSoC series adds seven `path.*` keys to `git repo info`: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, and `path.grafts` each with `.absolute` and `.relative` suffixes, plus single keys `path.git-prefix` and `path.cdup`. Today he posted v6, whose main change refactors `get_superproject_working_tree()` to take a `struct repository *` parameter instead of relying on `xgetcwd()`. Junio C Hamano noted that the null checks after `repo_get_index_file()` in patch 4/7 are dead code because that function `BUG()`s on an unready repository, asked what `path.cdup` returns when the current directory is outside the working tree, and suggested splitting patch 2/7 into a fix with tests in `t1500-rev-parse.sh` plus a separate patch adding the `path.superproject-root` keys with tests in `t1900-repo-info.sh`.

### sequencer: run auto maintenance once at the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 27 replies so far from 5 participants over 8 days; 2 messages that day.*

Thomas Bachem's series makes rebase, cherry-pick and revert run `git maintenance run --auto` exactly once when a sequence completes, and passes `maintenance.auto=false` to the commit, merge and exec commands the sequencer spawns so background maintenance cannot race the sequence for `MERGE_RR.lock` or delete open packs. Today Patrick Steinhardt objected that the `git_config_append_parameter()` comment is needlessly specific to `-c key=value`, proposing instead to describe appending a config option to a buffer exportable through `GIT_CONFIG_PARAMETERS`. He also disputed the claim that the sequencer lacks a single exit point: cherry-pick, revert, rebase, `--continue` and `--skip` all reach `pick_commits()` with abort as the exception, so he prefers calling auto maintenance when `pick_commits()` finishes and when aborting.

## The day in brief
Junio C Hamano queued D. Ben Knoble's `core.useNanosec` runtime-config series and decided to queue Jeff King's missed "ci: use system asciidoctor" patch on top of its topic branch. Patrick Steinhardt posted v3 of the pluggable object-database consistency-check series, while review continued on the receive-report hook, the `--force-if-includes` fix, and the sequencer auto-maintenance series.

## Notable threads

### ci: use system asciidoctor
Jeff King's patch replaces the pinned Asciidoctor 1.5.8 gem install in the CI Documentation job with the system package, since the build fixes for Asciidoctor 2 landed years ago. Today Jeff King explained the patch was missed when the `jk/ci-use-system-asciidoctor` topic was picked up, possibly because a lead-in patch got duplicated. Junio C Hamano initially concluded the patch had been merged, but Tuomas Ahola corrected that the third patch of the series appears never to have been picked up, and Junio C Hamano decided: "Let's queue it on top."

### push: fix --force-if-includes consulting the wrong ref
Tyler Cipriani's series fixes `--force-if-includes` checking the reflog of the same-named local branch instead of the ref actually being pushed, which caused false rejections for differently-named branches and detached HEAD and could allow data loss. Today Patrick Steinhardt and Junio C Hamano reviewed patch 1/2, questioning the "up-to-date branch" wording, the blind allowance of deletions, and the HEAD-only special case. Tyler Cipriani confirmed the reflog lookup behavior and agreed to a v4 that clarifies the wording and deletions rationale, adds `RESOLVE_REF_READING`, generalizes the check by resolving symrefs to a local branch, moves the `peer_ref` guard into `apply_push_cas()`, and drops a null guard that never ran in coverage.

### merge-ll: clean up external merge driver temporaries
Jeff King's three-patch series makes `merge-ll.c` use the tempfile API for external merge driver files so they are cleaned up after SIGINT and regular termination, reads the result back via strbuf, and catches `close()` errors. Today Michal Koutný confirmed the proposed tempfile variant works, and Jeff King posted v2 with the constness warning fixed and `close_tempfile_gently()` in place of `close()`. Elijah Newren spotted an LLP64 narrowing hazard where a result larger than `LONG_MAX` would be stored as a negative `long`, proposing a fail-fast `die()`; Junio C Hamano called the find sensible.

### odb: make consistency checks pluggable

*Long thread (root 2026/08/25/14-30-02): 21 replies so far from 3 participants over 18 days; 8 messages that day.*

Patrick Steinhardt's ten-patch series moves `git fsck` consistency checks into the object database sources, including loose-object verification into the loose source and reverse-index verification into the packed source. Today Toon Claes suggested each source decide itself whether to run based on `ODB_FSCK_FULL`/`--full`, early returns instead of a `ret` variable, and asked about `ODB_FSCK_VERBOSE` versus `ODB_FSCK_PROGRESS`. Patrick Steinhardt agreed to move the `ODB_FSCK_FULL || local` check into the "files" backend, to use early returns, and to call `prepare_repo_settings()` rather than `BUG()`, disagreed that the verbose and progress flags are mutually exclusive, and posted v3 with those changes.

### environment: clean up repository config handling
Tian Yuchen's three-patch cleanup of `environment.c`/`environment.h` drops redundant NULL checks in the config getters and clarifies documentation, with no functional changes intended. Today Patrick Steinhardt said he is not convinced retaining the NULL checks is correct, since they silently fall back to default values instead of global- or system-level configuration when the repository is uninitialized. He proposed redesigning the getters to accept an optional repository, `BUG()` on an uninitialized one, read that repository's configuration automatically, and fall back to a global variable when no repository is passed, saying the subsystem needs that redesign before further iteration can continue.

## In brief
- Harald Nordgren posted v2 of `git range-diff --matched-only`, which shows only commits with a counterpart in the other range; the docs no longer claim it equals `--left-only` and `--right-only` together, and the conflict check now uses `die_for_incompatible_opt3()`.
- Ravi Mistry proposed making `git blame` and `git annotate` use a regular `.git-blame-ignore-revs` file at the working-tree root by default in non-bare repositories, with `blame.ignoreRevsFile=""`, `--ignore-revs-file ""`, or `--no-ignore-revs-file` disabling it.
- Johannes Schindelin posted v2 of "Use Rust in the Windows CI jobs", fixing the UCRT64 case to `x86_64-pc-windows-gnu` since UCRT64 uses GCC; James Le Cuirot advised against setting an explicit Cargo target when not cross-compiling.
- Jeff King's patch bumps the CI `debian-11` job to `debian-12` after Debian 11 left LTS; he analyzed and rejected tracking the `oldstable` Docker tag, and Patrick Steinhardt and Junio C Hamano agreed to keep the numeric-version scheme.
- Patrick Steinhardt posted v3 of the 13-patch "odb: stop registering in-memory sources" series, improving one commit message, with two trivial merge conflicts remaining.
- Jiang Xin kicked off Git 2.56.0 translations after v2.56.0-rc0, with 163 catalog entries to translate before the update window closes on Sat, 26 Sep 2026.
- Patrick Steinhardt reviewed the `history` commit-signing series, suggesting commit-message trims and noting the new option documentation matches `git-rebase(1)`.
