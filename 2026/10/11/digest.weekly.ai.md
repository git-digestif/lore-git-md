## Long threads

### New `gitmergeconflicts(7)` manual page

*Long thread: 54 replies so far from 6 participants over 17 days.*

Julia Evans's six-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of repeating guidance in each command. The series reached v2 during the week, with Junio C Hamano reviewing each patch and objecting to author-added `Reviewed-by:` trailers in favor of `Helped-by:`. He suggested changes including co-author sign-off, explaining the `|||||||` marker, treating `zdiff3` as a failed experiment, and listing every conflict-causing command explicitly rather than ending with "and other operations". By week's end the series had absorbed detailed review and was approaching readiness.

### `uploadpack.lazyFetchTrusted` server-side lazy-fetch control

*Long thread: 36 replies so far from 3 participants over 89 days.*

Christian Couder's series introduces a protected `uploadpack.lazyFetchTrusted` config so server operators can mark repositories whose promisor remotes may be lazy-fetched during clones. After 89 days, Junio C Hamano asked whether the topic had reached diminishing returns, and Christian agreed the v5 series -- mostly refactorings plus the new config variable -- should be marked for `next`. The topic touches `builtin/upload-pack.c`, `upload-pack.c`, `promisor-remote.c`, and `setup.c`, and includes a NEEDSWORK about silently ignoring misspelled paths.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 33 replies so far from 5 participants over 59 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off. The topic was reverted out of `next` after a t5585 failure on a GnuTLS backend, and during the week Junio C Hamano bisected linux-* CI jobs spinning to 6-hour timeouts directly to this topic, confirming in isolation on Git 2.56 that it causes the timeouts. He tentatively ejected it from `seen`, noting the linux-TEST-vars/linux32 failures were unrelated Ubuntu 20.04 download issues.

### Fetching fewer branches from a new remote in shallow repos

*Long thread: 31 replies so far from 4 participants over 22 days.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository by adding `remote.<name>.refmap`, inferring branches from local upstreams, and defaulting to `--limited-fetch` in shallow clones. The week saw v7 fix a fatal `--refmap` configuration error and v8 split the work into a preliminary `collect_upstream_from_remote()` helper in `remote.c` plus a simplified refmap-inference commit, threading `struct repository *` through instead of hardcoding `the_repository`. Junio C Hamano reviewed v8, objecting that duplicating current-branch upstream logic belonged to the following step, and reported that combining the series with `seen` breaks t5505 and t5586 via interaction with the `ch/fetch-followremotehead-lazy-validation` topic.

### Rerere waiting on `MERGE_RR.lock`

*Long thread: 29 replies so far from 4 participants over 40 days.*

Thomas Bachem's series makes rerere wait up to `rerere.lockTimeout` milliseconds (default 1000) for `MERGE_RR.lock` before failing, and adds a hidden `--skip-locked` option to `git rerere gc` so auto-maintenance does nothing while the lock is held. The week saw v7 drop the earlier patch letting conflict-stopping commands proceed without rerere, and Junio C Hamano reviewed patch 1/2, accepting the millisecond unit and the call to `repo_hold_lock_file_for_update_timeout()` while questioning a doc sentence and test details that Bachem agreed to fix.

### Keeping `die()` out of the trace2 API

*Long thread: 26 replies so far from 6 participants over 84 days.*

This series introduces a root-level `banned-die.h` header included by trace2 API sources, banning die-capable helpers one at a time so low-level tracing code cannot trigger a recursive die. Derrick Stolee noted the topic has sat idle because the protections are incomplete, and that his RFC for making strbuf and friends more robust was similarly unsatisfying, though interesting alternative ideas exist.

### Linking CI failure and leak annotations to test scripts

*Long thread: 24 replies so far from 4 participants over 12 days.*

Harald Nordgren's series makes GitHub Actions annotations for test failures and leaks name the test script and line. After earlier debate about dropping the file/line linking patch because links fail when the file is unchanged in the PR, Nordgren posted a revision showing file and line as plain text in the annotation, omitting line numbers for leaks where they are unavailable. Phillip Wood reviewed it and said "Lets get this merged!"

