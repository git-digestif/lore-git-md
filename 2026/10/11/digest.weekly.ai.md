## Long threads

### New `gitmergeconflicts(7)` manual page
*Long thread: 54 replies so far from 6 participants over 17 days.*
Julia Evans's series adds a manual page explaining merge conflicts, linked from five commands. It reached v2 with detailed review from Junio C Hamano, who requested `Helped-by:` instead of self-added `Reviewed-by:`, formal constructions, and explicit listing of all conflict-causing commands. Earlier revisions refined the "OURS"/"THEIRS" section under review from Patrick Steinhardt and D. Ben Knoble.

### `uploadpack.lazyFetchTrusted` config
*Long thread: 36 replies so far from 3 participants over 89 days.*
Christian Couder's series adds the protected config for server-side lazy fetch. After v5 was rebased, Junio C Hamano asked if it had reached diminishing returns, and Christian agreed it should be marked for `next` as it adds mostly refactorings rather than substantial new features.

### `http.sslVerifyStatus` for stapled OCSP
*Long thread: 33 replies so far from 5 participants over 59 days.*
Grayson Gordon's patch adds the boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`. After being reverted from `next` over a GnuTLS failure, Junio C Hamano confirmed via bisection that it alone causes 6-hour linux-* CI timeouts and tentatively ejected it from `seen`.

### Shallow-repo limited fetch
*Long thread: 31 replies so far from 4 participants over 22 days.*
Harald Nordgren's series adds `remote.<name>.refmap` and `--limited-fetch` to avoid fetching every branch in shallow repos. It reached v8, splitting out a `collect_upstream_from_remote()` helper after Junio C Hamano reviewed v7 and requested naming and structural changes; remaining test failures were identified as an interaction with another topic.

### Rerere waiting on `MERGE_RR.lock`
*Long thread: 29 replies so far from 4 participants over 40 days.*
Thomas Bachem's series makes rerere wait up to `rerere.lockTimeout` for the lock and adds `--skip-locked` to `git rerere gc`. It reached v7, dropping the patch letting commands proceed without rerere; Patrick Steinhardt suggested a dedicated flags enum, and Junio C Hamano reviewed test details.

### Keeping `die()` out of trace2 API
*Long thread: 26 replies so far from 6 participants over 84 days.*
This series introduces `banned-die.h` to ban die-capable helpers in trace2 sources. Derrick Stolee noted the topic has sat idle because the protections are incomplete, and his RFC for making strbuf more robust was similarly unsatisfying.

### CI failure and leak annotations
*Long thread: 24 replies so far from 4 participants over 12 days.*
Harald Nordgren's series makes GitHub Actions annotations name the test script and line. After Phillip Wood agreed file/line links should be dropped, Harald posted a revision showing them as plain text, which Phillip reviewed with "Lets get this merged!"

### `git-bisect-rebase`
*Long thread: 22 replies so far from 6 participants over 8 days.*
Alejandro Colomar's tool bisects for the first commit a branch cannot cleanly rebase onto. He implemented `--abort`, `--continue`, and `--quit`, gave example sequences, and corrected the tree algorithm so `git rebase --onto A tmpA B` uses C instead of B.

### Ref copy and rename through transactions
*Long thread: 22 replies so far from 6 participants over 22 days.*
Maciej Ciemborowicz's series routes copy/rename through transactions so the hook sees the whole operation. V4 fixed test failures from v3, but reviewers criticized posting new versions without engaging feedback, with Patrick Steinhardt warning this is a red flag.

### `--delete-merged` squash detection
*Long thread: 22 replies so far from 4 participants over 13 days.*
Harald Nordgren's patch detects squash-merged branches via patch-id comparison. Reposted as v2, it shares diff results between branches and adds `--no-squashed`; Phillip Wood endorsed patch-id comparison as necessary since squash merges create single-parent commits not tree-same to the tip.

### Removing `gittutorial-2`
*Long thread: 21 replies so far from 4 participants over 9 days.*
This series deletes the stale tutorial page. Tuomas Ahola showed the interdiff adds it to `command-list.txt`, making `git(1)` advertise it; Junio C Hamano proposed ejecting both topics and having Julia Evans send a v3 without the entry.

### `--force-if-includes` checks pushed ref
*Long thread: 20 replies so far from 5 participants over 32 days.*
Tyler Cipriani's series makes the check consult the pushed ref's reflog, rejecting non-branch sources as unverifiable. He addressed Patrick Steinhardt's review by noting tags and detached HEAD are rejected because their reflogs lack integration information, and offered a rebased v7.

### `man git` navigation
*Long thread: 15 replies so far from 5 participants over 10 days.*
Julia Evans's patch rewrites `man git` to explain help navigation and points to `git help --guides`. The thread covered the git-scm.com migration, and Junio C Hamano asked the next iteration to justify dropping user-manual and datamodel references.

### Faster SHA-1 collision detection
*Long thread: 14 replies so far from 6 participants over 10 days.*
Scott Chacon's series ports the sha1dc Rust crate to C, claiming 2.7x faster hashing. Sebastian Thiel and Sam Reis welcomed it; D. Ben Knoble raised license compatibility, which Reis clarified by noting the UBC tables are autogenerated, and Junio confirmed GPL-2 can incorporate MIT code.

## The period in brief
The week of 2026/10/05 through 2026/10/11 saw heavy traffic across seven active days, with documentation and shallow-repository work dominating. Readers should not miss Julia Evans's `gitmergeconflicts(7)` series reaching v2 under detailed review, Harald Nordgren's v8 of the shallow-repo limited-fetch series, and the heated debate over Scott Chacon's RFC to allow AI-assisted contributions under the DCO. Junio C Hamano started the Git 2.98 cycle by flushing `next` to `master`.

