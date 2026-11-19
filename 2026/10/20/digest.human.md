# Git mailing list daily digest for 2026/10/20

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 31 replies so far from 4 participants over 32 days; 1 message that day.*

Harald Nordgren posted v10 of the five-patch series that keeps a shallow repository from fetching every branch of a newly added remote; the v10 changelog says it resolves a merge conflict after master integrated `ch/fetch-followremotehead-lazy-validation`. The series adds a per-remote `remote.<name>.refmap` config supplying the default for `--refmap`, makes a refspec-less `git fetch <name>` infer what to fetch from local branches whose `@{upstream}` is on that remote, adds `--limited-fetch` to `git remote add`, and defaults to it when adding a remote to an already shallow repository. Earlier, Junio C Hamano had asked for a `collect_upstream_from_remote()` helper in `remote.c` and other restructuring, which earlier revisions incorporated.

### doc: rewrite detached HEAD state explanation

*Long thread: 20 replies so far from 3 participants over 14 days; 2 messages that day.*

Julia Evans's standalone `gitdetached-head` guide rewrite continued its wording debate. She confirmed that many potential readers do not understand `HEAD` and are doing fine, and explained that she shared quotes from experienced users to show the language people actually use when describing Git, since she cannot accept suggested wordings verbatim; she asked for some trust in her sorting of user comments and added an XKCD reference to her point about unsettling comments. Earlier, Junio C Hamano had questioned what she aimed to illustrate with the quotes and had recorded replacement text for the opening paragraphs.

### doc: add new Git tutorial for beginners

*Long thread: 19 replies so far from 5 participants over 15 days; 13 messages that day.*

Julia Evans's v2 two-patch series replacing `gittutorial` drew review. Junio C Hamano called the 739-line beginner tutorial well-paced and free of common traps, with nitpicks: missing hyphens in the Step 3 and Step 10 heading underlines (which surfaced as build warnings), `ssh-keygen -t ed25519`, `cat ~/.ssh/id_ed25519.pub`, and capitalizing "Push your changes". Julia Evans agreed to the capitalization, leaned toward using SSH's default key type, and proposed Atx-style headings; Junio noted Setext underlines make level mistakes easier to spot. Earlier, the v2 patches had briefly failed to appear in the list archive.

### odb: introduce object storage extension

*Long thread: 14 replies so far from 3 participants over 2 days; 10 messages that day.*

Patrick Steinhardt's seven-patch series adding the `extensions.objectStorage` extension -- configuring the object database source as `<schema>[://<payload>]`, currently only `files`, whose payload can relocate `.git/objects` -- drew follow-up review. He answered Karthik Nayak's questions: relative payload paths stay for now, `gettext.h` is needed for translatable strings, `DEFAULT_REPO_FORMAT` is a protective measure, and he will look at not exposing the default format outside the repository/odb subsystems. He also explained the `object.format` vs `objects.storageFormat` naming clash and argued against a full 30-combination test matrix.

### github-ci: update macOS jobs from macos-14

*Long thread: 10 replies so far from 4 participants over 5 days; 8 messages that day.*

Junio C Hamano posted v4 of the patch updating the four `osx-*` jobs in `.github/workflows/main.yml` from `macos-14` -- `macos-15` for two and explicit `macos-26` for the other two, so GitHub and GitLab CI stay comparable -- then retracted it as bogus, having decided to use v2 instead. Earlier, Patrick Steinhardt had suggested mixing `macos-15` and `macos-26` for coverage, and Todd Zullinger had asked about `macos-latest`.

## The day in brief
Harald Nordgren posted v10 of the shallow-repo fetch series, resolving a merge conflict with recently integrated work. Julia Evans's new beginner tutorial and detached-HEAD guide both drew detailed review, and Patrick Steinhardt's object-storage extension answered its remaining review questions. Junio C Hamano retracted his v4 macOS CI patch as bogus and settled on v2.

## Notable threads

### read-cache: do not trust a size change when conversion is active
Torsten Bögershausen argued on the v5 patch -- which fixes `git status` reporting a file modified after a CRLF rewrite that `git diff` and `git add` treat as unchanged -- that a changed file size does not prove the filtered content changed, since `ce_modified_check_fs()` verifies content; he proposed dropping the size shortcut in `ie_modified()` entirely and reported all tests pass.

### odb/source-files: reload alternates on retry
Patrick Steinhardt posted a four-patch series restoring alternates reloading when an object is not found, a regression from 5946a564cd, with the last two patches bringing 20k missing-object lookups from 309ms back to 197ms via a hashmap-first check and `stat_validity` tracking. Junio C Hamano questioned whether the validity update should happen only after a successful read.

### object-file: fix ".keep" lockfile ownership in ODB transactions
Justin Tobler posted v3 of the four-patch series migrating `.keep` lockfile creation to ODB transaction commit so a `git receive-pack` process no longer deletes another process's lockfile. Patrick Steinhardt reviewed it as good overall and suggested setting `ret` explicitly on all exit paths, which Tobler agreed to fix in a new version.

## In brief
- Harald Nordgren posted v2 of a fix for slow fetches from shallow repositories over protocol v2, writing shallow lines through the packet writer.
- Justin Tobler discussed the v3 fetch patch committing references before backfilling tags, suggesting the pending transaction's refs could supplement the backfill "haves".
- Phillip Wood posted v3 of the stash series removing duplicate index loads; Junio C Hamano praised the temporary-index rework.
- 井上司 reported `git switch`/`checkout` stopping halfway when tracking-config writes fail, and proposed making `setup_tracking()` return an error.
- Patrick Steinhardt argued the reftable object-record overflow should be fixed in `reftable_alloc_size()` with `uintmax_t` parameters.
- Jason Newton posted v2 adding `git worktree add --reflink` and ephemeral-branch reaping; brian m. carlsson flagged the Windows ReFS arm as defective.
- Matt Hunter posted a patch restoring the absolute path in `git init`'s output message; discussion continued over whether a regression test is needed. *Long thread: 7 replies so far from 4 participants over 1 day; 8 messages that day.*
- Harald Nordgren's patch making `git status` compare against both `@{upstream}` and `@{push}` by default drew requests to explain the default change.
