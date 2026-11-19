# Git mailing list daily digest for 2026/10/05

## Long threads

### New `gitmergeconflicts` manual page

*Long thread: 36 replies so far from 6 participants over 12 days; 4 messages that day.*

Julia Evans's series adds `Documentation/gitmergeconflicts.adoc`, a manual page explaining merge conflicts with examples, linked from `git-merge`, `git-pull`, `git-rebase`, `git-revert` and `git-cherry-pick`. Today Julia revised the "OURS"/"THEIRS" section, defining "ours" as the current `HEAD` commit and mapping the terms to index stages 1/2/3. Junio C Hamano asked what is in scope, proposing minor code changes such as richer conflict-marker labels if they make the docs clearer; Julia noted `git status` recommends `git commit` rather than `git merge --continue` during a merge and proposed a NOTE calling it an older alternative. Earlier reviews from Patrick Steinhardt and Junio shaped the page and its new "WHAT IS A MERGE CONFLICT?" section.

### `uploadpack.lazyFetchTrusted` config for server-side lazy fetch

*Long thread: 35 replies so far from 3 participants over 88 days; 1 message that day.*

Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones, touching `builtin/upload-pack.c`, `upload-pack.c`, `promisor-remote.c` and `setup.c`. Today Junio C Hamano noted no one had reviewed beyond his own nitpicky v4 review and asked whether to declare diminishing returns and mark the topic for `next`. Earlier, v5 was rebased on Git 2.56 with documentation and test updates, including a NEEDSWORK about silently ignoring misspelled paths.

### Limited fetch for new remotes in shallow repositories

*Long thread: 25 replies so far from 4 participants over 17 days; 2 messages that day.*

Harald Nordgren's series makes a new remote in a shallow repository fetch only tracked branches, adding `remote.<name>.refmap` and `git remote add --limited-fetch`, defaulting to that mode in shallow repos. Today Junio C Hamano said he found the `--refmap` configuration error while writing basic tests, noted the topic textually interacts with Collin's followRemoteHAD work in `seen`, and asked Harald to double-check the merge result; Harald range-diffed against `seen` and reported everything looks normal. Earlier, Harald proposed that `remote.<name>.fetch` should win over `refmap` when both are set.

### CI failure and leak annotations

*Long thread: 23 replies so far from 4 participants over 11 days; 3 messages that day.*

Harald Nordgren's series links GitHub Actions failure and leak annotations to the test script in `t/test-lib-github-workflow-markup.sh`. Today Phillip Wood agreed the file/line linking patch should be dropped because links to the test output are more useful for debugging, while keeping the leak patch; Harald proposed instead putting the file name and line in the annotation text, which Phillip did not object to. Earlier, v5 removed percent-escaping after CI verification, and Phillip had noted the file/line link does not navigate to the test file when the file is unchanged in the PR.

### `--force-if-includes` checks the pushed ref

*Long thread: 20 replies so far from 5 participants over 32 days; 1 message that day.*

Tyler Cipriani's series makes `git push --force-if-includes` consult the reflog of the ref actually being pushed in `remote.c`, rejecting non-branch sources as unverifiable and deferring the check so fast-forward pushes are not rejected. Today Tyler added Patrick Steinhardt to CC and addressed his review: since v4 any source ref is resolved, and tags, oids and detached HEAD are rejected as unverifiable because their reflogs lack the integration information the check needs. He noted a textual conflict in `t5533` and offered a rebased v7. Earlier, D. Ben Knoble had flagged that rejecting non-branch pushes breaks workflows that only appear to work today.

## The day in brief

Junio C Hamano asked whether the `uploadpack.lazyFetchTrusted` topic has reached diminishing returns and should be marked for `next`, while Harald Nordgren confirmed the shallow-repo limited-fetch series range-diffs cleanly against `seen` after a textual interaction with another in-flight topic. The new `gitmergeconflicts` manual page was revised, the CI failure-annotation series discussed dropping the file/line link patch, and the SHA-256 signing RFC drew a sharp exchange over whether the SHA-256 default is ready for Git 3.0.

## Notable threads

### Signing a SHA-256 tree digest in commits and tags
Scott Chacon's RFC adds a `tree-sha256` header over the full tree when signing with `git commit -S --hash=sha256` or `git tag -s --hash=sha256`, as an alternative to a SHA-256 default object format. Today Scott argued the SHA-256 default transition is not ready for Git 3.0 and urged keeping it opt-in; Patrick Steinhardt countered that the ecosystem only moved once the mandatory 3.0 change was announced; brian m. carlsson replied that large institutions have asked for SHA-256 and noted his work was done personally with wide help.

### Defaulting `git blame` to `.git-blame-ignore-revs`
Ravi Mistry's patch makes `git blame` ignore revisions in a default `.git-blame-ignore-revs` file. Today Junio C Hamano proposed reading the `HEAD:.git-blame-ignore-revs` blob instead of the working-tree file to match hosting sites, and flagged that the parser would now read upstream-controlled content and must be audited. Ravi agreed and proposed v2 tightenings: rejecting lines with embedded NUL bytes and avoiding lazy promisor fetches when peeling tags in partial clones.

### Stale delta base cache entries in `packfile.c`
Patrick Steinhardt's series fixes corruption from delta base cache entries never evicted when their pack closes, reproducible via recursive submodule merges. Today Patrick corrected Jeff King's earlier suggestion that this was a use-after-free ASan would find: the cache key stores the freed `packed_git` address but never dereferences it, and ASan cannot reproduce it because it hands out unique addresses.

## In brief
- Patrick Steinhardt explained that in a shallow clone the boundary commit is rewritten to have no parents, so reverting it deletes all files, confirmed no safeguard warns when all tracked files are deleted, and agreed Git should warn about or refuse editing a shallow boundary commit by default.
- On ZIP timestamp bugs, René Scharfe explained DOS timestamps cover only 1980 onward while Unix timestamps start at 1970, suggested clamping out-of-range ZIP values instead of wrapping, and noted `fast-import` raw mode does little validation.
- Phillip Wood posted v2 of "checkout -m: recreate conflict labels", writing labels to `.git/MERGE_LABELS`; Johannes Sixt argued against also remembering the conflict style.
