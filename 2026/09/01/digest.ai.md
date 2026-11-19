# Git mailing list daily digest for 2026/09/01

## Long threads

### git-replay(1) `--linearize` option

*Long thread (root 2026/06/08/18-37-18): 39 replies so far from 7 participants over 86 days; 1 message that day.*

Toon Claes's series adds a `--linearize` option to `git-replay(1)` that drops merge commits and replays all reachable commits into a single linear history, mimicking `git-rebase(1)`'s default `--no-rebase-merges` behavior; it updates refs pointing to merges to the last replayed ancestor and rejects use with multiple branches or `--contained`. Earlier, Justin Tobler and Elijah Newren pressed for "multiple branches" wording instead of "multiple revision ranges" and simpler documentation of the restriction, and Claes agreed to those rewordings in v9. Today Elijah Newren reviewed the v9 series, stated "I think this version is good to go," and added a Reviewed-by trailer.

### runtime `core.useNanosec` configuration

*Long thread (root 2026/08/07/11-56-23): 38 replies so far from 6 participants over 26 days; 6 messages that day.*

D. Ben Knoble's series converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration setting, so distributed builds can enable nanosecond racy-git checks in `read-cache.c` and `statinfo.c` instead of relying on `#ifdef USE_NSEC`. Earlier reviewers settled the `#ifdef` strategy, the absence of a warning on `NO_NSEC` builds, and Patrick Steinhardt's `defined`->`enabled` comment nit, which landed in v6. Today Junio C Hamano flagged a "comparisions" typo in the v6 documentation, Jeff King reviewed the `#ifndef NO_NSEC` guard and found the rest as expected, and Knoble confirmed the typo was amended locally; Hamano agreed to amend locally and said a final reroll can be skipped if nothing else surfaces.

### packfile recovery when a multi-pack-index names a removed pack

*Long thread (root 2026/08/18/22-34-04): 36 replies so far from 5 participants over 15 days; 5 messages that day.*

Elijah Newren's series fixes objects being treated as missing despite being present when a stale multi-pack-index names a pack removed by a concurrent geometric repack; `find_pack_entry()` recovers by scanning the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ`, with `midx_fill_entry()` returning a tri-state enum. Earlier revisions dropped the QUICK-reader patch in favor of a small `git mktree --batch` change that stops passing `OBJECT_INFO_QUICK`. Today Derrick Stolee completed an inline review of v3 4/4 with "Good patch. LGTM." and said he supports the series; Newren clarified that the fallback scan is O(num_packs_in_the_midx), and Stolee accepted both clarifications.

### `receive-report` hook for git-receive-pack(1)

*Long thread (root 2026/08/18/07-55-55): 28 replies so far from 5 participants over 15 days; 3 messages that day.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client, receiving the pkt-line encoded report on stdin with its stdout replacing the report; a non-zero exit marks all references rejected. Earlier reviews renamed the hook from "report", moved report generation into `generate_report()`, and shaped the failure handling. Today Nayak posted v5, renaming `generate_response` to `generate_report`, overriding the cmd's error_strings, and adding documentation and a test for overriding the unpack status to fail the push. Junio C Hamano questioned the `bool detailed_report` parameter in 2/3, suggesting a protocol-version enum, and challenged the rationale for rejecting a reference-transaction hook, warning that divergent reported versus actual ref updates would desynchronize remote-tracking branches.

### trace2: stop allowing die()

*Long thread (root 2026/07/15/16-12-11): 25 replies so far from 6 participants over 49 days; 3 messages that day.*

Derrick Stolee's series makes the low-level trace2 API safe from `die()` by introducing a root-level `banned-die.h` header included by all trace2 API `*.c` files, banning die-capable helpers one at a time (`xsnprintf`, `xstrdup`, `ALLOC_ARRAY`, `xstrfmt`, `ALLOC_GROW`, `xcalloc`) so tracing cannot retrigger during die() handling. Earlier reviews fixed the `BANNED()` casing, the `tr2_sysenv_cb()` return value, and a redaction length bug. Today Jeff King argued that fully pruning dependencies would amount to a ground-up rewrite and that static analysis of the complete call tree could eventually make `banned-die.h` unnecessary, though he is not opposed to it in the meantime; Stolee said the header helps demonstrate progress during the refactor but is less useful as long-term protection.

## The day in brief
Elijah Newren gave the v9 `git-replay(1)` `--linearize` series a "good to go" review, Derrick Stolee approved the multi-pack-index removed-pack recovery patch, and Patrick Steinhardt posted a 12-patch series removing in-memory source registration from the object database.

## Notable threads

### revision: use-after-free of freed argv elements

