# Git mailing list daily digest for 2026/08/31

## Long threads

### git-replay(1) gains a --linearize option

*Long thread: 38 replies so far from 7 participants over 85 days; 2 messages that day.*

Toon Claes posted v9 of the series adding a `--linearize` option to `git-replay(1)` that drops merge commits and replays all reachable commits into a single linear history, mimicking `git-rebase(1)`'s `--no-rebase-merges` default; refs pointing at a merge are updated to its last replayed ancestor, and the option rejects multiple branches and `--contained`. Today's v9 rephrases "multiple revision ranges" as "multiple branches" in error messages and tests and simplifies the `replay.adoc` wording, after earlier reviewers pressed for exactly that wording and shorter documentation of the restriction.

### USE_NSEC becomes runtime core.useNanosec

*Long thread: 32 replies so far from 5 participants over 25 days; 8 messages that day.*

D. Ben Knoble posted v6 of the series converting the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration, so distributed builds can enable nanosecond stat data and reduce "racy git" problems in index-related commands like `git-diff`. The only v6 change is a comment tweak ("enabled" instead of "defined"); Patrick Steinhardt said he was otherwise happy with the patch and answered "Works for me" to the question of whether the series is ready for `next`. Earlier, Junio C Hamano had flagged a `repo_config_values()` ordering dependency with submodule recursion.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread: 31 replies so far from 5 participants over 14 days; 2 messages that day.*

Elijah Newren's v3 series fixes objects being reported missing despite being present when a concurrent geometric repack removes a pack that a stale multi-pack-index still names as owner. Patch 4/4 scans the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ`, with `midx_fill_entry()` returning a tri-state; patch 3/4 stops `git mktree --batch` from passing `OBJECT_INFO_QUICK`. Today Patrick Steinhardt added a +1 to the review, and Jeff King agreed with Junio C Hamano's proposal to mark the series for `next`.

### receive-report hook for git-receive-pack(1)

*Long thread: 26 replies so far from 5 participants over 14 days; 5 messages that day.*

Karthik Nayak's v4 series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client; its stdout replaces the pkt-line report, and a non-zero exit marks all references rejected. Today Patrick Steinhardt proposed reworded commit messages, a `githooks(5)`-derived documentation paragraph, and renaming `generate_response()` to `generate_report()`; Nayak accepted the messages and the rename.

### trace2 stops allowing die()

*Long thread: 22 replies so far from 6 participants over 48 days; 8 messages that day.*

Derrick Stolee posted v3 of the series making the low-level trace2 API die()-free via a root-level `banned-die.h` header that bans die()-capable helpers (`xsnprintf`, `xstrdup`, `ALLOC_ARRAY`, `xstrfmt`, `ALLOC_GROW`, `xcalloc`) one patch at a time. Today's v3 fixes the `BANNED()` macro case, returns 0 instead of -1 in `tr2_sysenv_cb()`, fixes the `":<REDACTED>"` length bug with new `t0212` coverage, and pulls a runtime fix out of patch 7.

## The day in brief

Toon Claes posted v9 of the `git-replay --linearize` series and v3 of the `last-modified` Bloom-filter speedups, while D. Ben Knoble's `core.useNanosec` runtime config reached v6 with Patrick Steinhardt's "Works for me" on readiness for `next`. Jeff King agreed to mark the MIDX removed-pack recovery patch for `next`, and Junio C Hamano marked the `worktree_basename()` fixes for `next`, though the `last-modified` series was ejected from `seen` after CI failed.

## Notable threads

### last-modified pre-filters commits with the pathspec's Bloom key
Toon Claes posted v3 of the six-patch series reusing `git-log`'s Bloom-filter machinery to speed up `git-last-modified(1)` on directories, adding a trace2 `bloom_queries` counter and test; benchmarks on ziglang/zig show about 1.95x speedup with `-t` and 3.74x without. Junio C Hamano reported that after the series was pushed to `seen`, CI failed reproducibly under the `ci/run-build-and-tests.sh` test variables and the topic was ejected from `seen`.

### fetch-pack parallelizing packfile URI fetches
Patrick Steinhardt's two-patch series prepares `git-fetch(1)` and `git-clone(1)` to fetch announced packfile URIs in parallel, which can significantly speed up fetches when a server announces many packfiles. Today he agreed with Justin Tobler to drop the series for now and resend once Tobler's ODB transaction series for fetch-pack has been merged.

### worktree add worktree_basename() fixes
The four-patch series fixes an out-of-bounds read, separator-only paths, slash trimming when deriving a branch name, and the string-copy return in `builtin/worktree.c`'s `worktree_basename()`. Junio C Hamano marked the topic for `next`, saying it was in good shape apart from a confusing log message already amended in his tree.

## In brief
- Phillip Wood reviewed Farid Zakaria's v4 making the sequencer honor `--empty` when a `fixup!`/`squash!` empties its target, calling the approach sound and "looking good", with fixup-chain dropping suggestions left for later.
- Harald Nordgren posted v2 printing a blank line after autostash conflict advice in `git-checkout(1)`; Junio C Hamano objected that a non-zero `run_command()` result does not reliably detect conflicts and said the patch needs more work.
- Patrick Steinhardt judged Michael Montalbo's v3 making the `t/lib-httpd` CGI helpers concurrency-safe "good enough"; Junio C Hamano and Steinhardt criticized the verbose AI-generated commit messages, and Montalbo committed to writing prose by hand. *Long thread: 20 replies so far from 3 participants over 55 days; 6 messages that day.*
- Patrick Steinhardt posted v2 of the series writing alternates at object-database creation time via `odb_source_create_on_disk()`, now using the lockfile API for `info/alternates` and removing the ad-hoc `write_alternates()` callback.
- Andrew Pleeter rerolled the new builtin `ident` command as v2, renaming it from `whoami` and adding `--porcelain` and `-z`/`--null` output options alongside the author/committer and name/email selectors.
