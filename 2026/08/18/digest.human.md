# Git mailing list daily digest for 2026/08/18

## Long threads

### `git repo info` path keys (GSoC series)

*Long thread: 24 replies so far from 4 participants over 34 days; 3 messages that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each with absolute and relative suffixes. Today he explained why the `path.toplevel` test does not use the `test_repo_info_path` helper: the helper assumes an `$expected_dir` suffix and builds `$ROOT/$expected_dir` and `../$expected_dir`, whereas toplevel is `$ROOT` and `../`. He also agreed with Lucas Seiki Oshiro that `path.git-prefix` is technically current-directory information rather than repository information, but kept it because it is useful path information for scripts operating within a repository and is already exposed by `git rev-parse --show-prefix`, offering to drop it if Oshiro thinks it should be kept separate. Junio C Hamano countered that `path.git-prefix` is repository information expressed as a relative path -- the inverse of a `path.cdup` -- and added that if `git-prefix` is offered, `cdup` should be offered as well.

## The day in brief

Patrick Steinhardt posted a series making corrupt-object handling backend-agnostic across object database sources. Elijah Newren posted a series addressing a race where a multi-pack-index naming a pack removed by geometric repacking made present objects look missing, including a `git replay` SIGSEGV. Kristofer Karlsson proposed making the connectivity check cost proportional to the incoming set rather than repository size.

## Notable threads

### Multi-pack-index recovery after geometric repack

Elijah Newren posted a two-patch series for objects treated as missing despite being present: a race where geometric repacking creates a new multi-pack-index and removes the pack an older multi-pack-index named as owner, so a running process that opened the old index cannot reach the object. The series makes `git replay` fail gracefully with an error message instead of SIGSEGV when a merge input is unreadable, and teaches `find_pack_entry()` to scan the multi-pack-index's covered packs directly for a surviving copy when the recorded owner is unavailable.

### Generic `OBJECT_INFO_DIE_IF_CORRUPT` handling

Patrick Steinhardt posted a seven-patch series moving `OBJECT_INFO_DIE_IF_CORRUPT` handling into `do_oid_object_info_extended()` generically, with object sources returning tri-state values (0 success, positive missing, negative corrupt) and bubbling up error messages, removing `has_packed_and_bad()` and the loose source's direct `die()`. Junio C Hamano questioned the positive-return convention for missing objects, preferring a negative value distinct from other errors, and suggested a named enum such as `ODB_ROI_MISSING = 1`; he also approved renaming the loose source's `corrupt:` label to `out:` and corrected patch 5/7's subject.

### Incoming-proportional connectivity check

Kristofer Karlsson proposed an RFC to make `check_connected()`, used by fetch and receive-pack, cost proportional to the incoming set rather than repository size, via opportunistic trusted-tree discovery and bounded boundary traversal seeded with old ref values, falling back to `--not --all` when the boundary is not closed. He reported prototype results on a repository with 3M commits and 200K refs dropping the check from about 1s to about 0.03s for a five-commit push.

## In brief

- D. Ben Knoble posted v3 converting build-time `USE_NSEC` into the runtime `core.useNanosec` config; Junio C Hamano suggested unconditionally initializing the member and simplifying `is_racy_stat()`.
- Grayson Gordon posted v6 of `http.sslVerifyStatus` for stapled OCSP responses, with documentation rephrased per Junio C Hamano's suggestion.
- Artur Bieniek submitted `git pull --hard`; Junio C Hamano expressed mild opposition, noting `git fetch` followed by `git reset --hard @{u}` already achieves the behavior safely.
- Karthik Nayak introduced a `report` hook for `git-receive-pack`; Junio C Hamano called the design "ugly, but may work OK" pending clearer documentation.
- Contributors debated surfacing editor failure in `git config -e`'s exit code; brian m. carlsson argued the nonzero status should be propagated.
- Kristoffer Haugsbakk posted v2 of the format-rev series adding `--abbrev`, `--color`, and `--date`.
- Justin Tobler accepted the patch raising the geometric-repack loose-object threshold to 6700.
- Patrick Steinhardt defended keeping an `extern` declaration in `bundle.c` while dropping `USE_THE_REPOSITORY_VARIABLE`.
