# Git mailing list daily digest for 2026/10/27

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread: 57 replies so far from 4 participants over 39 days; 6 messages that day.*

Harald Nordgren posted v12 of the series keeping a shallow repo from fetching every branch of a new remote: `remote.<name>.refmap` is renamed to `remote.<name>.fetchRefmap` across code, docs and tests, a missing `get_ref_map()` path now stores branches fetched by name via `remote.<name>.fetch` through the fetch refmap (with a "cabal" test), and the docs state that `remote.<name>.fetch` still decides what is fetched while the refmap only remaps it. Earlier, Phillip Wood and Junio C Hamano argued that "what to fetch" and "where to store it" need separate configuration; today Phillip Wood asked about dst-less refspecs, and Junio C Hamano answered they store only in `FETCH_HEAD`.

### reflog: name the branch of each reflog entry

*Long thread: 31 replies so far from 6 participants over 13 days; 5 messages that day.*

Harald Nordgren posted v3 of the reflog series, dropping v2's `--operations` and `--historical-branch` for a `%gb` placeholder in `--format` (e.g. `git log -g --format='%h %gd: (%gb) %gs'`) that names the branch HEAD was on when each entry was recorded, following switches, renames and rebases. Earlier, Phillip Wood preferred a format specifier and D. Ben Knoble agreed the reflog is not a true operations log. Today Ben Knoble argued detaching HEAD for `git am` would make such a change more invisible to users; Junio C Hamano called detaching a flexibility, and Harald Nordgren will use `git reflog @{0}` from now on.

### doc: add new git tutorial

*Long thread: 31 replies so far from 6 participants over 22 days; 3 messages that day.*

Julia Evans defended calling vi "extremely difficult" in her beginner tutorial to warn against learning it mid-tutorial, not to judge vim, and proposed a "show don't tell" approach. She agreed to say "bottom-left" for the tildes and to rephrase the escape steps as "press the Escape key, type ..., and press Enter." Earlier, Richard Kerry suggested ssh could be marked for Linux/Unix users after noting vim and ssh made Windows users feel excluded.

### odb: introduce object storage extension

*Long thread: 25 replies so far from 4 participants over 9 days; 1 message that day.*

Patrick Steinhardt posted v4 of the series adding the `extensions.objectStorage` extension, modeled on the ref storage extension with `<schema>[://<payload>]` syntax; only the "files" backend exists, whose payload can relocate `.git/objects` (e.g. `files://objs`). The series adds `GIT_OBJECT_STORAGE_FORMAT`, `git rev-parse --show-object-storage-format`, and `--object-storage-format=` for `git init`, `git clone` (explicit `--local` dies on non-default formats; implicit local falls back with a warning) and `git submodule`.

### diff-delta: use safe integer arithmetic

*Long thread: 22 replies so far from 5 participants over 5 days; 12 messages that day.*

Srujith A Poojary posted v9 replacing manual size arithmetic in `create_delta_index()` with the overflow-checked `st_mult`/`st_add`/`st_add3` helpers. After Pablo Sabater corrected the 32-bit struct size (12 bytes, not 16) and the entries cap (`0xfffffffeU / RABIN_WINDOW`), and Junio C Hamano challenged fixed-size claims as implementation-defined, v9 frames the change as defensive--future padding changes die on overflow instead of wrapping--while keeping the exact bounds (`hsize` capped at 2^26, minimum 16).

## The day in brief
Harald Nordgren posted new revisions of two long-running series: v12 of the shallow-fetch refmap work and v3 of the reflog `%gb` placeholder. Patrick Steinhardt advanced the object storage extension to v4 and fixed a `BUG()` crash in `pack-objects --exclude-promisor-objects-best-effort` on missing objects in partial clones.

## Notable threads

### pack-objects: handle missing objects when excluding promisor objects
Patrick Steinhardt fixed a `BUG()` crash in `git pack-objects --exclude-promisor-objects-best-effort` when the object walk hits missing objects, as in `tree:0` partial clones; `is_not_in_promisor_pack_obj()` now skips them instead of aborting, with regression tests in `t/t5616-partial-clone.sh`. Mark C. Chu-Carroll suggested defining promisor and non-promisor objects parenthetically in the opening paragraph.

### odb/files: reload alternates on retry

*Long thread: 13 replies so far from 3 participants over 8 days; 9 messages that day.*

Christian Couder reviewed v2 of Patrick Steinhardt's series restoring alternates reloading when an object lookup fails. He flagged a race in 6/6--`stat_validity_update()` runs after reading the alternates file, so an in-place rewrite during parsing could go unparsed--and proposed updating before reading as in `refs/packed-backend.c`; Steinhardt agreed and will adapt it. Couder also asked whether 5/6 still prints "object directory does not exist" for a removed alternate; Steinhardt said only when adding a nonexistent one.

### refs: fix loose-ref seeking across directory boundaries
Junio C Hamano asked for objections to making Venkat Ragavan S's loose-ref seeking fix ready for `next`, saying it seems ready aside from a test that confused a reader. The patch includes the trailing slash when comparing directory components so a seek past `refs/tags/releases/v1` descends into `releases/` instead of landing on `releases-z`.

## In brief
- Harald Nordgren posted v4 of "Fix imap-send with reordered message headers": headers may appear in any order, names match case-insensitively, and the `git send-email` special case is dropped; Patrick Steinhardt is happy with it.
- Phillip Wood posted a 3-patch series so `git rebase --continue` removes `REBASE_HEAD` and `AUTO_MERGE`; Junio C Hamano reviewed all three patches positively.
- Pablo Sabater proposed removing the 10k-OID limit on `cat-file remote-object-info` and batching internally (default 50k, `--remote-max-batch=<n>`), and discussed a server-side cap with Junio C Hamano.
