## Long threads

### New `gitmergeconflicts(7)` manual page
*Long thread: 54 replies so far from 6 participants over 17 days.*

Julia Evans's six-patch series adds a `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-revert`, `git-cherry-pick`, and `git-pull` instead of repeating guidance in each command. The series reached v2 on October 9, when Junio C Hamano objected that `Reviewed-by:` trailers on patch 1/6 had been added by the author rather than offered by reviewers; Evans agreed to use `Helped-by:` instead. Junio reviewed each patch, suggesting changes to the new page (co-author sign-off, explaining the `|||||||` marker, treating `zdiff3` as a failed experiment) and simplifications in the command docs, and said he would add his `Reviewed-by:` to patches 2/6, 3/6, and 6/6 as-is. On October 10 he asked that the new technical manual list every merge-conflict-causing command explicitly -- including `git am -3` and `git checkout -m` -- and use formal constructions instead of contractions throughout.

### `uploadpack.lazyFetchTrusted` config for server-side lazy fetch
*Long thread: 36 replies so far from 3 participants over 89 days.*

Christian Couder's series adds the protected `uploadpack.lazyFetchTrusted` config so server operators can mark repositories whose promisor remotes may be lazy-fetched when serving clones, touching `builtin/upload-pack.c`, `upload-pack.c`, `promisor-remote.c` and `setup.c`. On October 5 Junio C Hamano asked whether the topic had reached diminishing returns and should be marked for `next`; on October 6 Couder agreed the v5 series should be marked for `next`, since it adds mostly refactorings and a new protected config variable rather than substantial new code.

### `http.sslVerifyStatus` for stapled OCSP responses
*Long thread: 33 replies so far from 5 participants over 59 days.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off. After v7 was reverted out of `next` over a t5585 failure traced to a GnuTLS curl backend, Junio C Hamano bisected linux-* CI jobs spinning until their 6-hour timeout to this topic and tentatively ejected it from `seen` on October 7; on October 8 he confirmed in isolation on Git 2.56 that it causes the timeouts, while the linux-TEST-vars/linux32 failures are unrelated.

### Fetching fewer branches from new remotes in shallow repos
*Long thread: 31 replies so far from 4 participants over 22 days.*

Harald Nordgren's series avoids fetching every branch of a new remote in a shallow repository by adding `remote.<name>.refmap`, inferring branches to fetch from local upstreams, and defaulting to `--limited-fetch` in shallow clones. On October 10 he posted v8, a five-patch series splitting the work into a preliminary `collect_upstream_from_remote()` helper in `remote.c` plus a simplified refmap-inference commit, fixing a regression that could drop the current branch's `@{upstream}` when `remote.<name>.fetch` was narrowed, and threading `struct repository *` through instead of hardcoding `the_repository`. Junio reviewed patch 2/5, objecting that duplicating the current-branch upstream logic into the `remote.*.fetch` branch belongs to the following step, and called patch 4/5 (`git remote add --limited-fetch`) straightforward and nicely done.

### Rerere waiting on `MERGE_RR.lock`
*Long thread: 29 replies so far from 4 participants over 40 days.*

Thomas Bachem's series changes `rerere` so that a rebase hitting a conflict no longer dies at once when a background `git rerere gc` holds `MERGE_RR.lock`. On October 10 he posted v7, a two-patch series: the first makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) before failing, and the second adds a hidden `--skip-locked` option to `git rerere gc` that auto-maintenance passes so a background gc does nothing while the lock is held. On October 11 Junio reviewed patch 1/2, accepting the millisecond unit and the call to `repo_hold_lock_file_for_update_timeout()`, while questioning a doc sentence and test details; Bachem agreed to drop the sentence and adjust the tests.

### Keeping `die()` out of the trace2 API
*Long thread: 26 replies so far from 6 participants over 84 days.*

This series introduces a root-level `banned-die.h` header included by the trace2 API sources, banning die-capable helpers one at a time so the low-level tracing code cannot trigger a recursive die. On October 6 Derrick Stolee said the topic has sat idle because the `banned-die.h` protections are incomplete, and that his RFC for making strbuf and friends more robust was similarly unsatisfying, though interesting alternative ideas exist.

### CI failure and leak annotations
*Long thread: 24 replies so far from 4 participants over 12 days.*