### Running ref copy and rename through transactions

*Long thread: 22 replies so far from 6 participants over 22 days.*

Maciej Ciemborowicz's four-patch series routes reference copy and rename through ordinary transactions so the reference-transaction hook sees the whole operation, fixing v3 regressions in t0600/t5510 where a packed-refs error was overwritten. Patrick Steinhardt warned that posting new versions without replying to reviewers is a red flag, and the thread debated AI-assisted workflows. Ciemborowicz offered to let someone else write the patches if that would be more efficient, while brian m. carlsson advised that reporting the bug without a patch is valuable if the guidelines cannot be met.

## The period in brief

The week of 2026/10/05 through 2026/10/11 saw seven active days of heavy, eventful traffic on the Git mailing list. Readers should not miss Taylor Blau's notes from the 2026 Contributor's Summit outlining a possible Git 3.0 release in March 2027, the heated debate over Scott Chacon's RFC to allow responsible AI-assisted contributions, and Julia Evans's `gitmergeconflicts(7)` documentation series reaching v2 with detailed maintainer review.

## Key developments

### Git 3.0 planning from the 2026 Contributor's Summit

Taylor Blau shared notes from the summit covering seven topics, including a consensus to release Git 2.99.1 and 3.0 together in March 2027 with only `BREAKING_CHANGES` defaults flipped, plus discussions of the security process, pluggable ODB, AI policy, and protocol v2 for pushes. The notes provide the clearest public signal yet on the timing and scope of the next major version.

### AI assistance policy debate

Scott Chacon's RFC patch to `Documentation/SubmittingPatches` proposed allowing AI-assisted contributions under the existing DCO with human review and an `Assisted-by:` trailer, aligning with SFC, Linux, Xen, and Debian guidance. The proposal drew sharp objections: brian m. carlsson argued the DCO cannot honestly be asserted for LLM output without knowing all training sources, Luca Milanesio argued fully AI-generated contributions should be rejected, and Junio C Hamano critiqued the text as resting on an OpenAI model's output and giving maintainers less latitude than kernel guidelines. Patrick Steinhardt warned that loosening the AI policy without solutions would worsen the review bottleneck as AI-generated slop increases.

### SHA-256 signing and collision detection

Scott Chacon posted an RFC adding a `tree-sha256` header over the full tree when signing with `--hash=sha256`, arguing the SHA-256 default transition is not ready for Git 3.0 and should remain opt-in; Patrick Steinhardt countered that the ecosystem only moved once the mandatory 3.0 change was announced. Separately, Chacon's four-patch series porting Sam Reis's sha1dc Rust crate to C in `sha1dc-accel/` claimed roughly 2.7x faster hashing, with D. Ben Knoble raising license compatibility questions that Reis clarified by noting the UBC tables are autogenerated, and Junio C Hamano confirming Git's GPL-2 can incorporate MIT-licensed code.

### Defaulting `git blame` to `.git-blame-ignore-revs`

Ravi Mistry posted v3 of the series making `git-blame` and `git-annotate` automatically use the `HEAD:.git-blame-ignore-revs` blob when it exists, so local runs match hosting platforms without per-clone configuration. Junio C Hamano had earlier proposed reading the blob instead of the working-tree file and flagged that the parser would read upstream-controlled content requiring audit; Mistry agreed to v2 tightenings including rejecting lines with embedded NUL bytes. By week's end Junio said v3 looks good and would be marked ready for `next` unless objections arrived.

### Packfile corruption from stale delta base cache entries

Patrick Steinhardt's two-patch series fixes corruption from delta base cache entries never evicted when their pack closes, reproducible via recursive submodule merges. He corrected Jeff King's earlier suggestion that this was a use-after-free ASan would find, explaining the cache key stores the freed `packed_git` address but never dereferences it. Jeff King called the fix obviously correct, and Junio C Hamano marked the topic for `next`.

### `repack --dry-run` no longer rebuilds packs