## Key developments

### AI-assisted contributions policy debate
Scott Chacon's RFC proposed allowing responsible AI-assisted contributions under the DCO with an `Assisted-by:` trailer. The proposal drew sharp objections: brian m. carlsson argued the DCO cannot honestly be asserted for LLM output, Luca Milanesio argued fully AI-generated work should be rejected, and Junio C Hamano critiqued the text as unenforceable. Patrick Steinhardt warned that loosening the policy would worsen the review bottleneck as AI-generated slop increases.

### Packfile corruption from stale delta cache
Patrick Steinhardt's two-patch series fixes corruption from delta base cache entries never evicted when their pack closes, reproducible via recursive submodule merges. Jeff King called the fix obviously correct, and Junio C Hamano marked it for `next` after confirming the reproducer also fails on Alpine with musl.

### `git blame` default ignore-revs
Ravi Mistry's series makes `git blame` use `HEAD:.git-blame-ignore-revs` by default so local runs match hosting platforms. V3, which uses `sb->repo` instead of `the_repository`, was deemed ready for `next` by Junio C Hamano, with earlier revisions tightening the parser against NUL bytes and lazy promisor fetches.

### `repack --dry-run` fix
Siddharth Shrimali's patch fixes `git repack --drop-filtered --dry-run` rebuilding packs despite documentation saying it only lists objects. V2 returns right after listing candidates and adds a test that the pack directory is unchanged; D. Ben Knoble offered his Reviewed-by and Junio C Hamano marked it for `next`.

### SSH signing trust limits
Christian Ramos reported that `valid-before` is checked against a signer-controlled date and that a missing `gpg.ssh.revocationFile` fails open. Patrick Steinhardt argued there is no proper fix for the timestamp in a distributed system but agreed documentation should avoid false trustworthiness, and suggested failing by default on a missing revocation file with an optional escape hatch.

### Contributor Summit notes and Git 3.0 timeline
Taylor Blau shared notes from the 2026 Git Contributor's Summit covering security process, Git 3.0 timing (possible 2.99 and 3.0 in March 2027 with only `BREAKING_CHANGES` defaults flipped), pluggable ODB, and protocol v2 for pushes.

### Push resending common history after pre-push repack
Jens Röcker reported that a push resends common history when a pre-push hook repacks the object database. Patrick Steinhardt confirmed the mechanism as `odb_has_object()` false negatives after repack and provided an untested `send-pack.c` patch batching the negative-object checks.

## In brief

**`uploadpack.lazyFetchTrusted`** -- Christian Couder's series adding the protected config for server-side lazy fetch was agreed to be marked for `next`.

**`--force-if-includes`** -- Tyler Cipriani's series making the check consult the pushed ref's reflog addressed review by rejecting tags and detached HEAD as unverifiable.

**CI failure annotations** -- Harald Nordgren's revised patch showing file and line as plain text drew Phillip Wood's "Lets get this merged!"

**Keeping `die()` out of trace2** -- Derrick Stolee noted the `banned-die.h` protections remain incomplete and the topic has sat idle.

**Removing `gittutorial-2`** -- The series was set to be rerolled without the `command-list.txt` entry that advertised the obsolete tutorial.

**`man git` navigation** -- Julia Evans's patch explaining help navigation drew requests to justify dropping user-manual references.

**`git-bisect-rebase`** -- Alejandro Colomar implemented `--abort`, `--continue`, and `--quit` for the iterative rebase bisect tool.

**SHA-1 collision detection** -- Scott Chacon's port of the sha1dc crate to C was welcomed with license compatibility confirmed.

**Shallow fetch tag backfill** -- Karthik Nayak's patch fixes a regression where tag backfilling sent no `have` lines.

**Packed-refs optimization** -- Karthik Nayak's v4 of the `fwrite()` optimization showed ~20% faster deletions.

**Fetch commit-graph** -- Kristofer Karlsson's v3 writing commit-graph using updated refs only dropped fetch times from ~357 ms to under 10 ms.

**ODB source-files alternates** -- Karthik Nayak reviewed the 13-patch series and said it looks good.

**`checkout -m` conflict labels** -- Phillip Wood's v3 writing labels to `.git/MERGE_LABELS` was reviewed as "nicely done".

**`git status` merge advice** -- Julia Evans's patch changing advice to `git merge --continue` was approved for `next`.

**`fast-import` crash** -- Marc Becker fixed a crash when `from` references a commit created in the same session.

**`git repo structure` revs** -- Mark C. Chu-Carroll posted v4 adding revision filtering.

**Repack `.keep` race** -- Qin ShiCheng's series fixing objects lost to a mid-run `.keep` waits for `ps/odb-files-alternates`.

**ZIP timestamps** -- René Scharfe recommended clamping out-of-range values in `git archive --format=zip`.

**`git status` conversion disagreement** -- Curtis Allen Smith's series fixes `git status` disagreeing with `git diff` under active clean filters.

**Git 2.98 cycle** -- Junio C Hamano flushed `next` to `master` to start the new development cycle.

## Looking ahead
Harald Nordgren's v8 of the shallow-repo fetch series and Thomas Bachem's v7 of the rerere lock-wait series will see further review, as will Ravi Mistry's v3 blame ignore-revs series which is ready for `next`. The AI-assistance policy debate is likely to continue given Patrick Steinhardt's concerns about review bottlenecks. Qin ShiCheng's repack `.keep` race fix waits on the `ps/odb-files-alternates` topic, and the Git 3.0 timeline targeting March 2027 will shape upcoming discussions.