Harald Nordgren's series makes GitHub Actions annotations for test failures and leaks name the test script and line. On October 6 he posted a revision that reverts direct file/line linking -- which fails when the error points to a file unchanged in the PR -- in favor of showing the file and line as plain text in the annotation, and omits the line number for leaks where it is never available. Phillip Wood reviewed it and said "Lets get this merged!"

### Reference copy and rename through transactions
*Long thread: 22 replies so far from 6 participants over 22 days.*

Maciej Ciemborowicz's four-patch series routes reference copy and rename through ordinary transactions so the reference-transaction hook sees the whole operation, where files saw only the source deletion and reftable neither endpoint. On October 8 he posted v4, which preserves a packed-refs error by jumping to cleanup after the failed packed transaction, and reported the full suite passing on both backends. Patrick Steinhardt warned that posting new versions without replying to reviewers is a red flag; the thread then debated AI-assisted workflows. On October 10 Ciemborowicz offered to let someone else write the patches if that would be more efficient, and brian m. carlsson advised that it is better not to submit a patch at all if one cannot meet the SubmittingPatches guidelines.

### `--delete-merged` detecting squash-merged branches
*Long thread: 22 replies so far from 4 participants over 13 days.*

Harald Nordgren's patch makes `git branch --delete-merged` treat a branch as merged when an upstream commit since the fork point contains all its changes, printing where the work landed. On October 11 he reposted the series as v2, which diffs each upstream commit once and shares the result between all branches for speed, fixes a branch merged with a merge commit being reported incorrectly, uses `git merge --squash` as the example, and adds `--no-squashed` to skip the squash search, which is slower and may download blobs in a partial clone.

### `git-bisect-rebase` for iterative rebases
*Long thread: 22 replies so far from 6 participants over 8 days.*

Alejandro Colman's `git-bisect-rebase` bisects for the first commit a branch cannot cleanly rebase onto. On October 8 he implemented `--abort`, `--continue`, and `--quit` with semantics unlike `git-rebase(1)`, gave example sequences for moving a branch tree and a single branch, and corrected the tree algorithm so one `git rebase --onto A tmpA B` uses C instead of B.

### Removing `gittutorial-2`
*Long thread: 21 replies so far from 4 participants over 9 days.*

This documentation series deletes the stale `gittutorial-2` page, leaving a stub redirecting readers to `gitdatamodel` and `gitcore-tutorial`. On October 6 Tuomas Ahola showed the v1->v2 interdiff adds `gittutorial-2` to `command-list.txt`, making `git(1)` advertise the obsolete tutorial in its GUIDES list. Junio C Hamano proposed ejecting both topics and having Julia Evans send a v3 without the `command-list.txt` entry; Evans agreed, and Junio agreed to queue the reroll on a synthetic base.

### `--force-if-includes` checks the pushed ref
*Long thread: 20 replies so far from 5 participants over 32 days.*

Tyler Cipriani's series makes `git push --force-if-includes` consult the reflog of the ref actually being pushed in `remote.c`, rejecting non-branch sources as unverifiable and deferring the check so fast-forward pushes are not rejected. On October 5 he addressed Patrick Steinhardt's review: since v4 any source ref is resolved, and tags, oids and detached HEAD are rejected as unverifiable because their reflogs lack the integration information the check needs, and he offered a rebased v7.

### Teaching documentation navigation in `man git`
*Long thread: 15 replies so far from 5 participants over 10 days.*

Julia Evans's patch rewrites `man git` to explain that `git push -h` gives short help and `git push --help` or `git help push` the full documentation, drops the gittutorial and giteveryday references, and points to `git help`, `git help --guides`, `git help --user-interfaces` and `git help --web`. On October 7 the thread covered why git.github.io was replaced by git-scm.com, and Junio asked the next iteration to justify dropping the user-manual, datamodel and cli references.

### Faster SHA-1 collision detection via sha1dc-accel
*Long thread: 14 replies so far from 6 participants over 10 days.*

Scott Chacon's four-patch series ports Sam Reis's sha1dc Rust crate to C in `sha1dc-accel/`, claiming about 2.7x faster hashing. On October 8 Sebastian Thiel welcomed the speedup and Reis said he is happy to see it land. D. Ben Knoble raised license compatibility of the MIT-licensed derived code with GPL-2; Reis clarified the UBC tables are autogenerated from the Stevens/Shumow methodology, and Junio confirmed Git's GPL-2 can incorporate MIT-licensed code.

