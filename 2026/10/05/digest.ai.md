# Git mailing list daily digest for 2026/10/05

## Long threads

### New gitmergeconflicts manual page

*Long thread (root 2026/09/24/14-44-15): 36 replies so far from 6 participants over 12 days; 4 messages that day.*

This documentation series adds a new `gitmergeconflicts` man page explaining merge conflict resolution with examples, linked from `git-merge`, `git-rebase`, `git-cherry-pick`, `git-revert`, and `git-pull` instead of re-explaining the process in each command. Today Julia Evans revised the "OURS"/"THEIRS" section of the draft, defining "ours" as the current `HEAD` commit and "theirs" as the other side, covering the rebase "upside down" case and mapping the terms to index stages. Junio C Hamano asked what is in scope for the effort, proposing that minor code changes be considered if they make the docs easier to understand, such as updating conflict-marker labels and `git status`'s "deleted by us" phrasing. Julia Evans reported being surprised that `git status` recommends `git commit` rather than `git merge --continue` during a merge, proposed a NOTE calling `git commit` an older alternative, and brainstormed criticisms of the current conflict `git status` output.

### uploadpack.lazyFetchTrusted server-side lazy-fetch control

*Long thread (root 2026/07/10/08-51-34): 35 replies so far from 3 participants over 88 days; 1 message that day.*

This five-patch series introduces the protected `uploadpack.lazyFetchTrusted` config variable so server operators can mark trusted repositories whose promisor remotes may be lazy-fetched while serving clones and fetches, touching `builtin/upload-pack.c`, `promisor-remote.c`, `setup.c`, and `upload-pack.c`. Today Junio C Hamano noted that no one had provided more substantial reviews beyond his own nitpicky ones, and asked whether to declare that the point of diminishing returns had been reached and mark the topic for `next`.

### Fetching only tracked branches from a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 25 replies so far from 4 participants over 17 days; 2 messages that day.*

This series makes `git fetch` in a shallow repository avoid fetching every branch of a newly added remote, via a `remote.<name>.refmap` mechanism, inferring branches to fetch from local upstreams, a `git remote add --limited-fetch` option, and defaulting to it in shallow repositories. Today Junio C Hamano explained that he found the `--refmap` configuration error by thinking the process through while writing basic tests, noted that the topic textually interacts with Collin's followRemoteHAD work, and asked Harald Nordgren to double-check the result merged into `seen`. Harald range-diffed against `seen`, reported that everything looks normal, and asked whether he should hold off on sending a new version he has ready.

### CI annotations for test failures and leaks

*Long thread (root 2026/09/25/18-54-03): 23 replies so far from 4 participants over 11 days; 3 messages that day.*

This two-patch CI series links failure and leak annotations to the test script and stops leak-sanitizer jobs at their first failure via `--immediate`, so both can be found from the job summary. Today Phillip Wood agreed that the patch linking annotations to the test file should be dropped, because the current summary-page links to the test output are more useful for debugging than a diff that may not show the failed test, while affirming the leak patch is worth keeping. Harald Nordgren proposed instead showing the file name and line as part of the annotation text while leaving the linking as is, and Phillip Wood said he does not object.

### --force-if-includes checking the pushed ref

*Long thread (root 2026/09/04/21-01-20): 20 replies so far from 5 participants over 32 days; 1 message that day.*

This three-patch series makes `git push --force-if-includes` consult the reflog of the ref actually being pushed, rejects non-branch sources as unverifiable with a new `pushRefUnverifiable` advice, and defers the reflog check so valid fast-forward pushes are not rejected. Today Tyler Cipriani added Patrick Steinhardt to CC and addressed his review: since v4 the series resolves any passed source ref, checking that branch's reflog if it resolves to a branch and rejecting tags, oids, and detached HEAD as unverifiable. He noted a trivial textual conflict in `t5533` and a small conflict against the tip of `maint`, and offered to send a rebased v7.

## The day in brief

Junio C Hamano asked whether the `uploadpack.lazyFetchTrusted` series has reached diminishing returns and should be marked for `next`, and marked the gittutorial-2 removal for `next` after v2. The SHA-256 default transition debate intensified: Scott Chacon urged Git 3.0 ship without the default, Patrick Steinhardt countered that the ecosystem only moves once the breaking change is announced, and brian m. carlson argued the need is gradually urgent for large institutions.

## Notable threads

