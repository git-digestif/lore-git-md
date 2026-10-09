# Git mailing list daily digest for 2026/10/08

## The day in brief
The Git mailing list saw significant activity around security features, CI infrastructure, and performance optimizations. Key developments include confirmation of a CI timeout issue in the `http.sslVerifyStatus` patch, mentor reorganization for Outreachy, a new v3 series for the `git repack` data-loss fix, and continued debate about AI-assisted contributions. The `git blame` feature gained a new default ignore-revs file capability, and several documentation improvements were finalized.

## Notable threads

### http.sslVerifyStatus CI timeout confirmed intrinsic
[2026/10/08/04-57-39] Junio C Hamano confirmed that the CI timeout issue with the `http.sslVerifyStatus` patch is intrinsic to the patch itself, not an interaction with other topics. When tested in isolation on Git 2.56, the patch reproduces the same timeout in multiple Linux-based CI jobs (`linux-clang`, `linux-gcc`). The root cause remains unknown, but no new technical objections to the feature itself have been raised.

The patch adds a boolean `http.sslVerifyStatus` option (default `false`) that enables OCSP staple validation via libcurl's `CURLOPT_SSL_VERIFYSTATUS`, causing connections to fail if the server does not provide a staple. This security feature is motivated by government customers who mandate OCSP stapling. The feature was merged to `master` in June 2025 but reverted from `next` due to test infrastructure issues and CI timeouts.

### Outreachy mentor reorganization
[2026/10/08/13-20-21] Kaartic Sivaraam proposed and confirmed a mentor reorganization for Git's Outreachy December 2026 cohort. Git will proceed with three approved project ideas but can only mentor two interns due to mentor availability. The "Implement promisor remote fetch ordering" project will only be selected if no strong proposals are received for the other two projects: "Improve how command arguments and options are scanned and parsed" and "Reduce Git's global state to enable Git's libification."

This administrative update follows Pablo Sabater's withdrawal as a co-mentor and maintains Git's capacity for two interns while preserving all three project ideas. The Outreachy application deadline has passed, and intern selection is the next milestone.

### git repack data-loss fix v3
[2026/10/08/09-52-16] qeesung posted v3 of a bugfix series targeting a data-loss race in `git repack -d`. The race occurs when concurrent pushes create a `.keep` file after `repack` has decided which packs to delete but before `pack-objects` has finished building the replacement pack. The v3 series replaces the racy `--honor-pack-keep` mechanism with a snapshot-based approach, ensuring `pack-objects` uses the same `.keep` list observed at repack startup.

Key changes since v2 include moving the kept-pack cache invalidation logic from `builtin/pack-objects.c` to `packfile.c` to address Junio's layering concern. The series is now blocked on integration pending stabilization of Patrick Steinhardt's `ps/odb-files-alternates` topic, which introduces the object directory list that patch 5/5 must walk.

### git blame gains default ignore-revs file
[2026/10/08/21-07-22] Ravi Mistry introduced a v2 series that teaches `git blame` and `git annotate` to automatically use `HEAD:.git-blame-ignore-revs` as a default ignore-revs file when no explicit ignore file is configured. The series is split into two commits: (1) security hardening of the ignore-revs parser and tag-peeling logic, and (2) implementation of the default file lookup.

The security hardening rejects lines with embedded NUL bytes and prevents promisor fetches in partial clones. The default file is resolved via `get_oid_with_context()` and checked with `S_ISREG()` to skip non-regular tree entries. The implementation ensures user-configured ignore files take precedence, and empty configurations bypass the default blob entirely.

### AI-assisted contributions debate continues
[2026/10/08/19-06-32] Maciej Ciemborowicz disclosed a seven-step AI-assisted workflow in the stalled `reference-transaction` hook fix thread, framing participation as a moral dilemma about submitting patches not written by hand. The disclosure follows Junio C Hamano's critique of the patch's size and structure, which he attributed to LLM-generated code lacking thoughtful refactoring.

The debate centers on project norms for AI-assisted contributions, with Kristoffer Haugsbakk suggesting authors should disclose AI assistance upfront in their initial submission. The thread remains stalled on process concerns, with no clear path forward until Maciej demonstrates active engagement with feedback on v3.

### CI infrastructure updates
[2026/10/08/06-15-26] Patrick Steinhardt pushed back against Junio's proposal to remove the `linux32` and `linux-TEST-vars` jobs, arguing that i386 coverage (via Debian's supported images) and the `TEST-vars` job (which has caught regressions in the past) are still valuable. Steinhardt signaled willingness to send patches to preserve or adapt the jobs rather than remove them outright.

The `linux32` job provides 32-bit platform coverage, while `linux-TEST-vars` tests exotic configurations en masse. Junio initially proposed removing both due to lack of i386 support and questionable value, but Steinhardt's response highlights their continued utility for catching regressions.

### gitbreaking-changes(7) manpage ready for integration
[2026/10/08/19-27-15] Kristoffer Haugsbakk posted v2 of a documentation series converting Git's `BreakingChanges` document into a proper manpage (`gitbreaking-changes(7)`). The series is now ready for integration after addressing all review feedback, including URL formatting, structural improvements, and the addition of a living-draft admonition.

The goal is to make upcoming breaking changes more visible to end users via `git help breaking-changes` or `man gitbreaking-changes`. The series includes five patches that split the conversion into logical steps, replace message-IDs with clickable URLs, add a note clarifying the document's living-draft nature, and reorganize the "Adding new items" discussion.

## In brief
- **[2026/10/08/20-55-42]** Junio C Hamano provided a substantive review of Yoichi NAKAYAMA's `git worktree repair` bugfix series, engaging with the refactoring's correctness and edge-case behavior.
- **[2026/10/08/16-59-10]** Junio C Hamano reviewed Harald Nordgren's "matching-inspired" fetch mode series, suggesting documentation and code structure improvements for patch 2/4.
- **[2026/10/08/02-25-34]** Siddharth Shrimali posted a minimal fix for `git repack --drop-filtered --dry-run`, adding an early `return` to prevent repository modifications.
- **[2026/10/08/13-35-40]** Phillip Wood acknowledged a redundant index refresh inefficiency in `git stash push` and agreed with Junio that callers of `do_create_stash()` should be responsible for refreshing the index.
- **[2026/10/08/19-40-43]** Junio C Hamano queued Harald Nordgren's test fix for `t7004-tag.sh`, marking it as ready for integration.
- **[2026/10/08/18-18-14]** Junio C Hamano identified a substantive gap in test coverage: removing the `linux-reftable` job eliminates the only CI coverage for reftable's foreign-SCM interoperability tests.
- **[2026/10/08/22-10-37]** Karthik Nayak fixed a regression in shallow fetches with tag backfilling by splitting the transaction into two, ensuring the client reports the correct 'have' references during negotiation.