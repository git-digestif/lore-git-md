# Git mailing list daily digest for 2026/10/07

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread (root 2026/09/24/14-44-15): 39 replies so far from 6 participants over 14 days; 1 message that day.*

Julia Evans's series introduces a new manual page, `gitmergeconflicts`, explaining the merge-conflict process with examples, and links to it from the commands that can cause conflicts (`git-merge`, `git-rebase`, `git-cherry-pick`, `git-revert`, `git-pull`) instead of reexplaining the process each time. Earlier revisions refined the "WHAT IS A MERGE CONFLICT?" section and the "OURS"/"THEIRS" wording through review by Junio C Hamano and D. Ben Knoble. Today Junio C Hamano corrected the description of a custom `conflict-marker-size` setting: it is not about ignoring conflict markers but ensures payload lines that happen to look like markers are not mistaken for them (for example by rerere); he noted the patch is planned to be squashed into the step that introduces the new file and suggested the technique might be worth teaching in the merge-conflicts documentation series.

### `http.sslVerifyStatus` for stapled OCSP responses

*Long thread (root 2026/08/11/17-02-00): 32 replies so far from 5 participants over 58 days; 1 message that day.*

Grayson Gordon's v7 patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`, so libcurl requests the OCSP "Certificate Status Request" extension and honors stapled responses, including revoked ones; it defaults to false because enabling it would block connections to servers that do not staple responses. Earlier, the topic was reverted from `next` after SZEDER Gábor reported t5585 failures traced to a GnuTLS curl backend lacking OCSP support. Today Junio C Hamano reported that 'seen' with this topic ejected does not exhibit the linux-* CI timeout symptom, and he is tentatively ejecting it from `seen`, noting it might be interactions with other topics.

### Avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 25 replies so far from 4 participants over 19 days; 1 message that day.*

Harald Nordgren's series adds `remote.<name>.refmap` as a per-remote default for `--refmap`, infers branches to fetch from local upstreams when only a refmap is configured, adds `git remote add --limited-fetch`, and defaults to `--limited-fetch` in shallow repositories. Earlier revisions removed automatic default-branch fetching and renumbered the test file. Today he posted v7, fixing the fatal "--refmap option is only meaningful with command-line refspec(s)" error that fired when only `remote.<name>.refmap` was configured with nothing else to fetch, and adding tests showing that when both `remote.<name>.fetch` and `remote.<name>.refmap` are configured, a refspec-less fetch follows `.fetch` as usual while `.refmap` only remaps refs already being fetched by name.

### Teaching documentation navigation in `man git`

*Long thread (root 2026/09/28/20-32-54): 15 replies so far from 5 participants over 10 days; 8 messages that day.*

Julia Evans's v2 patch rewrites `man git` to explain that `git push -h` gives short help and `git push --help` or `git help push` gives full documentation, drops references to `gittutorial` and `giteveryday`, and points readers to `git help`, `git help --guides`, `git help --user-interfaces`, and `git help --web`. Today the discussion covered replacing the git.github.io link with git-scm.com (which adds grouping and usability features, though its freshness is uncertain), and Junio C Hamano asked the third iteration to justify also removing references to user-manual, datamodel, and cli; Julia Evans gave her reasoning that cli reads as developer-oriented, user-manual is outdated, and gitdatamodel seemed out of place once the others were removed.

## The day in brief

Junio C Hamano tentatively ejected the `http.sslVerifyStatus` topic from `seen` after reporting that `seen` without it does not exhibit the linux-* CI timeout symptom, Patrick Steinhardt's packfile stale-delta-base-cache fix was marked for `next` after Jeff King called it the obviously correct fix, and Scott Chacon's RFC proposal to allow responsible AI-assisted contributions drew an objection from brian m. carlsson and a critique from Junio C Hamano.

## Notable threads

### SubmittingPatches: allow responsible AI assistance
Scott Chacon submitted an RFC patch to `Documentation/SubmittingPatches` proposing to allow AI-assisted contributions under the existing DCO, requiring human review and testing plus an `Assisted-by:` trailer for substantial assistance, aligning with current SFC recommendations and the approaches of Linux, Xen, and Debian. brian m. carlsson objected that because major LLMs are trained on sources that prohibit copying and are known to reproduce training-set material, no one can honestly assert the DCO without knowing all training sources, citing legal-liability concerns as a distributor. Junio C Hamano critiqued the proposal as resting on OpenAI output for its legal-soundness claim, being largely unenforceable aspiration, and giving maintainers less latitude to reject contributions than the kernel guidelines do.

