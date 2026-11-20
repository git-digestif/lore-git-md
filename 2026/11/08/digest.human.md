# Git mailing list daily digest for 2026/11/08

## Long threads

### Detached HEAD guide rewrite in the documentation

*Long thread: 22 replies so far from 4 participants over 33 days; 1 message that day.*

Julia Evans's series rewrites the explanation of detached HEAD state into a standalone `gitdetached-head` guide, wired into `advice.c`, `command-list.txt`, and the build files so the advice printed on detaching can point users to `git help detached-head`. Earlier revisions addressed reviewer feedback on wording, examples, and the definition of `HEAD`. Today Junio C Hamano answered D. Ben Knoble's closing suggestions: he explained that the "parent of the next commit" mental model matches HEAD being "where you started from", appreciated the addition about not clobbering an existing branch or creating a temporary one, but objected to phrasing that calls it useful for HEAD "to be a commit ID", arguing that directly referring to a commit is the notable property while the hexadecimal name is an implementation detail. He suggested starting that paragraph with "directly referring to a commit (instead of indirectly referring to a commit by naming the current branch)", and said he hopes the thread is approaching a finish with the document moving in the right direction.

### `test-tool genbytes` to speed up macOS CI tests

*Long thread: 15 replies so far from 3 participants over 2 days; 13 messages that day.*

Kristofer Karlsson's series adds a `test-tool genbytes <hex-byte> [<count>]` helper and converts tests away from `genzeros | tr`, because t4205's EXPENSIVE tests spend most of their time in `tr(1)` on macOS. Today the remaining points were settled: Junio C Hamano retracted his earlier warning that renaming test-genzeros.c would break tests, while keeping his preference for a single rename-and-modify commit over a split pair; Johannes Sixt's `printf` alternatives were shown to fail on macOS, where `printf` ignores large padding, and Karlsson confirmed that forcing `LANG=C` does not help because test-lib-functions.sh already sets it. Karlsson posted v2 as two patches: patch 1 adds `genbytes` taking the byte as two lowercase hex digits (so NUL is `00`) and renames test-genzeros.c into test-genbytes.c, and patch 2 converts the remaining `genzeros` callers and removes it.

### Moving `assume_unchanged` into repository configuration

*Long thread: 6 replies so far from 2 participants over 1 day; 9 messages that day.*

Hussain Sitamouwala posted a patch moving the global `assume_unchanged` flag in `environment.c` into repository-specific configuration, adding a `repo_assume_unchanged()` getter and updating the callers in `builtin/update-index.c` and `read-cache.c`, so the setting is no longer shared across repositories. Earlier the same day he also posted v3 of a documentation patch describing promisor remote fetch order, after Junio C Hamano asked him to drop a redundant fallback sentence.

## The day in brief
Kristofer Karlsson posted v2 of the `test-tool genbytes` series, closing out the discussion of `printf` alternatives and the rename question for the macOS test slowdown. 井上司 reported a reftable bug where the HEAD reflog records a stale value, and separately proposed an all-or-none approach for `git switch` when tracking-config writes fail. brian m. carlson's patch defaulting the hash algorithm to `GIT_HASH_DEFAULT` drew a new concern from Junio C Hamano about reinitializing existing repositories.

## Notable threads

### Reftable HEAD reflog records a stale value
井上司 reported that with the reftable backend, a transaction that updates a branch and points HEAD at it records the branch's old value in the HEAD reflog, or writes no entry at all when the branch is created in the same transaction, unlike the files backend. He traced the cause to `write_transaction_table()` in `refs/reftable-backend.c`, which resolves the target with pre-transaction refs, and proposed taking the new oid from the same transaction. Karthik Nayak agreed the diagnosis is plausible, noted the files backend avoids the bug only because it lacks atomicity support, and suggested moving reflog logic into the generic `refs.c` layer so backends receive pre-resolved entries, while saying the specific fix would also resolve the issue and be welcome.

### Defaulting the hash algorithm to `GIT_HASH_DEFAULT`
brian m. carlson posted a patch to `setup.c` defaulting the repository hash algorithm to `GIT_HASH_DEFAULT` when nothing else selects one, so breaking-changes mode and Git 3.0 pick SHA-256, with a new test in `t/t0001-init.sh`. Junio C Hamano said the patch looks good for its stated goal but asked whether `HASH_UNKNOWN` should instead mark a not-yet-initialized limbo state, and later raised that the patch would clobber the object format when `git init` reinitializes an existing repository, outlining possible reinit behaviors and asking what happens with the reference storage format.

### All-or-none behavior for switch/checkout tracking-config failure
井上司 reported that when writing tracking config fails during `git switch` or `checkout` of a new branch, the worktree and index are already updated but HEAD still points at the old branch, leaving the user with unrelated staged changes. He tested two approaches on the files and reftable backends: holding one reference transaction across the worktree update breaks other ref updates in reftable, HEAD.lock conflicts in files, and `-m` autostash, while a check-only transaction aborted before the real updates leaves nothing locked during the worktree update. He proposes the second approach, writing the tracking config before the worktree update and removing it if a later step fails, and asks whether it is acceptable given it does not cover a race with another process.

## In brief
- Tanga Joris fixed `read-cache.c` so the shared index is never written with a zeroed checksum under `index.skipHash`, which previously lost split-index entries, adding two tests to `t/t1700-split-index.sh`.
- Tanga Joris reviewed the `git credential` OPT_SUBCOMMAND conversion, suggesting a helper for the repeated `parse_options()` calls and a test that subcommands reject extra arguments with exit code 129.
- Jeff King flagged that `[[name_only]]` anchors in the rewritten `git diff` EXAMPLES section work with Asciidoctor but not python AsciiDoc, and questioned the placement of the `[[ignore_all_space]]` anchor.
- Colin Hinton will adopt Matt Hunter's cleaner `get_follow_remote_head()` approach in his next re-roll of the deferred `remote.<name>.followRemoteHEAD` validation and is holding until `hn/fetch-refmap` lands.
- Farid Zakaria's credential-cache daemon race fix drew questions from Junio C Hamano about which existing arrangements could be ignored; Zakaria replied that he was unsure how to respond and had scoped his contribution to what he felt comfortable with.
- Torsten Bögershausen reported an alternative t0027 optimization cutting runtime from 71 to 54 seconds on an older macOS.
- Laveesh Bansal posted a `builtin/reflog` patch expiring `--all` and named refs in a single loop, keeping the name lookup inside the loop so output and exit status are unchanged.
- Cynthia Uche posted Outreachy proposal draft v3 targeting `git bisect` for parse-options conversion, with an updated candidate inventory, methodology section, and week-by-week timeline.
