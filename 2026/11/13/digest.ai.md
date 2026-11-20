# Git mailing list daily digest for 2026/11/13

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 77 replies so far from 6 participants over 56 days; 1 message that day.*

The v13 series introduces `remote.<name>.fetchRefmap` and `--limited-fetch` so a shallow repository does not fetch every branch of a new remote, and makes `@{push}` resolve through the refmap. Phillip Wood argued that when `remote.<name>.fetchRefmap` maps fetched ref names without `remote.<name>.fetch` selecting a default set, a command-line refspec with an explicit destination such as `feature:wip/feature` should not also update the fetchRefmap destination; he agreed any tweak to that long-standing double-store behavior should be a separate preliminary cleanup before the series, and stated that patch 3 should be dropped in favor of combining `.fetchRefmap` and `.fetch` to set default behavior when no command-line refspecs are given. Earlier, Junio C Hamano had agreed the `@{push}` failure could be a bug introduced by patch 3, and Matt Hunter identified the blocked topic as `ch/remote-defer-follow-remote-head-validation`, noting that merging up to patch 3 could resolve the textual conflicts.

### doc: rewrite the detached HEAD state explanation

*Long thread (root 2026/10/07/14-52-01): 42 replies so far from 5 participants over 38 days; 8 messages that day.*

Julia Evans's series rewrites the detached HEAD explanation into a standalone guide referenceable as `git help detached-head` from advice messages, with matching changes to `git-checkout.adoc` and `t/t2020-checkout-detach.sh`. Today Junio C Hamano argued that not clobbering branches when making a temporary commit on detached HEAD is the most important user-impacting consequence of HEAD not recording a branch, and that deliberately avoiding it is a disservice; Julia Evans proposed adding "You can also use this to make experimental commits without being on a branch." to the examples, identified the underlying conflict as writing for two audiences (users accidentally detached versus advanced users exploring), and offered to remove the `git-checkout.adoc` changes and reframe the series around helping confused users. Junio countered that a manual page must cater to both audiences and suggested a FAQ or survival guide might better serve inexperienced users.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread (root 2026/10/21/17-33-36): 38 replies so far from 8 participants over 24 days; 5 messages that day.*

Jasper Mayone's v5 series makes `git worktree remove` refuse a worktree in the middle of an operation without `--force`, documents reading `%(worktreepath)` safely, names the rebasing or bisecting worktree in `%(worktreepath)`, adds `%(currentworktree)`, and stops `git branch` from marking the rebased branch with `+`. Junio C Hamano praised the central `check_no_operation_in_progress()` design in patch 1 as very good and extensible, called patch 3 a nice loss of duplicated logic, and said patch 2 looks good, but stated he is a bit negative on patch 5, preferring the branch being bisected to be marked in the list and suggesting the series might stop at 3/5 if 4/5 exists solely to support that step.

### builtin/rerere: convert to use subcommands

*Long thread (root 2026/10/27/10-46-43): 27 replies so far from 6 participants over 18 days; 5 messages that day.*

Patrick Steinhardt's v4 series modernizes `git rerere` to parse subcommands with `OPT_SUBCOMMAND()` like other Git commands. Today he accepted Karthik Nayak's capitalization nit but declined to send a new version just for it, asking Junio to fix it up in his tree. Junio applied the fixup with `git commit --fixup` plus `git rebase --autosquash` and asked whether the last three steps could be simplified; Tuomas Ahola suggested a one-liner using `git rev-list --min-parents=2 --max-count=1 HEAD`, Phillip Wood suggested `--keep-base` with an upstream branch, and Junio rejected the upstream approach because he does not want upstream set for his many topic branches, instead wishing for a single `git fixup HEAD~10` command.

