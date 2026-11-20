# Git mailing list daily digest for 2026/10/25

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 45 replies so far from 4 participants over 37 days; 7 messages that day.*

Harald Nordgren's series changes `git remote add` and fetch refspec handling so that adding a remote to an already shallow repository does not commit to fetching every branch the remote has. Previously a shallow clone that later ran `git remote add` received a wildcard `remote.<name>.fetch` refspec, so a plain `git fetch` could hang negotiating history for all of the remote's branches. The series adds a `remote.<name>.refmap` config, makes a refspec-less fetch infer what to fetch from local branches whose `@{upstream}` is on that remote, and turns on `--limited-fetch` by default for shallow repos.

Today Harald Nordgren posted v11, which makes `remote_find_tracking()` fall back to `remote.<name>.refmap` when `remote.<name>.fetch` does not match, with tests that negative refspecs still exclude refs; the helpers now write straight into a strvec with the output parameter last; the followRemoteHEAD lookup is done once; and `use_limited_fetch` is a bool. Earlier, D. Ben Knoble had questioned whether the fallback could miss mapped refs for `--delete-merged`; Harald Nordgren answered that without the fallback `@{upstream}`, `--delete-merged` and `git branch --track` fail for refs fetched through the refmap, and agreed to add tests locking that down. Junio C Hamano described a use case for configuring both refmap and fetch on the same remote.

### reflog: add --operations and --historical-branch

*Long thread: 22 replies so far from 5 participants over 11 days; 3 messages that day.*

Harald Nordgren's three-patch series adds `--operations` (one entry per command, folding a rebase into the entry that finished or aborted it) and `--historical-branch` (decorating each entry with the branch HEAD was on when it was recorded) to `git reflog` and `git log -g`, because the default reflog lists every rebase step and branch switch. Earlier, Phillip Wood and D. Ben Knoble argued that `--operations` cannot truly show one entry per command and that `--historical-branch` might be better as a `%gb` format specifier.

Today Kristoffer Haugsbakk expressed doubt about his earlier motivation: testing `git reflog --no-decorate test-branch` on a five-commit rebase showed the per-branch reflog lists only the rebase-finish and branch-creation entries, so he wondered whether his "obscured reset points" complaint was "just an illusion" of habitually running reflog without a branch name, though he added that "that still leaves Patrick's problems with it." He also criticized the `--historical-branch` implementation, saying `reflog_walk_mark` taking two bool parameters with conditional calls inside the loop looks like a bad approach and suggesting the conditions be checked outside the loop; Harald Nordgren replied that it may not be worth reviewing the code now, since it is unknown whether any of this will be done.

## The day in brief

Harald Nordgren posted v11 of the shallow-repo fetch series and v3 of the status push-branch comparison patch, while Tyler Cipriani's chainlint/greplint merge cut `make test-lint` from about 25 seconds to 6. Discussion continued on the submodule `^/` URL syntax and the reflog options series.

## Notable threads

### t: run greplint checks inside chainlint

Tyler Cipriani posted a four-patch series teaching `t/chainlint.pl` to run the greplint checks while processing its own tokens, so `make test-lint` parses each of the roughly 1100 test scripts once, in parallel, instead of twice with greplint running serially. The cover letter reports five runs dropping from 25.158s to 5.972s. The series moves greplint's parsing into `t/lib-greplint.pl`, adds a `--greplint` option to chainlint, replaces bare `grep` with `test_grep` in `t/interop/i5700-protocol-transition.sh`, and adds an early return in greplint to skip test bodies without the string "grep".

### submodule: make '^/' work like '../' but with an absolute path

Stanislav Aleksandrov's RFC proposes resolving a submodule URL starting with `^/` against the superproject's remote from the server root, keeping protocol, user, host and port, so forks at a different depth (such as GitLab subgroups) do not break relative `../` URLs. Today Jacob Keller asked whether `.gitmodules` could carry rewrite rules and whether multiple `../` segments reach the server root; Stanislav Aleksandrov answered that the file is identical for everyone while the superproject's remote URL differs per clone, that `^/org/lib.git` must be written by hand or via `git submodule set-url`, and that `^/` resolves correctly at any depth. Jacob Keller concluded the concept makes sense.

### status: compare against both upstream and push branch by default

Harald Nordgren posted v3 of a patch making `git status` compare the current branch against both `@{upstream}` and `@{push}` by default, building on `status.compareBranches`. The strengthened commit message argues the push-branch comparison reminds a returning user how the branch differs from what was last pushed, reveals local rewrites needing a forced push, and validates what is about to be pushed.

## In brief

- Tanga Joris reported that `git apply --check` crashes on a patch whose new-file section has a blank line after the `diff --git` line, confirmed it is reproducible on master, and is working on a fix.
- Phillip Wood reviewed `git worktree list --branch`, suggesting preparatory patches to share code with `branch.c` and an `is-main-worktree` annotation in the output.
- Srujith A Poojary posted v5 of "diff-delta: use safe integer arithmetic", expanding the commit message with struct sizes and code-derived bounds after Junio C Hamano's critique.
- Junio C Hamano asked that the followRemoteHEAD resolution in `do_fetch()` be refactored into a `lazy_follow_remote_head()` helper to reduce indentation and ease reuse.
- Laveesh Bansal's fix making `git reflog expire --dry-run` say "would prune" for named refs drew a suggestion from Junio C Hamano to consolidate the two expiration loops; Laveesh Bansal agreed to look at that separately.
- Torsten Bögershausen posted v3 of the t1450 fix using `mv -f` when overwriting a read-only loose object on pre-12.3 macOS.
- Wasswa Edson acknowledged the security concern on the promisor-remote documentation patch and said he is redoing it.
- René Scharfe endorsed dropping `NO_PARSEOPT` from four builtins but asked whether completion still works given `git fast-import --git-completion-helper` waits for a line of input.
