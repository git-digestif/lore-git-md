# Git mailing list daily digest for 2026/10/02

## Long threads

### uploadpack.lazyFetchTrusted server-side lazy fetch control

*Long thread: 34 replies so far from 3 participants over 85 days; 4 messages that day.*

Christian Couder posted v5 of the series introducing the protected `uploadpack.lazyFetchTrusted` config variable, which lets server operators mark repositories whose git dir is trusted for lazy fetching when serving clones. The series touches `builtin/upload-pack.c`, `promisor-remote.c`, `setup.c`, and `upload-pack.c`. V5 is rebased on a018953688 with no functional changes from v4, only comment, documentation, test, and commit message updates, including a NEEDSWORK comment about silently ignoring misspelled allowlist paths and new tests for `--separate-git-dir`. Couder also corrected his earlier explanation of the documentation wording, clarifying that the variable controls the server lazily fetching missing objects _for_ the listed repository, not from it.

### gitmergeconflicts documentation guide

*Long thread: 29 replies so far from 6 participants over 9 days; 6 messages that day.*

Julia Evans drafted a new "WHAT IS A MERGE CONFLICT?" section for the v2 of the new `gitmergeconflicts` manual page, giving a light introduction to 3-way merge and explaining that Git combines non-overlapping edits but refuses to guess on overlapping lines. Junio C Hamano gave review reactions -- "since there's no overlap", "cannot guess", "it remembers them as conflicted" -- and offered an illustration of why `diff3` conflict style is often essential. Evans incorporated several suggestions and explained she views Git's conservative approach as a feature compared to more aggressive mergers; Hamano agreed but preferred calling it "more honest".

### format-patch --range-diff-notes

*Long thread: 23 replies so far from 4 participants over 40 days; 7 messages that day.*

Kristoffer Haugsbakk posted v3 of the series teaching `git format-patch` `--range-diff-notes=<ref>` and `--no-range-diff-notes` so the range diff can use a different notes ref than the patches. The only change from v2 is removing a redundant `test_when_finished`. Junio C Hamano reviewed both patches, calling the `get_notes_args` simplification good and asking for the conversational narrative to be stripped from commit messages and a condensed documentation paragraph; Haugsbakk agreed to all of the 2/2 suggestions.

### rerere MERGE_RR.lock waiting

*Long thread: 23 replies so far from 4 participants over 31 days; 1 message that day.*

Thomas Bachem posted v6 of the series making rerere wait for `MERGE_RR.lock` up to `rerere.lockTimeout` milliseconds before failing, adding a hidden `--skip-locked` option to `git rerere gc` used by `git maintenance run --auto` and `git gc --auto`, and letting commands that stop at a conflict warn and go on without rerere via the renamed `RERERE_WARN_LOCKED` flag while `git commit` and `git am --continue` still fail.

### git var -z, multiple variables, broken-out idents

*Long thread: 22 replies so far from 6 participants over 39 days; 4 messages that day.*

Andrew Pleeter's v9 series splits the work into four patches: string_list representation for multi-valued variables, a `-z` NUL-terminated output mode, support for multiple variable arguments, and new `GIT_AUTHOR_*`/`GIT_COMMITTER_*` identity variables. Phillip Wood reviewed the series and said it looks good, with only minor test-style nits about `test_env()` and collapsing per-component tests; Pleeter acknowledged both and said he would fold them in if a v10 happens.

## The day in brief
Christian Couder's v5 of `uploadpack.lazyFetchTrusted` and Thomas Bachem's v6 rerere lock handling both arrived as new revisions, while Patrick Steinhardt's packfile delta-base-cache fix drew discussion including an ASan suggestion from Jeff King. The refs copy/rename transaction patch hit a review impasse over apparently AI-generated content.

## Notable threads

### packfile stale delta base cache corruption fix

*Long thread: 7 replies so far from 5 participants over 1 day; 8 messages that day.*

Patrick Steinhardt posted a two-patch series fixing packfile corruption from stale delta base cache entries, which are keyed by `struct packed_git` address plus offset and were never evicted on pack close; the bug is reproducible via recursive submodule merges. Guillaume Chauvel proposed a deterministic test, which Steinhardt declined as too much infrastructure for the edge case; Jeff King characterized the core bug as use-after-free and suggested ASan would find it reliably.

### history: sign rewritten commits
Souma posted v4 of the series adding `-S`/`--gpg-sign` and `--no-gpg-sign` to the `git history` rewrite commands, now including `squash`, with a `sign_commit` option in the replay API. Junio C Hamano reported that with the topic merged, `seen` breaks t9902 and asked whether the completion output listing `--no-gpg-sign` is expected.

### refs copy/rename through transactions
Maciej Ciemborowicz's v2 patch represents branch copy and rename as reference transactions so the reference-transaction hook sees the logical updates. Patrick Steinhardt stopped reading, declaring the patch not reviewable and dominated by apparently AI-generated content; Ciemborowicz answered the technical questions, reported performance overhead measurements, apologized, and offered a v3 split into more commits.

## In brief
- Patrick Steinhardt posted a 13-patch series moving alternates into the "files" object database backend, dropping the `read_alternates` callback from `struct odb_source`.
- **fetch: commit-graph from updated refs only**: Kristofer Karlsson posted a series making fetch write the commit-graph from updated refs only; Patrick Steinhardt reviewed, noting no benchmark was provided, and the author answered with git.git numbers for a future reroll.

- Scott Chacon posted an RFC signing a SHA-256 digest of tree contents in commits and tags; brian m. carlson rejected it in favor of moving to SHA-256.
- Jeff King proposed a "limbo" object-format state for empty repositories so the server adopts the client's hash on first push.
- **Branch hook gaps**: Поникаров reported `git branch -C` and `reflog delete --updateref --rewrite` skip the reference-transaction hook while `-c`/`-m` invoke it without the new value, and carlsson linked a related patch thread.
