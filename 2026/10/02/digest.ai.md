# Git mailing list daily digest for 2026/10/02

## Long threads

### uploadpack.lazyFetchTrusted server-side lazy fetch control

*Long thread (root 2026/07/10/08-51-34): 34 replies so far from 3 participants over 85 days; 4 messages that day.*

Christian Couder posted v5 of the series introducing the protected `uploadpack.lazyFetchTrusted` config variable, which lets server operators mark repositories whose git dir is trusted for lazy fetching when serving clones. The series touches `builtin/upload-pack.c`, `promisor-remote.c`, `setup.c`, and `upload-pack.c`. V5 is rebased on a018953688 with no functional changes from v4, only comment, documentation, test, and commit message updates, including a NEEDSWORK comment about silently ignoring misspelled paths and new tests for `--separate-git-dir` and non-propagation of trust to promisor remotes. Couder also corrected his earlier explanation of the documentation wording, clarifying that the variable controls server-side lazy fetching where the server fetches missing objects for the listed repository, not from it.

### New gitmergeconflicts documentation page

*Long thread (root 2026/09/24/14-44-15): 29 replies so far from 6 participants over 9 days; 6 messages that day.*

Julia Evans's series adds a new `gitmergeconflicts` manual page explaining merge conflicts with examples and links to it from `git-merge`, `git-cherry-pick`, `git-pull`, `git-rebase`, and `git-revert`. Today she drafted a new "WHAT IS A MERGE CONFLICT?" section for v2 with a light introduction to 3-way merge, and put on her TODO list removing a sentence equating `git commit` with `git merge --continue` in favor of a vaguer statement. Junio C Hamano reviewed the section with wording suggestions ("since there's no overlap", "it marks them as conflicted") and offered an illustration of why `diff3` conflict style is often essential. Evans incorporated several suggestions and defended Git's conservative approach against more aggressive mergers like mergiraf, which Hamano agreed with but preferred to call "more honest".

### format-patch --[no-]range-diff-notes

*Long thread (root 2026/08/24/20-35-41): 23 replies so far from 4 participants over 40 days; 7 messages that day.*

Kristoffer Haugsbakk posted v3 of the series teaching `git format-patch` `--range-diff-notes=<ref>` and `--no-range-diff-notes` so the range diff can use a different notes ref than the patches. The only change from v2 is removing a redundant `test_when_finished`; patch 1/2 simplifies `get_notes_args` to take only `struct rev_info`, and patch 2/2 adds the options via a parse-options callback with seven new tests in `t/t3206-range-diff.sh`. Junio C Hamano reviewed both patches, calling the 1/2 simplification good and suggesting a condensed documentation paragraph and wrapping `parse_opt_string_list` for 2/2; Haugsbakk agreed to all of the 2/2 suggestions.

### rerere waiting for MERGE_RR.lock

*Long thread (root 2026/09/02/08-31-37): 23 replies so far from 4 participants over 31 days; 1 message that day.*

Thomas Bachem posted v6 of the series fixing rebase failures when a background `git rerere gc` holds `MERGE_RR.lock`. Patch 1/3 makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for the lock; patch 2/3 adds a hidden, undocumented `--skip-locked` option to `git rerere gc` passed by `git maintenance run --auto` and `git gc --auto`; patch 3/3 adds the `RERERE_WARN_LOCKED` flag so commands stopping at a conflict warn and go on without rerere while `git commit` and `git am --continue` still fail.

### git var -z, multiple variables, and identity variables

*Long thread (root 2026/08/25/20-46-42): 22 replies so far from 6 participants over 39 days; 4 messages that day.*

Andrew Pleeter's v9 series splits the earlier single patch into four: string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, support for multiple variable arguments printed as `VARIABLE=value` pairs, and broken-out `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables. Phillip Wood reviewed the series today and said it looks good to him, with only minor test nits (collapsing per-component identity tests, avoiding `test_env()` in favor of inline env assignment) that he did not think warranted a re-roll. Pleeter acknowledged both nits and said he would fold them in if a v10 happens, otherwise send them as a cleanup.

## The day in brief

Several series reached new revisions today: Christian Couder posted `uploadpack.lazyFetchTrusted` v5, Thomas Bachem posted rerere lock-handling v6, and Kristoffer Haugsbakk posted `format-patch` range-diff-notes v3 as refinements of earlier rounds. Patrick Steinhardt posted a fix for packfile corruption from stale delta base cache entries, and Junio C Hamano marked both Brigham Campbell's `git-contacts` stdin support and Harald Nordgren's `@{p}` shorthand for `next`.

