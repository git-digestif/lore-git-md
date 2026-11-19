# Git mailing list daily digest for 2026/10/06

## Long threads

### New `gitmergeconflicts` manual page

*Long thread: 38 replies so far from 6 participants over 13 days; 2 messages that day.*

This documentation series adds a new `gitmergeconflicts` manual page explaining merge conflicts with examples, linked from commands that cause conflicts. Julia Evans previously noted that `git status` recommends `git commit` rather than `git merge --continue` during a merge. Today D. Ben Knoble agreed, citing `wt-status.c:show_merge_in_progress()`, and proposed either changing the status output or documenting the commands as equivalent for merges; he noted a patch on this point is already in flight.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control

*Long thread: 36 replies so far from 3 participants over 89 days; 1 message that day.*

This series introduces a protected `uploadpack.lazyFetchTrusted` config variable so server operators can mark trusted repositories for lazy fetching during clone or fetch, with refactorings across `promisor-remote.c`, `setup.c`, and `builtin/upload-pack.c`. Today Christian Couder agreed the v5 series should be marked for `next`, since it adds mostly refactorings and a new protected config variable rather than substantial new code or features.

### Keeping `die()` out of the trace2 API

*Long thread: 26 replies so far from 6 participants over 84 days; 1 message that day.*

This series introduces a root-level `banned-die.h` header included by the trace2 API sources, banning die-capable helpers one at a time so the low-level tracing code cannot trigger a recursive die. Today Derrick Stolee said the topic has sat idle because the `banned-die.h` protections are incomplete, and that his RFC for making strbuf and friends more robust was similarly unsatisfying, though interesting alternative ideas exist.

### Linking CI failure and leak annotations to test scripts

*Long thread: 24 replies so far from 4 participants over 12 days; 2 messages that day.*

This CI series makes GitHub Actions annotations for test failures and leaks name the test script and line. Today Harald Nordgren posted a revision (labeled v6, with changes described as v7) that reverts direct file/line linking -- which fails when the error points to a file unchanged in the PR -- in favor of showing the file and line as plain text in the annotation, and omits the line number for leaks where it is never available. Phillip Wood reviewed it and said "Lets get this merged!"

### Removing `gittutorial-2`

*Long thread: 21 replies so far from 4 participants over 9 days; 5 messages that day.*

This documentation series deletes the stale `gittutorial-2` page, leaving a stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`. Today Tuomas Ahola showed the v1->v2 interdiff adds `gittutorial-2` to `command-list.txt`, making `git(1)` advertise the obsolete tutorial in its GUIDES list. Junio C Hamano proposed ejecting both topics and having Julia Evans send a v3 without the `command-list.txt` entry; Julia agreed, Tuomas said stacking his topic on top is cleaner than an evil merge, and Junio agreed to queue the reroll on a synthetic base.

## The day in brief

Taylor Blau shared contributor summit notes covering Git 3.0 timing, pluggable ODB, and protocol v2 for pushes. Christian Couder agreed the `uploadpack.lazyFetchTrusted` series is ready for `next`, and Harald Nordgren's revised CI annotation patch drew an enthusiastic review. A new bug report shows pushes resending common history when a pre-push hook repacks the object database.

## Notable threads

### Notes from the 2026 Git Contributor's Summit

*Long thread: 0 replies so far from 1 participant over 1 day; 8 messages that day.*

Taylor Blau shared notes covering seven topics: security process, Git 3.0 (possible 2.99 and 3.0 in March 2027; consensus to release 2.99.1 and 3.0 together with only `BREAKING_CHANGES` defaults flipped), documentation, Outreachy, pluggable ODB, AI policy, and protocol v2 for pushes.

### Fixing packfile corruption from stale delta base cache entries
Patrick Steinhardt posted v2 of a two-patch series that evicts delta base cache entries when closing a pack, fixing corruption reproducible via recursive merges with submodules whose packfiles look similar. Junio C Hamano reviewed the fix, noting the new test detects the bug only under the glibc allocator and questioning the value of an unreliable reproducer for an already-fixed bug.

### Push resending common history after a pre-push repack
Jens Röcker reported that a push resends common history when a pre-push hook repacks the local object database and removes loose common objects, reproduced on Apple Git 2.54.0 and upstream Git 2.56.0 with receiver pack sizes of 4,196,026 bytes versus 300 bytes without the repack. D. Ben Knoble asked for clarification on whether objects are being sent more than once.

## In brief
- Karthik Nayak reviewed the 13-patch `odb/source-files` alternates series, accepted Patrick Steinhardt's explanations, and said the whole series looks good. *Long thread: 10 replies so far from 2 participants over 5 days; 8 messages that day.*
- Kristofer Karlsson posted v2 of "fetch: write commit-graph using updated refs only", with benchmarks showing updated-ref fetches dropping from about 357 ms to under 10 ms.
- Kristofer Karlsson clarified the incremental connectivity check verifies incoming commit trees against their parents only, scaling with the number of changes rather than preexisting objects.
- Karthik Nayak posted v3 of the packed-refs `fwrite()` optimization; Patrick Steinhardt called the ~20% deletion win welcome, and Junio C Hamano's "sanitation" typo nit was confirmed.
- **Clean-merge label write questioned**: Junio C Hamano asked what would be written to MERGE_LABELS on clean merges, suggesting '(null)' lines.

- Phillip Wood reviewed the post-worktree lifecycle hook proposal, questioning the four-argument design and the usefulness of the worktree id.
- Harald Nordgren posted a 6-patch series making `git status` and `git push` suggest `git push --force-with-lease` after a clean rebase.
