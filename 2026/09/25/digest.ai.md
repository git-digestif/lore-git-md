# Git mailing list daily digest for 2026/09/25

## Long threads

### http.sslVerifyStatus for stapled OCSP responses

*Long thread (root 2026/08/11/17-02-00): 31 replies so far from 5 participants over 46 days; 2 messages that day.*

Grayson Gordon's series adds an `http.sslVerifyStatus` boolean mapping to libcurl's `CURLOPT_SSL_VERIFYSTATUS`, so git requests and checks stapled OCSP responses instead of ignoring them, including ones stating a certificate is revoked. Earlier, v7 was posted as the combined test version discussed with Junio C Hamano and Patrick Steinhardt, was reviewed and queued in `next`, then reverted after SZEDER Gábor reported the last test in the new t5585 failing on his system. Today SZEDER Gábor confirmed the cause: his system used the GnuTLS curl backend, which does not properly support OCSP, and switching to the OpenSSL backend makes t5585 succeed. Junio C Hamano thanked the contributors and noted the test's prerequisite setting still needs improvement.

### stash: in-core index merge for `apply --index`

*Long thread (root 2026/09/19/21-26-34): 21 replies so far from 3 participants over 7 days; 11 messages that day.*

D. Ben Knoble's four-patch series fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true`, replacing the file-system-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. Earlier, v2 added branch labels, ui merge options, and new tests, and Phillip Wood flagged a missing `merge_finalize()` leak. Today Junio C Hamano supplied a diff swapping the tree arguments to `merge_incore_nonrecursive()` to (merge_base, merge, head) plus a new test, noting "with the change to the code, it passes again." D. Ben Knoble confirmed the root cause was the differing parameter orders of the two merge functions, agreed to fix the confusing "Upstream index" label, and noted CI caught the leak. Phillip Wood said aligning the two functions' tree argument order is "definitely worth fixing," while D. Ben Knoble and Junio C Hamano declared that out of scope for this series.

### push: `--force-if-includes` with a reflog-less remote-tracking ref

*Long thread (root 2026/09/03/01-05-47): 20 replies so far from 6 participants over 23 days; 2 messages that day.*

Aleksei Sviridkin's patch fixes `is_reachable_in_reflog()` in the push code, where an uninitialized timestamp cut off the local reflog walk when the remote-tracking ref has no reflog, wrongly rejecting pushes with "remote ref updated since checkout"; the fix initializes the timestamp to 0. Earlier discussion debated whether 0 or a gc-based cutoff is the right fallback. Today Tyler Cipriani argued for `date=0` as always correct with usually equivalent runtime, giving a scenario where a cutoff yields the wrong answer; Junio C Hamano responded that with a never-expired reflog, using all available information works better than a cutoff.

### AsciiDoc cross-references and the tutorial rewrite

*Long thread (root 2026/09/22/19-29-02): 17 replies so far from 4 participants over 4 days; 8 messages that day.*

Julia Evans's patch changes documentation cross-references to the verbose `<<EXAMPLES,EXAMPLES>>` form so rendered text stays `EXAMPLES` across asciidoc implementations, at the cost of changing some heading HTML IDs. Today she reposted v2 with a rewritten commit message, and Jeff King said it looks good. Separately, she announced a plan to rewrite `gittutorial.adoc` covering only `git init`, `git add`, `git commit`, `git status`, `git diff`, `git push`, `git config`, and `git remote add`; Junio C Hamano said he is excited as long as learners do not end up reading three documents instead of two, agreed the current tutorial has outlived its time, and clarified that "replace" means dropping the stale file and freshly explaining salvageable topics elsewhere.

### New gitmergeconflicts(7) guide

*Long thread (root 2026/09/24/14-44-15): 12 replies so far from 5 participants over 2 days; 8 messages that day.*

Julia Evans's seven-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linking to it from the `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull` man pages. Today D. Ben Knoble flagged that the `git-merge.adoc` rewrite dropped what a conflict is and the `git log --merge -p <path>` tip; Julia Evans proposed adding the latter under "Tools to get more context", which Ben Knoble accepted. Junio C Hamano objected that the `git-cherry-pick.adoc` rewrite removed cherry-pick-specific details such as the `CHERRY_PICK_HEAD` pseudo-ref, and explained that `git log --merge` reads the index for conflicted paths; Ben Knoble added that the implied `HEAD...MERGE_HEAD` argument is the important half.

