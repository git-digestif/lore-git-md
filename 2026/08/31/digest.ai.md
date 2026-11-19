# Git mailing list daily digest for 2026/08/31

## Long threads

### git-replay(1) gains a --linearize option

*Long thread (root 2026/06/08/18-37-18): 38 replies so far from 7 participants over 85 days; 2 messages that day.*

Toon Claes posted v9 of the series teaching `git-replay(1)` to linearize merge commits. The series adds a `--linearize` option that drops merge commits and replays all reachable commits into a single linear history, mimicking `git-rebase(1)`'s `--no-rebase-merges` default; refs pointing at a merge are updated to the merge's last replayed ancestor, and the option rejects multiple branches and `--contained`. v9 rephrases "multiple revision ranges" to "multiple branches" in error messages and tests, rewords the commit message to say that without `--linearize` replaying multiple branches keeps shared history shared and divergent parts divergent, and moves the `git-rebase(1)` reference to a parenthetical in `replay.adoc`. v8 had added the multiple-branch and `--contained` restrictions following earlier reachability concerns in the thread.

### Convert build-time USE_NSEC to runtime core.useNanosec

*Long thread (root 2026/08/07/11-56-23): 32 replies so far from 5 participants over 25 days; 8 messages that day.*

D. Ben Knoble posted v6 of the `dk/use-nsec-runtime` series, converting the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration so index-related commands are less likely to hit racy-git problems on supported filesystems; the only v6 change is a comment tweak in patch 3 ("enabled" instead of "defined"). Patrick Steinhardt called that a micronit, said he was otherwise happy with the patch, and replied "Works for me" to Knoble's question whether the series might be ready for `next`. Earlier, Junio C Hamano had flagged an ordering dependency: `repo_config_values()` must work on repositories other than `the_repository` before a submodule index can be used in-process.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread (root 2026/08/18/22-34-04): 31 replies so far from 5 participants over 14 days; 2 messages that day.*

Elijah Newren's v3 series fixes objects being treated as missing despite being present due to a race with geometric repacking. Patch 4/4 makes `find_pack_entry()` recover when a stale multi-pack-index names a removed owning pack by scanning the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ`, with `midx_fill_entry()` returning a tri-state. Today Patrick Steinhardt added a +1 to his review of that patch, and Jeff King agreed with Junio C Hamano's proposal to mark it for `next`. Earlier, King had reviewed the MIDX recovery scan and the `mktree` QUICK-removal patch favorably.

### receive-report hook for git-receive-pack(1)

*Long thread (root 2026/08/18/07-55-55): 26 replies so far from 5 participants over 14 days; 5 messages that day.*

Karthik Nayak's v4 series introduces a `receive-report` hook that runs after the reference transaction but before the report is sent to the client, receiving the pkt-line encoded report on stdin with its stdout replacing the report; a non-zero exit marks all references rejected. Today Patrick Steinhardt suggested rewording the v4 1/3 commit message with a `githooks(5)`-derived paragraph, proposed renaming `generate_response()` to `generate_report()` in v4 2/3, and questioned the precedence of the new `ref_error` parameter in v4 3/3; Nayak accepted the commit-message and rename suggestions.

### trace2: stop allowing die()

*Long thread (root 2026/07/15/16-12-11): 22 replies so far from 6 participants over 48 days; 8 messages that day.*

Derrick Stolee posted v3 of the series making the low-level trace2 API die()-free, rebased onto `master`. It introduces a root-level `banned-die.h` header included by all trace2 API files, banning die()-capable helpers one at a time (`xsnprintf`, `xstrdup`, `ALLOC_ARRAY`, `xstrfmt`, `ALLOC_GROW`, `xcalloc`). The v3 changelog records fixes for earlier review findings: the `BANNED()` uppercase correction, returning 0 instead of -1 in `tr2_sysenv_cb()`, the `":<REDACTED>"` length fix with new `t0212` coverage, and the patch 6 runtime fix pulled out of patch 7. Earlier, Jeff King cautioned that the ban list is the tip of the iceberg since `strbuf` and `json-writer` also reach `die()`.

## The day in brief
Toon Claes posted v9 of the `git-replay` `--linearize` series and v3 of the `last-modified` Bloom-filter series, both queued and pushed by Junio C Hamano. D. Ben Knoble's v6 of the runtime `core.useNanosec` conversion drew a "Works for me" from Patrick Steinhardt, and Jeff King agreed to mark Elijah Newren's MIDX removed-pack recovery patch for `next`.

