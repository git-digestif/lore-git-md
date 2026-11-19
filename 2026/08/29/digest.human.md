# Git mailing list daily digest for 2026/08/29

## Long threads

### packfile recovery when a multi-pack-index names a removed pack

*Long thread: 28 replies so far from 5 participants over 12 days; 5 messages that day.*

Elijah Newren posted v3 of a four-patch series addressing objects reported missing despite being present, due to a race with geometric repacking. The series makes `replay` return an error instead of dereferencing a NULL tree when merge input is unreadable, plugs a per-tree leak in `git mktree --batch`, stops `mktree --batch` from passing `OBJECT_INFO_QUICK` so a resident reader recovers objects relocated by a concurrent repack via the normal reprepare, and makes `find_pack_entry()` recover when a stale multi-pack-index names a removed owning pack by scanning the MIDX's packs for a surviving copy on `OBJECT_INFO_SECOND_READ`. The rename of `fill_midx_entry()` to `midx_fill_entry()` with a tri-state return (`MIDX_FILL_MISS` / `MIDX_FILL_HIT` / `MIDX_FILL_OWNER_UNAVAILABLE`) forces callers to handle the new semantics. Earlier, Jeff King had argued that QUICK readers accept false negatives by design, while Patrick Steinhardt identified the real bug: a stale MIDX hides secondary copies in MIDX-covered packs from the non-MIDX lookup. Today Jeff King endorsed the decision to drop the QUICK-recovery patch, approved the rename, and reviewed patches 3/4 and 4/4 as looking good.

### converting build-time USE_NSEC into runtime core.useNanosec

*Long thread: 24 replies so far from 5 participants over 23 days; 1 message that day.*

D. Ben Knoble posted v5 of the three-patch `dk/use-nsec-runtime` series, which converts the build-time `USE_NSEC` compile option into the runtime `core.useNanosec` configuration setting so distributors can ship a single binary, reducing "racy git" hits in index-related commands such as `git diff` on supported filesystems. Today's reroll improves the commit message flow in patch 2. The series also adds a meson `man_base_url` option that fixes broken relative links in generated man pages by defaulting to `file://$(htmldir)/`, and reorders `repo_config_values_init` assignments to align with the `struct repo_config_values` declaration. The `NO_NSEC` strategy--ignoring the config variable while using the `use_nanosec` member unconditionally--was D. Ben Knoble's own proposal from earlier in the thread.

## The day in brief

Elijah Newren's v3 packfile-recovery series and D. Ben Knoble's v5 `core.useNanosec` reroll both arrived, with Jeff King reviewing the former's mktree and MIDX patches favorably. Smaller items include a varargs `die_for_incompatible_opts()` proposal that drew mixed reactions, fixes to the checkout/switch/restore refactoring draft, and a one-line documentation correction for the `pushcert` option.

## Notable threads

### die_for_incompatible_opts() accepting an unbounded number of options

Junio C Hamano proposed a variadic `die_for_incompatible_opts()` taking `<bool, name>` pairs terminated by `EOF` to replace the fixed `die_for_incompatible_optN()` wrappers for two to four options, rewriting the existing variants on top of it in `parse-options.c` and `parse-options.h`. Jeff King called the varargs interface less safe because there is no integer equivalent to `LAST_ARG_MUST_BE_NULL`, and said he is mildly negative on the signature, preferring to keep patch 1 with `LAST_ARG_MUST_BE_NULL`. Junio conceded the point and agreed to flip the pairs to `<name, set>` order; René Scharfe proposed a macro appending `EOF, NULL` to retain `LAST_ARG_MUST_BE_NULL`, or a type-safe `struct used_option` compound-literal array alternative.

### checkout/switch/restore code refactoring

Junio C Hamano locally amended the draft series repartitioning `builtin/checkout.c` so that `cmd_switch()` and `cmd_restore()` become orchestrators over refactored helpers instead of funneling into a monolithic `checkout_main()`: patch 8/8 now has `post_checkout_hook()` take a `struct repository *` parameter rather than using `the_repository`, and patch 7/8 corrects `die_for_incompatible_optN()` uses that had changed the order of options in error messages. D. Ben Knoble affirmed that the resulting code organization would be quite lovely.

### documenting pushcert if-asked for remote helpers

Lorenz Leutgeb submitted a one-line documentation patch adding `'if-asked'` as a valid value for the `pushcert` option in `Documentation/gitremote-helpers.adoc`, noting that `transport-helper.c` has implemented the value since commit 30261094b1f7fdcba3b7a1 while the documentation previously listed only `'true'` and `'false'`.
