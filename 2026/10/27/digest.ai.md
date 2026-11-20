# Git mailing list daily digest for 2026/10/27

## Long threads

### fetch: avoid fetching every branch of a new remote in a shallow repo

*Long thread (root 2026/09/19/14-47-26): 57 replies so far from 4 participants over 39 days; 6 messages that day.*

Harald Nordgren posted v12 of the series that keeps a shallow clone from fetching every branch of a newly added remote. The revision renames `remote.<name>.refmap` to `remote.<name>.fetchRefmap` across code, docs and tests, adds the missing `get_ref_map()` path so branches fetched by name via `remote.<name>.fetch` are stored through the fetch refmap (with a "cabal" test), rewords the docs to state that `remote.<name>.fetch` still decides what is fetched while the refmap only remaps it, and adds a prefetch test to `t5586-fetch-refmap.sh`. Earlier, Phillip Wood and Junio C Hamano had argued that "what to fetch" and "where to store it" need separate configuration; today Phillip Wood asked whether dst-less refspecs would make older Git versions update local branches, and Junio C Hamano answered that they store results only in `FETCH_HEAD`, keeping `git pull` working. D. Ben Knoble questioned whether the `git clone --single-branch` example in the `collect_upstream_from_remote()` commit message captures the same narrow-refspec remote.

### reflog: name the branch of each reflog entry with %gb

*Long thread (root 2026/10/15/13-13-55): 31 replies so far from 6 participants over 13 days; 5 messages that day.*

Harald Nordgren posted v3 of the reflog series, dropping v2's `--operations` and `--historical-branch` options in favor of a `%gb` placeholder for `--format` (e.g. `git log -g --format='%h %gd: (%gb) %gs'`) that names the branch HEAD was on when each entry was recorded, across branch switches, renames and rebases, with "detached at <commit>" for a detached HEAD; a companion patch defines the reflog marker strings once in `reflog.h` with no behavior change. Earlier, Phillip Wood had preferred a format specifier over a dedicated option, and Kristoffer Haugsbakk had retracted his original motivation after finding per-branch reflogs already show rebase entries. Today Ben Knoble argued that detaching HEAD to work around `git am` supports eliminating such workarounds, while Junio C Hamano called detaching HEAD a flexibility rather than a limitation; Harald Nordgren agreed on the bigger picture and said he will use `git reflog @{0}` from now on.

### doc: add new Git tutorial for beginners

*Long thread (root 2026/10/06/19-37-00): 31 replies so far from 6 participants over 22 days; 3 messages that day.*

Julia Evans' v3 series replacing `gittutorial` with a new 782-line beginner tutorial (covering `git init` through `git push`, without branches or collaboration) drew further review. Evans defended keeping "extremely difficult" to describe vi as a warning not to learn it mid-tutorial, agreed to change "left of the screen" to "bottom-left" and to rephrase the Escape instructions, and said she will link the appendix's editor list and consider a "show don't tell" explanation of vi. Earlier, Richard Kerry had asked that ssh and vim mentions not alienate Windows users, and Junio C Hamano had marked the topic "Expecting a reroll."

### odb: introduce the object storage extension

*Long thread (root 2026/10/19/08-55-55): 25 replies so far from 4 participants over 9 days; 1 message that day.*

Patrick Steinhardt posted v4 of the series introducing the `extensions.objectStorage` extension, mirroring the ref storage extension with `<schema>[://<payload>]` syntax and only a "files" backend whose payload can relocate the `.git/objects` directory (e.g. `files://objs`). The series adds `GIT_OBJECT_STORAGE_FORMAT`, `git rev-parse --show-object-storage-format`, `git repo info`'s `objects.storageFormat` field, and `--object-storage-format=` for `git init`, `git clone` and `git submodule`, with local clones falling back to non-local transport when either side uses a non-default format. V4 adds justification for the payload and removes the dead `repo_write_loose_object_map()`; Karthik Nayak had earlier reviewed v3 as good apart from nits.

### diff-delta: use safe integer arithmetic

*Long thread (root 2026/10/23/03-56-48): 22 replies so far from 5 participants over 5 days; 12 messages that day.*

Srujith A Poojary posted v9 replacing manual size arithmetic in `create_delta_index()` with overflow-checked `st_mult`/`st_add`/`st_add3` helpers. After Pablo Sabater corrected the 32-bit struct size (12, not 16 bytes) and the entries cap (`0xfffffffeU / RABIN_WINDOW`), and Junio C Hamano challenged the "exactly 16 bytes" claim as implementation-defined, v9 drops fixed struct-size claims and frames the change as defensive overflow checking, retaining the exact bounds (entries capped by `0xfffffffeU / RABIN_WINDOW`, `hsize` capped at 2^26 with minimum 16).