## The day in brief
Phillip Wood pushed to drop patch 3 of the fetchRefmap series and handle the double-store behavior as a separate cleanup, while brian m. carlsson posted a rework of hash-algorithm detection that refuses to reinitialize a repository with a different hash. Adithya Chakilam introduced `--filter-to-loose` for `git repack`, and Christian Couder posted fixes for the missing promisor-remote reply in fetch and a crash in `ls-remote` outside a repository.

## Notable threads

### setup: fix hash algorithm detection
brian m. carlsson posted v2 reworking hash selection in `setup.c` so a command-line `--object-format` takes precedence, environment, config, and default are consulted only for new or hash-less repositories, and reinitializing an existing repository with a different hash algorithm is refused loudly; the patch adds tests covering default object format selection, refusal to reinitialize with a different hash, and preservation of the existing hash when `init.defaultObjectFormat` conflicts.

### repack: write filtered-out objects as loose objects
Adithya Chakilam submitted a two-patch series adding `--filter-to-loose=<dir>` to `git repack`, writing filtered-out objects directly as loose objects into a destination object directory by streaming the filtered pack into `unpack-objects` instead of writing an intermediate pack; the second patch extends coverage to loose objects excluded by the filter, skipping objects already present in the destination and, with `-d`, removing the source copies after all objects are written successfully.

### cat-file: do not die on reflog entries out of range
Caspar Duregger posted a four-patch series so `git cat-file --batch-check` reports out-of-range reflog selectors as missing instead of dying: fixing the peel-suffix misparsing of `HEAD@{1}^{blob}`, scoping `GET_OID_GENTLY` to failing lookups only, making `read_ref_at()` fail gently on an empty reflog, and passing the flag in cat-file batch modes; `HEAD@{999999}~1` and `@{upstream}` on a detached HEAD still die.

### promisor-remote reply missing from fetch
Christian Couder's series fixes the missing `promisor-remote` reply sent with `fetch` protocol commands, which breaks servers that handle each request separately, such as Git over HTTP or JGit/Gerrit, and object-ID-only fetches. He explained that cleaning up the capability-sending API would require refactoring three duplicated places in `connect.c`, proposed that as a separate series, and offered a v2 with a NEEDSWORK comment; Junio C Hamano replied that the duplication is all the more reason to clean it up before piling more on top.

### promisor-remote: fix crash without a repository

Christian Couder proposed a fix for a crash when `git ls-remote` runs outside a repository with promisor config options that write to config, returning early from `filter_promisor_remote()` when `repo->gitdir` is NULL, with tests for both the auto-create and store-fields cases. Junio C Hamano suggested instead making `promisor_remote_reply()` a NOOP outside a repository and noted the code should comment on why punting when `.gitdir` is NULL is safe.

## In brief
- **for-each-ref --no-format segfault patch**: aeron-gh posted a patch to fix a segfault in `git for-each-ref --no-format` (shared with `refs list`) by assigning the default format after option parsing; Junio C Hamano called the segfault bad and suggested mimicking `git branch`'s handling of `--no-format`.

- Patrick Steinhardt's `builtin/repo: don't fully parse trees` counts tree entries from the raw object buffer instead of `parse_object_buffer()`, yielding a 20.3% wall-time speedup and 5.1x less peak memory for `git repo structure` in git.git.
- Kristofer Karlsson's v2 shrinks `struct commit` from 80 to 72 bytes on 64-bit platforms by moving `.index` into alignment padding; Junio C Hamano agreed the cast-direction comment was backwards and said "Will queue."
- Patrick Steinhardt's `builtin/repo: only parse objects as needed` skips parsing blobs and tags and reuses already-parsed commits from the revision walk, a 12.1% wall-time and 4.1% memory reduction for `git repo structure`.
- Patrick Steinhardt reviewed Miklos Vajna's patch making `--follow` for non-linear history optional, suggesting documentation nits and `git_parse_maybe_bool()`, and concluded it looks good aside from those nits.
- Patrick Steinhardt said he is happy with v2 of the `ls-remote --show-object-format` patch.