Jeff King posted a patch to fix a use-after-free in `setup_revisions()`: overwritten argv strings would be collected in a new `argv_to_free` strvec on `rev_info` and released in `release_revisions()` instead of freed immediately, addressing garbled `--src-prefix`/`--dst-prefix` output from `git stash show` (with a new test in `t3903-stash.sh`). He also posted a follow-up simplifying `mark_argv_for_free()` callers. Patrick Steinhardt reviewed the proposal favorably, and Junio C Hamano approved the cleanup.

### last-modified: Bloom filter pre-filtering for wildcard pathspecs

*Long thread (root 2026/07/17/15-46-58): 22 replies so far from 4 participants over 47 days; 4 messages that day.*

Toon Claes's series speeds up `git-last-modified(1)` on directory pathspecs by reusing the `git-log(1)` Bloom filter machinery to pre-filter commits, reporting about 1.95x speedup with `-t` and 3.74x without on ziglang/zig. Today Claes posted v4, overriding `GIT_TEST_COMMIT_GRAPH` when passing `-c core.commitGraph=` in the t8020 tests after Junio C Hamano pointed out the harness export made the `-c` ineffective; Hamano later noted the missing settings when queuing fixups.

### checkout: separate autostash conflict advice from branch-switch message
Harald Nordgren's patch makes `git checkout -m` print a blank line between the autostash conflict advice and the "Switched to branch" message, and makes `git stash apply`, `pop` and `branch` exit with status 2 when applying the stash entry resulted in conflicts (other failures exit 1), documented in the git-stash docs. Today Phillip Wood argued the exit-code convention is confusing since merge-tree, grep and diff use 1 for conflicts, and Junio C Hamano agreed that "1 if X, other non-zero if error" would be better.

### builtin/ident: new `ident` command
Andrew Pleeter's patch adds a builtin `git ident` command to display the resolved author and committer identity along with commit signing configuration, with scope and component selectors and porcelain output. Today Jeff King proposed instead teaching `git var` to provide broken-down identity components, arguing the new command's features could enhance `git-var` and that another specialized command is undesirable; Junio C Hamano endorsed that assessment.

### odb: stop registering in-memory sources

Patrick Steinhardt posted a 12-patch series removing the mechanism to link ad-hoc in-memory sources into the object database, so the source list is exactly the list of alternates, enabling moving alternates into the "files" backend later. It adapts cache-tree and submodule-config away from `the_repository` and drops `odb_add_to_alternates_memory()`. Today Junio C Hamano reviewed patches 01/12 and 06/12, suggesting a static wrapper for `cache_tree_fully_valid()` and noting patch 01/12 should pass `index_state->repo`.

## In brief
- Siddharth Asthana posted v2 of a `--missing-only` filter flag for `git rev-list` that requires `--missing=print` or `--missing=print-info` and dies with `--count` or `--disk-usage`; he explained Gitaly uses the missing list as a one-shot dependency set for transaction packing.
- Patrick Steinhardt requested eviction of the `ps/fetch-packfile-uris-parallel` topic due to conflicts with an upcoming object-transactions series for `git-fetch-pack(1)`; Junio C Hamano acknowledged.
- Karthik Nayak reviewed the checkout/switch/restore refactoring series; Junio C Hamano confirmed removing the monolithic `checkout_main()` makes `enum checkout_command` unnecessary. *Long thread (root 2026/08/27/18-29-00): 23 replies so far from 5 participants over 6 days; 6 messages that day.*
- Johannes Schindelin posted a two-patch series enabling Rust in the Windows CI jobs by selecting a GCC-compatible Cargo target under MSYS2/MinGW and running `rustup target add`, removing the `NO_RUST` opt-out.
- Elijah Newren posted v4 of the series refusing `git commit --amend` and partial commits during conflict resolution, with only a clarifying rename of the rebase-empty states. *Long thread (root 2026/08/26/05-21-21): 21 replies so far from 3 participants over 7 days; 1 message that day.*
- Skybuck Flying reported PNG corruption from a global `* text diff=lfclean` rule with a sed textconv stripping CR bytes from binary files; rsbecker recommended `core.autocrlf=input` and explicitly declaring binary files.
- Michael Montalbo posted v5 of the `t/lib-httpd` concurrency-safe CGI helper series, rewording the patch 3 advice after Patrick Steinhardt's review. *Long thread (root 2026/07/08/02-59-40): 22 replies so far from 3 participants over 56 days; 4 messages that day.*
- Patrick Steinhardt fixed the CI "documentation" job by explicitly installing Ruby; Jeff King suggested dropping the pinned Asciidoctor 1.5.8 in favor of v2.0.x.
- Toon Claes said the v2 "odb: write alternates at creation time" series looks good to him.
- D. Ben Knoble shared a link to benchmarks related to the v2.55.0 release for the Git Rev News edition 138 draft.
