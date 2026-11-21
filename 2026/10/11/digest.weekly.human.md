## Long threads

### New `gitmergeconflicts(7)` manual page

*Long thread: 54 replies so far from 6 participants over 17 days.*

Julia Evans's six-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of repeating guidance in each command. The series reached v2 on 2026/10/09, when Junio C Hamano objected that `Reviewed-by:` trailers on patch 1/6 were added by the author rather than offered by reviewers, and Julia agreed to use `Helped-by:` instead. Junio reviewed each patch, suggesting the new page list every merge-conflict-causing command explicitly -- including `git am -3` and `git checkout -m` -- and use formal constructions rather than contractions. By the end of the period the series had absorbed extensive review and stood ready for further iteration.

### `uploadpack.lazyFetchTrusted` config for server-side lazy fetch

*Long thread: 36 replies so far from 3 participants over 89 days.*

Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones. On 2026/10/05 Junio C Hamano asked whether the topic had reached diminishing returns, and on 2026/10/06 Christian agreed the v5 series should be marked for `next`, since it adds mostly refactorings and a new protected config variable. The topic thus graduated from a long-running discussion to the integration queue by midweek.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 33 replies so far from 5 participants over 59 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off. After earlier reverts from `next` over a t5585 failure, Junio C Hamano bisected linux-* CI jobs spinning until their 6-hour timeout to this topic and tentatively ejected it from `seen` on 2026/10/07. On 2026/10/08 he confirmed in isolation on Git 2.56 that the topic alone causes the timeouts, solidifying its removal from the integration branch.

### Fetch refmap inference for shallow repositories

*Long thread: 31 replies so far from 4 participants over 22 days.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository by adding `remote.<name>.refmap`, inferring branches from local upstreams, and defaulting to limited fetch in shallow clones. The series progressed through v7 on 2026/10/07, which fixed a fatal `--refmap` error, to v8 on 2026/10/10, which split out a preliminary `collect_upstream_from_remote()` helper in `remote.c` and threaded `struct repository *` through instead of hardcoding `the_repository`. Junio C Hamano reviewed v8, objecting that duplicating current-branch upstream logic belonged to a following step, and asked whether dropping a NULL-branch guard was intended; Harald acknowledged the oversight.

### Rerere waiting on `MERGE_RR.lock`

*Long thread: 29 replies so far from 4 participants over 40 days.*

Thomas Bachem's series makes rerere wait up to `rerere.lockTimeout` milliseconds for `MERGE_RR.lock` before failing, and adds a hidden `--skip-locked` option to `git rerere gc` so auto-maintenance does nothing while the lock is held. v7 posted on 2026/10/10 dropped the earlier patch letting conflict-stopping commands proceed without rerere. On 2026/10/11 Junio C Hamano reviewed patch 1/2, accepting the millisecond unit and the call to `repo_hold_lock_file_for_update_timeout()`, while questioning a doc sentence and test details that Thomas agreed to adjust.

### Keeping `die()` out of the trace2 API

*Long thread: 26 replies so far from 6 participants over 84 days.*

This series introduces a root-level `banned-die.h` header included by trace2 API sources, banning die-capable helpers one at a time so low-level tracing code cannot trigger a recursive die. On 2026/10/06 Derrick Stolee said the topic had sat idle because the `banned-die.h` protections remain incomplete, and that his RFC for making strbuf and friends more robust was similarly unsatisfying, though alternative ideas exist. The thread remained open without resolution by the end of the period.

### CI failure and leak annotations

*Long thread: 24 replies so far from 4 participants over 12 days.*

Harald Nordgren's series links GitHub Actions failure and leak annotations to the test script in `t/test-lib-github-workflow-markup.sh`. On 2026/10/06 he posted a revision reverting direct file/line linking -- which fails when the error points to a file unchanged in the PR -- in favor of showing the file and line as plain text. Phillip Wood reviewed it and said "Lets get this merged!", indicating the series had reached a mergeable state.

### Reference copy and rename through transactions

*Long thread: 22 replies so far from 6 participants over 22 days.*

Maciej Ciemborowicz's four-patch series routes reference copy and rename through ordinary transactions so the reference-transaction hook sees the whole operation, fixing a v3 regression where a packed-refs error was overwritten. On 2026/10/08 he posted v4 preserving that error, but Patrick Steinhardt warned that posting new versions without replying to reviewers is a red flag, and Karthik Nayak said he stopped reviewing the author's other patches because reviews were met with new versions rather than engagement. On 2026/10/10 Maciej offered to let someone else write the patches if that would be more efficient.

## The period in brief

The week of 2026/10/05 through 2026/10/11 saw heavy traffic across seven active days, with documentation, CI infrastructure, and long-running plumbing topics dominating the list. Readers should not miss Taylor Blau's notes from the 2026 Git Contributor's Summit outlining plans for Git 3.0 in March 2027, Patrick Steinhardt's packfile corruption fix marked for `next`, and the heated debate over whether to allow AI-assisted contributions under the DCO.

## Key developments

### Notes from the 2026 Git Contributor's Summit

