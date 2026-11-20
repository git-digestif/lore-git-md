# Git mailing list daily digest for 2026/10/25

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 45 replies so far from 4 participants over 37 days; 7 messages that day.*

This series by Harald Nordgren changes fetch and `git remote add` behavior in shallow repositories, where a plain `git fetch` on a newly added remote could negotiate history for every branch the remote has. The series adds a per-remote `remote.<name>.refmap` config that supplies the default for `--refmap`, makes a refspec-less fetch infer what to fetch from local branches whose `@{upstream}` is on that remote, adds a `--limited-fetch` option to `git remote add`, and turns that option on by default when adding a remote to an already shallow repository.

Today Harald Nordgren posted v11, whose changelog lists `remote_find_tracking()` falling back to `remote.<name>.refmap` when `remote.<name>.fetch` does not match (with negative-refspec exclusion and tests), helpers writing straight into a strvec with the output parameter last and the remote const, the followRemoteHEAD lookup done once, clearer commit messages and docs, and `use_limited_fetch` as a bool. Earlier in the day he answered D. Ben Knoble's questions about the fallback: without it, `@{upstream}`, `--delete-merged` and `git branch --track` fail for refs fetched through the refmap, and he will add tests locking that down, while noting `--limited-fetch` does not need the fallback. He also agreed to several of Knoble's earlier review suggestions (commit-message rewording, strvec instead of string_list, deduplicating the followRemoteHEAD lookup). Junio C Hamano described a use case for configuring both refmap and fetch on the same remote, fetching only selected branches from a "cabal" remote while storing them under `refs/remotes/cabal/`.

### reflog: add --operations and --historical-branch

*Long thread (root 2026/10/15/13-13-55): 22 replies so far from 5 participants over 11 days; 3 messages that day.*

This three-patch series by Harald Nordgren adds `--operations` (one entry per command, folding a rebase into the entry that finished or aborted it) and `--historical-branch` (decorating each entry with the branch HEAD was on when it was recorded) to `git reflog` and `git log -g`, because the default reflog lists every rebase step and branch switch, making it hard to see what was done and on which branch.

Today Kristoffer Haugsbakk retracted his earlier motivation for the series: after testing `git reflog --no-decorate test-branch` on a five-commit rebase, he found the per-branch reflog shows only the rebase-finish and branch-creation entries, so his "obscured reset points" complaint may have been an illusion caused by running reflog without a branch name. He also criticized the `--historical-branch` implementation, saying `reflog_walk_mark` taking two `bool` parameters and doing conditional calls inside the for-each loop looks like a bad approach, and suggested checking each condition first and then running the loop with an unconditional call per item. Harald Nordgren replied that it may not be worth reviewing the code now, since it is unknown whether any of this will be done.

## The day in brief

Harald Nordgren posted v11 of the shallow-repo fetch series, incorporating review feedback on the `remote_find_tracking()` refmap fallback and helper structure. Tyler Cipriani's series merging greplint checks into chainlint cut `make test-lint` from about 25 seconds to under 6 seconds in the cover letter's benchmark. Tanga Joris identified the cause of a `git apply --check` crash: a blank line after the `diff --git` line in a new-file section.

## Notable threads

### t: run greplint checks inside chainlint

Tyler Cipriani posted a four-patch series teaching `t/chainlint.pl` to run the greplint checks while processing its own tokens, so `make test-lint` parses each of the roughly 1100 test scripts once, in parallel, instead of running two linters that both open and tokenize the same files. The cover letter reports five runs of `make test-lint` dropping from 25.158s to 5.972s (nproc=16), and local testing with `-j$(nproc)` dropping from about 4.5s to under 1s. The series also moves greplint's parsing into `t/lib-greplint.pl`, adds an early return in greplint to skip test bodies without the string "grep" (cutting standalone greplint cost by about half), replaces five bare `grep` calls with `test_grep` in `t/interop/i5700-protocol-transition.sh`, and keeps standalone greplint for `GIT_TEST_CHAIN_LINT=0`.

### submodule: make '^/' work like '../' but with an absolute path