## Notable threads

### last-modified: pre-filter commits with the pathspec's Bloom key
Toon Claes posted v3 of the 6-patch series speeding up `git-last-modified(1)` by reusing the Bloom-filter machinery behind `git-log(1)`, adding a trace2 `bloom_queries` counter and a test asserting the count for a top-level wildcard pathspec; benchmarks on ziglang/zig showed about 1.95x speedup with `-t` and 3.74x without. Junio C Hamano reported that after merging the series to `seen` and pushing, CI failed reproducibly with test variables from `ci/run-build-and-tests.sh`, and ejected the topic from `seen`.

### fetch-pack: parallelizing packfile URI fetches
Patrick Steinhardt's series prepares `git-fetch(1)` and `git-clone(1)` to fetch packfile URIs in parallel. Justin Tobler suggested landing his own series extending the ODB transaction interface to fetch-pack first, and Steinhardt agreed to drop this series for now and resend once Tobler's series has been merged.

### worktree add: worktree_basename() fixes
Junio C Hamano marked the `rs/worktree-add-basename-fixes` topic for `next`, saying it was in good shape except for a confusing part of the log message already amended in his tree. The series fixes an out-of-bounds read in `worktree_basename()` in `builtin/worktree.c`, rejects separator-only paths, trims slashes when deriving branch names, and lets `worktree_basename()` return a string copy.

### sequencer: honor --empty when a fixup!/squash! empties its target
Phillip Wood reviewed Farid Zakaria's v4 patch, which makes `git rebase --autosquash` honor `--empty=drop` when a `fixup!` or `squash!` commit empties its target, and judged the approach sound and "looking good", with suggestions including setting `ALLOW_EMPTY` unconditionally, moving fixup-target tracking into `pick_commits()`, and adding a test for a fixup chain whose final fixup becomes empty.

### checkout: print blank line after autostash conflict advice
Harald Nordgren posted v2 of the patch making the autostash apply functions return 1 when the stash could not be applied due to conflicts, so `git checkout -m` prints a blank line between the conflict advice and the branch-switch message. Junio C Hamano objected that treating any non-zero `run_command()` result as 1 does not reliably detect conflicts, and said the patch needs more work enumerating and remapping the stash subcommands' exit values.

## In brief
- Diogo Castro's fix for negative pathspecs in `git ls-files` and `git add`: Junio C Hamano agreed to ignore negative pathspecs when computing the common prefix, strip it only from positive pathspecs, and match full paths against negative ones.
- Patrick Steinhardt posted v2 of "odb: write alternates at creation time", using a lockfile to write `info/alternates` during `odb_source_create_on_disk()` and removing the ad-hoc `write_alternates()` callback.
- Patrick Steinhardt posted v2 of the 10-patch "odb: make consistency checks pluggable" series with commit-message improvements only; Karthik Nayak reviewed it and said the changes look good. *Long thread (root 2026/08/25/14-30-02): 14 replies so far from 2 participants over 7 days; 8 messages that day.*
- Andrew Pleeter rerolled the new builtin `git ident` command as v2, renaming it from `whoami` and adding `--porcelain`, `-z`/`--null`, and composable identity and component selectors.
- Patrick Steinhardt judged Michael Montalbo's v3 lib-httpd concurrency-safety series "good enough" despite verbose AI-generated commit messages, which Junio C Hamano also criticized. *Long thread (root 2026/07/08/02-59-40): 20 replies so far from 3 participants over 55 days; 6 messages that day.*
- Patrick Steinhardt confirmed Grayson Gordon's v6 `http.sslVerifyStatus` tests already cover the fail-close case Junio C Hamano asked about. *Long thread (root 2026/08/11/17-02-00): 20 replies so far from 4 participants over 21 days; 4 messages that day.*
- Harald Nordgren posted v3 of the CI change canceling stale pull request workflow runs; Junio C Hamano agreed no cross-repo solution is needed since concurrency groups do not span repos.
- Junio C Hamano asked whether Johannes Schindelin's "mingw: rely on MSYS2's metadata" topic could be concluded soon.
- D. Ben Knoble clarified that the porcelain commands create `refs/heads/@`, which `check-ref-format --branch` currently accepts, and suggested `--branch` mode could be stricter.
- Junio C Hamano queued and pushed v9 of `tc/replay-linearize` and v3 of `tc/last-modified-bloom`.