Taylor Blau shared notes on 2026/10/06 covering seven topics: the security process, Git 3.0 timing (with consensus to release 2.99.1 and 3.0 together in March 2027 with only `BREAKING_CHANGES` defaults flipped), documentation, Outreachy, pluggable ODB, AI policy, and protocol v2 for pushes. The notes provided the clearest public signal yet on the project's near-term release schedule and architectural direction.

### Packfile corruption from stale delta base cache entries

Patrick Steinhardt's two-patch series fixes corruption caused by delta base cache entries never evicted when their pack closes, reproducible via recursive submodule merges. On 2026/10/07 he confirmed the reproducer also fails on Alpine with musl, Jeff King called the fix obviously correct, and Junio C Hamano marked the topic for `next`. The fix addresses a subtle memory-lifetime bug where the cache key stores a freed `packed_git` address.

### AI-assisted contributions policy debate

Scott Chacon's RFC proposing to allow responsible AI-assisted contributions under the existing DCO with human review and an `Assisted-by:` trailer drew sharp disagreement. brian m. carlsson objected that the DCO cannot honestly be asserted for LLM output without knowing all training sources, while Junio C Hamano critiqued the text as resting on an OpenAI model's output and giving maintainers less latitude than kernel guidelines. Patrick Steinhardt argued the review bottleneck is tightening as AI-generated slop increases, warning that loosening the policy without solutions would make the problem worse.

### `git blame` defaulting to `HEAD:.git-blame-ignore-revs`

Ravi Mistry posted v3 of his series on 2026/10/11 making `git-blame` and `git-annotate` automatically use the `HEAD:.git-blame-ignore-revs` blob when it exists, so local runs match hosting platforms. Junio C Hamano said v3 looks good and would mark the topic ready for `next` unless objections arrived within a few days. Earlier iterations had addressed security concerns about parsing upstream-controlled content by rejecting lines with embedded NUL bytes and avoiding lazy promisor fetches.

### `git repack --dry-run` no longer rebuilds packs

Siddharth Shrimali's patch fixes `git repack --drop-filtered --dry-run`, which was documented to list objects without rebuilding any pack but actually wrote a new promisor pack and deleted old ones because the `--dry-run` block fell through into regular repack code. On 2026/10/11 Junio C Hamano marked the topic for `next` after D. Ben Knoble offered his `Reviewed-by:`.

### Git 2.98 cycle begins

On 2026/10/10 Junio C Hamano flushed topics waiting in `next` down to `master` to start the Git 2.98 cycle and rewound the `next` branch, marking the formal start of the new development window.

### Push resending common history after a pre-push repack

Jens Röcker reported on 2026/10/06 that a push resends common history when a pre-push hook repacks the local object database and removes loose common objects, reproduced on Apple Git 2.54.0 and upstream Git 2.56.0. On 2026/10/07 Patrick Steinhardt confirmed the mechanism -- `odb_has_object()` false negatives after a pre-push repack -- and provided an untested `send-pack.c` patch batching the negative-object checks.

## In brief

**SHA-256 signing RFC** -- Scott Chacon proposed adding a `tree-sha256` header over the full tree when signing with `--hash=sha256`, arguing the SHA-256 default transition is not ready for Git 3.0; Patrick Steinhardt countered that the ecosystem only moved once the mandatory 3.0 change was announced.

**SSH signing trust limits** -- Christian Ramos reported that `valid-before` is checked against a date the signer controls and that a missing `gpg.ssh.revocationFile` fails open; Patrick Steinhardt suggested failing by default on a missing revocation file with an optional escape hatch, and Phillip Wood endorsed both points.

**`git status` suggesting `git merge --continue`** -- Julia Evans's v3 patch changing the merge-conflict advice in `wt-status.c` was marked for `next` by Junio C Hamano on 2026/10/09 with Phillip Wood's `Acked-by:`.

**`checkout -m` recreating conflict labels** -- Phillip Wood posted v3 writing conflict labels to `.git/MERGE_LABELS`; Junio C Hamano reviewed it as "nicely done" and said "Will replace."

**`fast-import` lookup crash** -- Marc Becker fixed a crash when the `from` command references a commit created during the same import session; Aniket Gupta tested it and requested a test in `t9300`.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll posted v4 adding a `[<revs>...]` argument to limit the set of objects processed.

**Status conversion disagreement** -- Curtis Allen Smith's series fixes `git status` disagreeing with `git diff` and `git add` when a clean filter is active, because `ie_match_stat()` trusts a size difference as proof of change.

**Contributor Summit follow-ups** -- Taylor Blau's notes also covered pluggable ODB and protocol v2 for pushes as active summit topics.

**Small fixes and cleanups** -- René Scharfe analyzed ZIP timestamp bugs and proposed clamping out-of-range values; Karthik Nayak posted v4 of the packed-refs `fwrite()` optimization; Kristofer Karlsson posted v3 of "fetch: write commit-graph using updated refs only"; Marc Becker posted v2 of a wincred refactor; Xiangfei Ding fixed `imap-send.c` to use `ASN1_STRING_length`; and Jon Simons documented the preference for submitting bug-fix tests in the same commit as the code change.