Siddharth Shrimali's patch fixes `git repack --drop-filtered --dry-run`, which was documented to list objects without rebuilding but actually wrote a new promisor pack and deleted old packs because the `--dry-run` block fell through into regular repack code. His v2 returns right after listing candidates, and D. Ben Knoble offered his `Reviewed-by:`. Junio C Hamano marked the topic for `next`.

### SSH signing trust limits

Christian Noé Ramos López reported that `valid-before` is checked against the committer/tagger date the signer controls, so a backdated commit bypasses key expiry, and that a configured but missing `gpg.ssh.revocationFile` only warns while verification proceeds. Patrick Steinhardt argued there is no proper fix for the timestamp in a distributed system but agreed documentation should avoid a false sense of trustworthiness, and suggested failing by default on a missing revocation file with a `:(optional)` path prefix as an escape hatch; Phillip Wood endorsed both points.

### Push resending common history after a pre-push repack

Jens Röcker reported that a push resends common history when a pre-push hook repacks the local object database and removes loose common objects, reproduced on Apple Git 2.54.0 and upstream Git 2.56.0 with receiver pack sizes of 4,196,026 bytes versus 300 bytes without the repack. Patrick Steinhardt confirmed the mechanism as `odb_has_object()` false negatives after a pre-push repack and provided an untested `send-pack.c` patch batching the negative-object checks.

## In brief

**`git status` suggesting `git merge --continue`** -- Julia Evans posted v3 changing the merge-conflict advice in `wt-status.c` from `git commit` to `git merge --continue`, consistent with rebase, revert, and cherry-pick; Junio C Hamano said he will apply it with Phillip Wood's `Acked-by:` and mark the topic for `next`.

**`checkout -m` recreating original conflict labels** -- Phillip Wood posted v3 of his series teaching the ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` recreates conflicts with original labels; Junio reviewed it as "nicely done" and said "Will replace."

**Shallow clone boundary commit revert warning** -- Patrick Steinhardt explained that in a shallow clone the boundary commit is rewritten to have no parents, so reverting it deletes all files, and agreed Git should warn about or refuse editing a shallow boundary commit by default.

**ZIP timestamp bugs** -- René Scharfe explained DOS timestamps cover only 1980 onward while Unix timestamps start at 1970, suggested clamping out-of-range ZIP values, and recommended rejecting non-representable timestamps in `git archive --format=zip`.

**`odb/source-files` alternates series** -- Karthik Nayak reviewed the 13-patch series, accepted Patrick Steinhardt's explanations, and said the whole series looks good.

**Fetch commit-graph using updated refs only** -- Kristofer Karlsson posted v3 with benchmarks showing updated-ref fetches dropping from about 357 ms to under 10 ms.

**Packed-refs `fwrite()` optimization** -- Karthik Nayak posted v4 of the optimization showing roughly 20% faster deletions; Patrick Steinhardt called the win welcome.

**Shallow fetch tag backfill** -- Karthik Nayak proposed a patch fixing a regression where tag backfilling sent no `have` lines, making the server send all objects.

**`fast-import` lookup crash** -- Marc Becker posted a patch fixing a crash when the `from` command references a commit created during the same import session.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll posted v4 adding a `[<revs>...]` argument to limit the set of objects processed.

**Wincred refactor** -- Marc Becker posted v2 splitting credential blob dissection into separate methods and fixing deterministic line splitting and CRLF bleed-over.

**Status/diff/add conversion disagreement** -- Curtis Allen Smith's series fixes `git status` disagreeing with `git diff` and `git add` when a clean filter is active under `text eol=lf`.

**`git stash create` options** -- Kazumasa Shigeta proposed limiting the current patch to `-m`/`-q`/`-u`/`-a`, leaving `--patch`, `--staged`, and pathspec support for follow-ups.

**Mergesort tests to Clar** -- Junio C Hamano asked for clarification on the commit message of the patch moving mergesort tests to Clar.

**fsck plural-message microproject** -- Aniket Gupta introduced himself as a GSoC applicant and proposed an fsck plural-message microproject using `Q_()`.