## The period in brief

The week of 2026/10/05 -- 2026/10/11 saw seven active days of steady traffic, dominated by long-running documentation and infrastructure series absorbing detailed review. Junio C Hamano started the Git 2.98 cycle by flushing `next` down to `master` on October 10, while several topics -- the `gitmergeconflicts(7)` page, `uploadpack.lazyFetchTrusted`, the `repack --dry-run` fix, and the blame ignore-revs default -- were marked ready for `next`. A heated debate over AI-assisted contributions and a Contributor Summit summary covering Git 3.0 timing rounded out a routine but eventful week.

## Key developments

### Contributor Summit notes and Git 3.0 planning
Taylor Blau shared notes from the 2026 Git Contributor's Summit covering seven topics: security process, Git 3.0 (possible 2.99 and 3.0 in March 2027; consensus to release 2.99.1 and 3.0 together with only `BREAKING_CHANGES` defaults flipped), documentation, Outreachy, pluggable ODB, AI policy, and protocol v2 for pushes.

### AI-assisted contributions policy debate
Scott Chacon's RFC patch to `Documentation/SubmittingPatches` proposed allowing AI-assisted contributions under the existing DCO with human review and an `Assisted-by:` trailer, aligning with SFC, Linux, Xen and Debian guidance. brian m. carlsson objected that the DCO cannot honestly be asserted for LLM output without knowing all training sources, citing legal liability; Junio C Hamano critiqued the text as resting on an OpenAI model's output and giving maintainers less latitude than the kernel guidelines. Patrick Steinhardt argued the review bottleneck is tightening as AI-generated slop increases, warning that loosening the AI policy without solutions would make the problem significantly worse.

### Packfile corruption fix lands in `next`
Patrick Steinhardt's two-patch series fixes corruption from delta base cache entries never evicted when their pack closes, reproducible via recursive submodule merges. Jeff King called the fix obviously correct, and on October 7 Junio C Hamano marked the topic for `next`.

### Push resending common history after pre-push repack
Jens Röcker reported that a push resends common history when a pre-push hook repacks the local object database and removes loose common objects. Patrick Steinhardt confirmed the mechanism (`odb_has_object()` false negatives after a pre-push repack) and provided an untested `send-pack.c` patch batching the negative-object checks.

### `repack --dry-run` no longer rebuilds packs
Siddharth Shrimali's patch fixes `git repack --drop-filtered --dry-run`, which was documented to list objects without rebuilding any pack or deleting anything but did both because the `--dry-run` block fell through into the regular repack code. On October 10 he posted v2 returning right after listing the drop candidates; D. Ben Knoble offered his `Reviewed-by:`, and on October 11 Junio marked the topic for `next`.

### Blame defaults to `HEAD:.git-blame-ignore-revs`
Ravi Mistry posted v3 of the series making `git-blame` and `git-annotate` automatically use the `HEAD:.git-blame-ignore-revs` blob when it exists, so local runs match hosting platforms. On October 11 Junio said v3 looks good and would be marked ready for `next` unless objections arrive.

### `git status` suggests `git merge --continue`
Julia Evans posted v3 of her patch changing the merge-conflict advice in `wt-status.c` from `git commit` to `git merge --continue`, consistent with rebase, revert, and cherry-pick. Phillip Wood said the version looks good, and Junio said he will apply it with Phillip's `Acked-by:` and mark the topic for `next`.

### SSH signing trust limits
Christian Ramos reported that `valid-before` is checked against a date the signer writes and that a missing `gpg.ssh.revocationFile` fails open, so an SSH signing key cannot be reliably stopped from being trusted. Patrick Steinhardt argued there is no proper fix for the timestamp in a distributed system but agreed documentation should avoid a false sense of trustworthiness, and suggested failing by default on a missing revocation file with a `:(optional)` path prefix as an escape hatch.

## In brief

**`checkout -m` recreating conflict labels** -- Phillip Wood posted v3 of his two-patch series teaching the ort machinery to write conflict labels to `.git/MERGE_LABELS` so `git checkout -m` recreates conflicts with the original labels; Junio reviewed it as "nicely done" and said "Will replace."

