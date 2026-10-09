# Git mailing list daily digest for 2026/10/08

## The day in brief

The Git mailing list saw significant activity around security features, CI infrastructure, and performance optimizations. Key developments include confirmation of a CI timeout issue in the `http.sslVerifyStatus` patch, mentor reorganization for Git's Outreachy participation, and a fix for a data-loss race in `git repack`. The day also featured ongoing discussions about AI-assisted contributions and their place in the project.

## Notable threads

### http.sslVerifyStatus CI timeout confirmed intrinsic to patch
[2026/10/08/04-57-39 by Junio C Hamano]

Junio C Hamano confirmed that the CI timeout issue with the `http.sslVerifyStatus` patch is intrinsic to the patch itself, not an interaction with other topics. When tested in isolation on Git 2.56, the patch reproduces the same timeout in multiple Linux-based CI jobs (`linux-clang`, `linux-gcc`). This confirmation rules out the possibility that the timeout was caused by interactions with other in-flight topics in the `seen` integration branch.

The patch adds a boolean `http.sslVerifyStatus` option (default `false`) that enables OCSP staple validation via libcurl's `CURLOPT_SSL_VERIFYSTATUS`, causing connections to fail if the server does not provide a staple. While the core functionality remains intact in `master`, the revert and ejection from `next` and `seen` were procedural to allow test infrastructure fixes. The root cause of the CI timeout remains unknown, requiring further debugging to identify which part of the patch triggers the hang.

### Outreachy mentor reorganization finalized
[2026/10/08/13-20-21 by Kaartic Sivaraam]

Git's participation in the Outreachy December 2026 cohort has been finalized with a mentor reorganization. The project will proceed with three approved project ideas but can only mentor two interns due to mentor availability:

1. "Improve how command arguments and options are scanned and parsed" (co-mentors: Christian Couder, Siddharth Asthana)
2. "Reduce Git's global state to enable Git's libification" (co-mentors: Kaartic Sivaraam, Usman Akinyemi)
3. "Implement promisor remote fetch ordering" (mentor: Kaartic Sivaraam, with Christian Couder as fallback)

The "Implement promisor remote fetch ordering" project will only be selected if no strong proposals are received for the other two projects. This reorganization follows Pablo Sabater's withdrawal as a co-mentor and maintains Git's capacity for two interns.

### git repack data-loss race fix advances
[2026/10/08/09-52-16 by qeesung via GitGitGadget]

Qeesung posted v3 of a bugfix series targeting a data-loss race in `git repack -d`. The race occurs when concurrent pushes can cause the repack machinery to delete packs whose objects were never copied elsewhere, resulting in data loss. The v3 revision addresses Junio C Hamano's layering concern by moving the kept-pack cache invalidation logic from `builtin/pack-objects.c` to `packfile.c`, grouping the downcast with existing ones.

The series replaces the racy `--honor-pack-keep` mechanism with a snapshot-based approach, ensuring `.keep` files are managed consistently. The final patch (5/5) eliminates the race by passing `pack-objects` a snapshot of `.keep` packs observed at repack startup via `--keep-pack-from-file`, replacing the mid-repack directory scan that could miss concurrent `.keep` file creation.

Junio has blocked integration pending stabilization of Patrick Steinhardt's `ps/odb-files-alternates` topic, preferring not to carry a temporary merge resolution in `seen` while the ODB topic is still evolving.

### CI infrastructure debate: preserve or remove contested jobs?
[2026/10/08/06-15-26 by Patrick Steinhardt]

Patrick Steinhardt pushed back against Junio C Hamano's proposal to remove the `linux32` and `linux-TEST-vars` jobs from Git's CI infrastructure. Steinhardt argued that i386 coverage (via Debian's supported images) and the `TEST-vars` job (which has caught regressions in the past) are still valuable. He signaled willingness to send patches to preserve or adapt the jobs rather than remove them outright.

The `linux32` job provides 32-bit platform coverage, while `linux-TEST-vars` exercises Git with non-default "exotic" options like `OPENSSL_SHA1_UNSAFE` and `GIT_TEST_SPLIT_INDEX`. Junio had proposed removing both jobs, citing the lack of i386 support in Ubuntu 20.04 and questioning the value of testing exotic configurations en masse. Steinhardt's response highlights the pragmatic trade-offs between CI maintenance burden and test coverage.

### git blame default ignore-revs file implementation
[2026/10/08/21-07-22 by Ravi Mistry via GitGitGadget]

Ravi Mistry posted a v2 series that teaches `git blame` and `git annotate` to automatically use `HEAD:.git-blame-ignore-revs` as a default ignore-revs file when no explicit ignore file is configured. The series addresses maintainer feedback by splitting the change into two commits: (1) security hardening of the ignore-revs parser and tag-peeling logic, and (2) implementation of the default file lookup.

The core change is security-focused: the patch rejects lines with embedded NUL bytes (using `memchr` instead of `strchr`) and prevents promisor fetches in partial clones by passing `OBJECT_INFO_SKIP_FETCH_OBJECT` and `OBJECT_INFO_QUICK` when peeling tags. The default file is resolved via `get_oid_with_context()` and checked with `S_ISREG()` to skip non-regular tree entries (e.g., committed symlinks). The implementation ensures user-configured ignore files take precedence, and empty configurations (`blame.ignoreRevsFile=""` or `--no-ignore-revs-file`) bypass the default blob entirely.

### AI-assisted contributions debate continues
[2026/10/08/19-06-32 by Maciej Ciemborowicz]

The debate about AI-assisted contributions to Git continued with Maciej Ciemborowicz disclosing a seven-step workflow that relies heavily on AI agents for code generation, testing, and review. Ciemborowicz expressed discomfort with acting as a "meat proxy" for externally generated patches, framing his participation as a moral dilemma about submitting patches he did not write by hand.

The disclosure comes in the context of a stalled patch series fixing the `reference-transaction` hook's omission of new refs during branch rename/copy operations. Junio C Hamano and Patrick Steinhardt have reinforced the expectation that authors engage substantively with reviewer feedback before posting new versions, framing this as a long-standing project norm. The discussion highlights the tension between leveraging AI tools for productivity and maintaining the project's review culture.

## In brief

- **[PATCH v2] blame: default to HEAD:.git-blame-ignore-revs** [2026/10/08/21-07-23]: Security hardening of the ignore-revs parser and tag-peeling logic to reject malformed input and prevent promisor fetches in partial clones.
- **[PATCH v2] blame: implement default ignore-revs file lookup** [2026/10/08/21-07-24]: Teaches `git blame` to automatically use `HEAD:.git-blame-ignore-revs` as a default ignore-revs file when no explicit ignore file is configured.
- **[PATCH 0/8] ci: housekeeping updates for CI configurations** [2026/10/08/10-01-18]: Modernizes Git's CI configurations by removing outdated dependencies, eliminating redundant jobs, and updating supporting scripts.
- **[PATCH 0/2] Fix index refresh inconsistency with line-ending conversions** [2026/10/08/20-45-02]: Addresses a long-standing inconsistency where `git status` reports files as modified when `git diff` and `git add` correctly ignore them due to clean filters.
- **[PATCH] t7004-tag.sh: fix flaky test on Alpine Linux** [2026/10/08/09-07-45]: Fixes a flaky test by replacing filesystem-modifying cleanup with an environment override to simulate a missing public key.
- **[PATCH] fetch: fix shallow fetches with tag backfilling** [2026/10/08/22-10-37]: Fixes a regression in shallow fetches when backfilling tags by splitting the transaction into two, ensuring correct 'have' references during negotiation.