### packfile: fix corruption from stale delta base cache entries
Patrick Steinhardt's series fixes corruption when delta base cache entries, keyed by the in-core `packed_git` pointer and pack offset, survive pack closure and collide with a newly allocated pack at the same address, a bug reproducible via recursive merges with submodules that have similar packfiles. Today he confirmed the reproducer also fails on Alpine Linux with musl libc, Jeff King analyzed the bug and called the patch the obviously correct fix, and Junio C Hamano marked the topic for `next`.

### refs: run copy and rename through transactions
Maciej Ciemborowicz posted v3 of a four-patch series routing reference copy and rename through ordinary transactions so the reference-transaction hook can observe and reject the complete operation (previously the files backend saw only the source deletion and reftable saw neither endpoint), adding `ref_transaction_replace_reflog()` and removing the backend-specific copy/rename callbacks. Junio C Hamano reported that applying the series to master breaks t0600.16 and t5510.35, where `git update-ref -d` and `git fetch --prune` unexpectedly succeed despite `.git/packed-refs.new` being pre-created.

### faster SHA-1 collision detection (`sha1dc-accel`)
Scott Chacon's series ports Sam Reis's Rust sha1dc approach to C, hashing roughly 2.7x faster on the tested machines; the author notes the code is mainly AI generated. Today Johannes Schindelin suggested reusing sha1dc's existing compression function, offered a Perl script to regenerate the UBC tables from the Rust sources, and offered a SHA-NI patch closing the performance gap to the Rust crate to about 1%; Junio C Hamano asked how the AI-generated code can be confirmed not to borrow from incompatible sources, and Scott Chacon answered that he discussed the port with Sam Reis and is confident signing off under the DCO.

### push resends common history after repack during pre-push
A bug report shows a push resending common history when its pre-push hook repacks the local object database and removes previously loose common objects. Today Patrick Steinhardt confirmed the mechanism--`odb_has_object()` can have false negatives by default, so after a concurrent repack `append_negative_object()` silently skips the removed objects and the graph walk pushes too many objects--and proposed an untested `send-pack.c` patch that batches the negative-object existence checks into an `oidset` and calls `odb_reprepare()` once only if any are missing.

## In brief
- Muhammed Dilshad A posted v2 of "combine-diff: honor relative paths consistently", printing outside-prefix parent paths as `../there/file` and fixing the fast multi-tree scan that bypassed the relative-prefix filter; Junio C Hamano called the behavior the most sensible within the `--relative` constraint.
- Junio C Hamano marked Colin Hinton's "fetch.c: defer fetch.followRemoteHEAD validation" topic for `next`, accepting that `remote.*.followRemoteHEAD` remains inconsistent.
- Karthik Nayak posted v4 of "packed-refs: use `fwrite()` when passing refs verbatim" with a reworded comment and inlined `fwrite()`, showing a consistent ~20% improvement; Patrick Steinhardt said he is happy with the version.
- Ravi Mistry said he will put up a v2 of the default `.git-blame-ignore-revs` patch with tightened OID parsing so security-interested reviewers can chime in.
- Kristofer Karlsson posted v3 of "fetch: write commit-graph using updated refs only" with a trimmed commit message and a simplified t5537 test.
- René Scharfe analyzed the ZIP timestamp bugs, recommending rejecting non-representable timestamps when creating ZIP files and arguing against rejecting negative timestamps in fast-import.
- Junio C Hamano reported recurring ubuntu 20.04 CI failures and plans to drop the linux32 job and rework linux-TEST-vars to ubuntu:rolling.
- Review of Julia Evans's new standalone `gitdetachedhead(7)` guide produced wording suggestions from Junio C Hamano and Kristoffer Haugsbakk.
- D. Ben Knoble gave extensive feedback on Julia Evans's new beginner tutorial, concluding it is excellent overall.
- **odb backend design discussion**: Patrick Steinhardt argued the files backend's internals shouldn't leak into callers, instead endorsing a separate self-contained merged-source backend.

- Phillip Wood agreed to make callers of `do_create_stash()` responsible for refreshing the index in the stash duplicate-check removal series.
- Phillip Wood acknowledged `write_merge_labels()` would be called even for clean merge results and said he will fix it.
