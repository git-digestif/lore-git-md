# Git mailing list daily digest for 2026/08/29

## Long threads

### packfile recovery when a multi-pack-index names a removed pack

*Long thread (root 2026/08/18/22-34-04): 28 replies so far from 5 participants over 12 days; 5 messages that day.*

Elijah Newren posts v3 of a four-patch series addressing objects treated as missing despite being present, due to a race with geometric repacking in the packfile and multi-pack-index code. The revision drops the earlier patch that handled QUICK readers and instead inserts a fix for `git mktree --batch` to stop passing `OBJECT_INFO_QUICK` (with a new testcase in `t1010`), renames `fill_midx_entry()` to `midx_fill_entry()` so callers must handle a new tri-state return value (`MIDX_FILL_MISS` / `MIDX_FILL_HIT` / `MIDX_FILL_OWNER_UNAVAILABLE`), plugs a per-tree leak in `mktree --batch`, and makes `find_pack_entry()` recover when a stale multi-pack-index names a removed owning pack by scanning the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ`. Jeff King endorses the decision to drop the QUICK-recovery patch, approves the `midx_fill_entry` rename as fitting the subsystem naming order, and reviews v3 3/4 and 4/4 as looking good, reasoning that the MIDX-specific scan loop is the right approach since MIDX-covered packs are not all in the regular pack list. Earlier, Jeff King had argued that QUICK readers work as designed and accept false negatives, while Patrick Steinhardt maintained the patch fixes a real bug: an object present in two MIDX-covered packs is not found at all when the preferred pack vanishes concurrently, because the non-MIDX lookup skips all MIDX-covered packs. Derrick Stolee had pointed to `git multi-pack-index expire` as the existing mechanism for this race.

### converting build-time USE_NSEC into runtime core.useNanosec

*Long thread (root 2026/08/07/11-56-23): 24 replies so far from 5 participants over 23 days; 1 message that day.*

D. Ben Knoble posts v5 of the three-patch `dk/use-nsec-runtime` series, which converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration setting, replacing `#ifdef` checks in `is_racy_stat()` and `match_stat_data()` in `read-cache.c` and `statinfo.c` with runtime checks on `cfg->use_nanosec` and updating the racy-git documentation, so that index-related commands like `git diff` are less likely to hit racy-git problems on supported filesystems. The series also adds a meson `man_base_url` option that changes the default base URL for relative links in manuals to `file://$(htmldir)/` to fix broken links in generated man pages, and reorders assignments in `repo_config_values_init` to align with the `struct repo_config_values` declaration with no behavioral changes. Earlier, Junio C Hamano and Patrick Steinhardt had discussed whether the `use_nanosec` member should be initialized unconditionally and whether a warning should be emitted on `NO_NSEC` builds, with agreement that no warning should be added since users sharing a `$HOME/.gitconfig` across machines with different builds would be annoyed.

## The day in brief

Elijah Newren's v3 packfile recovery series advanced with Jeff King's approval, and D. Ben Knoble posted v5 of the `core.useNanosec` runtime-config series. Jeff King pushed back on a varargs `die_for_incompatible_opts()` proposal in `parse-options.c`, while Lorenz Leutgeb documented the `if-asked` value of the `pushcert` remote-helper option.

## Notable threads

### die_for_incompatible_opts() for an unbounded number of options

Junio C Hamano's patch introduces a variadic `die_for_incompatible_opts()` in `parse-options.c` and `parse-options.h` that takes `<bool, name>` pairs terminated by `EOF` and rewrites the existing `die_for_incompatible_opt2/3/4()` wrappers to use it, answering what a developer should do when more than four mutually incompatible options must be checked. Jeff King is mildly negative on the varargs signature because the compiler cannot verify an `EOF` sentinel is present, since there is no integer equivalent to `LAST_ARG_MUST_BE_NULL`, and says keeping the counted wrappers with `LAST_ARG_MUST_BE_NULL` would be preferable. Junio C Hamano concedes that point and agrees it makes sense to flip the pairs to `<name, set>` order, while René Scharfe proposes keeping `LAST_ARG_MUST_BE_NULL` via a macro that appends `EOF, NULL`, or alternatively a type-safe `struct used_option` compound-literal array terminated by `{ NULL }`.

### checkout/switch/restore code refactoring

Junio C Hamano locally amends the eight-patch refactoring of `builtin/checkout.c` that repartitions `cmd_checkout()`, `cmd_switch()`, and `cmd_restore()` away from the monolithic `checkout_main()` choke point: patch 8/8 removes the `the_repository` dependency from `post_checkout_hook()` by giving it a `struct repository *` parameter, and patch 7/8 corrects uses of `die_for_incompatible_optN()` that had changed the order of options in error messages. D. Ben Knoble affirms that the resulting organization--trimming `cmd_foo()` implementations into orchestrators that call refactored helper functions--would be quite lovely.

### documenting pushcert if-asked for remote helpers

Lorenz Leutgeb submits a one-line documentation patch to `Documentation/gitremote-helpers.adoc` adding `if-asked` as a documented value for the `pushcert` option, noting that `transport-helper.c` already implements this value while the documentation previously listed only `true` and `false`.