## The day in brief
SZEDER Gábor traced the t5585 failure that got the `http.sslVerifyStatus` topic reverted from `next` to the GnuTLS curl backend. Junio C Hamano found transposed tree arguments in the new in-core stash index merge. Julia Evans advanced both the AsciiDoc cross-reference patch and a plan to rewrite `gittutorial.adoc` down to eight commands.

## Notable threads

### `git history` during an interactive rebase breaks the repo
Nikita Bobko reported that running `git history` mid-rebase updates the branch currently being rebased, so `git rebase --continue` fails with a ref lock error and leaves a state even `git branch --force` cannot repair. He proposed five potential solutions, from forbidding `git history` during rebase to updating all branches except the one being rebased, and noted related interactions with cherry-pick, the rebase-todo file, and `--update-refs`.

### Registering temporary pack files for cleanup

Royce Remer posted a patch registering temp pack files from `git gc` and `git maintenance` in `pack-write.c` and `pack-bitmap-write.c` with the tempfile subsystem so they are unlinked on exit. Motivation: disk exhaustion in Kubernetes pods running Gitea mirrors, where killed gc left orphaned `tmp_pack_*` files (~60 GiB) and killed repacks left `.tmp-*-pack-*` files (~118 GiB, ~39 GiB).

### fetch: limited branch fetching in shallow repos
Harald Nordgren posted v3 of a four-patch series avoiding fetching every branch of a new remote in a shallow repository, now using a `remote.<name>.refmap` mechanism instead of v2's special `+:` refspec. Junio C Hamano reviewed the first two patches, calling the code good but objecting to always including the remote's default branch in the inferred fetch set, since Git should not fetch the remote's HEAD unasked.

### parse_revision_opt() bugfixes
Jeff King posted a two-patch series fixing `parse_revision_opt()` in `revision.c`: one stops malformed options like `--default` from producing a spurious "unknown option" message, the other reads the unknown option from the consolidated argv so `git shortlog -n --no-such-option` reports the option name instead of `(null)`. Junio C Hamano reviewed and said both patches make sense.

### GIT_WORK_TREE exported as '.' for the post-checkout hook
Phil Sainty reported that testing Junio C Hamano's patch shows `GIT_WORK_TREE` exported as `.` rather than the actual worktree path, so `git rev-parse --show-toplevel` still returns the CWD; he suggested `discovery.worktree` may not be the right value and pointed at `setup.c` lines setting that `.`.

## In brief
- Colin Hinton posted v3 of deferring `fetch.followRemoteHEAD` validation to `do_fetch()`, applying the `xstrdup_or_null` fix from review.
- Patrick Monette added `-S`/`--gpg-sign` signing to `git replay`, fixing a FIXME and a commit-failure handling bug.
- Brigham Campbell posted v4 of `git-contacts` reading a patch from stdin via `-`; Junio C Hamano accepted and asked for doc updates.
- Junio C Hamano argued `git fetch-pack` should not dwim its ref arguments and that an explicit way to send `want-ref <oid>` may be needed.
- Tamir Duberstein posted v2 using `cmp` for a huge-output test and aligning CI job counts across providers.
- Kaartic Sivaraam reviewed the series making `create_repository()`'s repo a pure out-parameter and said it looks good.
- Post-2.56 version naming discussion included a proposal for a tool recommending modern config alternatives.
- **Junio on CI warning patch**: Junio C Hamano said checking the correctness of the patch scoping perforce/git-lfs/JGit warnings to platforms that install them would be a "no brainer."

- Harald Nordgren posted a patch making leak-sanitizer CI failures point at the actual test file and line.
- Kristofer Karlsson posted v2 trimming whitespace-only trailing lines from `git log -L :funcname:` ranges.
- Johannes Sixt queued the git-gui Afrikaans translation.
- Kaartic Sivaraam advised using `--in-reply-to` from the next version of the t40* modernization series.
