# Git mailing list daily digest for 2026/10/06

## Long threads

### New `gitmergeconflicts` manual page

*Long thread (root 2026/09/24/14-44-15): 38 replies so far from 6 participants over 13 days; 2 messages that day.*

Julia Evans's series adds a new `gitmergeconflicts` manual page explaining merge conflicts with examples, and links to it from `git-merge`, `git-pull`, `git-rebase`, `git-revert` and `git-cherry-pick` instead of re-explaining the process in each command. Earlier revisions refined a "WHAT IS A MERGE CONFLICT?" section and an "OURS"/"THEIRS" section under review from Junio C Hamano and D. Ben Knoble. Julia Evans previously reported that `git status` recommends `git commit` rather than `git merge --continue` during a merge; today D. Ben Knoble agreed, citing `wt-status.c:show_merge_in_progress()`, and proposed either changing the status output or documenting the two commands as equivalent for merges, noting that a patch addressing the point is already in flight.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control

*Long thread (root 2026/07/10/08-51-34): 36 replies so far from 3 participants over 89 days; 1 message that day.*

Christian Couder's v5 series introduces the protected `uploadpack.lazyFetchTrusted` config variable so server operators can mark trusted repositories -- vouching for their promisor remotes, configuration and hooks -- letting `builtin/upload-pack.c` skip setting `GIT_NO_LAZY_FETCH` for them; it also refactors `promisor-remote.c` and adds recursion-depth tracking. Earlier, Junio C Hamano asked whether the point of diminishing returns had been reached. Today Couder agreed the series should be marked for `next`, saying it adds mostly refactorings plus the new config variable, documentation and tests rather than substantial new code.

### trace2: stop allowing die()

*Long thread (root 2026/07/15/16-12-11): 26 replies so far from 6 participants over 84 days; 1 message that day.*

Derrick Stolee's series introduces a root-level `banned-die.h` included by the trace2 API sources, banning die()-capable helpers one at a time so the low-level trace2 API cannot trigger a recursive die() while tracing. Earlier reviewers including Jeff King and Elijah Newren noted the ban list is whack-a-mole, since transitive calls through strbuf and json-writer remain. Today Stolee stated the topic has sat idle because the `banned-die.h` protections are incomplete, and that his RFC for a more robust strbuf was similarly unsatisfying.

### CI annotations for test failures and leaks

*Long thread (root 2026/09/25/18-54-03): 24 replies so far from 4 participants over 12 days; 2 messages that day.*

Harald Nordgren's series makes GitHub CI annotations for test failures and leaks name the test script and line. Earlier, Phillip Wood objected that direct file/line links fail when the file is unchanged in the PR. Today Nordgren posted a revision (labeled v6, with changes described as v7) that reverts direct linking in favor of showing the file and line as plain text in the annotation, omits the line number for leaks, keeps `--immediate` for leak-sanitizer jobs, and adds a leak annotation with the sanitizer report in a log group; Phillip Wood reviewed it and said "Lets get this merged!"

### Remove `gittutorial-2`

*Long thread (root 2026/09/28/20-25-19): 21 replies so far from 4 participants over 9 days; 5 messages that day.*

Julia Evans's v2 series removes `gittutorial-2` and its references, leaving a stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`; Junio C Hamano marked it for `next` earlier. Today Tuomas Ahola showed the interdiff adds `gittutorial-2` to `command-list.txt`, making git(1) advertise the obsolete tutorial in its GUIDES list; Junio proposed ejecting both topics and having Evans resubmit without the command-list entry, which Evans agreed to do in a v3, while Ahola said stacking his topic on top is cleaner than an evil merge and that `gittutorial-2` should not be re-added; Junio agreed to queue Ahola's reroll on a synthetic base.

## The day in brief
Christian Couder agreed the `uploadpack.lazyFetchTrusted` series should be marked for `next`, and Harald Nordgren's revised CI annotation series drew Phillip Wood's "Lets get this merged!". Julia Evans will resubmit the `gittutorial-2` removal without advertising the obsolete tutorial in git(1)'s guides list.

## Notable threads