## The day in brief
Several series advanced with new revisions: Harald Nordgren posted v12 of the shallow-repo fetch refmap series and v3 of the reflog `%gb` series, Patrick Steinhardt posted v4 of the object storage extension, and Srujith A Poojary posted v9 of the diff-delta overflow fix. Patrick Steinhardt also fixed a `BUG()` crash in `git pack-objects --exclude-promisor-objects-best-effort` with missing objects, and Julia Evans announced she would drop her glossary "object ID" patch after the discussion.

## Notable threads

### pack-objects: handle missing objects when excluding promisor objects
Patrick Steinhardt submitted a patch fixing a `BUG()` crash in `git pack-objects --exclude-promisor-objects-best-effort` when the object walk reaches missing objects, as happens with `tree:0` partial clones; `is_not_in_promisor_pack_obj()` now skips missing objects instead of aborting, with regression tests in `t/t5616-partial-clone.sh`. Mark C. Chu-Carroll suggested rewording the opening paragraph.

### move the Rust gitcore crate into a rust/ subdirectory
Junio C Hamano noted that Mike Hommey's patch moving all Rust-specific files into a dedicated `rust/` subdirectory (so a top-level `Cargo.toml` no longer implies a bare `cargo build` works) has drawn no comments in over a month and is not near `next`, and questioned how much the project wants the topic.

### glossary: "object ID" versus "object name"
Julia Evans' patch making "object ID" the canonical glossary term met resistance: Ben Knoble argued "ID" should mean only the hash while other spellings are notations, and Junio C Hamano defended "object name" as the term established in 2005, saying "main" is not an object ID but can be used where one is called for. Evans acknowledged the change was more controversial than she thought and announced she would drop the patch.

### odb/files: reload alternates on retry

*Long thread (root 2026/10/20/08-54-51): 13 replies so far from 3 participants over 8 days; 9 messages that day.*

Christian Couder reviewed Patrick Steinhardt's v2 series restoring alternates reloading when an object lookup misses: he flagged a race in 6/6 where `stat_validity_update()` runs after reading the alternates file, so an in-place rewrite during parsing could go unparsed, and Steinhardt agreed to adapt the fix. Couder also asked whether 5/6's stat skip drops the "object directory does not exist" error for removed alternates; Steinhardt said it still prints when adding a nonexistent alternate but not when an added one disappears later.

### object-file: fix ".keep" lockfile ownership in ODB transactions

*Long thread (root 2026/10/16/02-28-58): 21 replies so far from 4 participants over 12 days; 2 messages that day.*

Justin Tobler agreed to add an `unlink_or_warn()` warning in the error path when writing the ".keep" lockfile fails, but noted the v4 series fixing quarantine ".keep" deletion by concurrent receive-pack processes is already in `next`, so he will send the change as a separate patch.

## In brief
- Pablo Sabater proposed removing the 10k-OID limit on `cat-file remote-object-info` and batching internally (default 50k) over one connection; Junio C Hamano asked about deployed servers, and Sabater proposed a server-side cap advertised via handshake.
- Patrick Steinhardt posted an 8-patch series consolidating CI test variables; Phillip Wood called it nice, and Jeff King's simpler `make FEATURE_TESTBED=1` alternative for the last patch was endorsed by Junio C Hamano. *Long thread (root 2026/10/27/08-18-20): 7 replies so far from 4 participants over 1 day; 8 messages that day.*
- Patrick Steinhardt posted a 12-patch series converting `builtin/rerere` to `OPT_SUBCOMMAND()`; Junio C Hamano found it well done.
- Jeff King proposed running clar unit tests through `test-lib.sh` via a new `t/t0011-unit-tests.sh`, and (RFC) dropping the shell JUnit XML reporting code.
- Harald Nordgren posted v4 making CI fail when perforce, git-lfs or JGit installs fail; Patrick Steinhardt said the version looks good. *Long thread (root 2026/09/12/14-38-02): 21 replies so far from 3 participants over 46 days; 2 messages that day.*
- Harald Nordgren posted v4 of the imap-send header-order fix (any header order, case-insensitive names); Patrick Steinhardt is happy with it.
- Mark C. Chu-Carroll posted v3 of `git repo structure` concern reporting, restructured to two patches with configurable `repo.concern.*` thresholds.
- Phillip Wood posted a 3-patch series removing `REBASE_HEAD` and `AUTO_MERGE` at the end of a rebase; Junio C Hamano called it overall very good.
