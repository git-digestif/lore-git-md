# Git mailing list daily digest for 2026/11/13

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 77 replies so far from 6 participants over 56 days; 1 message that day.*

The v13 series separates which refs to fetch from where they are stored via `remote.<name>.fetchRefmap`, and resolves `@{push}` through the refmap for `--limited-fetch` remotes. Phillip Wood argued that when `remote.<name>.fetchRefmap` maps fetched names without `remote.<name>.fetch` setting a default set, a command-line refspec with an explicit destination such as `feature:wip/feature` should not also update the refmap destination, and agreed any tweak to that long-standing double-store behavior should be a separate preliminary cleanup. He also stated that patch 3 should be dropped and the combination of `.fetchRefmap` and `.fetch` used instead to set default behavior when no refspecs are on the command line.

### doc: rewrite detached HEAD state explanation

*Long thread: 42 replies so far from 5 participants over 38 days; 8 messages that day.*

Julia Evans's series rewrites the detached HEAD explanation into a standalone guide referenceable as `git help detached-head` from advice messages. Today's exchange centered on how much detail to include about commits made in detached HEAD state not landing on a branch: Junio C Hamano argued that not clobbering branches when making a temporary commit is the most important user-impacting consequence and that deliberately avoiding it is a disservice, while Evans proposed adding "You can also use this to make experimental commits without being on a branch." to the examples. Evans identified the underlying conflict as writing for two audiences -- users accidentally detached versus advanced users exploring -- and offered to drop the `git-checkout.adoc` changes and reframe the series around confused users; Hamano countered that a manual page must cater to both.

### worktree: treat a branch being rebased or bisected as checked out

*Long thread: 38 replies so far from 8 participants over 24 days; 5 messages that day.*

Jasper Mayone's v5 series makes `git worktree remove` refuse a worktree mid-operation without `--force`, documents reading `%(worktreepath)` safely, names the rebasing or bisecting worktree in `%(worktreepath)`, adds `%(currentworktree)`, and stops `git branch` marking the rebased branch with `+`. Junio C Hamano praised the central `check_no_operation_in_progress()` design as very good and extensible, called patch 3 a "nice loss of duplicated logic", and said patch 2 looks good; he questioned a phrase in patch 4 and said he is "a bit negative" on patch 5, suggesting the series might stop at 3/5 if 4/5 exists only to support it.

### builtin/rerere: convert to use subcommands

*Long thread: 27 replies so far from 6 participants over 18 days; 5 messages that day.*

Patrick Steinhardt's v4 series modernizes `git rerere`'s custom subcommand parsing with `OPT_SUBCOMMAND()`. Today Steinhardt accepted Karthik Nayak's capitalization nit but declined to resend just for it, asking Junio to fix it up. Junio C Hamano applied the fixup with `git commit --fixup` plus `git rebase --autosquash` and asked whether the steps could be simplified; Tuomas Ahola suggested a `git rev-list`-based one-liner and Phillip Wood suggested `--keep-base`, but Hamano rejected the upstream-based approach and wished for a single `git fixup HEAD~10` command.

## The day in brief

Phillip Wood pushed the shallow-repo fetch series toward dropping its third patch in favor of combining `.fetchRefmap` with `.fetch`, while the detached HEAD documentation rewrite grappled with serving both confused and advanced users. New series were posted for making `cat-file` survive out-of-range reflog selectors, fixing hash algorithm detection on re-init, and writing repack-filtered objects directly as loose objects.

## Notable threads

### cat-file: report out-of-range reflog entries as missing
Caspar Duregger's four-patch series makes `git cat-file --batch-check` report out-of-range reflog selectors as missing instead of dying: fixing peel-suffix misparsing in `object-name.c`, scoping `GET_OID_GENTLY` to only failing lookups, failing gently on an empty reflog in `read_ref_at()`, and passing the flag in cat-file batch modes.

### setup: fix hash algorithm detection
brian m. carlsson's v2 reworks hash selection so a command-line choice takes precedence, environment, config, and default are consulted only for new or hash-less repositories, and reinitializing an existing repository with a different hash algorithm is refused, with tests covering each case.

### repack: write filtered-out objects as loose objects
Adithya Chakilam's two-patch series adds `--filter-to-loose=dir` to `git repack`, streaming the filtered pack into `unpack-objects` to write filtered-out objects directly as loose objects in a destination object directory, with the second patch extending coverage to loose objects excluded by the filter.

## In brief
- Christian Couder fixed a crash when `git ls-remote` runs outside a repository with promisor config that writes to config, returning early from `filter_promisor_remote()` when `repo->gitdir` is NULL.
- **for-each-ref --no-format patch**: aeron-gh proposed moving default format assignment after parsing to fix the segfault.

- Kristofer Karlsson's patch removes padding from `struct commit` on 64-bit platforms, shrinking it from 80 to 72 bytes; reviewers refined the cast-direction and date comments.
- Patrick Steinhardt sped up `git repo structure` in huge repositories by only parsing objects as needed and not fully parsing trees.
- Philippe Blain discussed `git replay` support for 2-parent merges, noting `--ancestry-path` covers replaying a merge with only one parent changed.
- Tatchou Yopa Nelly's `promisor.fetchOrder` patch stays an RFC prototype pending Outreachy selections.
- 井上司's patch fixes the reflog of a symref whose target is updated in the same transaction.
- Christian Couder's series fixes the missing promisor-remote reply in fetch protocol v2.