**Shallow clone boundary commit warning** -- Patrick Steinhardt explained that in a shallow clone the boundary commit is rewritten to have no parents, so reverting it deletes all files, and agreed Git should warn about or refuse editing a shallow boundary commit by default.

**ZIP timestamp bugs** -- René Scharfe explained DOS timestamps cover only 1980 onward while Unix timestamps start at 1970, suggested clamping out-of-range ZIP values, and later recommended rejecting non-representable timestamps in `git archive --format=zip`.

**`odb/source-files` alternates series** -- Karthik Nayak reviewed the 13-patch series, accepted Patrick Steinhardt's explanations, and said the whole series looks good.

**Fetch commit-graph using updated refs** -- Kristofer Karlsson posted v3 of the series with benchmarks showing updated-ref fetches dropping from about 357 ms to under 10 ms.

**Packed-refs `fwrite()` optimization** -- Karthik Nayak posted v4 of the series with a ~20% deletion win; Patrick Steinhardt called it welcome.

**Shallow fetch tag backfill** -- Karthik Nayak proposed a patch fixing a regression where tag backfilling sent no `have` lines, making the server send all objects.

**`repack` `.keep` race** -- Qin ShiCheng's five-patch series fixes a race where a concurrent push installs a `.keep` after `git repack -d` decided which packs to delete; Junio suggested waiting for the `ps/odb-files-alternates` topic before merging.

**`git-bisect-rebase`** -- Alejandro Colman implemented `--abort`, `--continue`, and `--quit` semantics for iterative rebases.

**`git repo structure` revision filtering** -- Mark C. Chu-Carroll posted v4 adding a `[<revs>...]` argument to limit the set of objects processed.

**`fast-import` same-session lookup** -- Marc Becker submitted a patch fixing a crash when the `from` command references a commit created during the same import session.

**Status/diff/add conversion disagreement** -- Curtis Allen Smith's series fixes `git status` disagreeing with `git diff` and `git add` when a clean filter is active under `text eol=lf`.

**`git config --global` XDG support** -- Delilah Ashley Wu posted v3 making `git config --global` read both `$HOME/.gitconfig` and the XDG config file.

**Sparse checkout untracked files** -- Muhammed Dilshad A posted a patch making sparse checkout refuse to overwrite untracked files like ordinary checkout.

**Push refspec matching speedup** -- Jon Simons posted a 15-patch series speeding up client-side push refspec matching; Kristoffer Haugsbakk objected to separating failing tests from their fixes.

**`git stash create` options** -- Kazumasa Shigeta proposed limiting the current patch to `-m`/`-q`/`-u`/`-a`, leaving `--patch`, `--staged`, and pathspec support for follow-ups.

**Wincred refactor** -- Marc Becker posted v2 splitting credential blob dissection into separate methods and fixing CRLF bleed-over.

**`imap-send.c` ASN1 handling** -- Xiangfei Ding's patch uses `ASN1_STRING_length` and `strncasecmp` instead of assuming null-terminated `ASN1_STRING` data.

**Archive-zip UT field** -- René Scharfe's patch makes `archive-zip.c` omit the UT extra field when the commit time exceeds the signed 32-bit range.

**Worktree repair error behavior** -- Yoichi NAKAYAMA explained adjusted error-case behavior and said he will split the commits to separate modified from unchanged behavior.

**SubmittingPatches test preference** -- Jon Simons documented the preference for submitting a bug fix's test in the same commit as the code change.

**GSoC fsck microproject** -- Aniket Gupta introduced himself as a GSoC applicant and proposed an fsck plural-message microproject using `Q_()`.

## Looking ahead

Harald Nordgren's v8 of the shallow-repo limited-fetch series and Thomas Bachem's v7 of the rerere lock-wait series were both posted late in the week and will see further review next period. The `gitmergeconflicts(7)` documentation series awaits a v3 addressing Junio's October 10 requests for explicit command lists and formal constructions. Qin ShiCheng's `repack` `.keep` race fix is held pending the `ps/odb-files-alternates` topic, and Scott Chacon's SHA-256 signing RFC and sha1dc-accel series will likely continue drawing review attention.