## Notable threads

### packfile stale delta base cache corruption fix

*Long thread (root 2026/10/02/07-34-05): 7 replies so far from 5 participants over 1 day; 8 messages that day.*

Patrick Steinhardt posted a two-patch series fixing corruption caused by stale delta base cache entries: the cache is keyed by `struct packed_git` address plus offset and entries were never evicted on pack close, so a reallocated pack at the same address could reuse stale entries, reproducible via recursive submodule merges. Patch 2/2 adds `delta_base_cache_evict_entry()` called from `close_pack()`. Guillaume Chauvel proposed a deterministic test forcing address reuse, but Steinhardt declined, saying the infrastructure for such a specific edge case was not reasonable; Jeff King characterized the bug as a user-after-free that ASan would find reliably.

### refs copy and rename through transactions
Maciej Ciemborowicz's v2 patch represents reference copy and rename as transactions so the reference-transaction hook sees the logical updates. Patrick Steinhardt reviewed it but stopped reading, declaring the patch not reviewable and dominated by apparently AI-generated content, and said he did not want to invest time in the topic. Ciemborowicz responded with explanations of the design (operation state attached to the destination update, backend-specific reflog handling retained, new source/destination revalidation after backend locks) and performance measurements showing overhead from +3.4% to +137.7% depending on backend and hook presence, apologized, and offered a v3 split into more commits without guaranteeing it would meet standards.

### history: sign rewritten commits
Souma posted v4 of the series adding `-S`/`--gpg-sign[=<key-id>]` and `--no-gpg-sign` to `git history drop`, `fixup`, `reword`, `split`, and `squash`, with new signing support for `squash` including replayed descendants, via a `sign_commit` option in `replay_revisions_options` passed to `commit_tree_extended()`. Junio C Hamano reported that with the topic merged, `seen` breaks t9902, and asked whether the `git history split --git-completion-helper` output listing `--no-gpg-sign` after `--` is expected.

### fetch: avoid fetching every branch in shallow repos

*Long thread (root 2026/09/19/14-47-26): 21 replies so far from 4 participants over 14 days; 2 messages that day.*

Harald Nordgren posted v5 of the four-patch series adding `remote.<name>.refmap`, inferring branches to fetch from local upstreams, adding `git remote add --limited-fetch`, and defaulting to `--limited-fetch` in shallow repositories; the only change is renumbering the test file from t5585 to t5586. Junio C Hamano reviewed patch 3/4 and noted its commit message still says "plus the remote's default branch", which should have been removed when the series stopped fetching the default branch.

### repack corner cases for cruft-less MIDXs

*Long thread (root 2026/09/30/01-28-34): 21 replies so far from 5 participants over 3 days; 7 messages that day.*

Taylor Blau's v2 series addresses corner cases when `repack.midxMustContainCruft=false`, adding tree and tag roots, retaining cruft for ordinary incremental repacks, and including required packs in incremental MIDX writes so bitmap generation does not fail. Jeff King reviewed several patches today: he approved 4/8, suggested a De Morgan-equivalent form for the double-negation in 5/8, called 6/8 cleaner, but admitted he had a hard time following 8/8 and would rely on the functional tests.

## In brief
- Patrick Steinhardt posted a 13-patch series moving alternates into the "files" object database backend, dropping the `read_alternates` callback from `struct odb_source` and handling `GIT_OBJECT_DIRECTORY` and `GIT_ALTERNATE_OBJECT_DIRECTORIES` in the files backend.
- Kristofer Karlsson posted a series optimizing the post-fetch commit-graph write to seed from only newly updated refs, with a three-mode enum skipping no-op fetches, writing incrementally from updated tips, and falling back to a full scan; Patrick Steinhardt's review questions were answered with benchmark numbers on git.git.
- Karthik Nayak posted v2 of the packed-refs optimization using `fwrite()` via a new `write_packed_entry_raw()` when passing unchanged refs verbatim, noting the loss of sanitation as a side effect.
- Junio C Hamano noted Patrick Monette's `git replay -S` patch closely resembles another mid-July topic and asked him to review that instead, salvaging the remainder afterward.
- Brigham Campbell posted v6 of `git-contacts` stdin support (`git contacts - <patch`); Junio C Hamano acknowledged it and marked the topic for `next`.
- Harald Nordgren reposted v2 accepting `@{p}` as short for `@{push}`; Junio C Hamano called the commit message very well written and marked the patch for `next`.
- Diogo Castro reported the negative pathspec fix for `git ls-files`/`git add` has been incorporated into the referenced thread and closed the PR.
