## Long threads

### `git history squash` subcommand series

*Long thread: 86 replies so far from 9 participants over 68 days.*

Harald Nordgren's series adding `git history squash <revision-range>` to fold a commit range into one reached v14, an eight-patch split covering parsing, range validation, branch protection, commit creation, and message editing. Review attention this week centered on details rather than design: Phillip Wood flagged missing `Signed-off-by` trailers from his fixup patches and their ordering relative to Harald's, Kristoffer Haugsbakk advised a specific trailer sequence, and Junio C Hamano corrected both his own earlier framing and the series on `istarts_with()` matching hex object IDs in `fixup!` subjects, insisting `fixup! ABCdef` must not be accepted and that the `| tr` in the tests come out. The series remains in active review.

### Moving libgit.a sources into a `lib/` directory

*Long thread: 38 replies so far from 12 participants over 124 days.*

Michael Montalbo's RFC to move roughly 700 libgit.a source files into a `lib/` directory continued with Junio C Hamano responding to a mechanical grouping experiment. Junio observed that the earliest group -- `connect.c`, `fetch-pack.c`, `remote-curl.c`, `send-pack.c`, `transport.c`, `upload-pack.c` -- forms a coherent "transport" group matching intuition, while the other groups are less clear-cut. He suggested carving conceptually clean groups into their own directories, as was done with `builtin/`, `refs/`, or `odb/`, without waiting to classify the remainder, and raised whether bisect and revision machinery could be separated. No restructuring landed this week.

### Pluggable packfile writes for `git-receive-pack`

*Long thread: 35 replies so far from 3 participants over 16 days.*

Justin Tobler's series introducing the generic `odb_transaction_write_pack()` interface advanced from v4 to v5, making `git-receive-pack(1)` write incoming packfiles through the ODB transaction layer so the existing child-process logic spawning `git-index-pack` and `git-unpack-objects` becomes the "files" backend implementation. Patrick Steinhardt approved passing the unpack limit via `odb_transaction_flags` and asked for a test covering the full `.keep` file lifecycle from quarantine to migration; Justin agreed to add it. Junio C Hamano objected that reading `receive.unpackLimit` and `transfer.unpackLimit` in generic object-layer code is a layering violation and suggested `odb-file.unpackLimit` as a more accurate name. Patrick said he is happy with v5.

### Eagerly loading ODB alternates

*Long thread: 31 replies so far from 5 participants over 12 days.*

Patrick Steinhardt's series making alternates eagerly initialized at object-database creation reached v3, adding a first patch that creates the ref and object databases only after the complete repository configuration is written in `init_db()`, addressing Jeff King's timing hazard about caching `core.ignoreCase` too early. Karthik Nayak affirmed the rationale, noting `core.sharedRepository` must be known when the ODB is created. Junio C Hamano questioned the `eq()` semantics for object databases on filesystems with differing case sensitivity and whether `o->repo->initialized` is true in that path; Patrick defended keeping the existing hashing, and Junio concluded the code can stay as-is since inconsistent casing only causes a redundant duplicate search.

### Pluggable packfile generation via the object database

*Long thread: 31 replies so far from 6 participants over 15 days.*

Patrick Steinhardt's series hoisting pack-generation machinery shared by `git-upload-pack`, `git-send-pack`, and `git-bundle` into a new `odb_generate_pack()` interface reached v4 with an improved error message and a stylistic fix. The v3 round fixed a use-after-scope bug Elijah Newren found in `odb_source_files_generate_pack()`, where a stale pointer to a function-local struct could be dereferenced on abnormal exit. Karthik Nayak reviewed v4 and said everything looks good; Junio asked whether `opts->pack_fd` should be closed when the primary source lacks the `generate_pack` callback, and Patrick confirmed the generator consumes it and a missing callback dies.

### Ambiguous remote branch message in `worktree add`

*Long thread: 27 replies so far from 6 participants over 15 days.*

Yoichi NAKAYAMA's series improving the error when `git worktree add` or `git checkout` matches multiple remote-tracking branches with the same name reached v7, rewording the advice to "Branch name '%s' appears in multiple remotes:" followed by the matched remotes, applied to both commands. Junio C Hamano reviewed v7, called the changes sensible, said he very much likes the more concise message, and said "Will queue." He classified the multiple-match-but-success case in `git worktree add` as the same class of bug, fine to leave for a follow-up, noting that creating the branch from HEAD on multiple matches is buggy and should eventually error out.

### `git repo info` path keys (GSoC series)

*Long thread: 25 replies so far from 4 participants over 37 days.*

