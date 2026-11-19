# Git mailing list daily digest for 2026/08/18

## Long threads

### `git repo info` path keys (GSoC series)

*Long thread (root 2026/07/16/01-21-31): 24 replies so far from 4 participants over 34 days; 3 messages that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each with absolute and relative suffixes -- and removes a now-unused `setup.h` include from `builtin/repo.c` after the prefix member moved into `struct repository`. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, dropped `path.objects`, and added relative-path tests. Today K Jayatheerth explained why the `path.toplevel` test does not use the `test_repo_info_path` helper: the helper assumes an `$expected_dir` suffix and builds `$ROOT/$expected_dir` and `../$expected_dir`, whereas toplevel is `$ROOT` and `../`. He also agreed that `path.git-prefix` is technically current-directory information rather than repository information, but kept it because it is useful path information for scripts operating within a repository and is already exposed by `git rev-parse --show-prefix`, offering to drop it if Lucas Seiki Oshiro prefers. Junio C Hamano argued that `path.git-prefix` is repository information expressed as a relative path -- the inverse of a `path.root-of-the-working-tree.relative` (aka `path.cdup`) -- drawing parallels to `git rev-parse --show-cdup` and `path.commondir.relative`, and added that if `git-prefix` is offered, `cdup` should be offered as well.

## The day in brief

Patrick Steinhardt posted a seven-patch series making `OBJECT_INFO_DIE_IF_CORRUPT` handling backend-agnostic in the object database, Elijah Newren fixed a race where a multi-pack-index naming a removed pack made present objects look missing, and Kristofer Karlsson proposed an RFC to make `check_connected()` cost proportional to the incoming set. Grayson Gordon's `http.sslVerifyStatus` reached v6, and Artur Bieniek proposed a `--hard` mode for `git pull`.

## Notable threads

### Generic `OBJECT_INFO_DIE_IF_CORRUPT` handling in the ODB

Patrick Steinhardt posted a seven-patch series that moves `OBJECT_INFO_DIE_IF_CORRUPT` handling into `do_oid_object_info_extended()` generically, extending `odb_source_read_object_info()` to distinguish missing objects (positive return) from corrupt ones (negative return) and to bubble up error messages, removing `has_packed_and_bad()` and the loose source's direct `die()` so the check is no longer tied to the "files" backend. The series adapts the in-memory, packed, loose, and files sources to the tri-state convention, adds an `errmsg` parameter to the callback, and includes a test that a corrupted loose commit can still be read from an alternate. Junio C Hamano questioned the positive-return convention for missing objects, preferring "missing" as a negative value distinct from other errors, suggested a named enum such as `ODB_ROI_MISSING = 1` instead of a hardcoded `1`, thanked Patrick for fixing the case where an already-marked-bad packed object was reported as missing, approved renaming the `corrupt:` label to `out:`, and corrected the subject of patch 5/7 ("mark" should be "missing").

### Recovering from a multi-pack-index naming a removed pack

Elijah Newren posted a two-patch series addressing objects being treated as missing despite being present due to a race between geometric repacking and multi-pack-index updates, which he found evidence of in at least eight server-side operations across seven git commands including replay (SIGSEGV), merge-tree, diff, rev-list --count, merge-base, object resolution, and repository repair. The first patch makes `git replay` fail gracefully with an error message instead of SIGSEGV when a merge input is unreadable, and the second teaches `find_pack_entry()` to recover when a multi-pack-index names a pack removed by a concurrent geometric repack by scanning the MIDX's covered packs directly for a surviving copy, fixing spurious missing-object reports for all readers including `OBJECT_INFO_QUICK` callers.

### RFC: incoming-proportional `check_connected()`

Kristofer Karlsson proposed an RFC to make `check_connected()` -- used by fetch and receive-pack -- cost proportional to the incoming set vs repository size, defining "trusted object" and "boundary commit". He proposed two optimizations: opportunistic trusted-tree discovery that caches verified objects and a parallel tree walk against trusted parents, and bounded boundary traversal seeded with old ref values and a walk budget, falling back to `--not --all` if unclosed. He reported prototype results on a large repo (3M commits, 200K refs) with the check dropping from ~1s to ~0.03s for a 5-commit push and ~0.007s for a 1-commit push. The appendix notes replace objects and deepening fetches fall back to the rev-list path, while promisor remotes use the promisor-pack fast path and shallow clones get full closure verification.

### `git pull --hard`

Artur Bieniek submitted a patch adding a `--hard` mode to `git pull` that, after fetching, requires a single integration candidate and resets the current branch, index, and working tree to it, while preserving quiet and submodule recursion behavior and rejecting options that cannot be honored by a hard reset. Junio C Hamano expressed mild opposition, arguing the behavior is already safely and explicitly achieved by `git fetch` followed by `git reset --hard @{u}`, that the core purpose of `git pull` is history integration, and that adding a destructive option makes it easier for users with uncommitted work to trigger data loss by accident.

### `http.sslVerifyStatus` for stapled OCSP responses

Grayson Gordon posted v6 of a patch adding an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`, since git never sets that option and stapled OCSP responses -- including ones stating a certificate has been revoked -- are ignored. The documentation now reads that the default false allows connections without validating revocation, while enabling it refuses certificates with a status other than "good" and servers that return no stapled response. Earlier in the day Patrick Steinhardt reviewed v4, questioning backend-specific claims and suggesting `curl_easy_strerror()` in the die() message; Grayson Gordon posted v5 with the error string and a reworked commit message, and Junio C Hamano objected to describing the option as "fail-closed", leading to the v6 rephrasing.

## In brief

- Karthik Nayak introduced a `report` hook for `git-receive-pack(1)` that receives the pkt-line status report on stdin and whose stdout replaces the report sent to the client; Junio C Hamano called the design "ugly, but may work OK" but said it needs clearer documentation.
- D. Ben Knoble posted v3 converting build-time `USE_NSEC` into runtime `core.useNanosec`; Junio C Hamano suggested unconditionally initializing the config member and simplifying `is_racy_stat()`.
- **git config -e exit code**: Discussion continued on surfacing editor failure from `git config -e`; brian m. carlson argued the error should propagate since, other than ed(1) and POSIX vi(1), editors exit nonzero only on error.

- Kristoffer Haugsbakk posted v2 of a series adding `--abbrev`, `--color`, and `--date` to format-rev, including a fix to release parsed date data.
- Lutz Lengemann pushed a v2 of zsh completion support for `git -C <path>` to his GitHub repo; D. Ben Knoble noted it would need a resubmit to reach the list.
- Justin Tobler accepted Patrick Steinhardt's patch raising the geometric-repack loose-object threshold from 100 to 6700 to match `git-gc(1)`.
- Patrick Steinhardt defended keeping an `extern` declaration in `bundle.c` while removing `USE_THE_REPOSITORY_VARIABLE`.
- Junio C Hamano suggested simplifying Friel's pack-objects trace patch by incrementing `bytes_written` in the common code path.
- Ernst Reissner requested that git incorporate Gittyup's linewise file indexing feature.