### packfile stale delta base cache corruption fix
Patrick Steinhardt posted v2 of a two-patch series fixing corruption caused by delta base cache entries not evicted when a pack is closed: stale entries keyed by `struct packed_git` address and offset can be reused by a new pack at the same address, reproducible via recursive merges with submodules that have similar packfiles. Patch 2 adds `delta_base_cache_evict_entry()` called from `close_pack()` plus a test in `t/t6437-submodule-merge.sh`. Junio C Hamano reviewed, noting the test only detects the bug under the glibc allocator and calling an unreliable reproducer for an already-fixed bug of dubious value.

### "safe" strbuf API RFC set aside
Derrick Stolee's RFC proposed a `strbuf-safe` API that cannot die() on allocation failure, motivated by trace2's recursive die() risk via json-writer. Today Stolee set the RFC down for now, acknowledging Jeff King's earlier feedback would require starting over on the direction and that he does not currently have time, though he might revisit it.

### Evicting `ds/trace2-tolerate-failed-timestamp`
Stolee recommended evicting the `ds/trace2-tolerate-failed-timestamp` topic, saying it delivers little value for the complexity it adds and that `banned-die.h` in the repo may suggest stronger protections than it really promises.

### post-worktree lifecycle hook review
The v3 series adds a `post-worktree` hook with add/move/remove events for tools managing per-worktree environments. Today Phillip Wood affirmed the hook is a useful addition, since wrapper scripts are easy to forget and hard to coordinate, but questioned the four-argument design with empty-string padding, whether the worktree id is useful given git commands do not accept it (proposing the absolute git-dir instead), whether serial execution is necessary, and why separate tests are needed for paths with spaces.

### merge: remember conflict labels for `checkout -m`
The v2 series teaches the ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` recreates the original labels. Today Phillip Wood agreed to tighten `write_merge_labels()` to take `const char *labels[3]` and committed to several fixes; Junio C Hamano questioned a potential bug where `write_merge_labels()` is called on the clean-merge path while `labels[]` is only populated for conflicted results, possibly writing "(null)" lines.

## In brief
- Kristofer Karlsson clarified that the incremental connectivity check verifies only incoming commit trees against their parents' root trees, scaling with the number of changes rather than preexisting objects; Patrick Steinhardt accepted that die() in the rev-list subprocess is fine.
- Karthik Nayak's v3 of "packed-refs: use `fwrite()` when passing refs verbatim" (a ~20% win on ref deletion) drew nits from Patrick Steinhardt and Junio C Hamano ("sanitation" should be "sanitization"); Nayak agreed to amend the message and inline the wrapper.
- Jens Röcker reported that a push resends common history when a pre-push hook repacks and removes loose common objects (4,196,026 vs 300 bytes on the receiver), suspecting a stale pack catalogue makes `odb_has_object()` miss the base; D. Ben Knoble asked for clarification.
- Kristofer Karlsson posted v2 of "fetch: write commit-graph using updated refs only", skipping no-op fetches and writing incrementally from updated tips, with benchmarks (1 ref updated: 357 ms -> 9.3 ms).
- Junio C Hamano agreed Git should warn about or refuse editing a shallow boundary commit, while committing an empty tree is a normal operation that should not be warned about.
- Junio C Hamano objected to the proposed clock-skew warning, arguing it could annoy users cloning a future-dated tip and that time-based options cannot be fully reliable due to unsynchronized clocks.
- Discussion of the SHA-256 tree-signing RFC: brian m. carlsson detailed his ~212-patch `sha256-interop` branch, its missing pieces, and an estimate of four release cycles to upstream it; Johannes Schindelin argued migration reluctance stems from effort rather than urgency.
- Harald Nordgren posted a 6-patch series making `git status` and `git push` handle a push branch rebased onto a newer upstream, with `git push --force-with-lease` hints.
- Karthik Nayak reviewed Patrick Steinhardt's 13-patch `odb/source-files` series moving alternates into the backend, found the changes good, and asked for another review. *Long thread (root 2026/10/02/10-08-11): 10 replies so far from 2 participants over 5 days; 8 messages that day.*
- Patrick Steinhardt reviewed Mark C. Chu-Carroll's "repo structure" filtering options, suggesting synopsis and terminology changes and additional tests.