K Jayatheerth's GSoC series adding `path.*` keys to `git repo info` -- toplevel, superproject root, hooks, index, grafts, git-prefix, each with absolute and relative suffixes -- continued with discussion of why the `path.toplevel` test does not use the `test_repo_info_path` helper. K agreed `path.git-prefix` is technically current-directory information but kept it as useful for scripts, while Junio countered that it is repository information expressed as a relative path, the inverse of a `path.cdup`, and that `cdup` should be offered too. K committed to adding a `cdup` key in the next version.

### diff hunk provider interface rethought

*Long thread: 25 replies so far from 4 participants over 94 days.*

Michael Montalbo retracted the "diff provider" abstraction from RFC v7, saying it shoehorns several concepts together, and clarified that "chain order is authority" was an unclear description of earlier providers overriding xdiff's default. He confirmed the underlying goal: a single point where every in-process diff computes hunks, with a "cousin" interface to xdiff-interface for user-configured alternatives via `.gitattributes` and `diff.<driver>.process`, OID-only operation for callers needing only hunk headers, and a consistent pkt-line interface for plugging in xdiff alternatives. He decided to return to the v6 line for future re-rolls and eject the diff-hunks store patches for now.

## The period in brief

The week of 2026/08/17 through 2026/08/23 saw seven active days of steady, technical traffic dominated by Patrick Steinhardt's object-database pluggability work and Harald Nordgren's long-running `git history squash` series. Readers should not miss Elijah Newren's fix for the multi-pack-index race with geometric repacking, Patrick Steinhardt's parallel packfile-URI fetch series with benchmarked speedups, and the controversy over an AI-generated treewide patch that brian m. carlsson and Junio C Hamano declined to review on Developer's Certificate of Origin grounds.

## Key developments

### Multi-pack-index race with geometric repacking

Elijah Newren posted a two-patch series for objects treated as missing despite being present: a race where geometric repacking creates a new multi-pack-index and removes the pack an older index named as owner, so a running process cannot reach the object. The series makes `git replay` fail gracefully instead of SIGSEGV when a merge input is unreadable, and teaches `find_pack_entry()` to scan the multi-pack-index's covered packs for a surviving copy. Junio C Hamano confirmed the replay diagnosis -- any non-zero value was mistakenly treated as OK -- and gave the verdict "Well spotted and fixed," while noting he would prepare an evil-merge for an API change from another in-flight topic. Elijah later identified that the race also affects `OBJECT_INFO_QUICK` callers and found a hole in his own fix when only the `.pack` vanishes.

### Incoming-proportional connectivity check

Kristofer Karlsson proposed an RFC making `check_connected()`, used by fetch and receive-pack, cost proportional to the incoming set rather than repository size, via opportunistic trusted-tree discovery and bounded boundary traversal seeded with old ref values, falling back to `--not --all` when the boundary is not closed. His prototype on a repository with 3M commits and 200K refs dropped the check from about 1s to about 0.03s for a five-commit push. The proposal drew review interest but no decision this week.

### Parallel packfile URI fetches

Patrick Steinhardt submitted a two-patch series letting `git-fetch(1)` and `git-clone(1)` fetch packfile URIs in parallel, refactoring `fetch-pack.c` into preparation, fetch, and aggregation phases and adding the `fetch.packfileURIThreads` config (default 1, 0 for auto-detect). Benchmarks showed speedups up to 7.58x on a throttled connection. The series was under review at the end of the period.

### Worktree command fixes queued

Yoichi NAKAYAMA landed two fixes reviewed and queued by Junio C Hamano. One stops `git worktree add -b <new-branch> <path> <branch>` from DWIM-ing `<branch>` to a remote-tracking branch, a regression from Git 2.42, adding an `else if (ac == 2 && new_branch)` branch in `builtin/worktree.c`. The other, covered under Long threads above, improves the ambiguous-remote-branch message. Separately, Toon reported that `git history fixup HEAD` in one worktree leaves another showing staged changes it never made; Patrick Steinhardt agreed `git-history(1)` should refuse updating branches checked out elsewhere and created GitLab work item 772.

### Reftable stack reload avoidance

Karthik Nayak's series removing `REFTABLE_STACK_NEW_ADDITION_RELOAD` and avoiding reloading the reftable stack when already locked within a reference transaction cut the `fstat()` calls that dominated ref creation. Jeff King reported a ~25% speedup for `update-ref --stdin` with 50,000 refs, and Karthik reproduced it using `refs/tags/*`. Patrick Steinhardt noted libgit2 also uses the flag and suggested renaming `reftable_stack_new_addition()`; Karthik confirmed no "don't reload" flow remains. Junio raised an unresolved concern that a second `reftable_addition` on a locked stack would release a shared lock, which Karthik addressed by proposing per-addition lock ownership tracking.

