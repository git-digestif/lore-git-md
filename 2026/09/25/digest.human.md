# Git mailing list daily digest for 2026/09/25

## Long threads

### http.sslVerifyStatus OCSP stapling in the HTTP transport

*Long thread: 31 replies so far from 5 participants over 46 days; 2 messages that day.*

Grayson Gordon's series adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off because enabling it would block connections to servers that do not staple. Earlier, the combined v7 was reverted out of `next` after SZEDER Gábor reported the new t5585 test failing on his system. Today SZEDER Gábor confirmed his setup used the GnuTLS curl backend, which does not properly support OCSP, and that switching to the OpenSSL backend makes t5585 pass; Junio C Hamano thanked the contributors and noted the test's prerequisite setting still needs improvement.

### In-core index merge in builtin/stash.c

*Long thread: 21 replies so far from 3 participants over 7 days; 11 messages that day.*

D. Ben Knoble's four-patch series fixes a bug in the interaction between autostashing, staged index entries, and `stash.index=true` by replacing the file-system-based index merge in `builtin/stash.c` with an in-core `merge_incore_nonrecursive()` call. Earlier, Phillip Wood flagged a missing `merge_finalize()` leak and Junio C Hamano questioned the tree argument order. Today Junio diagnosed that the new test fails because the trees are passed in the wrong order for `merge_incore_nonrecursive()`, whose parameter order differs from `merge_ort_nonrecursive()`; he supplied a fix plus a new test, and D. Ben Knoble confirmed the root cause, agreed to fix the confusing "Upstream index" label, and noted CI caught the leak. Phillip Wood agreed the two functions should take trees in the same order, but both agreed that change is out of scope for this series.

### --force-if-includes with a reflog-less remote-tracking ref

*Long thread: 20 replies so far from 6 participants over 23 days; 2 messages that day.*

Aleksei Sviridkin's patch fixes `push --force-if-includes` rejecting valid pushes when the remote-tracking ref has no reflog, by initializing the cutoff timestamp to 0 in `is_reachable_in_reflog()` so the local reflog walk is not cut off at the first entry. Earlier discussion weighed 0 against a gc-based cutoff. Today Tyler Cipriani argued for `date=0` as always correct with usually equivalent runtime, giving a clone/reset/wait/commit/push scenario where a cutoff produces the wrong answer; Junio C Hamano responded that with a never-expired reflog, using all available information works better than any cutoff.

### AsciiDoc cross-references and the tutorial rewrite in Documentation/

*Long thread: 17 replies so far from 4 participants over 4 days; 8 messages that day.*

Julia Evans's patch replaces "see EXAMPLES below" with `<<EXAMPLES,EXAMPLES>>` cross-references in the man pages so the rendered text is controlled across asciidoc implementations, at the cost of changing some heading HTML IDs. Today she reposted v2 with a corrected commit message and Jeff King said it looks good to him; she also announced a planned rewrite of `gittutorial.adoc` covering only `git init`, `git add`, `git commit`, `git status`, `git diff`, `git push`, `git config`, and `git remote add`. Junio C Hamano agreed the current tutorial has outlived its time but stressed that new guides must be sufficient to replace it, with removing the stale document as the primary goal.

### New gitmergeconflicts(7) guide

*Long thread: 12 replies so far from 5 participants over 2 days; 8 messages that day.*

Julia Evans's seven-patch series adds `Documentation/gitmergeconflicts.adoc` explaining how to handle merge conflicts and links to it from the `git merge`, `git rebase`, `git revert`, `git cherry-pick`, and `git pull` pages. Today D. Ben Knoble flagged that the `git-merge.adoc` rewrite dropped both an explanation of what a conflict is and the `git log --merge -p <path>` tip; Julia Evans proposed restoring the latter under "Tools to get more context", and Junio C Hamano objected that the `git-cherry-pick.adoc` rewrite removed cherry-pick-specific details such as `CHERRY_PICK_HEAD`.

## The day in brief
The `http.sslVerifyStatus` OCSP failure was traced to a GnuTLS curl backend without OCSP support, with the test prerequisite still needing work. D. Ben Knoble's in-core stash index merge was diagnosed as transposed tree arguments in `merge_incore_nonrecursive()`, with a fix and new test supplied. Jeff King fixed two `parse_revision_opt()` bugs so unknown options are reported by name instead of `(null)`, and Julia Evans advanced both the AsciiDoc cross-reference patch and a plan to rewrite the Git tutorial.

## Notable threads

### git-history during an interactive rebase
Nikita Bobko reported that running `git history` mid-rebase updates the branch currently being rebased, so `git rebase --continue` fails with a ref lock error and leaves a state even `git branch --force` cannot repair. He proposed five possible fixes, from forbidding the command during rebase to updating all branches except the one being rebased, and noted related interactions with cherry-pick, the `git-rebase-todo` file, and `rebase --update-refs`.

### GIT_WORK_TREE missing for the post-checkout hook
Phil Sainty reported that in a worktree the post-checkout hook sees `GIT_DIR` but not `GIT_WORK_TREE`, so `git rev-parse --show-toplevel` returns the CWD instead of the real toplevel. Today he tested Junio C Hamano's patch and found `GIT_WORK_TREE` is exported as `.` rather than the actual worktree path, so the problem persists; he pointed at setup.c as the source of the `.` value.

### parse_revision_opt() bugfixes in revision.c
Jeff King posted two fixes for `parse_revision_opt()`: treating a negative return from `handle_revision_opt()` as an already-reported error so malformed options are no longer called "unknown", and reading the unknown option from the consolidated argv so `git shortlog -n --no-such-option` reports the option name instead of `(null)`. Junio C Hamano reviewed the series and said both patches make sense.

## In brief
- Royce Remer registers the temporary pack files from `pack-write.c` and `pack-bitmap-write.c` with the tempfile subsystem so killed `git gc`/`git maintenance` runs no longer leave orphaned files in `objects/pack/`.
- Harald Nordgren posted v3 of the shallow-repo fetch series, replacing the special `+:` refspec with a `remote.<name>.refmap` mechanism; Junio C Hamano objected to always including the remote's default branch in the inferred fetch set.
- Patrick Monette added `-S`/`--gpg-sign` signing to `git replay` and fixed `pick_regular_commit()` to mark commit-creation failures.
- Colin Hinton posted v3 deferring `fetch.followRemoteHEAD` validation to `do_fetch()`, applying the `xstrdup_or_null` fix.
- SZEDER Gábor advised that custom `EXTRA_CPPFLAGS` affecting `git-compat-util.h` should also add `-Wno-invalid-pch` when using the precompiled header.
- Tamir Duberstein posted v2 using `cmp` for the huge t4205 comparison and aligning CI job counts across providers.
- Brigham Campbell posted v4 of `git-contacts` reading a patch from stdin via `-`; Junio C Hamano accepted the approach and asked for a doc update.
- Nathan Froyd's patch sets `exact_oid` in `builtin/fetch-pack.c`; Junio C Hamano argued fetch-pack should pass its ref arguments through exactly as given.