### Signing a SHA-256 tree digest in commits and tags
This RFC series adds a `tree-sha256` header computed over the full tree contents when signing commits or tags with SHA-256, proposed as an alternative to making SHA-256 the default object format in Git 3.0. Today Scott Chacon argued the SHA-256 default transition is not ready -- interop is unusable, pack index v3 and compat pieces are incomplete, and major forges lack practical support -- and urged shipping Git 3.0 without the default. Patrick Steinhardt countered that the ecosystem only began moving once the mandatory 3.0 change was announced. brian m. carlson replied that the need becomes gradually urgent for large institutions, that he has had inquiries from government agencies and corporations wanting SHA-256, and clarified that his SHA-256 work was done in a personal capacity.

### Defaulting blame to .git-blame-ignore-revs
This patch makes `git blame` default to ignoring revisions listed in `.git-blame-ignore-revs`, which has become the de facto standard across hosting platforms. Today Junio C Hamano argued that defaulting to the working-tree file would not match hosting sites, which operate on the `HEAD:.git-blame-ignore-revs` blob, and proposed adding that blob as the initial ignore-revs entry in bare and non-bare repositories alike; he also flagged a new security threat vector and said the parser must be audited. Ravi Mistry agreed that reading the committed blob makes sense and proposed parser tightenings for v2, including rejecting lines with NUL bytes and avoiding lazy promisor fetches in partial clones.

### Incremental connectivity check
This series adds an incremental mode for the connectivity check behind `transfer.connectivityCheck=incremental`, verifying incoming trees against trusted parent trees instead of marking all reachable objects uninteresting, so cost scales with the number of changes rather than preexisting objects. Today Patrick Steinhardt's review asked the documentation to spell out the reverse invariant about unreferenced objects, warned against referring to code like `check_connected()` in docs, questioned whether the approach helps when there are many references, and objected to `die()` on cycle detection since receive-pack callers expect connectivity errors to bubble up. Junio C Hamano wondered whether connectivity roots other than refs, such as index entries used by fsck, should also be paid attention to.

### Running ref copy and rename through transactions
This patch represents reference copy and rename operations as reference transactions so the reference-transaction hook sees the logical updates, retaining backend-specific reflog handling for the files and reftable backends. Today Patrick Steinhardt identified reflog caveats -- no way to delete many old reflog entries yet and potential performance impact -- rejected the source/destination revalidation as not sensible because the "preparing" hook runs before locking by design, and said the patch is not in a shape ready for review and needs to be split up.

### Removing gittutorial-2
This two-patch documentation series replaces the stale `gittutorial-2` page with a short stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`, and updates references in six other documentation files. Today Julia Evans posted v2 with reworded commit messages; Junio C Hamano said the titles look much better, that he will replace the topic, and marked it for `next`. Kristoffer Haugsbakk pointed out a typo in the first commit message.

## In brief
- Hanan Arshad retracted the `git stash publish` proposal after Johannes Sixt and Junio C Hamano noted that branches already cover sharing, narrowing the RFC to porcelain for `git stash list --remote`, `git stash get`, and `git stash remove`.
- Phillip Wood posted v2 of "checkout -m: recreate conflict labels", writing conflict labels to `.git/MERGE_LABELS` for reuse; Johannes Sixt argued against also remembering the conflict style.
- Patrick Steinhardt explained that in a shallow clone the boundary commit looks like a root commit, so reverting it deletes every file, and agreed Git should warn about or refuse editing shallow boundary commits by default.
- Patrick Steinhardt corrected the characterization of the stale delta base cache bug: the cache key stores a freed `packed_git` pointer's address but never dereferences it, so ASan cannot reproduce it.
- Mark C. Chu-Carroll posted v2 of filtering options for `git repo structure`, rewritten to accept include/exclude revision queries as understood by `git log`.
- Junio C Hamano judged Colin Hinton's v5 deferred `fetch.followRemoteHEAD` validation very good, with only a commit-message wording fix and a leaked string remaining.
- Devi Srinivas Vasamsetti submitted a patch adding an `advice.clockSkew` setting that warns when a new commit is dated before its parent.
- Johannes Schindelin announced Git for Windows 2.56.0(2), with OpenSSL v3.5.9 and fixes for `GIT_CONFIG_GLOBAL=NUL` suppression, lost Authenticode signatures, and a client-certificate push assertion.