Stanislav Aleksandrov's RFC proposes resolving a submodule URL starting with `^/` against the superproject's remote from the server root, so `https://host/me/super.git` plus `^/org/lib.git` yields `https://host/org/lib.git`, keeping protocol, user, host and port. Unlike `../`, it fails for local-path clones and `<transport>::<address>` remotes. Today Jacob Keller asked whether `.gitmodules` could carry insteadOf-style rewrites and whether multiple `../` segments reach the server root; Aleksandrov answered that `.gitmodules` is identical for everyone while the superproject's remote URL differs per clone, that `^` is a valid directory name rejected by `git submodule add`, and that `^/` resolves correctly at any fork depth where `../` counts must match the superproject's depth. Keller acknowledged the low breakage risk given `protocol.file.allow` defaults and concluded the `^/` concept makes sense.

### Git crashed on `git apply --check`

Tanga Joris traced the `git apply --check` crash to a blank line after the `diff --git` line in a new-file patch. He confirmed the issue is reproducible on master and said he is working on a fix.

### status: compare against both upstream and push branch by default

Harald Nordgren posted v3 of a patch making `git status` compare the current branch against both `@{upstream}` and `@{push}` by default, building on the earlier opt-in `status.compareBranches` config. The strengthened commit message argues the push-branch comparison reminds returning users how the branch differs from what was last pushed, reveals local rewrites needing a forced push, and validates what is about to be pushed; it also notes triangular-workflow users currently rely on custom aliases.

### worktree list: add --branch to select worktrees by branch

This patch teaches `git worktree list` a repeatable `--branch=<branch>` option listing only worktrees with that branch checked out, where a worktree rebasing or bisecting a branch counts. Phillip Wood reviewed it, noting scripts wanting per-branch checkout status would still match names themselves, asking that the commit message spell out the rebase/bisect counting, and proposing preparatory refactoring in `branch.c` (a `struct worktree` member in `struct checked_out_branch`, a hashmap keyed by branch name, and a `branch_find_checkout()` function). He also suggested an `is-main-worktree true` annotation so scripts can avoid removing the main worktree, and questioned why nonexistent branch names are accepted while syntactically invalid ones are rejected.

## In brief

- Srujith A Poojary posted v5 of "diff-delta: use safe integer arithmetic", replacing manual size arithmetic in `create_delta_index()` with `st_mult`/`st_add`/`st_add3` and expanding the commit message with the 32-bit struct sizes and code-derived bounds after Junio C Hamano criticized v4 for parroting final numbers.
- Junio C Hamano asked that the follow_remote_head resolution logic in `do_fetch()` be refactored into a separate `lazy_follow_remote_head()` helper, supplying a diff, because the deeply indented code would conflict with another topic.
- On the patch fixing `git reflog expire --dry-run --verbose` to say "would prune" for named refs, Junio C Hamano suggested consolidating the `--all` and named-ref loops into one; Laveesh Bansal agreed it is doable but proposed keeping the bugfix as-is and doing the restructuring separately.
- Dorine Tipo posted a patch making `partial_clone_register()` and `partial_clone_get_default_filter_spec()` take a `struct repository *` instead of using `the_repository`, with callers in `builtin/clone.c` and `builtin/fetch.c` passing it explicitly.
- Torsten Bögershausen posted v3 of "t1450: use 'mv -f' when overwriting a read-only file", with the shortened commit message Junio C Hamano proposed, noting macOS 12.3 fixed the interactive prompt.
- Omri Sarig posted v4 of "doc: git-rebase: update incompatible options", rebased onto maint and asking whether to add the new `--onto` "See also INCOMPATIBLE OPTIONS" line in the linkless aligned form or the linked final form.
- René Scharfe endorsed dropping `NO_PARSEOPT` from the four builtins that now use parse-options, but asked whether completion still works seamlessly for `git fast-import`, whose `--git-completion-helper` output requires a line of input first.
- Wasswa Edson acknowledged the security concern about server-advertised promisor remotes as valid, stated the client must explicitly configure acceptance policies before any advertised remote is accepted, and said he is redoing the documentation patch.
- Aniket Gupta resent v2 of "object-name: use Q_() for the reflog entry count" as a reply to v1, with t1506 now matching "entr" so a one-entry reflog passes.
- Junio C Hamano noted that the v2 reflog-count patch must be sent as a reply to the v1 message.