### AI-generated treewide patch declined on DCO grounds

A treewide series migrating from the POSIX-removed `utime()` and `<utime.h>` to `utimensat()` drew a refusal to review: brian m. carlsson declined because the cover letter disclosed the series was generated by Gemini Flash, arguing this cannot satisfy the Developer's Certificate of Origin requirement, and noted he had not read the series to avoid being influenced by code the project cannot include. Junio C Hamano endorsed the stance as prudent, explaining the project bears the cost if DCO-covered work later proves legally unusable. brian m. carlsson distinguished honest disclosure of AI use from misleading the project about provenance. Related discussion covered AI-assistance disclosure in patches and a gitk README patch discouraging AI contributions.

### `git organize` RFC and `git config --global` fix

Michael Montalbo posted a 14-patch RFC introducing `git organize`, which records a project's file-placement guideline in a tracked `.gitorganize` file and moves out-of-place files, with `status` and `apply` subcommands and configurable labeler/organizer commands; the final patch moves 22 transport C sources and 21 headers into `transport/`. Delilah Ashley Wu posted v2 of a three-patch series making `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`, fixing the inconsistency where `--global` read only the former; Chris Torek asked which file `git config --global --edit` edits.

## In brief

- **Use-after-free fix in `get_oid_with_context_1()`** -- Shlok Kulshreshtha's v3 collapses two exits into one so `free(new_path)` happens after the last use of `cp`; Junio C Hamano reviewed it as "nicely described" and said "Will queue."
- **Valueless `submodule.active` warning** -- Junio C Hamano rejected the warn-and-continue proposal, arguing the existing hard error is sufficient and early messages scroll away unnoticed.
- **`http.sslVerifyStatus` for stapled OCSP** -- Grayson Gordon posted v4 and v6; Junio reviewed and said he will replace the patch.
- **Generic `OBJECT_INFO_DIE_IF_CORRUPT` handling** -- Patrick Steinhardt's v2 restructures around `enum odb_read_status` with negative error codes and an `errmsg` parameter; Junio accepted the explanation but noted the reasoning relies on each object appearing at most once per pack.
- **`report` hook for `git-receive-pack`** -- Karthik Nayak's v2 was refined with documentation in `githooks(5)`; Junio agreed a failed report hook should explicitly tell the client the push was declined.
- **`git pull --hard` vs `--reset`** -- Discussion of Artur Bieniek's proposal shifted to Phillip Wood's `git pull --reset [--discard-changes | --autostash]` alternative, which Artur now prefers.
- **Stop recognizing URLs as trailers** -- Kristoffer Haugsbakk's v3 fails trailer parsing on `://` separators; Junio marked the topic for `next`.
- **Shallow-clone push optimization** -- Elijah Newren's `send-pack` patch adds reachable shallow grafts as negative pack tips via `push.shallowExcludeBoundary` (default true); Patrick Steinhardt's review questions were answered.
- **Runtime `core.useNanosec`** -- D. Ben Knoble's v4 exposes the config at runtime; Junio called the struct alignment a "great improvement" and agreed no warning on `NO_NSEC` builds.
- **Many-packfile loading performance** -- Jeff King's synthetic test showed `git rev-list --count` growing from 18 ms with 500 packs to 6351 ms with 16000 packs; he concluded MRU updates should use constant-time movement in a doubly-linked list.
- **credential/libsecret explicit loading** -- Daniel Martí defended unconditionally loading secrets with `secret_item_load_secret_sync()` over a NULL-check fallback, arguing the flag discards errors from the second round trip.
- **`stash.ref` config knob** -- Vladimir Sitnikov proposed per-worktree stash stacks; Kristoffer Haugsbakk endorsed opting in to `refs/worktree/stash` and confirmed no ref namespace is special for GC.
- **Subtree/rebase bug report** -- Gabriel Ford reported `git subtree add --squash` followed by `git rebase` conflicting on same-relative-path files in separate directories.
- **Miscellaneous** -- Kristoffer Haugsbakk posted v4 of the format-rev documentation fix and v2 of the gitdatamodel(7) advertising series; Friel's `pack-objects` `wrote_bytes` Trace2 patch got "looks good" verdicts; Lutz Lengemann posted v2 of the zsh completion fix for `git -C`; Yoichi NAKAYAMA posted v3 of the `worktree repair` relative-path fix; Volodymyr Vriukalo fixed a BUG in `git branch --recurse-submodules`; Oswald Buddenhagen proposed an opt-out for Unicode box-drawing in `git repo structure`, which Junio agreed is needed.