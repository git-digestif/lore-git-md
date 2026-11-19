# Git mailing list daily digest for 2026/09/01

## Long threads

### git-replay(1) `--linearize` option

*Long thread: 39 replies so far from 7 participants over 86 days; 1 message that day.*

Toon Claes's series adds a `--linearize` option to `git-replay(1)` that drops merge commits and replays all reachable commits into a single linear history, mimicking what `git rebase --no-rebase-merges` does; earlier revisions reworked the error messages and documentation around the multiple-branches restriction after review. Today Elijah Newren reviewed the v9 series, said "I think this version is good to go," and added a Reviewed-by trailer.

### runtime `core.useNanosec` configuration

*Long thread: 38 replies so far from 6 participants over 26 days; 6 messages that day.*

D. Ben Knoble's series converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` setting, replacing `#ifdef` checks in `read-cache.c` and `statinfo.c` so index racy-git comparisons can use nanosecond timestamps when the platform supports them. Today Junio C Hamano flagged a "comparisions" typo in the v6 documentation, Jeff King reviewed the `NO_NSEC` build guard and found it reasonable, and Hamano agreed to amend the typo locally, saying a final reroll can be skipped if nothing else surfaces.

### packfile recovery when a MIDX names a removed pack

*Long thread: 36 replies so far from 5 participants over 15 days; 5 messages that day.*

Elijah Newren's series fixes objects being reported missing despite being present when a stale multi-pack-index names a pack removed by a concurrent geometric repack; v3 renames `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return and rescans the MIDX's packs for a surviving copy on second read. Today Derrick Stolee completed his review of patch 4/4 with "Good patch. LGTM." and said he supports the series, and Newren clarified that the fallback scan runs over the packs in the MIDX rather than all packs.

### `receive-report` hook for git-receive-pack(1)

*Long thread: 28 replies so far from 5 participants over 15 days; 3 messages that day.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client, letting the hook's stdout replace the pkt-line report and a non-zero exit reject all references. Today he posted v5, renaming `generate_response()` to `generate_report()` and adding documentation and a test for overriding the unpack status to fail the push; Junio C Hamano questioned the `bool detailed_report` parameter, suggesting a protocol-version enum, and challenged the rationale for rejecting a reference-transaction hook.

### trace2: stop allowing die()

*Long thread: 25 replies so far from 6 participants over 49 days; 3 messages that day.*

Derrick Stolee's series bans die()-capable helpers from the low-level trace2 API via a root-level `banned-die.h`, so tracing cannot retrigger a process-ending failure while die() is being handled. Today Jeff King argued that chasing transitive die() calls would need static analysis or a ground-up rewrite, though he is not opposed to `banned-die.h` in the meantime, and Stolee said the header helps demonstrate progress during the refactor but could be dropped in a v4.

## The day in brief
Elijah Newren gave the `git-replay` `--linearize` series a Reviewed-by, Derrick Stolee approved the MIDX removed-pack recovery patch, and Jeff King posted a fix for a use-after-free in `setup_revisions()` that garbled `git stash show` prefix output. Elsewhere, v5 of the `receive-report` hook and v4 of the commit-conflict-refusal series were posted, and Patrick Steinhardt asked to evict a conflicting fetch-packfile-uris topic from `seen`.

## Notable threads

### proposed builtin `ident` command
Andrew Pleeter proposed a new `git ident` command to display the resolved author and committer identity plus signing configuration in one place. Today Jeff King argued those features belong in `git var` as broken-down identity components rather than another specialized command, and Junio C Hamano endorsed that assessment.

### use-after-free in `setup_revisions()`

Jeff King posted a patch for a use-after-free where `setup_revisions()` freed argv strings that options like `--src-prefix` still referenced, garbling `git stash show` output; it collects overwritten strings in an `argv_to_free` strvec released in `release_revisions()`. He also posted a cleanup simplifying `mark_argv_for_free()` callers, approved by Patrick Steinhardt and Junio C Hamano.

### What's cooking in git.git
In the August "What's cooking" thread, Patrick Steinhardt asked to evict the `ps/fetch-packfile-uris-parallel` topic because it conflicts with an upcoming object-transaction series for `git-fetch-pack(1)`, saying he will resend once that lands; Junio C Hamano replied "Will do."

## In brief
- Harald Nordgren's v3 makes `git stash apply`/`pop`/`branch` exit 2 on conflicts, but Phillip Wood and Junio C Hamano argued 1-for-conflicts matches the merge-tree, grep and diff conventions better.
- Siddharth Asthana posted v2 of `rev-list --missing-only`, a filter flag requiring `--missing=print` or `--missing=print-info`; he explained Gitaly uses the missing list as a one-shot dependency set for transaction packing.
- Toon Claes posted v4 of the `last-modified` Bloom-filter pre-filtering series, fixing `GIT_TEST_COMMIT_GRAPH` overrides in the t8020 tests. *Long thread: 22 replies so far from 4 participants over 47 days; 4 messages that day.*
- Patrick Steinhardt posted a 12-patch series removing in-memory source registration from the object database so the source list is exactly the alternates list.
- Elijah Newren posted v4 refusing `git commit --amend` and partial commits during conflict resolution, with a clarifying rename of the empty-rebase-pick states. *Long thread: 21 replies so far from 3 participants over 7 days; 1 message that day.*
- Johannes Schindelin posted a two-patch series enabling Rust in the Windows CI jobs by selecting a GCC-compatible Cargo target under MSYS2/MinGW.
